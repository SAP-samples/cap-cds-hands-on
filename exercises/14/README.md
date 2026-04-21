# 14 - Declarative contraints and assertions

Now that we've become somewhat familiar with annotations at a basic level, it's
time to explore some more. We'll remain on our declarative trajectory here and
look at how we can add annotations to our CDS model to impose
[constraints](https://cap.cloud.sap/docs/guides/services/constraints) and
checks.

## Constrain the allowable discount values

In a previous exercise we [added a bound
action](../12#declare-the-bound-action) called `applyDiscount` to our `Simple`
service, using a custom type for the percentage value:

```cds
using workshop from '../db/schema';

@protocol: 'odata'
@path    : '/simple'
service Simple {
  entity Products  as projection on workshop.Products
    actions {
      action applyDiscount(percent: Percentage) returns Products:price
    };

  ...

}

type Percentage : Integer;

```

Values for this `Percentage` type logically can only be between 1 and 100. But
there's nothing stopping us from applying a 200% discount right now,
effectively doubling the price!

### Consider the existing scenario

👉 Check the current price for product 1 ("Chai"):

```bash
curl \
  --silent \
  --url localhost:4004/simple/Products/1/price_amount \
  | jq .
```

Right now, the value is 18 (the original price, as our CAP server has been
restarted and thus data has been redeployed to the in-memory persistence
mechanism):

```json
{
  "@odata.context": "../$metadata#Products(1)/price_amount",
  "value": 18
}
```

👉 Apply a 200% discount:

```bash
curl \
  --data '{"percent":200}' \
  --silent \
  --header 'Content-Type: application/json` \
  --url localhost:4004/simple/Products/1/applyDiscount \
  | jq .
```

This should emit something like this:

```json
{
  "@odata.context":"../$metadata#Simple.return_Simple_Products_applyDiscount",
  "price_amount":36
}
```

Whoops, that "discount" was accepted!

### Constrain the values for the Percentage type

We can address that declaratively, in the CDS model, with no custom coding
required. Let's do that now.

👉 Extend the definition of the custom `Percentage` type in `srv/ecommerce.cds`
with an assert-based constraint:

```cds
type Percentage : Integer @assert.range: [
    1,
    100
];
```

The CAP server should, as usual, restart once this modification has been saved
(remember that as we're running with the local development default of an
in-memory SQLite persistence mechanism, the price of "Chai" will be reset to
what it was, i.e. 18).

👉 Now retry that discount:

```bash
curl \
  --data '{"percent":200}'
  --silent \
  --header 'Content-Type: application/json` \
  --url localhost:4004/simple/Products/1/applyDiscount \
  | jq .
```

This time, the discount of 200% is rejected:

```json
{
  "error": {
    "message": "Enter a value between 1 and 100.",
    "target": "percent",
    "code": "ASSERT_RANGE",
    "@Common.numericSeverity": 4
  }
}
```

> If you were to ask for the response headers:
>
> ```bash
> curl \
>   --data '{"percent":200}'
>   --include \
>   --header 'Content-Type: application/json` \
>   --url localhost:4004/simple/Products/1/applyDiscount
> ```
>
> then you'd see that the status code is appropriately set to 400:
>
> ```text
> HTTP/1.1 400 Bad Request
> X-Powered-By: Express
> OData-Version: 4.0
> Connection: keep-alive
> Keep-Alive: timeout=5
> ...
> ```

Nice - modelling our intent with `@assert.range` is all we need, another
example of how CDS allows us to focus on "what, not how". And errors can
be [surfaced to the UI,
too](https://cap.cloud.sap/docs/guides/services/constraints#served-to-fiori-uis).

## Explore more flexible constraint options

The relatively new[<sup>1</sup>](#footnotes) "general assertion" mechanism
lends a lot more flexibility for constraints than we have had so far with
`@assert.range` and its siblings such as `@assert.format` and so on.

This general assertion mechanism combines the simple `@assert` annotation with
an entire sub language in the CDS family, namely the CDS Expression Language
([CXL](https://cap.cloud.sap/docs/cds/cxl))[<sup>2</sup>](#footnotes).

Given that flexibility, there's a lot to explore. But let's just, erm,
_constrain_ ourselves to one example, to keep this exercise at a reasonable
length.

Let's say we want to ensure that a supplier name should be of a reasonable
length. We can add a constraint to this effect using `@assert` with a "case"
style expression, and we should explore doing this in a separate CDS file,
rather than clutter the `ecommerce.cds` file of service definitions.

### Add the constraint

👉 Create `srv/checks.cds` with the following content:

```cds
using Simple from './services';

annotate Simple.Suppliers with {
    company @assert: (case
                          when trim(company)   = ''
                               then 'Company name cannot be empty'
                          when length(company) < 2
                               then 'Company name must be at least 2 characters'
                      end);
}
```

How you end up organising files that make up your CDS model is up to you, but
adding constraints like this to a separate file in the `srv/` directory makes
some sense. Also, we learn how to use the `annotate` keyword to _add_ a
contraint, referring to its target.

It's [good practice to enclose the CXL expression in
brackets](https://cap.cloud.sap/docs/cds/cdl#expressions-as-annotation-values),
not least to give the language server some help in being able to identify,
parse and enhance the expression for us in the editor.

### Try out the constraint

Once the CAP server has restarted, try this constraint definition out.

👉 First, send a write request with an empty company name:

```bash
curl \
  --silent \
  --request PUT \
  --url localhost:4004/simple/Suppliers/1 \
  --header 'content-type: application/json' \
  --data '{"company":""}' \
  | jq .
```

The first error message is appropriately returned:

```json
{
  "error": {
    "message": "Company name cannot be empty",
    "code": "ASSERT",
    "target": "company",
    "@Common.numericSeverity": 4
  }
}
```

👉 Also send another request, this time with a name that is too short:

```bash
curl \
  --silent \
  --request PUT \
  --url localhost:4004/simple/Suppliers/1 \
  --header 'content-type: application/json' \
  --data '{"company":"Z"}' \
  | jq .
```

This time, we receive:

```json
{
  "error": {
    "message": "Company name must be at least 2 characters",
    "code": "ASSERT",
    "target": "company",
    "@Common.numericSeverity": 4
  }
}
```

With the combination of the declarative assertion mechanism and CXL, we can
[shift left](https://qmacro.org/blog/posts/2026/02/09/shift-left-with-cap/)
with our CAP modelling and avoid writing code with moving parts and that needs
maintaining. We'll see another example of this in the next and final
exercise[<sup>3</sup>](#footnotes).

---

[Next](../15/)

---

## Footnotes

1. It's in [Gamma](https://cap.cloud.sap/docs/releases/index#status-badges)
   status at the time of writing, at CAP Node.js version 9.8.

1. See the live stream series on the CDS Expression Language for lots of detail
   and deep diving into the topic: episode replays and accompanying notes are
   available via the blog post [A new Hands-on SAP Dev mini-series on the core
   expression language in
   CDS](https://qmacro.org/blog/posts/2025/12/09/a-new-hands-on-sap-dev-mini-series-on-the-core-expression-language-in-cds/)

1. For another example of a `case` expression like this, see the section [A
   first look at
   expressions](https://qmacro.org/blog/posts/2026/03/05/cds-expressions-in-cap-notes-on-part-1/#a-first-look-at-expressions)
   in the notes to part 1 of our series on CXL.
