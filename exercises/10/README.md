# 10 - Explore projections with a second service

In this exercise we'll begin to understand what the service layer is and what
it's for, by extending the rudimentary definitions we already have in there.

## Understand where services fit

This diagram from the [Service-Centric
Paradigm](https://cap.cloud.sap/docs/guides/providing-services#service-centric-paradigm)
section of the Providing Services topic in Capire illustrates services and
their relation to Consumers, and also to where we've been thus far - at the
Domain Models part:

![service centric paradigm diagram](assets/service-centric-paradigm.png)

We're now moving up the down-arrow to the Service / API Models part.

In many ways, services are where the rubber meets the road, providing
differently shaped APIs to consumers, in different forms, all based upon the
foundation that is the domain model. It's also the context in which we can
provide custom domain (business) logic beyond what's already provided out of
the box for us by the CAP framework.

One might think of the domain model (conventionally at the `db/` layer) as
being fairly static, i.e. declarative definitions that form the source of truth
for artifacts in the database and for how queries are resolved at runtime.

In contrast, services (conventionally at the `srv/` layer) are dynamic. They
marshal, constrain, reimagine, expose and control access to data and functions
at the domain model via cheap, lightweight declarative definitions that
describe facades in different forms.

These facades, or APIs, are dynamic in the sense that they are reified in the
context of wire protocols such as plain HTTP ("REST"), OData and GraphQL. So at
that level there are moving parts. But at a level below there are moving parts
too, built into the framework, to provide full and complete support for the
built-in and protocol-specific facilities.

## Look beyond the pass-through projections

> While some of what's presented in this section could equally apply to domain
> modelling (at the `db/` level), it is especially useful to know and to have
> in mind when considering how cheap services are to define and how flexible
> they can be to present flexible and focused facades on the business data for
> different consumption contexts and purposes.

👉 Open the `srv/simple.cds` file in the editor and take a look, to reveal:

```cds
using workshop from '../db/schema';

@protocol: 'odata'
@path    : '/simple'
service Simple {
    entity Products  as projection on workshop.Products;
    entity Suppliers as projection on workshop.Suppliers;
    entity Orders    as projection on workshop.Orders;
}
```

Let's start to explore, or at least to scratch the surface of, what we can do
beyond just the plain "pass-through" projections we have so far.

### Define a new service for accounting

Imagine we have an internal team looking at cost accounting, and want to
offer a service for them relating to the products on file. While we could just
as easily define this new service in a separate file, let's keep it in the same
file for simplicity and ease of viewing.

> Exploration of separation of concerns, mixins, and general reuse and
> definition management, as well as role based access control, are topics for future exercises.

### Change the name of the service CDS file

👉 Rename the `srv/simple.cds` file to `srv/services.cds` to reflect the fact
that (shortly) there will be more than one service, not just the `Simple` service, defined:

```bash
mv srv/simple.cds srv/services.cds
```

### Add a second service definition

👉  Add a second `Accounting` service definition as shown:

```cds
using workshop from '../db/schema';

@protocol: 'odata'
@path    : '/simple'
service Simple {
  entity Products  as projection on workshop.Products;
  entity Suppliers as projection on workshop.Suppliers;
  entity Orders    as projection on workshop.Orders;
}

service Accounting {
  entity Valuations as
    projection on workshop.Products {
      ID                   as ProductID,
      name                 as ProductName,
      stock * price.amount as StockValue : Decimal,
      price.currency.name  as Currency,
      supplier.company     as Source
    }
}
```

There's a lot to unpack here, but first, let's look at the CAP server's log
output.

👉 Pay attention to the log output from the CAP server when it restarts, where
you should see that indeed there are two services being served, on two separate
paths, from the same server base (listening on port `4004` on `localhost`):

```log
[cds] - serving Simple {
  at: [ '/simple' ],
  decl: 'srv/services.cds:5',
  impl: 'node_modules/@sap/cds/srv/app-service.js'
}
[cds] - serving Accounting {
  at: [ '/odata/v4/accounting' ],
  decl: 'srv/services.cds:13',
  impl: 'node_modules/@sap/cds/srv/app-service.js'
}
[cds] - server listening on { url: 'http://localhost:4004' }
```

### Examine the details in the valuations entity definition

Now let's turn our attention back to the definition of the `Valuations` entity
in our new `Accounting` service:

- we're still employing a
  [projection](https://cap.cloud.sap/docs/cds/cdl#as-projection-on) here
- however, we're not using the [inferred elements
  signature](https://cap.cloud.sap/docs/cds/cdl#views-with-inferred-signatures)
  (i.e. an implicit "all elements of this projectee" pass-through) that
  we've defaulted to for earlier projections (in the `Simple` service)
- instead, there is an explicit signature within the structure block (`{ ...
  }`), containing different element expressions

What are the features of those element expressions?

Well, there are a few in play:

- with `as`, the elements are presented with aliased names (e.g. `ID` is
  aliased as `ProductID`)
- `StockValue` is a [calculated
  element](https://cap.cloud.sap/docs/cds/cdl#calculated-elements); this one in
  particular is of the [on-read](https://cap.cloud.sap/docs/cds/cdl#on-read)
  variety, not stored in the database
- as the compiler does not infer a type from an expression (`stock *
  price.amount` in this case), we use a [cast](https://cap.cloud.sap/docs/cds/cql#casts-in-cdl) to set the type for `StockValue`
  explicitly (to `Decimal`)
- two elements (plus one part of the expression forming `StockValue`) have
  values which are defined via dotted multi-path names: these are [path
  expressions](https://cap.cloud.sap/docs/cds/cql#path-expressions)
  multi-path names)

> Path expressions are part of CAP's query language
> ([CQL](https://cap.cloud.sap/docs/cds/cql)), as you'll see from the [topic
> within which they are
> described](https://cap.cloud.sap/docs/cds/cql#path-expressions).

👉 Make sure the CAP server has restarted after the addition of this new
service definition, and then visit the CAP server at <http://localhost:4004>,
to see the second service presented:

![second service](assets/second-service.png)

### See the valuations definition as it occurs in the OData context

👉 Check the `Valuations` entityset made available at <http://localhost:4004/odata/v4/accounting/Valuations>, which should look something like this:

```json
{
  "@odata.context": "$metadata#Valuations",
  "value": [
    {
      "ProductID": 1,
      "ProductName": "Chai",
      "StockValue": 702,
      "Currency": "Pound",
      "Source": "Exotic Liquids"
    },
    {
      "ProductID": 2,
      "ProductName": "Chang",
      "StockValue": 323,
      "Currency": "Pound",
      "Source": "Exotic Liquids"
    },
    {
      "ProductID": 3,
      "ProductName": "Aniseed Syrup",
      "StockValue": 130,
      "Currency": "Pound",
      "Source": "Exotic Liquids"
    },
    {
      "ProductID": 4,
      "ProductName": "Chef Anton's Cajun Seasoning",
      "StockValue": 1166,
      "Currency": "Pound",
      "Source": "New Orleans Cajun Delights"
    },
    {
      "ProductID": 5,
      "ProductName": "Chef Anton's Gumbo Mix",
      "StockValue": 0,
      "Currency": "Pound",
      "Source": "New Orleans Cajun Delights"
    },
    {
      "ProductID": 6,
      "ProductName": "Grandma's Boysenberry Spread",
      "StockValue": 3000,
      "Currency": "Pound",
      "Source": "Grandma Kelly's Homestead"
    }
  ]
}
```

This is a very nice collection of flat entities, the elements of which are
calculated and even determined from related entities.

👉 Look at the `Valuations` service's metadata document at
<http://localhost:4004/odata/v4/accounting/$metadata> and identify the entity
type definition, which looks like this:

```xml
<EntityType Name="Valuations">
  <Key>
    <PropertyRef Name="ProductID"/>
  </Key>
  <Property Name="ProductID" Type="Edm.Int32" Nullable="false"/>
  <Property Name="ProductName" Type="Edm.String"/>
  <Property Name="StockValue" Type="Edm.Decimal" Scale="variable"/>
  <Property Name="Currency" Type="Edm.String" MaxLength="255"/>
  <Property Name="Source" Type="Edm.String"/>
</EntityType>
```

This really brings home the "flatness" and power of even this simple set of definitions.

### Follow the path expressions

How do the path expressions work, what's going on there? We have `price.amount` in:

```cds
stock * price.amount as StockValue : Decimal
```

and `price.currency.name` in:

```cds
price.currency.name as Currency
```

and `supplier.company` in:

```cds
supplier.company as Source
```

In exercises
earlier in this workshop, as we built out our domain model, we took the
occasional look at the CSN. Now that we've got used to that, let's do it again,
as the insights it can give us in understanding what's going on here are
valuable.

#### Look at the CSN for detail

👉 Request the CSN, in YAML format as always, for the `srv/services.cds` resource:

```bash
cds compile --to yaml srv/services.cds
```

If you pick out the `Accounting.Valuations` definition, you'll uncover a wealth
of information:

```yaml
Accounting.Valuations:
  kind: entity
  projection:
    from: { ref: [workshop.Products] }
    columns:
      - { ref: [ID], as: ProductID }
      - { ref: [name], as: ProductName }
      - {
          xpr:
            [
              { ref: [stock] },
              "*",
              { ref: [price, amount] },
            ],
          as: StockValue,
          cast: { type: cds.Decimal },
        }
      - { ref: [price, currency, name], as: Currency }
      - { ref: [supplier, company], as: Source }
  elements:
    ProductID: { key: true, type: cds.Integer }
    ProductName: { type: cds.String }
    StockValue:
      { "@Core.Computed": true, type: cds.Decimal }
    Currency:
      {
        "@title": { i18n>Name },
        localized: true,
        type: cds.String,
        length: 255,
      }
    Source: { type: cds.String }
```

> Digging into the detail of this is beyond the scope of this workshop, but it's
important to know that it exists, and is the gateway to further understanding,
especially in the context of expression notation
([CXN](https://cap.cloud.sap/docs/cds/cxn)).

What's happening here is that the path expressions are the declarative, human
readable version of multi-level view references.

Simple alias references such as:

```cds
ID as ProductID
```

are single level references, as can be seen from the equivalent section in the CSN:

```yaml
{ ref: [ID], as: ProductID }
```

The path expression:

```cds
supplier.company as Source
```

is a two level mapping, as can be seen from the equivalent section in the CSN:

```yaml
{ ref: [supplier, company], as: Source }
```

#### Visualise the expression traversal through the CDL

If we consider how this is resolved, we can think of our domain model in CDL,
and how that expression traverses it (start at the bottom!):

```text
        using {Currency} from '@sap/cds/common';

        namespace workshop;

        aspect cuid {
          key ID : Integer;
        }

        type Price {
          amount   : Decimal;
          currency : Currency;
        }
  +--------------+
  |              |
  |              V
  |     entity Products : cuid {
  |              |
  |   +----------+
  |   |
  |   |   name     : String;
  |   |   stock    : Integer;
  |   |   price    : Price;
  |   +-> supplier : Association to Suppliers;
  |     }                               |
  |                                     |
  |                +--------------------+
  |                |
  |                V
  |     entity Suppliers : cuid {
  |                |
  |          +-----+
  |          |
  |          V
  |       company  : String;
  |       products : Association to many Products
  |                    on products.supplier = $self;
  |     }
  |
  |     entity Orders : cuid {
  |       date  : Date default $now;
  |       items : Composition of many {
  |                 key pos      : Integer;
  |                     product  : Association to Products;
  |                     quantity : Integer;
  |               }
  |     }
  |
  +-----------------------------------+
                                      |
        service Accounting {          |
          entity Valuations as        |
    +---->  projection on workshop.Products {
    |         ID                   as ProductID,
    |         name                 as ProductName,
    |         stock * price.amount as StockValue : Decimal,
    |         price.currency.name  as Currency,
    |         supplier.company     as Source
    |       } ----------------
    |   }             |
    +-----------------+
```

Phew!

---

[Next](../11/)
