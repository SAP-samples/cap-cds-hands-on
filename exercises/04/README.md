# 04 - Design and use custom types for a richer entity definition

In this exercise we'll learn about custom types in CDL, and how
and when to employ them (and when not to).

## Consider the existing product element definitions

Right now we have an extremely simple `Products` entity, the
definition for which is in `db/schema.cds`, which contains:

```cds
namespace workshop;

entity Products {
  key ID    : Integer;
      name  : String;
      stock : Integer;
}
```

Note how the elements are defined using simple built-in types `Integer` and `String`.

> "Element" is the term in CDS modelling for what we might call a "property" or
> "field" in other contexts.

### Explore definition abstraction

There are some schools of thought that would promote the use of
custom types for even these scalar elements, like this:

```cds
namespace workshop;

type Stock : Integer;

entity Products {
  key ID    : Integer;
      name  : String;
      stock : Stock;
}
```

> There's a `define` keyword that can be used here, like this:
>
> ```cds
> define type Stock : Integer;
> ```
>
> but it's optional and usually left off. The same goes for `define
> entity` too, for that matter.

This looks neat and has an academic and abstract appeal, especially perhaps to
those schooled in ABAP development in the context of the all important Data
Dictionary, where there are Domains, Data Elements and Data Types supplying metadata
at different layers, bringing about this kind of relationship:

```text
Field <- Data Element <- Domain <- Type
```

### Keep it simple

But on the whole this is considered bad practice in CAP, where there is a fresh
approach to design and no (need for a) Data Dictionary. In domain
modelling terms, CAP [encourages the KISS
approach](https://cap.cloud.sap/docs/guides/domain-modeling#keep-it-simple-stupid).

This custom type `Stock` merely causes us to have to think harder to understand what
we're looking at:

```text
stock <- Stock <- Integer
```

than if we'd simply had:

```text
stock <- Integer
```

That said, there are some circumstances where types add value, such as
when elements belong together.

## Add price information

Let's add price information. While a price is typically represented
as a decimal, it is meaningless without a currency.

What does good look like here? Well, it depends. But for the sake of
learning about types, let's explore a custom [structured
type](https://cap.cloud.sap/docs/cds/cdl#structured-types).

### Use an ad hoc structure

👉 Add a new element `price` described by a structure like this:

```cds
namespace workshop;

entity Products {
  key ID    : Integer;
      name  : String;
      stock : Integer;
      price : {
        amount   : Decimal;
        currency : String;
      }

}
```

👉 Look at what this turns into from a CSN point of view:

```bash
cds compile --to yaml srv/ # specifying the entire srv/ dir
```

This shows us:

```yaml
namespace: workshop
definitions:
  workshop.Products:
    kind: entity
    elements:
      ID: { key: true, type: cds.Integer }
      name: { type: cds.String }
      stock: { type: cds.Integer }
      price:
        {
          elements:
            {
              amount: { type: cds.Decimal },
              currency: { type: cds.String },
            },
        }
meta: { creator: CDS Compiler v6.4.6, flavor: inferred }
$version: 2.0
```

This anonymous type structure is effectively ad hoc, as it cannot be
reused anywhere else we might want to have an element representing a monetary 
value (that is, without repeating it each time).

### Use a named type

👉 To address this, declare a named custom type and use that, like this:

```cds
namespace workshop;

type Price {
  amount   : Decimal;
  currency : String;
}

entity Products {
  key ID    : Integer;
      name  : String;
      stock : Integer;
      price : Price;
}
```

If we were to run the `cds compile` command again to get the CSN, it would look
like this:

```yaml
namespace: workshop
definitions:
  workshop.Price:
    {
      kind: type,
      elements:
        {
          amount: { type: cds.Decimal },
          currency: { type: cds.String },
        },
    }
  workshop.Products:
    kind: entity
    elements:
      ID: { key: true, type: cds.Integer }
      name: { type: cds.String }
      stock: { type: cds.Integer }
      price: { type: workshop.Price }
meta: { creator: CDS Compiler v6.4.6, flavor: inferred }
$version: 2.0
```

Note how this named custom type is a first class citizen now, in the form of
`workshop.Price`.

The advantage of this approach is of course that this new custom type can be
used in other entity definitions as the model grows.

> [!TIP]
> Try to remain aware of CDS modelling best practices, one of which is
> to [prefer flat
> models](https://cap.cloud.sap/docs/guides/domain-modeling#prefer-flat-models).
> Avoid complexity when something simpler will do. There's always a balance
> to be found between "too simple" and "over engineered". If we were to
> consider this for our `Products` entity, it might even look something like
> this:
>
> ```cds
> namespace workshop;
> 
> entity Products {
>   key ID       : Integer;
>       name     : String;
>       stock    : Integer;
>       price    : Decimal;
>       cost     : Decimal;
>       currency : String;
> }
> ```
>
> In other words, having `currency` flattened into the entity means that it can
> be shared between multiple currency-based values such as (here) `price` and
`cost`.

---

[Next](../05/)
