# 05 - Explore reuse and standard common definitions

In this exercise you'll learn about reuse and how we can (and should) avoid
re-inventing the wheel for common building blocks.

## Improve the currency definition

Right now, in our `db/schema.cds` file, the currency definition is a simple
`String` type:

```cds
namespace workshop;

type Price {
  amount   : Decimal;
  currency : String; // <---
}

entity Products {
  key ID    : Integer;
      name  : String;
      stock : Integer;
      price : Price;
}
```

In the context of this deliberately simple model, that might be fine. But in
the real world there is more to the concept of currencies. Just look at the
family of tables in the core ERP system; here are just a few of them:

- TCURX Decimal places in currencies
- TCURR Exchange rates
- TCURF Conversion factors
- TCURC Currency codes
- TCURT Currency text

Relegating the `currency` component of our `Price` type to a `String` brings
about guaranteed technical debt from the outset, as there's a lot of work
needed to support currencies with such a plain definition.

## Explore common reuse types

👉 Head over to Capire and navigate to the [Common Reuse Types and
Aspects](https://cap.cloud.sap/docs/cds/common) topic:

![Common Reuse Types and Aspects section of
Capire](assets/common-reuse-types-and-aspects-in-capire.png)

This is a great resource that is worth reading through after this workshop.
We'll look at reuse aspects in the next exercise; in this exercise we'll look
at reuse types, which are introduced in the [Common Reuse
Types](https://cap.cloud.sap/docs/cds/common#code-types) section.

Before we do, though, it's worth understanding why this concept and
implementation of common reuse types exists, and [the points from this topic in
Capire](https://cap.cloud.sap/docs/cds/common#why-use-sap-cds-common) are worth
reproducing here:

- Concise and comprehensible models
- Foster interoperability between all applications
- Proven best practices captured from real applications
- Streamlined data models with minimal entry barriers
- Optimized implementations and runtime performance
- Automatic support for localized code lists and value helps
- Extensibility using Aspects
- Verticalization through third-party extension packages

## Import and use Currency from the reuse library

The common reuse facility described in this Capire topic is commonly known as
`@sap/cds/common`, a "module" or "library" reference that is in the form of:

```text
namespace (@sap) / module (cds) / location (common)
```

> Referring to locations within modules like this in other related contexts
> (such as handler logic) is sometimes to be avoided, but here at the CDS
> modelling level we are OK.

Being a location within `@sap/cds` which is the core runtime for CAP Node.js,
this facility is always and implicitly available.

`Currency` is a type that's available in this facility. Let's explore it by
using it.

👉 In `db/schema.cds`, import the `Currency` type from this facility, and use
it to define the `currency` element of our custom `Price` type, like this:

```cds
using Currency from '@sap/cds/common';

namespace workshop;

type Price {
  amount   : Decimal;
  currency : Currency;
}

entity Products {
  key ID    : Integer;
      name  : String;
      stock : Integer;
      price : Price;
}
```

> [!INFO]
> Here we see a new CDL construct - the
> [using](https://cap.cloud.sap/docs/cds/cdl#using) directive, which we're
> employing to import a definition from another CDS model (`@sap/cds/common`).

Right now this new definition is all a bit opaque; what have we really got here now?

👉 Take a look at the resulting CSN:

```bash
cds compile --to yaml db/schema.cds
```

The resulting YAML is rather overwhelming! That's because as well as loading
the contents of our `db/schema.cds`, the compiler will load the entirety of
`@sap/cds/common`, which includes a lot more than just the `Currency` type.

Instead, let's take a look at the sources of `@sap/cds/common`, as it will help
us understand what is going on and what we will be getting with this `Currency`
type. It will also introduce us to some other CDL features.

👉 Open up the file `node_modules/@sap/cds/common.cds` in your editor and take
a look; there's a lot of content, here's what's relevant for us and our use of
the `Currency` type:

```cds
type Country : Association to sap.common.Countries;

context sap.common {

  entity Currencies : CodeList {
    key code      : String(3) @(title : '{i18n>CurrencyCode}');
        symbol    : String(5) @(title : '{i18n>CurrencySymbol}');
        minorUnit : Int16     @(title : '{i18n>CurrencyMinorUnit}');
  }

  aspect CodeList @(
    cds.autoexpose,
    cds.persistence.skip : 'if-unused'
  ) {
    name  : localized String(255)  @title : '{i18n>Name}';
    descr : localized String(1000) @title : '{i18n>Description}';
  }

}
```

> [!INFO]
> There are quite a few more CDL features here:
>
> - `Association to`
> - `context { ... }`
> - `aspect`
> - `@...`
> - `localized`
> - the `:` symbol between `Currencies` and `CodeList`
>
> Most are relevant to our fundamental understanding of
> `Currency` and will be explained briefly in the following section.
>
> Others such as the `:` symbol and the `aspect` keyword will
> be explained in more detail in the next exercise. Others still, such
> as the `Association to` construct, will be explained in a subsequent
> part of this workshop.

---

## TODO

- <https://qmacro.org/blog/posts/2024/03/12/iso-content-for-common-cap-types/>
