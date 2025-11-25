# 08 - Define contained-in relationships with compositions

In the previous exercise we explored relationships between essentially
independent entities: products and suppliers. But there are other
relationships that we can think of as having a "parent-child" style. This
is often referred to more generically as a "contained-in" relationship.

In this exercise we'll learn about CAP's support here.

## Consider a contained-in relationship

The classic example of such a "contained-in" relationship in ERP is the
"document", which has a header and items. Think of the header as the
container, and the items as the containees. If the container is removed,
the containees should be too, i.e. they cannot exist independently.

With "Compositions", CAP supports such "contained-in" relationships, from a
modelling perspective, and also [from a runtime
perspective](https://cap.cloud.sap/docs/guides/domain-modeling#compositions):

- Deep Insert / Update automatically filling in document structures
- Cascaded Delete is when deleting Composition roots
- Composition targets are auto-exposed in service interfaces

> While using regular Association constructs would go some way to modelling
> such relationships, it's the wrong way to go, unless we want a whole load of
> extra, unnecessary work and complexity to achieve what CAP provides for us
> out of the box with Compositions.

## Model a simple purchase order facility

To illustrate the support and the use of
[Compositions](https://cap.cloud.sap/docs/cds/cdl#compositions) in CDL, let's
add a parent-child construct for a purchase order entity.

👉 To the list of entities we have so far in `db/schema.cds`, add the
(deliberaely simple) `Orders` entity, paying close attention to how the order
items are modelled:

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

entity Orders : cuid {
  date  : Date default $now;
  items : Composition of many {
            key pos      : Integer;
                product  : Association to Products;
                quantity : Integer;
          }
}
```

> [!NOTE]
> With the `default` keyword we can specify a default value if none is supplied
> on creation; here, we use the [pseudo
> variable](https://cap.cloud.sap/docs/guides/domain-modeling#pseudo-variables)
> `$now` (we saw this [in a previous exercise
> too](../06#use-the-managed-aspect-for-basic-data-tracking)).

### Take a moment to think about what's been defined

It's worth
[staring](https://qmacro.org/blog/posts/2017/02/19/the-beauty-of-recursion-and-list-machinery/#initial-recognition)
at this new definition, as there's plenty that presents itself. Let's unpack
what we see:

- the `Orders` entity gets a simple primary key via our custom local `cuid`
  aspect, just like the other entities
- if no value is supplied for the `date` element in a creation scenario, it
  will be defaulted (to the date of creation)
- the only other element is `items`, described as a `Composition of many`

So far so good. But note:

- the target of the composition is not, like the association describing
  `Suppliers:products` earlier, another entity ... it's a structure (`{ ... }`)
- that structure is an [anonymous inline aspect](https://cap.cloud.sap/docs/guides/domain-modeling#composition-of-aspects)

What does that anonymous inline aspect describe? It describes the structure of
the child, the containee - in this case, the structure of the order item.
Clearly, an order item, the existence of which cannot be outside the context of
aparent order, will usually have two key elements. Let's look at the elements
of this structure to see where they are:

- there's a `pos` element representing the item position (item number)
- there are also a couple of basic order item elements, one being a managed
  to-one association to the `Products` entity, the other being a simple integer
  representing the order quantity for the given product

So where's the other key element - the one for the parent order?

- there isn't one defined explicitly ... the primary key for the parent order
  is implicit, part of how this is another sort of "managed" relationship

Let's dig in to see this for ourselves.

### Dig in to the effect of the composition

👉 First, have a look at what gets generated in CSN:

```bash
cds compile --to yaml db/schema.cds
```

Picking out the relevant parts of the structure, we see:

```yaml
namespace: workshop
definitions:
  workshop.cuid:
    {
      kind: aspect,
      elements: { ID: { key: true, type: cds.Integer } },
    }
  workshop.Orders:
    kind: entity
    includes: [workshop.cuid]
    elements:
      ID: { key: true, type: cds.Integer }
      date: { type: cds.Date, default: { ref: [$now] } }
      items:
        type: cds.Composition
        cardinality: { max: "*" }
        targetAspect:
          elements:
            pos: { key: true, type: cds.Integer }
            product:
              {
                type: cds.Association,
                target: workshop.Products,
                keys: [{ ref: [ID] }],
              }
            quantity: { type: cds.Integer }
        target: workshop.Orders.items
        on: [{ ref: [items, up_] }, "=", { ref: [$self] }]
  workshop.Orders.items:
    kind: entity
    elements:
      up_:
        key: true
        type: cds.Association
        cardinality: { min: 1, max: 1 }
        target: workshop.Orders
        keys: [{ ref: [ID] }]
        notNull: true
      pos: { key: true, type: cds.Integer }
      product:
        {
          type: cds.Association,
          target: workshop.Products,
          keys: [{ ref: [ID] }],
        }
      quantity: { type: cds.Integer }
```

There's an almost overwhelming amount to dwell on here. Let's pick out just a
few important parts:

- the anonymous inline structure is indeed an `aspect` (see the `targetAspect`
  of the `items` property for the `workshop.Orders` definition)
- a new entity `workshop.Orders.items` has been generated automatically
- the link between the parent (see the `on` condition) and the child (the
  generated entity) is via an element called `up_`. This generated element is
  part of how this composition-based relationship is "managed" for us, in a
  similar way to how the foreign key `supplier_ID` [was generated in the
  previous exercise](07#define-the-relationship).

> For more on how this fits together, see [Modelling contained-in relationships
> with compositions in
> CDS](https://qmacro.org/blog/posts/2025/10/14/modelling-contained-in-relationships-with-compositions-in-cds/).

