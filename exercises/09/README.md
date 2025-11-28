# 09 - Try out deep inserts and cascaded deletes

In this exercise we'll round out our exploration of relationships by taking
our new parent-child order construct for a data test drive.

## Add one more pass-through entity to the service definition

At the end of the previous exercise we deferred the final tyre-kicking of our
contained-in relationship, as we didn't yet have the `Orders` entity exposed
for in the OData service that represented the current service definition in
`srv/simple.cds`.

👉 Do that now; start by adding another projection within the `Simple` service
definition, so it looks like this:

```cds
using workshop from '../db/schema';

service Simple {
  entity Products  as projection on workshop.Products;
  entity Suppliers as projection on workshop.Suppliers;
  entity Orders    as projection on workshop.Orders;
}
```

We touched on this pattern in a previous exercise when
we [first created the contents of this
file](../03#rework-the-content-of-servicescds-into-the-service-and-persistence-layers),
but we didn't dwell on it. Let's think about what's
happening here.

While the declaration in this file is static, the outcome is a dynamic
artifact, in our case a full fat OData service with full support for all
OData operations (Create, Read, Update, Delete and Query) out of the box:

```log
[cds] - serving Simple {
  at: [ '/odata/v4/simple' ],
  decl: 'srv/simple.cds:3',
  impl: 'node_modules/@sap/cds/srv/app-service.js'
}
```

This is why we were able to quickly try out such operations on our fledgling
data model.

The service definition itself by default will be
presented as an OData service, but for the sake of
illustration we can be explicit and use [the appropriate
annotation](https://cap.cloud.sap/docs/node.js/cds-serve#protocol).
While we're at it, we can also use an annotation to tell
the CAP server to make the OData service available on a
different
[path](https://cap.cloud.sap/docs/node.js/cds-serve#path)
to the default (shown in the `at:` property in the log
output above). Let's do that.

> Annotations in general will be covered in a future exercise.

👉  Add the annotations as shown:

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

> There's a short form of the `@protocol: 'odata'` annotation: `@odata` (see the [cds.protocols](https://cap.cloud.sap/docs/node.js/cds-serve#cds-protocols) section in Capire).

As the CAP server should still be running in watch mode, it will notice this
change and restart, whereupon we should see the custom path `/simple`:

```log
[cds] - serving Simple {
  at: [ '/simple' ],
  decl: 'srv/simple.cds:5',
  impl: 'node_modules/@sap/cds/srv/app-service.js'
}
```

Now we can perform a few experiments on our latest additions to the model
relating to the order construct.

## Explore the order construct as it manifests in the service

Now that we've added `Orders` as a projection, let's have a quick explore.

### Look at the service's metadata document

👉 First, have a look at the OData service's metadata document (at
<http://localhost:4004/simple/$metadata>), and pick out the relevant parts for
the order construct.

The `Orders` entity type should be defined like this:

```xml
<EntityType Name="Orders">
  <Key>
    <PropertyRef Name="ID"/>
  </Key>
  <Property Name="ID" Type="Edm.Int32" Nullable="false"/>
  <Property Name="date" Type="Edm.Date"/>
  <NavigationProperty Name="items" Type="Collection(Simple.Orders_items)" Partner="up_">
    <OnDelete Action="Cascade"/>
  </NavigationProperty>
</EntityType>
```

👉 Take a moment to study the detail here:

- there's a navigation property `items` which corresponds to the `items`
  element in our model
- that navigation property is described as a collection of another entity type
  `Orders_items`

> The `Simple` name prefix is just there because the service name permeates the
> metadata as the namespace of the entire schema:
>
> ```xml
> <Schema xmlns="http://docs.oasis-open.org/odata/ns/edm" Namespace="Simple">
> ```

We also see that:

- there's an `OnDelete` element which augments the navigation property
  definition, stating (via the `Cascade` value) that related entities (items)
  will be deleted when the containing entity (the order) is deleted; this part
  of the entity data model definition has been added by the CAP server based on
  the composition, and serves to inform consumers about what will happen

> For more information on this `OnDelete` element, see the [relevant section of
> the OData V4 CSDL
> specification](https://docs.oasis-open.org/odata/odata/v4.0/os/part3-csdl/odata-v4.0-os-part3-csdl.html#_Toc372793928).

Additionally we see the `Orders_items` entity type:

```xml
<EntityType Name="Orders_items">
  <Key>
    <PropertyRef Name="up__ID"/>
    <PropertyRef Name="pos"/>
  </Key>
  <NavigationProperty Name="up_" Type="Simple.Orders" Nullable="false" Partner="items">
    <ReferentialConstraint Property="up__ID" ReferencedProperty="ID"/>
  </NavigationProperty>
  <Property Name="up__ID" Type="Edm.Int32" Nullable="false"/>
  <Property Name="pos" Type="Edm.Int32" Nullable="false"/>
  <NavigationProperty Name="product" Type="Simple.Products">
    <ReferentialConstraint Property="product_ID" ReferencedProperty="ID"/>
  </NavigationProperty>
  <Property Name="product_ID" Type="Edm.Int32"/>
  <Property Name="quantity" Type="Edm.Int32"/>
</EntityType>
```

👉 Take a moment to study the detail here:

- there are two key properties as we'd expect and thought about already in the
  previous exercise: `up__ID` (the order ID) and `pos` (the item number)
- the target of the `up_` navigation property is described not as a
  `Collection( ... )` this time, but as a (single) `Orders` entity type
- for both the navigation properties, each of which involve the use of foreign
  key values, there are referential contraints that ensure that the navigation
  leads to the appropriate target entity instance
  - the value of the `Order_items` element `up__ID` and the value of the target
    `Orders` element `ID` need to match
  - the value of the `Order_items` element `product_ID` and the value of the
    target `Products` element `ID` need to match

### Request some OData operations

👉 Copy the two order related CSV files to the `db/data/` directory and check
that the CAP server restarts:

```bash
cp ../exercises/09/assets/workshop-Orders*.csv db/data/
```

The pair of CSV files contain data for a simple initial order with a couple of items.

> On a trivia note, do you know the significance of the order date chosen?

#### Make an OData query with expanded navigation

👉 Visit <http://localhost:4004/simple/Orders?$expand=items> to perform an
OData query operation to retrieve this order and its corresponding items, which
should look something like this:

```json
{
  "@odata.context": "$metadata#Orders",
  "value": [
    {
      "ID": 1,
      "date": "1992-07-06",
      "items": [
        {
          "up__ID": 1,
          "pos": 1,
          "product_ID": 1,
          "quantity": 10
        },
        {
          "up__ID": 1,
          "pos": 2,
          "product_ID": 2,
          "quantity": 10
        }
      ]
    }
  ]
}
```

> For a bonus exploration, add an nested expansion of the `product` navigation
> property on each item, and a further nested expansion to get the currency details:
> <http://localhost:4004/simple/Orders?$expand=items($expand=product($expand=price_currency))>.

#### Make an OData create operation with header and items

Now it's time to try an OData create operation, supplying a JSON payload representing
a new order with three items. A so-called "deep-insert".

The data is in a file called `order.json` and looks like this:

```json
{
  "ID": 100,
  "items": [
    {
      "up__ID": 100,
      "pos": 10,
      "product_ID": 1,
      "quantity": 5
    },
    {
      "up__ID": 100,
      "pos": 20,
      "product_ID": 2,
      "quantity": 5
    },
    {
      "up__ID": 100,
      "pos": 30,
      "product_ID": 3,
      "quantity": 5
    }
  ]
}
```

👉 Make the request now:

```bash
curl \
  --include \
  --header 'Content-Type: application/json' \
  --data @../exercises/09/assets/order.json \
  --url http://localhost:4004/simple/Orders
```

The output should look something like this:

```log
HTTP/1.1 201 Created
X-Powered-By: Express
OData-Version: 4.0
location: Orders(100)
Content-Type: application/json; charset=utf-8
Content-Length: 240

{"@odata.context":"$metadata#Orders/$entity","ID":100,"date":"2025-11-26","items":[{"up__ID":100,"pos":10,"product_ID":1,"quantity":5},{"up__ID":100,"pos":20,"product_ID":2,"quantity":5},{"up__ID":100,"pos":30,"product_ID":3,"quantity":5}]}
```

All the signs from this response show that the creation of this order was
successful, including:

- the appropriate [201 HTTP status code](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Status/201)
- the corresponding `Location` header that accompanies a 201 response, showing
  the address `Orders(100)` of the new resource

> The address `Orders(100)` doesn't begin with a forward slash, meaning it's relative, becoming `/simple/Orders(100)` as the complete URL path.

👉 Check for yourself, either by revisiting the previous URL
<http://localhost:4004/simple/Orders?$expand=items> or by following the path to
this specific new resource as pointed to by the `Location` header, i.e.
<http://localhost:4004/simple/Orders(100)?$expand=items>

### Make an OData delete operation and check that cascading deletes happen

Curious to see the effect of a successful cascading delete?

Let's revisit [a
previous bonus activity](../02#deployments-bonus) and deploy the data model to
a persistent database file. Then we can delete the single "1992-07-06" order with
an OData delete operation, and check in the database that the items have been
deleted too.

👉 Stop the currently running CAP server (with `Ctrl-C`).

👉 Now deploy the model to SQLite, this time without specifying a name for the
actual database file (previously we specified `test.db`), so that the default
of `db.sqlite` will be used (so that we can benefit from convention over configuration again):

```bash
cds deploy --to sqlite
```

This will emit something similar to this:

```log
  > init from db/data/workshop-Suppliers.csv
  > init from db/data/workshop-Products.csv
  > init from db/data/workshop-Orders.items.csv
  > init from db/data/workshop-Orders.csv
  > init from db/data/sap.common-Currencies.texts.csv
  > init from db/data/sap.common-Currencies.csv
/> successfully deployed to db.sqlite
```

We need to tell the CAP server to use a persistent file (rather than in-memory), and can do that
temporarily with a configuration parameter.

👉 Specify the configuration parameter by adding the following to a new file
called `.env` in the project root:

```env
cds.requires.db.kind=sqlite
```

Let's now "monitor" the item data in this database file. Remember that we have
[a single order](#request-some-odata-operations) in our initial data CSV files,
and that is what will be served but also what has been deployed to `db.sqlite`
(note the corresponding `> init from db/data/workshop-Orders...` lines in the
log output just above).

👉 Use the SQLite CLI to list the order items:

```bash
sqlite3 db.sqlite 'select * from workshop_Orders_items;'
```

This should show the two records representing the items:

```text
1|1|1|10
1|2|2|10
```

> Remember, these two item records in the database came directly from the
> [workshop-Orders.items.csv](assets/workshop-Orders.items.csv) initial data
> CSV file.

👉 Now start up the CAP server, specifying that you want to see debug level
output for the SQL activities:

```bash
DEBUG=sql cds watch
```

and note in the log output that a connection is made to the persistent
`db.sqlite` file this time rather than an in-memory store:

```log
[cds] - connect to db > sqlite { url: 'db.sqlite' }
```

OK, we're ready to delete the single order, the parent of the items that are
"contained-in" it, the items that should also be deleted due to the cascade
delete action.

👉 Send an OData delete operation specifying this single order (which has an
`ID` of `1`):

```bash
curl \
  --include \
  --request DELETE \
  --url localhost:4004/simple/Orders/1
```

This should produce something like this:

```log
HTTP/1.1 204 No Content
X-Powered-By: Express
OData-Version: 4.0
Connection: keep-alive
Keep-Alive: timeout=5
```

and you should see a corresponding entry in the CAP server's log output, plus
some extra SQL debug log records which show the items being deleted in the same
transaction as the order header itself:

```log
[odata] - DELETE /simple/Orders/1
[sql] - BEGIN
[sql] - DELETE FROM workshop_Orders_items as "$i" WHERE exists (SELECT 1 as "1" FROM workshop_Orders as "$O" WHERE "$O".ID = "$i".up__ID and ("$O".ID) in (SELECT "$O2".ID FROM Simple_Orders as "$O2" WHERE "$O2".ID = ?)) [ 1 ]
[sql] - DELETE FROM workshop_Orders as "$O" WHERE ("$O".ID) in (SELECT "$O2".ID FROM Simple_Orders as "$O2" WHERE "$O2".ID = ?) [ 1 ]
[sql] - COMMIT
```

👉 Check the items again via the SQLite CLI:

```bash
sqlite3 db.sqlite 'select * from workshop_Orders_items;'
```

This should show that there are now ... no item records!

👉 One last thing: after stopping
the currently running CAP server (with `Ctrl-C`), remove the `.env`-based
configuration file and restart the server:

```bash
rm .env; cds watch
```

This should bring us back to in-memory persistence:

```log
[cds] - connect to db > sqlite { url: ':memory:' }
  > init from db/data/workshop-Suppliers.csv
  > init from db/data/workshop-Products.csv
  > init from db/data/workshop-Orders.items.csv
  > init from db/data/workshop-Orders.csv
  > init from db/data/sap.common-Currencies.texts.csv
  > init from db/data/sap.common-Currencies.csv
/> successfully deployed to in-memory database.
```

Great stuff. In the [final
part](../../#part-4---exposing-models-via-services---interfaces-for-the-outside-world)
of this workshop, we'll turn our attention to the service layer and start to
explore what else we can do there.
