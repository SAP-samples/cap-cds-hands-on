# 14 - Declarative contraints and assertions

Now that we've become somewhat familiar with annotations at a basic level, it's
time to explore some more. We'll remain declarative here and look at how we can
add annotations to our CDS model to impose
[constraints](https://cap.cloud.sap/docs/guides/services/constraints) and
checks.

## Constrain the allowable discount values

In [exercise 12](../12/) we added a bound action called `applyDiscount` to our
`Simple` service, using a custom type for the percentage value:

```cds
@protocol: 'odata'
@path    : '/simple'
service Simple {
  @cds.redirection.target
  entity Products           as projection on workshop.Products
    actions {
      action applyDiscount(percent: Percentage) returns Products:price;
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

Right now, the value is 18:

```json
{
  "@odata.context": "../$metadata#Products(1)/price_amount",
  "value": 18
}
```

👉 Apply a 200% discount:

```bash
curl \
  --data '{"percent":200}'
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

👉 Extend the definition of the custom `Percentage` type in `srv/services.cds`
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

```bash
curl \
  --data '{"percent":200}'
  --include \
  --header 'Content-Type: application/json` \
  --url localhost:4004/simple/Products/1/applyDiscount
```

then you'd see that the status code is appropriately set to 400:

```text
HTTP/1.1 400 Bad Request
X-Powered-By: Express
OData-Version: 4.0
Connection: keep-alive
Keep-Alive: timeout=5
...
```

Nice - modelling our intent with `@assert.range` is all we need, another
example of how CDS allows us to focus on "what, not how".

