# 08 - Define contained-in relationships with compositions

In the previous exercise we explored relationships between essentially
independent entities: products and suppliers. But there are other
relationships that we can think of as having a "parent-child" style. These
are often referred to more generically as "contained-in" relationships.

In this exercise we'll learn about CAP's great support for such
relationships, in the form of compositions.

## Consider a contained-in relationship

The classic example of such a "contained-in" relationship in ERP is the
document, which has a header and items. Think of the header as the
container, and the items as the containees. If the container is removed,
the containees should be too, i.e. they cannot exist independently.

With compositions, CAP supports such "contained-in" relationships, from a
modelling perspective, and also [from a runtime
perspective](https://cap.cloud.sap/docs/guides/domain-modeling#compositions):

- Deep Insert / Update automatically filling in document structures
- Cascaded Delete is when deleting composition roots
- Composition targets are auto-exposed in service interfaces

> While using regular Association constructs would go some way to modelling
> such relationships, it's the wrong approach, unless we want a whole load of
> extra and unnecessary work and complexity to achieve what CAP provides for us
> out of the box with compositions.

## Model a simple order facility

To illustrate the support and the use of
[compositions](https://cap.cloud.sap/docs/cds/cdl#compositions) in CDL, let's
add a parent-child construct for an order entity.

👉 To the list of entities we have so far in `db/schema.cds`, add the
(deliberately simple) `Orders` entity, paying close attention to how the order
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
> With the `default` keyword for `date` we can specify a default value if
> none is supplied on creation; here, we use the [pseudo
> variable](https://cap.cloud.sap/docs/guides/domain-modeling#pseudo-variables)
> `$now` (we saw this [in a previous exercise
> too](../06#use-the-managed-aspect-for-basic-data-tracking)).

### Consider what's been defined

It's worth
[staring](https://qmacro.org/blog/posts/2017/02/19/the-beauty-of-recursion-and-list-machinery/#initial-recognition)
at this new definition for a moment or two, as there's plenty to think about.
Let's unpack what we see:

- the `Orders` entity gets a simple primary key via our custom local `cuid`
  aspect, just like the other entities
- if no value is supplied for the `date` element in a creation scenario, it
  will be defaulted (to the date of creation)
- the only other element is `items`, described as a `Composition of many`

So far so good. Let's dig in further:

- unlike the Association describing `Suppliers:products` earlier, the target
  composition is not another entity ... it's a structure (`{ ... }`)
- that structure is an [anonymous inline aspect](https://cap.cloud.sap/docs/guides/domain-modeling#composition-of-aspects)

What does that anonymous inline aspect describe? It describes the structure of
the child, the containee - in this case, the structure of the order item.

An order item, the existence of which cannot be outside the context of a parent
order, will usually have a key made up of two elements: one for the parent, and one for the
item (the child). Let's look at all the elements of this structure to see where
they are:

- there's a `pos` element representing the item via its unique position (item
  number)
- there are also a couple of basic order item elements, one being a managed
  to-one association to the `Products` entity, the other being a simple integer
  representing the order quantity for the given product

So where's the other key element - the one for the parent order?

There isn't one defined explicitly ... the primary key for the parent order is
implicit. This is another form of "managed" relationship, where the intent is
of primary concern, and the implementation left to the system.

Let's dig in to see this for ourselves.

### Take a look at the effect of the composition

👉 First, have a look at what gets generated in CSN:

```bash
cds compile --to yaml db/schema.cds
```

👉 Pick out the relevant parts of the structure, to see:

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

There's an almost overwhelming amount to dwell on here. Let's focus on just a
few important parts:

- the anonymous inline structure is indeed an `aspect` as we can see from the
  `targetAspect` of the `items` property for the `workshop.Orders` definition
- a new entity `workshop.Orders.items` has been generated automatically, with the name being constructed from the namespace and the source entity and element
- the link between the parent (see the `on` condition) and the child (the
  generated entity) is via an element called `up_`. This generated element is
  part of how this composition-based relationship is "managed" for us, in a
  similar way to how the foreign key `supplier_ID` [was generated in the
  previous exercise](07#define-the-relationship).

> For more on how this fits together, see [Modelling contained-in relationships
> with compositions in
> CDS](https://qmacro.org/blog/posts/2025/10/14/modelling-contained-in-relationships-with-compositions-in-cds/).

### Take a look from a CSV headers point of view

👉 To drive home the appearance of this `up_` element, ask for the generation
of CSV files with headers, specifically for this new `Orders` definition (using
the `--filter` option):

```bash
cds add data --filter Orders \
  && head db/data/workshop-Orders*.csv
```

This will produce something like this:

```log
adding data
adding headers only, use --records to create random entries
  creating db/data/workshop-Orders.csv
  creating db/data/workshop-Orders.items.csv

successfully added features to your project
==> db/data/workshop-Orders.csv <==
ID,date
==> db/data/workshop-Orders.items.csv <==
up__ID,pos,product_ID,quantity
```

The managed foreign key for the order item records has been constructed in the
same way as `supplier_ID` before - the source element name `up_` and the
target's key element `ID`, joined with an underscore.

In the next exercise we'll explore this construct with some data.

So far, so good!

---

[Next](../09/)
