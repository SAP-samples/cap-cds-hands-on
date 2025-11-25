# 07 - Link entities together with associations

In this exercise we'll learn how to use associations in CDL to relate entities together.

## Review what we have and what's missing

Right now in our `db/schema.cds` file we have a couple of entities, `Products`
and `Suppliers`:

```cds
entity Products : cuid {
  name  : String;
  stock : Integer;
  price : Price;
}

entity Suppliers : cuid {
  company : String;
}
```

Logically there is a relationship between these two entities, as demonstrated
in the original
[Northbreeze](https://developer-challenge.cfapps.eu10.hana.ondemand.com/odata/v4/northbreeze)
service.

[Each product comes from a certain supplier](https://developer-challenge.cfapps.eu10.hana.ondemand.com/odata/v4/northbreeze/Products?$top=5&$select=ProductName&$expand=Supplier($select=CompanyName)):

```json
{
  "@odata.context": "$metadata#Products(ProductName,ProductID,Supplier(CompanyName,SupplierID))",
  "value": [
    {
      "ProductName": "Chai",
      "ProductID": 1,
      "Supplier": {
        "SupplierID": 1,
        "CompanyName": "Exotic Liquids"
      }
    },
    {
      "ProductName": "Chang",
      "ProductID": 2,
      "Supplier": {
        "SupplierID": 1,
        "CompanyName": "Exotic Liquids"
      }
    },
    {
      "ProductName": "Aniseed Syrup",
      "ProductID": 3,
      "Supplier": {
        "SupplierID": 1,
        "CompanyName": "Exotic Liquids"
      }
    },
    {
      "ProductName": "Chef Anton's Cajun Seasoning",
      "ProductID": 4,
      "Supplier": {
        "SupplierID": 2,
        "CompanyName": "New Orleans Cajun Delights"
      }
    },
    {
      "ProductName": "Chef Anton's Gumbo Mix",
      "ProductID": 5,
      "Supplier": {
        "SupplierID": 2,
        "CompanyName": "New Orleans Cajun Delights"
      }
    }
  ]
}
```

Also, [suppliers can have more than one product available](https://developer-challenge.cfapps.eu10.hana.ondemand.com/odata/v4/northbreeze/Suppliers?$top=3&$select=CompanyName&$expand=Products($select=ProductName)):

```json
{
  "@odata.context": "$metadata#Suppliers(CompanyName,SupplierID,Products(ProductName,ProductID))",
  "value": [
    {
      "CompanyName": "Exotic Liquids",
      "SupplierID": 1,
      "Products": [
        {
          "ProductID": 1,
          "ProductName": "Chai"
        },
        {
          "ProductID": 2,
          "ProductName": "Chang"
        },
        {
          "ProductID": 3,
          "ProductName": "Aniseed Syrup"
        }
      ]
    },
    {
      "CompanyName": "New Orleans Cajun Delights",
      "SupplierID": 2,
      "Products": [
        {
          "ProductID": 4,
          "ProductName": "Chef Anton's Cajun Seasoning"
        },
        {
          "ProductID": 5,
          "ProductName": "Chef Anton's Gumbo Mix"
        },
        {
          "ProductID": 65,
          "ProductName": "Louisiana Fiery Hot Pepper Sauce"
        },
        {
          "ProductID": 66,
          "ProductName": "Louisiana Hot Spiced Okra"
        }
      ]
    },
    {
      "CompanyName": "Grandma Kelly's Homestead",
      "SupplierID": 3,
      "Products": [
        {
          "ProductID": 6,
          "ProductName": "Grandma's Boysenberry Spread"
        },
        {
          "ProductID": 7,
          "ProductName": "Uncle Bob's Organic Dried Pears"
        },
        {
          "ProductID": 8,
          "ProductName": "Northwoods Cranberry Sauce"
        }
      ]
    }
  ]
}
```

What we really need in our model is a similar relationship, one that goes both ways:

- `Products` -> `Suppliers`
- `Suppliers` -> `Products`

## Use an association to link products to suppliers

In CDS modelling, there is the concept of
[associations](https://cap.cloud.sap/docs/cds/cdl#associations) whose purpose
is to describe relationships between entities.

Our first task, to declare a `Products` -> `Suppliers` relationship, can be
achieved with a so-called "managed to-one association". What does that name mean?

The "to-one" part of the name is half of the classic
[one-to-one](https://en.wikipedia.org/wiki/One-to-one_(data_model))
relationship:

```text
+-----+  1:1  +-----+
|  A  |<----->|  B  |
+-----+       +-----+
```

Why only half of it? Well, a one-to-one relationship is "_a type of cardinality
that refers to the relationship between two entities A and B in which one
element of A may only be linked to one element of B, **and vice versa**_". And
while a product may only have one supplier, a supplier may have more than one
product.

Hence if `A` is `Products` and `B` is `Suppliers`, then the "managed
to-one association" is this part:

```text
+-----+   :1  +-----+
|  A  |   --->|  B  |
+-----+       +-----+
```

The "managed" part of the name tells us that CAP manages the technical details
of the relationship's implementation, in that the foreign key details and
persistence level query operations are automatically taken care of, without us
having to describe how to make the relationship a reality; remember, CDS
domain modelling is about [capturing intent - what, not
how](https://cap.cloud.sap/docs/guides/domain-modeling#capture-intent-%E2%80%94-what-not-how).

### Define the relationship

👉 Add a new `supplier` element to the `Products` entity, using the managed
to-one association syntax to describe it, like this:

```cds
entity Products : cuid {
  name     : String;
  stock    : Integer;
  price    : Price;
  supplier : Association to Suppliers;
}

entity Suppliers : cuid {
  company : String;
}
```

> We may sometimes see `Association to one` out there in the wild, but the
> `one` is optional, and it reads better without given the plural naming
> convention for the targets.

It's as simple as that.

What effect does this actually have? Well, let's take a look.

👉 Compile the `db/schema.cds` contents to CSN again, asking for a YAML
representation, and pick out the `workshop.Products` definition:

```bash
cds compile --to yaml db/schema.cds
```

The important parts of the definition we're looking for are here:

```yaml
namespace: workshop
definitions:
  workshop.Products:
    kind: entity
    includes: [workshop.cuid]
    elements:
      ID: { key: true, type: cds.Integer }
      name: { type: cds.String }
      stock: { type: cds.Integer }
      price: { type: workshop.Price } supplier:
        {
          type: cds.Association,
          target: workshop.Suppliers,
          keys: [{ ref: [ID] }],
        }
  workshop.Suppliers:
    kind: entity
    includes: [workshop.cuid]
    elements:
      ID: { key: true, type: cds.Integer }
      company: { type: cds.String }
```

Observe that the `supplier` element is defined thus:

```yaml
{
  type: cds.Association,
  target: workshop.Suppliers,
  keys: [{ ref: [ID] }],
}
```

This shows us that:

- `type`: `Association` is effectively a built-in type too
- `target`: any sort of relationship needs to declare where it's pointing
- `keys`: the referenced `ID` here is the name of the key element of the target
  (the `ID` element in `workshop.Suppliers`)

Moreover, we can see the effect of this association if we ask for CSV headers
to be re-generated at this point ...

👉 Do that now:

```bash
cds add data --force && head db/data/workshop*.csv
```

This produces:

```log
using '--force' ... existing files will be overwritten

adding data
adding headers only, use --records to create random entries
  overwriting db/data/sap.common-Currencies.csv
  overwriting db/data/sap.common-Currencies.texts.csv
  overwriting db/data/workshop-Products.csv
  overwriting db/data/workshop-Suppliers.csv

successfully added features to your project
==> db/data/workshop-Products.csv <==
ID,name,stock,price_amount,price_currency_code,supplier_ID,createdAt,createdBy,modifiedAt,modifiedBy
==> db/data/workshop-Suppliers.csv <==
ID,company,createdAt,createdBy,modifiedAt,modifiedBy
```

There are two important things to note here:

- the `db/data/workshop-Products.csv` header has a new field `supplier_ID`,
  constructed by default (in the "managed" mode) from the source element name
  `supplier` and the target key element's name `ID`, joined with an underscore
- the `db/data/workshop-Suppliers.csv` has -- and needs -- nothing for this
  relationship

### Add some supplier and product data

To see the effect of this relationship, let's add some data - just a handful of
products and suppliers from the Northbreeze service.

👉 Copy the two CSV files `workshop-Products.csv` and `workshop-Suppliers.csv`
from this exercise's [assets/](assets/) directory into the `db/data/`
directory:

```bash
cp ../exercises/07/assets/workshop-*.csv db/data/
```

👉 Now start up the CAP server again using `cds watch` as you did in part 1 of
this workshop:

```bash
cds watch
```

Looking at our service definition in `srv/simple.cds`, which looks like this:

```cds
using workshop from '../db/schema';

service Simple {
  entity Products as projection on workshop.Products;
}
```

then we remember that we'll get this service exposed as an OData V4 service by
default, indeed we can see this from the CAP server log output:

```log
[cds] - serving Simple {
  at: [ '/odata/v4/simple' ],
  decl: 'srv/simple.cds:3',
  impl: 'node_modules/@sap/cds/srv/app-service.js'
}
```

Given that, let's put the association to the test.

👉 Retrieve the products entityset, requesting an expansion on the supplier in
each case, via this URL:
<http://localhost:4004/odata/v4/simple/Products?$select=name&$expand=supplier>

The resulting entityset should look like this:

```json
{
  "@odata.context": "$metadata#Products",
  "value": [
    {
      "name": "Chai",
      "supplier": {
        "ID": 1,
        "company": "Exotic Liquids"
      },
      "ID": 1
    },
    {
      "name": "Chang",
      "supplier": {
        "ID": 1,
        "company": "Exotic Liquids"
      },
      "ID": 2
    },
    {
      "name": "Aniseed Syrup",
      "supplier": {
        "ID": 1,
        "company": "Exotic Liquids"
      },
      "ID": 3
    },
    {
      "name": "Chef Anton's Cajun Seasoning",
      "supplier": {
        "ID": 2,
        "company": "New Orleans Cajun Delights"
      },
      "ID": 4
    },
    {
      "name": "Chef Anton's Gumbo Mix",
      "supplier": {
        "ID": 2,
        "company": "New Orleans Cajun Delights"
      },
      "ID": 5
    },
    {
      "name": "Grandma's Boysenberry Spread",
      "supplier": {
        "ID": 3,
        "company": "Grandma Kelly's Homestead"
      },
      "ID": 6
    }
  ]
}
```

## Define the reverse relationship from suppliers to products

What if we wanted to try to follow the relationship the other way round, from
suppliers to the products they have?

Well, first we should add the `Suppliers` entity to the `Simple` service we
have, as right now only `Products` is exposed. This is so that we'll be able
to test out our second relationship definition.

👉 Edit `srv/simple.cds` and add another projection so that it looks like this:

```cds
using workshop from '../db/schema';

service Simple {
  entity Products  as projection on workshop.Products;
  entity Suppliers as projection on workshop.Suppliers;
}
```

The CAP server, running in [watch
mode](https://cap.cloud.sap/docs/tools/cds-cli#cds-watch) should detect this
change and automatically restart.

👉 At this point, start by requesting the suppliers entityset via this URL:
<http://localhost:4004/odata/v4/simple/Suppliers>

This should return:

```json
{
  "@odata.context": "$metadata#Suppliers",
  "value": [
    {
      "ID": 1,
      "company": "Exotic Liquids"
    },
    {
      "ID": 2,
      "company": "New Orleans Cajun Delights"
    },
    {
      "ID": 3,
      "company": "Grandma Kelly's Homestead"
    }
  ]
}
```

👉 Now try adding a `$expand` for the products navigation property with this
URL: <http://localhost:4004/odata/v4/simple/Suppliers?$expand=products>

Well, we should already be able to guess what will happen. Where did we get the
`products` navigation property from? It was a logical guess, but it doesn't
(yet) exist! Sure enough, this is returned:

```json
{
  "error": {
    "message": "Navigation property \"products\" is not defined in Simple.Suppliers",
    "code": "400",
    "@Common.numericSeverity": 4
  }
}
```

Indeed, looking at the service's [metadata
document](http://localhost:4004/odata/v4/simple/$metadata), we can see that the
`Products` entity type looks like this:

```xml
<EntityType Name="Products">
    <Key>
        <PropertyRef Name="ID"/>
    </Key>
    <Property Name="ID" Type="Edm.Int32" Nullable="false"/>
    <Property Name="name" Type="Edm.String"/>
    <Property Name="stock" Type="Edm.Int32"/>
    <Property Name="price_amount" Type="Edm.Decimal" Scale="variable"/>
    <NavigationProperty Name="price_currency" Type="Simple.Currencies">
        <ReferentialConstraint Property="price_currency_code" ReferencedProperty="code"/>
    </NavigationProperty>
    <Property Name="price_currency_code" Type="Edm.String" MaxLength="3"/>
    <NavigationProperty Name="supplier" Type="Simple.Suppliers">
        <ReferentialConstraint Property="supplier_ID" ReferencedProperty="ID"/>
    </NavigationProperty>
    <Property Name="supplier_ID" Type="Edm.Int32"/>
</EntityType>
```

with a `NavigationProperty` of `supplier`.

However, the `Suppliers` entity type is a little simpler at this point, with no
navigation properties expressed in this OData context:

```xml
<EntityType Name="Suppliers">
    <Key>
       <PropertyRef Name="ID"/>
    </Key>
    <Property Name="ID" Type="Edm.Int32" Nullable="false"/>
    <Property Name="company" Type="Edm.String"/>
</EntityType>
```

There's nothing yet in the CDS model at this point that would cause a
navigation property to be made present in this entity type! Let's address that
next.

### Consider the cardinality and association type needed

Remembering that a supplier can have more than one product, we cannot use the
same association type as before.

Fortunately there is also the [to-many
association](https://cap.cloud.sap/docs/guides/domain-modeling#to-many-associations).
This is based on the
[one-to-many](https://en.wikipedia.org/wiki/One-to-many_(data_model))
relationship, where the to-many part is denoted by the N, which represents
"zero or more":

```text
+-----+  N:1  +-----+
|  A  |<----->|  B  |
+-----+       +-----+
```

(where `A` is `Products` and `B` is `Suppliers`).

In contrast to the managed to-one association we used from `Products` ->
`Suppliers`, this to-many association is unmanaged, in the sense that we must
supply some information that will determine how the relationships should be
determined, how the queries should traverse the objects at the persistence
layer. That means providing an `on` clause that describes a join condition.

### Add the association

👉 Add this to-many association as a new element `products` in the `Suppliers`
entity, like this:

```cds
entity Products : cuid {
  name     : String;
  stock    : Integer;
  price    : Price;
  supplier : Association to Suppliers;
}

entity Suppliers : cuid {
  company  : String;
  products : Association to many Products
               on products.supplier = $self;
}
```

Here's how to think about this `on` condition `products.supplier = $self`:

- `products` refers to the `Suppliers:products` element
- `supplier` refers to the `Products:supplier` entity
- `$self` refers to the given `Suppliers` entity instance

```text
    entity Products : cuid {
      name     : String;
      stock    : Integer;
      price    : Price;
 +--> supplier : Association to Suppliers;
 |  }                               |
 |             +--------------------+
 |             |
 |             V
 |  entity Suppliers : cuid {
 |    company  : String;
 +--- products : Association to many Products
         ^         on products.supplier = $self;
    }    |            -----------------
         |                    |
         +--------------------+
```

### Check the CSV header requirements

Has this new to-many association caused any changes at the CSV header level?

👉 Use the `cds add data` command again, but this time with the `--out` option
to supply a different ("throwaway") target directory for the CSV file
generation:

```bash
mkdir /tmp/tempcsv \
  && cds add data --out /tmp/tempcsv \
  && head /tmp/tempcsv/workshop-*.csv
```

As we can see from what the last command in this chain produces:

```text
==> /tmp/tempcsv/workshop-Products.csv <==
ID,name,stock,price_amount,price_currency_code,supplier_ID
==> /tmp/tempcsv/workshop-Suppliers.csv <==
ID,company
```

there are no "artificially constructed" (managed) header fields beyond what was
already there in the form of `supplier_ID`.

From a modelling perspective, this is all we need. From a data loading
perspective, this is all we need too.

But what about from a service exposure and data retrieval perspective? Let's see.

### Re-visit the supplier to products navigation

👉 First, check the service metadata again at this URL:
<http://localhost:4004/odata/v4/simple/$metadata>, and find the `Suppliers`
entity type.

It should now look like this:

```xml
<EntityType Name="Suppliers">
    <Key>
        <PropertyRef Name="ID"/>
    </Key>
    <Property Name="ID" Type="Edm.Int32" Nullable="false"/>
    <Property Name="company" Type="Edm.String"/>
    <NavigationProperty Name="products" Type="Collection(Simple.Products)" Partner="supplier"/>
</EntityType>
```

Great - the CAP server, specifically the support for OData service provision
and handling, has made a `NavigationProperty` element available for the
`Suppliers` entity type. Note that our previous "guess" as to what this would
be named, "products", was correct, i.e. based on the `products` element in the
`Suppliers` entity:

```cds
entity Suppliers : cuid {
  company  : String;
  products : Association to many Products
               on products.supplier = $self;
}
```

👉 Try that previous suppliers to products navigation again with this URL:
<http://localhost:4004/odata/v4/simple/Suppliers?$expand=products($select=name)>
(to keep things brief a `$select` query option has been applied to the expanded
navigation property).

This time we should see something like this:

```json
{
  "@odata.context": "$metadata#Suppliers",
  "value": [
    {
      "ID": 1,
      "company": "Exotic Liquids",
      "products": [
        {
          "name": "Chai",
          "ID": 1
        },
        {
          "name": "Chang",
          "ID": 2
        },
        {
          "name": "Aniseed Syrup",
          "ID": 3
        }
      ]
    },
    {
      "ID": 2,
      "company": "New Orleans Cajun Delights",
      "products": [
        {
          "name": "Chef Anton's Cajun Seasoning",
          "ID": 4
        },
        {
          "name": "Chef Anton's Gumbo Mix",
          "ID": 5
        }
      ]
    },
    {
      "ID": 3,
      "company": "Grandma Kelly's Homestead",
      "products": [
        {
          "name": "Grandma's Boysenberry Spread",
          "ID": 6
        }
      ]
    }
  ]
}
```

So not only does this to-many association bring about the requisite navigation
property in the exposed OData service, but also the appropriate query is being
made at the persistence layer to resolve the data request represented by the
OData query operation transmitted.

> We're still defaulting to an in-memory SQLite persistence layer, but that's
> still a very valid and capable database platform, and well suited to
> design-time development.

> If you're extra curious, you can see the `SELECT` statement that is generated
> to resolve this query, by setting the `DEBUG` environment variable to `sql`
> before starting the `cds watch` command, like this, for example: `DEBUG=sql
> cds watch`.
>
> If you do, you'll see something like this:
>
> [odata] - GET /odata/v4/simple/Suppliers { '$expand': 'products($select=name)' }
> [sql] - BEGIN
> [sql] - SELECT json_insert('{}','$."ID"',ID,'$."company"',company,'$."products"',products->'$') as _json_ FROM (SELECT "$S".ID,"$S".company,(SELECT jsonb_group_array(jsonb_insert('{}','$."name"',name,'$."ID"',ID)) as _json_ FROM (SELECT "$p".name,"$p".ID FROM Simple_Products as "$p" WHERE "$S".ID = "$p".supplier_ID)) as products FROM Simple_Suppliers as "$S" ORDER BY "$S".ID ASC LIMIT ?) [ 1000 ]
> [sql] - COMMIT
>
> As this is quite hard to read, here's that `SELECT` statement nicely formatted:
>
> ```sql
> SELECT Json_insert('{}', '$."ID"', id, '$."company"', company, '$."products"',
>        products
>               -> '$') AS _json_
> FROM   (SELECT "$S".id,
>                "$S".company,
>                (SELECT Jsonb_group_array(Jsonb_insert('{}', '$."name"', name,
>                                          '$."ID"',
>                                          id)) AS
>                        _json_
>                 FROM   (SELECT "$p".name,
>                                "$p".id
>                         FROM   simple_products AS "$p"
>                         WHERE  "$S".id = "$p".supplier_id)) AS products
>         FROM   simple_suppliers AS "$S"
>         ORDER  BY "$S".id ASC
>         LIMIT  ?) [ 1000 ] 
> ```

