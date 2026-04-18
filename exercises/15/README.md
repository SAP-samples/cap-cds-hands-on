# 15 - Shift left, avoid procedural code

In this last exercise, we'll double down on the declarative approach we've
explored thus far, and look to see how else we can use the same approach to
avoid custom code, and to embrace more powerful facilities that CDS modelling
has to offer, linked to a key reason to use CAP - [the code is in the
framework, not outside of
it](https://qmacro.org/blog/posts/2024/11/07/five-reasons-to-use-cap/#1-the-code-is-in-the-framework-not-outside-of-it).

## Replace the function with a declarative infix filter

The function we chose to specify and implement in [exercise 11](../11/) was
deliberately simple, of course. But did you know that we don't even need custom
code for such a facility?

One of the best features of developing with the CAP framework is that it allows
us to push out logic and mechanics to the extremities:

- upwards to the declarative surface area of our domain model (and related
  service definitions)
- downwards to the persistence layer where complex queries can be handled
  directly and natively by the database systems

Let's make available that same feature (the listing of products that are out of
stock) without having to write a single line of custom code.

### Remove the custom implementation

👉 Start out by deleting the `srv/services.js` file that we created when we
[provided the implementation in exercise
11](../11/README.md#provide-an-implementation) as we don't need it any more.
Deleting code (while the app or service still does what you want) is a much
underrated power move! For reference and comparison with what we're about to
define, here's the salient part of the custom code (although the point also
is that in order for this custom code to exist in the right calling context,
more code was needed):

```javascript
this.on('outOfStockProducts', async () => {
    return await SELECT.from(Products).where({ stock: 0 })
})
```

#### Redefine the facility as a projection

👉 Next, remove the `outOfStockProducts()` function definition from the
`Simple` service, replacing it with another entity projection called
`OutOfStockProducts` as shown below. Also, add the annotation
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

![OutOfStockProducts entity exposed](assets/outOfStockProducts-entity.png)

👉 Select that entity link to get to the entityset resource, which should
reflect the same data as the function did: the products "Chai" and "Chef
Anton's Gumbo Mix".
