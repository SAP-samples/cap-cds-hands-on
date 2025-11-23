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

> [!NOTE]
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

> [!NOTE]
> There are quite a few more CDL features here:
>
> - `Association to`
> - `context { ... }`
> - `aspect`
> - `@...`
> - `localized`
> - the `:` symbol between `Currencies` and `CodeList`
>
> Some are relevant to our fundamental understanding of
> `Currency` and will be explained in brief in the following section.
>
> Others such as the `:` symbol and the `aspect` keyword will
> be explained in more detail in the next exercise. Others still, such
> as the `Association to` construct, will be explained in a subsequent
> part of this workshop.

## Consider a reduced version of the Currency definition

First, we can and should ignore those sections of the CDL source above that
start with `@` - they are annotations which we will cover in a later part of
this workshop.

We can also ignore `localized`, which is a construct that allows
the model to reflect the reality of internationalisation where we can and
should provide texts in different locales for different audiences (think of the
tables that are suffixed with `T` in the core ERP system, such as `TCURT` as
mentioned earlier in this exercise).

This leaves us with a simpler version.

👉 Add this simpler version to a new file `db/common.cds` and then take a
moment to
[stare](https://qmacro.org/blog/posts/2017/02/19/the-beauty-of-recursion-and-list-machinery/#initial-recognition)
at it.

```cds
type Currency : Association to sap.common.Currencies;

context sap.common {

  entity Currencies : CodeList {
    key code      : String(3);
        symbol    : String(5);
        minorUnit : Int16;
  }

  aspect CodeList {
    name  : String(255);
    descr : String(1000);
  }

}
```

👉 Now temporarily modify the existing import in `db/schema.cds` from:

```cds
using Currency from '@sap/cds/common';
```

to

```cds
using Currency from './common';
```

to use our `Currency` definition in this simpler version.

> This is purely illustrative and deliberately simplified to aid comprehension.
> In normal modelling we would use the `Currency` as-is from `@sap/cds/common`.
> A bonus side effect of this simplified illustration is that it shows us the similarity
> between importing from a CDS model in a module, and from a CDS model in a file.

## Study the component parts of the Currency construct

Let's take the definitions one by one.

The [context](https://cap.cloud.sap/docs/cds/cdl#context) directive is similar
to the `namespace` directive we already know about. The `context`
allows us to create definitions in different namespaces (and even nest them)
in the same `.cds` file. We can guess how this works, because
following the context's name there's a block construct (`{ ... }`) to enclose
those definitions that are to be in the scope of that context's name.

The upshot of this `context sap.common { ... }` is that the entity `Currencies`
and the aspect `CodeList` are both actually in that `sap.common` scope and are
therefore have these fully qualified names:

- `sap.common.Currencies`
- `sap.common.CodeList`

Knowing this helps us to understand the "target" of the `type` definition:

```cds
type Currency : Association to sap.common.Currencies;
```

The word "target" is relevant here, as the `Association to` part is a so-called
[managed to-one
association](https://cap.cloud.sap/docs/guides/domain-modeling#managed-1-associations),
a type of relationship. Here, it means that the possible currencies themselves
are maintained elsewhere, and a "currency key" pointing to a specific, single
("to-one") currency with the rest of that currency's details is what is to be
stored in an element that is described with this `Currency` type.

We'll look at associations and other relationships in a later exercise.

### Generate initial data CSV headers

Earlier in this workshop we [added some initial
data](../01#add-some-initial-data) for our fledgling `Products` entity. We
used `cds add data` to generate the file, pre-populated for us
with appropriate header line:

```csv
ID,name,stock
```

Asking for this to be done for us again, based on our new definitions, can be illustrative.

👉 Do that now, using the `--force` option to overwrite the existing
`db/data/workshop-Products.csv` file:

```bash
cds add data --force
```

This should emit something like this:

```log
using '--force' ... existing files will be overwritten

adding data
adding headers only, use --records to create random entries
  creating db/data/sap.common-Currencies.csv
  overwriting db/data/workshop-Products.csv

successfully added features to your project
```

What is the result of this?

👉 Open the two files to take a look at the headers.

In `db/data/workshop-Products.csv` we'll see:

```csv
ID,name,stock,price_amount,price_currency_code
```

And in `db/data/sap.common-Currencies.csv`, we'll see:

```csv
code,symbol,minorUnit,name,descr
```

> Note how the elements of our custom `Price` type have been referenced, with
> the names prefixed with the name of the entity's element that it describes.
>
> For example, the `amount` element from the `Price` type, which itself has
> been used to describe the `price` element in the `Products` entity, becomes
> `price_amount` as a CSV field.

Given that contents of our `db/data/sap.common-Currencies.csv` might represent
the initial data for our base currency configuration (`TCURC` et al.), we might
have some core currency information in that file, and then references to that
information, by currency code, in the `db/data/workshop-Products.csv` file,
like this:

```text
db/data/workshop-Products.csv:
ID,name,stock,price_amount,price_currency_code
----------------------------------------------
1,Chai,39,18,GBP
2,Chang,17,19,EUR
3,Aniseed Syrup,13,10,GBP
                       |    db/data/sap.common-Currencies.csv:
                       |    code,symbol,minorUnit,name,descr
                       |    --------------------------------
                       +--> GBP,£,2,Pound,Great British Pound
                            EUR,€,2,Euro,European Currency Unit
                            USD,$,2,Dollar,United States Dollar
```

> Note also how the names of the CSV files themselves are constructed - from
> the "scope"-prefixed entity names, whether that scope was defined using the
> `namespace` directive (in the case of `workshop-Products`) or the `context`
> directive (in the case of `sap.common-Currencies`).

## Take the briefest of looks at the aspect construct

The feature that we haven't looked at any real level yet is the [CodeList
aspect](https://cap.cloud.sap/docs/cds/common#aspect-codelist).

👉 Before finishing this exercise, take a first look, by first revisiting our definitions:

```cds
type Currency : Association to sap.common.Currencies;

context sap.common {

  entity Currencies : CodeList {
    key code      : String(3);
        symbol    : String(5);
        minorUnit : Int16;
  }

  aspect CodeList {
    name  : String(255);
    descr : String(1000);
  }

}
```

This appears twice in our `sap.common` context:

- as a definition (`aspect CodeList { ... }`)
- in use (`entity Currencies : CodeList`)

> The order in which these appearances are actually made also teaches us that
> in CDS models, definitions don't have to come before their first use.

👉 For now, think of aspects as a sibling of types. Like types, they can be
anonymous, or be given a name (as `CodeList` here). Unlike types, they cannot
be "scalar", i.e. they must contain elements (i.e. have a `{ ... }` structure).

Aspects can be used to extend existing structures, most commonly entities. And
the shortest, most colloquial way to do this is with a `:` symbol, a [shortcut
syntax construct](https://cap.cloud.sap/docs/cds/cdl#includes) that says "oh,
and include the elements in this aspect too".

The upshot of this is that the `Currencies` entity, when fully defined, has the
three elements directly defined with it (`code`, which is a key element, and
`symbol` & `minorUnit`) and, in addition, the two elements from the `CodeList`
aspect (`name` and `descr`).

👉 Consider the five fields in the header for the corresponding initial CSV
data file, where their origin now should make sense:

```csv
code,symbol,minorUnit,name,descr
```

We'll look at aspects in more detail in the next exercise.

---
