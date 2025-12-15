# 12 - Add a further operation in the form of a bound action

In the previous exercise we explored a custom unbound function, before
replacing it with a declarative alternative in the form of a projection with an
infix filter. In this exercise we'll continue our exploration of actions and
functions by definining and implementing a simple bound action, to see what
that looks and feels like.

## Recall what the previous function definition looked like

In the first half of the previous exercise, our `Simple` service looked like
this, after the addition of the function declaration:

```cds
@protocol: 'odata'
@path    : '/simple'
service Simple {
  entity Products  as projection on workshop.Products;
  entity Suppliers as projection on workshop.Suppliers;
  entity Orders    as projection on workshop.Orders;
  function outOfStockProducts() returns many Products;
}
```

Being unbound, the function `outOfStockProducts` was simply listed alongside
the entities.

## Declare the bound action

For a bound function or action, we need a little bit more syntax to be able to
connect it to the entity to which it should be bound.

👉 Define a bound action `applyDiscount` for the `Products` entity, so that it
ends up looking like this:

```cds
@protocol: 'odata'
@path    : '/simple'
service Simple {
  @cds.redirection.target
  entity Products           as projection on workshop.Products
    actions {
      action applyDiscount(percent: Percentage) returns Products:price;
    };

  entity Suppliers          as projection on workshop.Suppliers;
  entity Orders             as projection on workshop.Orders;
  entity OutOfStockProducts as projection on workshop.Products[stock <= 0];

}
```

👉 The expected `percent` value is defined as type `Percentage`, which is custom,
so add that to the bottom of the `services.cds` file too:

```cds
type Percentage : Integer @assert.range: [
  1,
  100
];
```

There are a couple of things worth noticing here:

- the action declaration is directly connected to the `Products`
entity projection via the `actions { ... }` construct
- the `Percentage` type definition is annotated with a [range
  assertion](https://cap.cloud.sap/docs/guides/providing-services#assert-range),
  a feature of CAP's multifaceted [input
  validation](https://cap.cloud.sap/docs/guides/providing-services#input-validation)
  where we can yet again express intent (we're expecting a percentage value)
  and let the framework do the rest

## Add the action's implementation

In a similar way to how we added an implementation for the custom unbound
function in the previous exercise, we need to add an implementation, again in
the [on
phase](https://cap.cloud.sap/docs/guides/providing-services#hooks-on-before-after).

👉 Do that now, recreating `services.js` in the `srv/` directory, with this content:

```javascript
const cds = require('@sap/cds')

class Simple extends cds.ApplicationService {
  init() {
    this.on('applyDiscount', async (req) => {
      const result = await UPDATE(req.subject)
        .set`price_amount = price_amount * ${req.data.percent / 100}`
      if (!result) return failed(req)
      return await SELECT.columns`price_amount`.from(req.subject)
    })
    return super.init()
  }
}
module.exports = { Simple }
```

This is very similar to the implementation for the unbound function
`outOfStockProducts` in the previous exercise, not least in its deliberate
simplicity. There are a couple of points worthy of note here. But before we
look at those, let's try out the bound action.

## Make a call to the bound action

Being bound, this action is applicable in the context of a specific entity,
i.e. via its address (URL).

### Retrieve an entity

So let's start out by picking a specific entity, say product with ID 1, and
requesting an OData read operation.

👉 Do that:

```bash
curl \
  --silent \
  --url localhost:4004/simple/Products/1 \
  | jq .
```

This should emit something like this:

```json
{
  "@odata.context": "$metadata#Products/$entity",
  "ID": 1,
  "name": "Chai",
  "stock": 0,
  "price_amount": 18,
  "price_currency_code": "GBP",
  "supplier_ID": 1
}
```

Note that the price of this product is currently 18 GBP.

### Invoke the bound action on this product

Invoking the previous unbound function could be done with a simple HTTP GET
request, like this:

```bash
curl \
  --silent \
  --url localhost:4004/simple/outOfStockProducts
```

This time, we need the HTTP POST method (as actions can have side effects, and
so GET is not permitted), and the URL needs to be constructed from the address
of a specific product.

👉 Do this:

```bash
curl \
  --silent \
  --data '{"percent":50}' \
  --url localhost:4004/simple/Products/1/applyDiscount \
  | jq .
```

> Using the `--data` option will cause `curl` to automatically use the POST
> method rather than the default GET method.

This should result in something like this:

```json
{
  "@odata.context": "../$metadata#Simple.return_Simple_Products_applyDiscount",
  "price_amount": 9
}
```

Additionally, another GET request to the entity (at
<http://localhost:4004/simple/Products/1>) should reveal that the price has
indeed been changed.

### Try an invalid percent value

Do we need to provide an implementation to ensure the percent range restriction
is heeded?

👉 Try this, to find out:

```bash
curl \
  --silent \
  --data '{"percent":999}' \
  --url localhost:4004/simple/Products/1/applyDiscount \
  | jq .
```

This should result in something like this:

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

Nice - modelling our intent with `@assert.range` is all we need, another
example of how CDS allows us to focus on "what, not how".
