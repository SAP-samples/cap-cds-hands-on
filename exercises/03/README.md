# 03 - Separate out the data model from the service definition

In this exercise we'll tease apart the separate components of what we have so
far into different layers, and understand why.

## Review what we have

In our IDE we can see the files and directories that we have in our project so
far, either in an Explorer style column or via a traditional command in the shell
such as `tree -F -I node_modules`, which will reveal:

```log
./
├── README.md
├── app/
├── db/
│   └── data/
│       └── Simple.Products.csv
├── db.sqlite
├── eslint.config.mjs
├── package-lock.json
├── package.json
├── services.cds
└── srv/

5 directories, 7 files
```

The simple OData service we have so far is in a single file `services.cds` at
the project root level, and based upon a few declarative lines:

```cds
service Simple {
  entity Products {
    key ID    : Integer;
        name  : String;
        stock : Integer;
  }
}
```

Going back to the directories in our project, we have `app/`, `srv/` and `db/`.
These were created when the project was initialised (with `cds init`) and are
the standard locations that CAP Node.js uses to find:

Location|Contains
-|-
`app/`|Frontend (UI) assets such as HTML, CSS and JavaScript assets, often UI5 / Fiori based
`srv/`|Service definitions (plural, as defining [single-purposed services](https://cap.cloud.sap/docs/guides/providing-services#single-purposed-services) is a best practice)
`db/`|The data model, predominantly in the form of entity definitions and relations between them

This is the first glimpse into, and a 30,000 feet level example of, one of
CAP's fundamental design tenets - [separation of
concerns](https://cap.cloud.sap/docs/cds/aspects#separation-of-concerns).

> We already noticed the use of `db/` as the default location for a `data/`
> directory to hold [initial data](../01#add-some-initial-data), and we'll
> revisit that later in this exercise.

This workshop is focusing on what CAP and in particular CDS modelling can
bring, so we can safely ignore the `app/` directory for the rest of the
exercises.

## Rework the content of services.cds into the service and persistence layers

👉 Before making these changes, stop (with `Ctrl-C`) any currently running CAP
server (i.e. the server you started with `cds watch`) - this is just so we
don't get too many log messages during the restarts that will take place as we
create files and edit their content.

Examining the content of `services.cds` we see the keywords `service` and
`entity`; these logically belong at separate levels, so let's adjust that now.

👉 Create a file `schema.cds` in the `db/` directory, with the following
contents, i.e. extracting the `entity` definition into this new file:

```cds
namespace workshop;

entity Products {
  key ID    : Integer;
      name  : String;
      stock : Integer;
}
```

> [!NOTE]
> Here we come across the
> [namespace](https://cap.cloud.sap/docs/cds/cdl#the-namespace-directive)
> directive which is used to define a prefix for all subsequent definition
> names in the file; thus the fully qualified name of the entity will be
> `workshop.Products`.

👉 Now create another file `simple.cds` in the `srv/` directory to define the
`Simple` service, bringing in the `Products` entity definition from where it
now is:

```cds
using workshop from '../db/schema';

service Simple {
  entity Products as projection on workshop.Products;
}
```

> [!NOTE]
> The [using](https://cap.cloud.sap/docs/cds/cdl#using) directive is a key
> enabler of componentisation, separation of concerns and model reuse. The CDL
> in this `simple.cds` file starts by importing the definitions from the
> `schema.cds` file at the `db/` layer, by their top-level name (namespace).

👉 Now delete the original `services.cds` file, and re-align the name of
the CSV file to fit the namespaced entity name so it will be picked up
and used for initial data:

```bash
rm services.cds
mv db/data/Simple.Products.csv db/data/workshop-Products.csv
```
