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

While we touched on this pattern in a previous exercise when we [first created
the contents of this
file](../03#rework-the-content-of-servicescds-into-the-service-and-persistence-layers),
we didn't dwell on it. Let's think about what's happening here.

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

The service definition itself by default will be presented as an OData service,
but we can be explicit and use [the appropriate
annotation](https://cap.cloud.sap/docs/node.js/cds-serve#protocol). While we're
at it, we can also use an annotation to tell the CAP server to make the OData
service available on a different
[path](https://cap.cloud.sap/docs/node.js/cds-serve#path) to the default (shown
in the `at:` property in the log output above). Let's do that.

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

