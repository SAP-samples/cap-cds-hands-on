# 06 - Understand and use aspects

In this exercise we'll dig into aspects a little more, and learn how we can put
them to good use in our modelling.

## See how aspects are related to extending definitions more generally

At the end of the previous exercise we were using a simplified custom and
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

### Try using extend with an anonymous aspect

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

### See how the modelling effect is the same

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

Let's explore this alternative extension scenario for a bit longer to drive
home another feature we have already learned about.

### Understand the importance of context and scoped names

👉 First, restore the definition of the named aspect `CodeList`, without adding it back as an include to the `Currencies` entity:

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

## Explore common reuse aspects

While the `CodeList` aspect we've looked at so far is useful and was helpful to
gain an initial understanding, we can think of it more as a building block for
underlying structures and extensions. Now that we have that understanding,
let's take a look at some more immediately useful aspects from
`@sap/cds/common` and how they're often employed.

### Go back to @sap/common/cds

👉 Before we continue, let's edit the `using` line to import from the proper
common source, rather than our own custom one:

```cds
using Currency from '@sap/cds/common';
```

👉 To keep things tidy and avoid duplicate definition errors, delete the `db/common.cds` file.

### Add a second entity Suppliers

To drive the reuse theme home, let's add a second entity.

👉 Add `Suppliers` as an entity to the `db/schema.cds` file so that it looks
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
> that also include the recommendation to capitalise such names, while keeping
> element names in lower case, as is also evident here.

### Get to know the cuid and managed aspects

Notice that both entities have a single primary key `ID`, defined as an
`Integer`. This is fine for such simple examples, but numeric (integer) IDs
have their challenges (to which as anyone who has worked with number range
management and value generation can attest). A primary key like this is common,
and there is an aspect that can be applied to both entities here that can
replace the explicit and manual definition of such. That aspect is `cuid`.

Also, often, in modelling a business domain, there will be a requirement for
basic data tracking to record creation and modification. There's a simple
one-word aspect for this too, which is `managed`.

Both these aspects help with the [what not
how](https://cap.cloud.sap/docs/guides/domain-modeling#capture-intent-%E2%80%94-what-not-how)
intent-based modelling approach that CAP exhorts. Rather than include
implementation details, technical mechanics, to achieve these everyday
requirements, definitions can be constructed with beauty and simplicity, but most of
all with minimum fuss and fanfare, allowing the primary goal of modelling to
continue.

Let's try out both of these aspects.

#### Use the cuid aspect for a primary key

👉 Modify the contents of `db/schema.cds` to also import `cuid` from
`@sap/cds/common`, and use it in place of the explicit `ID` elements (which you
should remove):

```cds
using {
  Currency,
  cuid
} from '@sap/cds/common';

namespace workshop;

type Price {
  amount   : Decimal;
  currency : Currency;
}

entity Products : cuid {
  name  : String;
  stock : Integer;
  price : Price;
}

entity Suppliers : cuid {
  company : String;
}
```

> Note that in order to import more than one artifact we need to enclose the
> list in a `{ ... }` block.

Outwardly, the entity definitions become smaller. But more importantly the domain model becomes simpler on the surface, moving away from "implementation" and further towards "intent".

Using the `cuid` aspect helps us to follow [best practices relating to primary keys](https://cap.cloud.sap/docs/guides/domain-modeling#primary-keys) for performance, simplicity and consistency reasons.

> If you're curious as to what this `cuid` definition brings, good! Curiosity
> is the key. If you were to look in the file `@sap/cds/common.cds` (inside
> `node_modules/` in your project directory) you'd find this:
>
> ```cds
> /**
>  * Aspect for entities with canonical universal IDs
>  *
>  * See https://cap.cloud.sap/docs/cds/common#aspect-cuid
>  */
> aspect cuid {
>   key ID : UUID; //> automatically filled in
> }
> ```
>
> In other words, the aspect contains a single element `ID` marked as `key`,
> with the built-in type `UUID`.

#### Use the managed aspect for basic data tracking

With the addition of `cuid` to the `using` line in our `db/schema.cds` we're on
a roll, and it's straightforward to continue on this
[mixin-based](https://cap.cloud.sap/docs/cds/cdl#aspects:~:text=They%27re%20based%20on%20a%20mixin%20approach%20as%20known%20from%20Aspect%2Doriented%20Programming%20methods)
trajectory.

The aspect is defined in `@sap/cds/common` thus:

```cds
/**
 * Aspect to capture changes by user and name
 *
 * See https://cap.cloud.sap/docs/cds/common#aspect-managed
 */
aspect managed {
  createdAt  : Timestamp @cds.on.insert : $now;
  createdBy  : User      @cds.on.insert : $user;
  modifiedAt : Timestamp @cds.on.insert : $now  @cds.on.update : $now;
  modifiedBy : User      @cds.on.insert : $user @cds.on.update : $user;
}
```

> [!NOTE]
> Here we see some `@` and `$` prefixed constructs for the first time. The
> former are annotations which we'll cover generally in a later exercise, and
> the latter are [pseudo
> variables](https://cap.cloud.sap/docs/guides/domain-modeling#pseudo-variables)
> which resolve as you'd probably expect, given their names.

👉 Import and use the `managed` aspect in `db/schema.cds` like this:

```cds
using {
  Currency,
  cuid,
  managed
} from '@sap/cds/common';

namespace workshop;

type Price {
  amount   : Decimal;
  currency : Currency;
}

entity Products : cuid, managed {
  name  : String;
  stock : Integer;
  price : Price;
}

entity Suppliers : cuid, managed {
  company : String;
}
```

Note there's no outward increase in complexity, no sign of "implementation
details" - just a single descriptive word "managed" that suffices at this CDS
modelling level.

👉 Take a look at what this produces at the CSN level, what the actual
underlying implementation (for the CAP server and the persistence layer) is:

```bash
cds compile --to yaml db/schema.cds
```

There will be quite a bit of output; scroll through it and you will see what
effect this `managed` aspect (and indeed the `cuid` aspect) has. Here's the CSN
for the `workshop.Suppliers` entity:

```yaml
namespace: workshop
definitions:
  workshop.Suppliers:
    kind: entity
    includes: [cuid, managed]
    elements:
      ID: { key: true, type: cds.UUID }
      createdAt:
        "@cds.on.insert": { "=": $now }
        "@UI.HiddenFilter": true
        "@UI.ExcludeFromNavigationContext": true
        "@Core.Immutable": true
        "@title": { i18n>CreatedAt }
        "@readonly": true
        type: cds.Timestamp
      createdBy:
        "@cds.on.insert": { "=": $user }
        "@UI.HiddenFilter": true
        "@UI.ExcludeFromNavigationContext": true
        "@Core.Immutable": true
        "@title": { i18n>CreatedBy }
        "@readonly": true
        "@description": { i18n>UserID.Description }
        type: User
        length: 255
      modifiedAt:
        "@cds.on.insert": { "=": $now }
        "@cds.on.update": { "=": $now }
        "@UI.HiddenFilter": true
        "@UI.ExcludeFromNavigationContext": true
        "@title": { i18n>ChangedAt }
        "@readonly": true
        type: cds.Timestamp
      modifiedBy:
        "@cds.on.insert": { "=": $user }
        "@cds.on.update": { "=": $user }
        "@UI.HiddenFilter": true
        "@UI.ExcludeFromNavigationContext": true
        "@title": { i18n>ChangedBy }
        "@readonly": true
        "@description": { i18n>UserID.Description }
        type: User
        length: 255
      company: { type: cds.String }
```

Even if we don't yet fully understand the `@`-prefixed annotations right now,
we can see and appreciate the effect that these two aspects have, and how
useful they are in modelling!

## Re-simplify our model for the remaining exercises

In order to cut down on information and data that might otherwise cause "noise"
and get in the way of our understanding, let's re-simplify our model by going
back to a numeric key field for both entities, and doing away with the
tracking information.

### Restore the key ID field

The model we're building in this workshop is deliberately simple and also based
on the classic
[Northwind](https://services.odata.org/V4/Northwind/Northwind.svc/) dataset, in
the form of a cut-down version called
[Northbreeze](https://developer-challenge.cfapps.eu10.hana.ondemand.com/odata/v4/northbreeze).

As we want to reuse the Northbreeze data in our simple model, we have to align
the types as much as we can. This means that while using the standard `cuid`
aspect is best practice, we'll use our own custom version that defines the
element as an `Integer` type instead of a `UUID` type. This reflects the key
properties in the [corresponding Northbreeze
service](https://developer-challenge.cfapps.eu10.hana.ondemand.com/odata/v4/northbreeze/$metadata),
such as this `Products` entity type definition, where the `ProductID` property has the (OData entity data model) integer type `Edm.Int32`:

```xml
<EntityType Name="Products">
  <Key>
    <PropertyRef Name="ProductID"/>
  </Key>
  <Property Name="ProductID" Type="Edm.Int32" Nullable="false"/>
  <Property Name="ProductName" Type="Edm.String"/>
  <Property Name="..." Type="..."/>
</EntityType>
```

👉 Remove the import of `cuid` from `@sap/cds/common` and instead add a custom
`cuid` definition after the `namespace` declaration, so that the
`db/schema.cds` file looks like this:

```cds
using {
  Currency,
  managed
} from '@sap/cds/common';

namespace workshop;

aspect cuid {
  key ID : Integer;
}

type Price {
  amount   : Decimal;
  currency : Currency;
}

entity Products : cuid, managed {
  name  : String;
  stock : Integer;
  price : Price;
}

entity Suppliers : cuid, managed {
  company : String;
}
```

### Remove the managed aspect

Again, to keep things simple for the remainder of this workshop, let's remove the use of the `managed` aspect, so that we're not inundated with `createdAt`, `createdBy`, `modifiedAt` and `modifiedBy` elements and their default values as we continue through the exercises.

👉 Remove all references to `managed`, both from the `using` directive and from the inclusion in each entity, so that the final version of `db/schema.cds` looks like this:

```cds
using {Currency} from '@sap/cds/common';

namespace workshop;

aspect cuid {
  key ID : Integer;
}

type Price {
  amount   : Decimal;
  currency : Currency;
}

entity Products : cuid {
  name  : String;
  stock : Integer;
  price : Price;
}

entity Suppliers : cuid {
  company : String;
}
```

Now we're all set to move on to [the next
part](../../#part-3---describing-relationships-with-associations-and-compositions)
of this workshop - good work!
