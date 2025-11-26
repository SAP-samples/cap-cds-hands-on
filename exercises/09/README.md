# 09 - Explore service definitions

In this exercise we'll begin to understand what the service layer, by extending
the rudimentary definition we already have in there.

## Understand where services fit

This diagram from the [Service-Centric
Paradigm](https://cap.cloud.sap/docs/guides/providing-services#service-centric-paradigm)
section of the Providing Services topic in Capire illustrates services and
their relation to Consumers, and also to where we've been thus far - at the
Domain Models part:

![service centric paradigm diagram](assets/service-centric-paradigm.png)

In many ways, services are where the rubber meets the road, providing
differently shaped APIs to consumers, in different forms, all based upon the
foundation that is the domain model.

One might think of the domain model (conventionally at the `db/` layer) as
being fairly static, i.e. declarative definitions that form the source of truth
for artifacts in the database and for how queries are resolved at runtime.

In contrast, services (conventionally at the `srv/` layer) are dynamic. They
marshal, constrain, expose and control access to data and functions at the
domain model via cheap, lightweight declarative definitions that describe
facades in different forms.

These facades, or APIs, are dynamic in the sense that they are reified in the
context of wire protocols such as plain HTTP ("REST"), OData and GraphQL. So at
that level there are moving parts. But at a level below there are moving parts
too, built into the framework, to provide full and complete support for the
protocol specific facilities out of the box.

## Consider what we have so far

Purely as a means to an end - to enable us to make OData calls to understand
the effect of our domain model design - we have a very simple service
definition in `srv/simple.cds`.

👉 Open the `srv/simple.cds` file in the editor and take a look.

This is what you should see:

```cds
using workshop from '../db/schema';

service Simple {
  entity Products  as projection on workshop.Products;
  entity Suppliers as projection on workshop.Suppliers;
}
```
