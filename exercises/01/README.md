# 01 - The simplest thing that could possibly work

In this exercise we'll create a very basic definition and see how that becomes
the heart of everything. [The simplest thing that could possibly
work](https://creators.spotify.com/pod/profile/tech-aloud/episodes/The-Simplest-Thing-that-Could-Possibly-Work--A-conversation-with-Ward-Cunningham--Part-V---Bill-Venners-e5dpts),
pretty much. After all, every technical solution to a business problem has a
well thought out model at its core.

## Start a new CAP Node.js project

👉 Within the `hands-on-with-cap-cds/` directory, initialise a new CAP project
and change to the directory that is created:

```bash
cds init simple \
  && cd $_ \
  && npm install \
  && cds watch
```

> The only reason we're running `npm install` is so that we'll get
> project-local instances of the CAP runtime meaning that the log output will
> be a little less verbose as the paths to the implementations and default
> files should be shorter and relative to the current directory.

This should emit something like this:

```log
creating new CAP project in ./simple

adding nodejs

successfully created project – continue with cd simple

find samples on https://github.com/capire/samples
learn about next steps at https://cap.cloud.sap

cds serve all --with-mocks --in-memory?
( live reload enabled for browsers )

        ___________________________

    No models found in db/,srv/,app/,schema,services.
    Waiting for some to arrive...

```

The CAP server is started but is telling us (correctly) that there are no
models defined in any place it expects, so will not start listening for any
incoming requests as there is nothing to wrap a service around.

> The words "model" and "service" are chosen specificially and their meaning
> and distinction will become clear later on.

## Define a simple domain model

👉 Add the following content to a new file in the `simple/` directory called `services.cds`:

```cds
service Simple {
  entity Products {
    key ID    : Integer;
        name  : String;
        stock : Integer;
  }
}
```

> Technically we're combining a model definition inside a simple service
> definition here, but again, that distinction is for later.

As we've started the CAP server in [watch
mode](https://cap.cloud.sap/docs/tools/cds-cli#cds-watch), it should notice
these changes and restart, and this time, the log output includes, amongst
others (which have been redacted to keep things simple), these extra lines:

```log
[cds] - loaded model from 2 file(s):

  services.cds
  node_modules/@sap/cds/srv/outbox.cds

[cds] - connect to db > sqlite { url: ':memory:' }
/> successfully deployed to in-memory database.

[cds] - serving Simple {
  at: [ '/odata/v4/simple' ],
  decl: 'services.cds:1',
  impl: 'node_modules/@sap/cds/srv/app-service.js'
}

[cds] - server listening on { url: 'http://localhost:4004' }
```

> From the `odata` component of the relative URL path (`/odata/v4/simple`) we
> can correctly surmise that, by default, services such as this are made
> available in OData form, which is often going to be exactly what we need to
> provide a service to support an extension or new app to enhance our
> enterprise capabilities.

## Add some initial data

The `Simple` service is fully functional as the CAP framework provides
everything for a complete CRUD (Create, Read, Update, Delete) out of the box.
But let's add some data to make it a little easier to explore.

👉 Use the `data` facet of `cds add` to have a CSV file with a header line
added for the entities (just `Products` in this simple setup):

```bash
cds add data
```

The log output from this:

```log
adding data
adding headers only, use --records to create random entries
  creating db/data/Simple.Products.csv

successfully added features to your project
```

tells us where file is.

👉 Open the file and add the following records to it:

```csv
1,Chai,39
2,Chang,17
3,Aniseed Syrup,13
```

## Explore the service

👉 Open up `http://localhost:4004/` in your browser to see the default CAP
server landing page:

![Default CAP server landing page](assets/default-cap-server-landing-page.png)

👉 Examine some of the resources available via the various hyperlinks:

- [http://localhost:4004/odata/v4/simple](http://localhost:4004/odata/v4/simple) (the OData service
  document)
- [http://localhost:4004/odata/v4/simple/\$metadata](http://localhost:4004/odata/v4/simple/$metadata)
  (the OData metadata document)
- [Fiori preview](http://localhost:4004/$fiori-preview/Simple/Products#preview-app) (a basic Fiori elements List Report app where the Products data can be viewed)
- [http://localhost:4004/odata/v4/simple/Products](http://localhost:4004/odata/v4/simple/Products) (an entityset with the Products data)

👉 Riff on this last resource by trying out some standard OData URL-based query mechanisms (admittedly these are somewhat limited, given the rather limited nature of our dataset), such as:

- [http://localhost:4004/odata/v4/simple/Products/3](http://localhost:4004/odata/v4/simple/Products/3) (retrieve the Product with ID 3)
- [http://localhost:4004/odata/v4/simple/Products?\$filter=contains(name,%27Syrup%27)](http://localhost:4004/odata/v4/simple/Products?$filter=contains(name,'Syrup')) (return an entityset with products that have "Syrup" in their names)
- [http://localhost:4004/odata/v4/simple/Products?\$select=name](http://localhost:4004/odata/v4/simple/Products?$select=name) (all the products, just with the name property for each entity)

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
