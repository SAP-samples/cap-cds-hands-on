# 03 - Separate out the data model from the service definition

In this exercise we'll tease apart the separate components of what we have so
far into different layers, and understand why.

## Review what we have

In our working environment we can see the files and directories that we
have in our project so far, either in an Explorer style column or via a
traditional command in the shell such as `tree -F`, which will reveal:

```log
./
├── app/
├── db/
│   └── data/
│       └── Simple.Products.csv
├── readme.md
├── services.cds
├── srv/
└── test.db

5 directories, 4 files
```

The simple OData service we have so far is the result of definitions in a
single file `services.cds` at the project root level, and based upon a few
declarative lines:

```cds
service Simple {
  entity Products {
    key ID    : Integer;
        name  : String;
        stock : Integer;
  }
}
```

## Get to know the standard layers

Going back to the directories in our project, we have `app/`, `srv/` and `db/`.
These were created when the project was initialised (with `cds init`) and are
the standard locations that CAP Node.js uses to find:

Location|Contains
-|-
`app/`|Frontend (UI) assets such as HTML, CSS and JavaScript files, often UI5 / Fiori based
`srv/`|Service definitions (plural, as defining [single-purposed services](https://cap.cloud.sap/docs/guides/providing-services#single-purposed-services) is a best practice - see the corresponding link in the [Related resources for part 1](../../?tab=readme-ov-file#related-resources-for-part-1) to this part of the workshop)
`db/`|The data model, predominantly in the form of entity definitions and relations between them

This is the first glimpse into, and a 30,000 feet level example of, one of
CAP's fundamental design tenets - [separation of
concerns](https://cap.cloud.sap/docs/cds/aspects#separation-of-concerns).

> We already noticed the use of `db/` as the default location for a `data/`
> directory to hold [initial data](../01#add-some-initial-data), and we'll
> revisit that later in this exercise.

This workshop is focusing on what CAP and in particular CDS modelling can
bring, so we can safely ignore the `app/` directory for the rest of the
workshop.

## Rework the content of services.cds into the service and persistence layers

👉 Before making these changes, stop (with `Ctrl-C`) any currently running CAP
server (i.e. the server you started with `cds watch` in a previous exercise) -
this is just so we don't get too many log messages during the restarts that
will take place as we create files and edit their content.

Examining the content of `services.cds` we see the keywords `service` and
`entity`; these logically belong at separate levels, so let's adjust that now.

### Define the schema

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
> directive which is used to define a prefix, or scope, for all subsequent
> definition names in the file; thus the fully qualified name of the entity
> will be `workshop.Products`.

### Define the service

👉 Now create another file `simple.cds` in the `srv/` directory to define the
`Simple` service, bringing in the `workshop` name (through which we can
reference the `Products` entity definition within) from where it
now is:

```cds
using workshop from '../db/schema';

service Simple {
  entity Products as projection on workshop.Products;
}
```

> [!NOTE] The [using](https://cap.cloud.sap/docs/cds/cdl#using) directive is a
> key enabler of componentisation, separation of concerns and model reuse. The
> CDL in this `simple.cds` file starts by importing the definitions from the
> `schema.cds` file at the `db/` layer, by their top-level name
> (namespace)[<sup>1</sup>](#footnotes).
>
> Moreover, we have `as projection on` which is [one of
> two](https://cap.cloud.sap/docs/cds/cdl#views-projections) variants that
> allow us to derive new entities from existing ones. Here the simplest case is
> in play, where there's a 1:1 relationship, a "pass-through" projection. But
> the power to limit, re-imagine and otherwise manipulate existing entities as
> new ones is great. Think of projections as
> [views](https://en.wikipedia.org/wiki/View_(SQL)) in relational databases.

👉 Now delete the original `services.cds` file, and re-align the name of
the CSV file to fit the namespaced entity name so it will be picked up
and used for initial data:

```bash
rm services.cds
mv db/data/Simple.Products.csv db/data/workshop-Products.csv
```

## Examine the data definition language constructs

Before we leave the depths of the persistence layer and the corresponding Data
Definition Language
([DDL](https://en.wikipedia.org/wiki/Data_definition_language)) statements,
let's remind ourselves of what the initial incarnation of our service
definition [translated to in DDL](../02#sql-and-ddl) translated to:

```sql
CREATE TABLE Simple_Products (
  ID INTEGER NOT NULL,
  name NVARCHAR(255),
  stock INTEGER,
  PRIMARY KEY(ID)
);
```

Now, with our first steps towards separating the concerns, and the layers,
things look different.

👉 Request a compilation to SQL again, like this:

```bash
cds compile --to sql srv/simple.cds
```

This time there are two distinct objects - a table, and a view:

```sql
CREATE TABLE workshop_Products (
  ID INTEGER NOT NULL,
  name NVARCHAR(255),
  stock INTEGER,
  PRIMARY KEY(ID)
);

CREATE VIEW Simple_Products AS SELECT
  Products_0.ID,
  Products_0.name,
  Products_0.stock
FROM workshop_Products AS Products_0;
```

The reification of the projection as a view at the persistence layer is what we
expected, given the explanation of `as projection on` earlier.

## Check the service works as before

Before finishing this exercise and this first part, let's make sure the
modifications we've made still result in what we intend, i.e. the simple OData
service exposing product information.

👉 Start the CAP server up again:

```bash
cds watch
```

> This can be shortened to `cds w`, helpful if you're typing things in manually.

👉 Revisit <http://localhost:4004> and explore the service [like you did in the
previous exercise](../02#explore-the-service).

Well done! We can now move on to [the next
part](../../?tab=readme-ov-file#part-2---more-on-structure-with-types-aspects-and-reuse)
of this workshop.

---

## Footnotes

1. Technically we don't have to use a namespace, we could have written the
   definitions like this in `db/schema.cds`:

    ```cds
    entity Products {
      ...
    }
    ```

    and then used the definition in `srv/simple.cds` like this:

    ```cds
    using Products from '../db/schema';

    service Simple {
        entity MyProducts as projection on Products;
    }
    ```

    However, there are many advantages to using namespacing, for example:

    - multiple entities can be imported at once with a single "handle" instead
      of in a comma separated list within braces (`{ Products, Suppliers,
      Customers, ... }`)
    - a clean separation can be made between projection and projectee names
    - ownership can be aligned along namespace lines
    - naming clashes can be avoided
