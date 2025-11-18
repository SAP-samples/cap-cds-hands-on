# 02 - Understand the basic model, service and persistence features

In the previous exercise we conjured up a simple but fully functional OData
service from a few lines of declarative model definition. In this exercise
we'll explore what we have to understand the central importance of the model,
what the service is, and how the definition relates to a database layer.

## Explore the service

👉 Open up `http://localhost:4004/` in your browser to see the default CAP
server landing page:

![Default CAP server landing page](assets/default-cap-server-landing-page.png)

👉 Examine some of the resources available via the various hyperlinks:

- [http://localhost:4004/odata/v4/simple](http://localhost:4004/odata/v4/simple)
  (the OData service document)
- [http://localhost:4004/odata/v4/simple/\$metadata](http://localhost:4004/odata/v4/simple/$metadata)
  (the OData metadata document)
- [Fiori
  preview](http://localhost:4004/$fiori-preview/Simple/Products#preview-app) (a
  basic Fiori elements List Report app where the Products data can be viewed)
- [http://localhost:4004/odata/v4/simple/Products](http://localhost:4004/odata/v4/simple/Products)
  (an entityset with the Products data)

👉 Riff on this last resource by trying out some standard OData URL-based query
mechanisms (admittedly these are somewhat limited, given the rather limited
nature of our dataset), such as:

- [http://localhost:4004/odata/v4/simple/Products/3](http://localhost:4004/odata/v4/simple/Products/3)
  (retrieve the Product with ID 3)
- [http://localhost:4004/odata/v4/simple/Products?\$filter=contains(name,%27Syrup%27)](http://localhost:4004/odata/v4/simple/Products?$filter=contains(name,'Syrup'))
  (return an entityset with products that have "Syrup" in their names)
- [http://localhost:4004/odata/v4/simple/Products?\$select=name](http://localhost:4004/odata/v4/simple/Products?$select=name)
  (all the products, just with the name property for each entity)

> By default the key property, in this case `ID`, is returned as well.

👉 At the shell prompt, add a new product:

```bash
lastid="$(
  curl -s 'localhost:4004/odata/v4/simple/Products?$orderby=ID%20desc&$top=1' | \
    jq '.value|first|.ID'
)" # get latest ID
nextid="$((lastid + 1))" # increment it
curl -d '{"ID":'"$nextid"',"name":"New Product ('"$nextid"')","stock":10}' \
  --url 'localhost:4004/odata/v4/simple/Products' # use for a new record
```

👉 Change the name of product with ID 3:

```bash
curl --request PATCH \
  --data '{"name": "Aniseed Sauce"}' \
  --url 'localhost:4004/odata/v4/simple/Products/3'
```

👉 Remove the "Chai" product:

```bash
curl --request DELETE \
  --url 'localhost:4004/odata/v4/simple/Products/1'
```

## Understand how the definition is used

It's hard to imagine now how much work it was to get an OData service like this
up and running before CAP. But that's not the point of this exercise nor this
workshop. Instead, let's take a quick look at what "descends" from the
definition.

The definition is written using the Conceptual Definition Language
([CDL](https://cap.cloud.sap/docs/cds/cdl)), the human-readable form of the
declarative language designed to be used by domain experts and developers to
build a solution based on the foundation of the domain model that underpins it.

### Core Schema Notation

The CAP server uses the CDS model definition to provide an appropriate OData
service here, out of the box. But it uses it in a more readily machine-readable
form called Core Schema Notation ([CSN](https://cap.cloud.sap/docs/cds/csn),
pronounced "season") and which can come in two common representations - JSON and
YAML.

Let's remind ourselves of the CDS model we have, written in CDL:

```cds
service Simple {
  entity Products {
    key ID    : Integer;
        name  : String;
        stock : Integer;
  }
}
```

👉 Create the CSN equivalent of this model, in a JSON representation:

```bash
cds compile --to json services.cds
```

This emits:

```json
{
  "definitions": {
    "Simple": {
      "@source": "services.cds",
      "kind": "service"
    },
    "Simple.Products": {
      "kind": "entity",
      "elements": {
        "ID": {
          "key": true,
          "type": "cds.Integer"
        },
        "name": {
          "type": "cds.String"
        },
        "stock": {
          "type": "cds.Integer"
        }
      }
    }
  },
  "meta": {
    "creator": "CDS Compiler v6.4.6",
    "flavor": "inferred"
  },
  "$version": "2.0"
}

```

> This is a very common request and so can also be produced with the shorter
> `cds c .`, where `.` is a reference to the current directory, which only
> contains a single `services.cds` source file at this point anyway.

While JSON is arguably "the default", YAML is easier on the eye so we'll use
that as our go-to representation throughout this workshop.

👉 Re-create the CSN equivalent of this model, this time in a YAML representation:

```bash
cds compile --to yaml services.cds
```

This emits:

```yaml
definitions:
  Simple: { kind: service }
  Simple.Products:
    kind: entity
    elements:
      {
        ID: { key: true, type: cds.Integer },
        name: { type: cds.String },
        stock: { type: cds.Integer },
      }
meta: { creator: CDS Compiler v6.4.6, flavor: inferred }
$version: 2.0
```

> Here, for purposes of display and readability in this workshop, the YAML has
> been passed through [Prettier](https://prettier.io/), "an opinionated code
> formatter".

While we won't need to look much further at CSN in this workshop, it's
important to understand that it exists and is the "processable" version of the
definitions we construct in our CDS models.

### SQL

At some point the data model part of our definitions will need to have some
form of presence at a persistence layer. In other words, we'll want to store
the data in our model, the records that make up the data in our entities, and
the projections upon them, and so on.

When we started our CAP server with `cds watch` earlier, this compiled the CDL
into SQL, in order to be able to deploy the data model to a persistence layer.

In development mode, by default, this persistence layer is provided by the
quietly powerful and ubiquitous [SQLite](https://sqlite.org/) database engine.
Also by default in this context, it will be started in "in-memory" mode, i.e.
ephemeral persistence (if that is not an oxymoron) for the duration of the CAP
server's lifetime. This is incredibly useful for rapid turnaround and
local-first development.

> If you want to learn more about what CAP has to offer for local-first
> development, come to the technical talk [Local-first: A more efficient
> development strategy for extending with the SAP Cloud Application Programming
> Model](https://virtual.oxfordabstracts.com/event/75555/submission/127)
> tomorrow (Mon 01 Dec).

When looking at the log output from the CAP server earlier, you may have
noticed these lines:

```log
[cds] - connect to db > sqlite { url: ':memory:' }
/> successfully deployed to in-memory database.
```

This reflects what's happening - the model is compiled and deployed to a SQLite
powered in-memory persistence layer.

👉 Take a look at what that looks like by asking for the SQL equivalent:

```bash
cds compile --to sql services.cds
```

This emits:

```sql
CREATE TABLE Simple_Products (
  ID INTEGER NOT NULL,
  name NVARCHAR(255),
  stock INTEGER,
  PRIMARY KEY(ID)
);
```

> There's also the HANA specific equivalent for when deployment is to an SAP
> HANA powered persistence layer, which can be summoned with `cds compile --to
> hana services.cds` and looks like this:
>
> ```sql
> ----- Simple.Products.hdbtable -----
> COLUMN TABLE Simple_Products (
>   ID INTEGER NOT NULL,
>   name NVARCHAR(5000),
>   stock INTEGER,
>   PRIMARY KEY(ID)
> )
> ```
