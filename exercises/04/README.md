# 04 - Design and use custom types for a richer entity definition

In this exercise we'll learn about custom types in CDL, and how
and when to employ them (and when not to).

## Consider the existing Products entity

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

> "Element" is CAP's term for what we might call a "property" or
> "field" in other contexts.

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

While this looks neat and has an academic and abstract appeal,
this is considered bad practice. In domain modelling terms, CAP
[encourages the KISS
approach](https://cap.cloud.sap/docs/guides/domain-modeling#keep-it-simple-stupid).
This custom type merely causes us to have to think harder to
understand what we're looking at:

```text
stock -> Stock -> Integer
```

than if we'd simply had:

```text
stock -> Integer
```

That said, there are some circumstances where types add value, such as
when values belong together.

## Add price information

Let's add cost information. While a price is typically represented
as a decimal, they are meaningless without a currency.

What does good look like here? Well, it depends. But for the sake of
learning about types, let's explore a custom [structured
type](https://cap.cloud.sap/docs/cds/cdl#structured-types).

👉 Add a custom type `Price` to `db/schema.cds` and use it for a new element `price`:

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
