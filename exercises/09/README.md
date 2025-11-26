# 09 - Explore service definitions

In this exercise we'll begin to understand what the service layer is and what
it's for, by extending the rudimentary definition we already have in there.

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

## Consider what we have so far

Purely as a means to an end - to enable us to make OData calls to understand
the effect of our domain model design as we worked through the exercises thus
far- we have a very simple service definition in `srv/simple.cds`.

👉 Open the `srv/simple.cds` file in the editor and take a look.

This is what you should see:

```cds
using workshop from '../db/schema';

service Simple {
  entity Products  as projection on workshop.Products;
  entity Suppliers as projection on workshop.Suppliers;
}
```

## Add one more pass-through entity

At the end of the previous exercise we deferred the final tyre-kicking of our
contained-in relationship, as we didn't yet have the `Orders` entity exposed
for in the OData service that represented the current service definition in
`srv/simple.cds`.

👉 Do that now; start by adding another projection within the `Simple` service
definition, so it looks like this:

```cds using workshop from '../db/schema';

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
artifact, in our case a fully fledged OData service with full support for all
OData operations (create, read, update, delete and query) out of the box:

```log [cds] - serving Simple {
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

As the CAP server should still be running in watch mode, it will notice this
change and restart, whereupon we should see the custom path:

```log
[cds] - serving Simple {
  at: [ '/simple' ],
  decl: 'srv/simple.cds:5',
  impl: 'node_modules/@sap/cds/srv/app-service.js'
}
```

Now we can perform a few experiments on our latest additions to the model
relating to the purchase order construct.

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
> ```xml <Schema xmlns="http://docs.oasis-open.org/odata/ns/edm"
> Namespace="Simple"> ```

We also see that:

- there's an `OnDelete` element which augments the navigation property
  definition, stating (via the `Cascade` value) that related entities will be
  deleted when the containing entity (the order) is deleted; this part of the
  entity data model definition has been added by the CAP server based on the
  composition, and serves to inform consumers on what will happen

> For more information on this `OnDelete` element, see the [relevant section of
> the OData V4 CSDL
> specification](https://docs.oasis-open.org/odata/odata/v4.0/os/part3-csdl/odata-v4.0-os-part3-csdl.html#_Toc372793928).

Additionally we see there's an `Orders_items` entity type:

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
  key values, there are referential contraints that ensure that the "pointer"
  goes to the appropriate target entity instance
  - the value of the `Order_items` element `up__ID` and the value of the target
    `Orders` element `ID` need to match
  - the value of the `Order_items` element product_ID` and the value of the
    target `Products` element `ID` need to match

### Request some OData operations

👉 Copy the two order related CSV files to the `db/data/` directory and check
that the CAP server restarts:

```bash
cp ../exercises/08/workshop-Orders*.csv db/data/
```

The pair of CSV files contain data for a simple initial order with a couple of items.

> On a trivia note, do you know the significance of the order date chosen?

👉 Visit <http://localhost:4004/simple/Orders?$expand=items> to perform an OData query operation to retrieve this order and its corresponding items, which should look something like this:

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

> For a bonus exploration, add an expansion of the `product` navigation property on each item: <http://localhost:4004/simple/Orders?$expand=items($expand=product)>.
