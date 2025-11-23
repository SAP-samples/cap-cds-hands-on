# 06 - Understand and use aspects

In this exercise we'll dig into aspects a little more, and learn how we can put
them to good use in our modelling.

## See how aspects are related to extending definitions more generally

At the end of the previous exercise we were considering a simplified custom and
cut down version of some of the content from the `@sap/cds/common` reuse
module, in `db/common.cds`, which looks like this:

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

At this point we already are starting to understand how
[aspects](https://cap.cloud.sap/docs/cds/cdl#aspects) are collections of
elements that are to be used to extend existing entities. Let's drive this home
a little by modifying how our custom `Currencies` entity inherits the `name`
and `descr` elements that are defined in the `CodeList` aspect.

👉 Modify the definitions inside the `sap.common` context in `db/common.cds` so
it looks like this:

```cds
context sap.common {

  entity Currencies {
    key code      : String(3);
        symbol    : String(5);
        minorUnit : Int16;
  }

  extend Currencies with {
    name  : String(255);
    descr : String(1000);
  }

}
```

This modification:

- removes the inclusion of the `CodeList` aspect in the `Currencies` definition
- removes the definition of `CodeList` as an aspect
- replaces it with the use of the `extend` directive

> [!NOTE]
> This is the first time we're seeing the somewhat imperative
> [extend](https://cap.cloud.sap/docs/cds/cdl#the-extend-directive) directive
> but it's really what's happening behind the [syntactic
> sugar](https://en.wikipedia.org/wiki/Syntactic_sugar) scenes when we employ
> the shorter and more declarative `:`. Note that to use the `:` construct we
> need a named aspect to which we can refer, rather than the anonymous
> structure we have with
>
> ```cds
> extend Currencies with {
>   name  : String(255);
>   descr : String(1000);
> }
> ```

👉 To see that this is the same as we had before, regenerate the CSV files:

```bash
cds add data --force
```

👉 Now re-examine the header record of `db/data/sap.common-Currencies.csv`,
which should be the same as before:

```csv
code,symbol,minorUnit,name,descr
```

The end result is the same - the three elements from `Currencies` plus the two
from the structure in the `extend`.

A key advantage of named aspects is that they can be used and reused in
different places.

Let's riff on this alternative extension scenario for a bit longer to drive
home another feature we have already learned about.

👉 First, restore the definition of the named aspect `CodeList`:

```cds
context sap.common {

  entity Currencies {
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

In this state, `Currencies` will only have the three elements `code`, `symbol`
and `minorUnit` (and `CodeList` will remain unused and unloved).

👉 Now, after the `sap.common` context block finishes, try to add an `extend`
directive thus:

```cds
context sap.common {

  entity Currencies {
    key code      : String(3);
        symbol    : String(5);
        minorUnit : Int16;
  }

  aspect CodeList {
    name  : String(255);
    descr : String(1000);
  }

}

extend Currencies with CodeList;
```

If your editor doesn't already, then the compiler itself will have something to
say about this:

```log
[ERROR] db/common.cds:18:8-18: No artifact has been found
with name “Currencies” (in extend:“Currencies”)
```

This is because context (literally!) matters. If we want to refer to
definitions that are inside a context, from outside of it, we need to use their
fully qualified (scoped) names.

👉 Fix the issue by rewriting the `extend` line like this:

```cds
context sap.common {

  entity Currencies {
    key code      : String(3);
        symbol    : String(5);
        minorUnit : Int16;
  }

  aspect CodeList {
    name  : String(255);
    descr : String(1000);
  }

}

extend sap.common.Currencies with sap.common.CodeList;
```

> Of course, placing the `extend` within the `context` scope would allow us to
> omit the scope name:
>
> ```cds
> context sap.common {
> 
>   entity Currencies {
>     key code      : String(3);
>         symbol    : String(5);
>         minorUnit : Int16;
>   }
> 
>   aspect CodeList {
>     name  : String(255);
>     descr : String(1000);
>   }
>
>   extend Currencies with CodeList;
>
> }
>
> ```
>
> But at this point the `:` shortcut syntax is likely the better choice anyway.

## Explore common aspects

While the `CodeList` aspect we've looked at so far is useful and was helpful to
gain an initial understanding, we can think of it more as a building block for
underlying structures and extensions. Now that we have that understanding,
let's take a look at a couple of more immediately useful aspects from
`@sap/cds/common` and how they're often employed.

Before we start, let's add a second entity.

👉 Add a new entity `Suppliers` to the `db/schema.cds` file so that it looks
like this:

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

entity Suppliers {
  key ID      : Integer;
      company : String;
}
```

> You may have spotted at this point that the `type` name is in the singular
> and the `entity` names are in the plural. This is not a coincidence, it
> follows domain modelling [naming
> conventions](https://cap.cloud.sap/docs/guides/domain-modeling#naming-conventions)
> (that also include the recommendation to capitalise such names, while keeping
> element names in lower case, as is also evident here).

Notice that both entities have a single primary key `ID`, defined as an
`Integer`. This is fine for such simple examples, but numeric (integer) IDs
have their challenges (as anyone who has worked with number range management
and value generation can attest to).

A primary key like this is common, and there is an aspect that can be applied
to both entities here that can replace the explicit and manual definition of
such. That aspect is `cuid`.

## TODO - cuid and managed
