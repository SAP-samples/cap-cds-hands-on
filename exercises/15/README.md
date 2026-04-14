# 11 - Shift left, avoid procedural code

TAKEN OUT OF THE END OF EXERCISE 11

## Replace the function with a declarative infix filter

The function we chose to implement was deliberately simple, of course. But did
you know that we don't even need a function for such a facility?

One of the best features of developing with the CAP framework is that it allows
us to push out logic and mechanics to the extremities:

- upwards to the declarative surface area of our domain model (and related
  service definitions)
- downwards to the persistence layer where complex queries can be handled
  directly and natively by the database systems

To round off this exercise, let's make that same feature available (the listing
of products that are out of stock) without having to write a single line of
custom code.

### Remove the custom implementation

👉 Start out by deleting the `srv/services.js` file as we don't need it any more.

#### Redefine the facility as a projection

👉 Next, remove the `outOfStockProducts()` function definition from the
`Simple` service, replacing it with another entity projection called
`OutOfStockProducts` as shown. Also, add the annotation
`@cds.redirection.target` to the `Products` entity projection. Once you're
done, the `Simple` service definition should look like this:

```cds
@protocol: 'odata'
@path    : '/simple'
service Simple {
  @cds.redirection.target
  entity Products           as projection on workshop.Products;

  entity Suppliers          as projection on workshop.Suppliers;
  entity Orders             as projection on workshop.Orders;
  entity OutOfStockProducts as projection on workshop.Products[stock <= 0];
}
```

#### Examine what we've done

What have we done here? Importantly, we have:

- moved from a procedural approach that required custom business logic,
  to a purely declarative one using the power of CAP's domain modelling
  language CDL
- that power specifically here is the `[stock <= 0]` part which is an [infix
  filter](https://cap.cloud.sap/docs/cds/cdl#publish-associations-with-filter)

Unrelated directly to the use of an infix filter, and more related to the fact
that we have defined a second projection on the same base entity
(`workshop.Products`), we have also:

- added the annotation
  [@cds.redirection.target](https://cap.cloud.sap/docs/cds/cdl#using-cds-redirection-target-annotations)
  to help the compiler resolve any ambiguity between the two possible
  destinations for association based relationships

Now that we've made these changes and got rid of the `srv/services.js` file,
make sure the CAP server has restarted and visit the CAP server home page again
at <http://localhost:4004/>, where this new resource is exposed, as an entity
this time of course, and not as a function:

![OutOfStockProducts entity exposed](assets/OutOfStockProducts-entity.png)

👉 Select that entity link to get to the entityset resource, which should
reflect the same data as the function did: the products "Chai" and "Chef
Anton's Gumbo Mix".
