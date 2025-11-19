# 03 - Separate out the data model from the service definition

In this exercise we'll tease apart the separate components of what we have so
far into different layers, and understand why.

## Review what we have

In our IDE we can see the files and directories that we have in our project so
far, either in the Explorer column or via a traditional command in the shell
such as `tree -F -I node_modules`, which will reveal:

```log
./
├── README.md
├── app/
├── db/
├── db.sqlite
├── eslint.config.mjs
├── package-lock.json
├── package.json
├── services.cds
└── srv/

4 directories, 6 files
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

This workshop is focusing on what CAP and in particular CDS modelling can
bring, so we can safely ignore the `app/` directory for the rest of the
exercises.

## Rework the content of services.cds into the service and persistence layers

👉 Before making these changes, stop (with `Ctrl-C`) any currently running CAP
server (i.e. the server you started with `cds watch`) - this is just so we
don't get too many log messages during the restarts that will take place as we
create files and edit their content.

Teasing apart the content of `services.cds` we see the keywords `service` and
`entity`; these logically belong at separate levels, so let's remedy that now.

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

👉 Now create another file `simple.cds` in the `srv/` directory to define the
`Simple` service, bringing in the `Products` entity definition from where it
now is:

```cds
using workshop from '../db/schema';

service Simple {
  entity Products as projection on workshop.Products;
}
```

👉 Now delete the original `services.cds` file:

```bash
rm services.cds
```
