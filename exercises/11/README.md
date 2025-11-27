# 11 - Take a look at domain specific custom operations

In this exercise we'll take a look at how we can define and implement custom
functions in your services.

What does "custom" mean here? It depends on the context. Consider API styles
based on the HTTP application protocol, which translate to two of the protocols
supported out of the box by CAP, namely "REST" and OData.

> Technically speaking REST is more of an architectural style than a protocol;
> so where you see "REST", think
> [HTTP](https://www.youtube.com/watch?v=Ic37FI351G4)

In this context, the surface area of an API is made up of a small number of
standard "verbs" (HTTP methods) and an almost infinite number of "nouns"
(resources, addressed via URLs). Consequently, standard APIs look similar, in
that each offer CRUD (create, read, update and delete) operations on resources
that normally represent business data.

The entities in our domain model, exposed via projections in our `Simple`
service, are a perfect example of that, and through that service we are able to
send requests of each operational type to create, read, update and delete
instances of the entities (products, suppliers, and so on).

While this approach is aligned with the philosophy of the underlying
application protocol (HTTP), there are some circumstances where a more "remote
procedure call" ([RPC](https://en.wikipedia.org/wiki/Remote_procedure_call))
style of API facility is desirable, where the endpoint (target resource) is
more opaque and the semantics of the operation do not align cleanly with the
HTTP method used; in fact, HTTP is relegated to a transport protocol in this
case.

> Yes, I care deeply about Representational State Transfer (REST), indeed my
> [narrowboat home](https://qmacro.org/tags/narrowboat/)'s name is "FULLY
> RESTFUL".

## Consider OData's actions and functions

If we look at OData (specifically V4), we see that beyond the HTTP oriented
support for the standard operations (CRUD plus Q for "query", an OData specific
form of read), there are provisions for such "out-of-band" mechanisms in the
form of actions and functions.

And CAP has [first class
provision](https://cap.cloud.sap/docs/guides/providing-services#actions-functions)
for such mechanisms.

> Not only can actions and functions be defined and employed in the context of
> services that are explicitly or implicitly served via OData, but also via the
> slightly more generic
> [HTTP](https://cap.cloud.sap/docs/advanced/publishing-apis/).

## Understand the scope of actions and functions

Actions and functions are for different purposes, and each can be bound or unbound.

👉 Visit Capire's [Actions &
Functions](https://cap.cloud.sap/docs/guides/providing-services#actions-functions)
topic page where you'll see:

- Actions modify data in the server
- Functions retrieve data
- Unbound actions/functions are like plain unbound functions in JavaScript.
- Bound actions/functions always receive the bound entity's primary key as
  implicit first argument, similar to this pointers in Java or JavaScript

If you really must, you can think of the difference between bound and unbound
as similar to the difference between instance and static methods in object
oriented programming.

## Define an unbound function in the Simple service

Let's take our first steps in this regard with a simple function that should return just those products that are out of stock.

Because this is going to be read-only, a function is appropriate. Because we
want to have products returned to us from the entire set, the function is to be
unbound rather than bound.

👉 In `srv/services.cds`, add the definition for an `outOfStockProducts` function within the `Simple` service as shown:

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

Note that this unbound function is as simple as it can be for the purposes of
this introduction, and expects no arguments (there is nothing defined within
the `()` signature).

👉 Ensure that the CAP server has restarted after this change, and visit
<http://localhost:4004> to see the service endpoints; pay particular attention
to the entries for the `Simple` service, which should now include this
function:

![outOfStockProducts() listed in the service](assets/outOfStockProducts-function.png)

Because it's a function, rather than an action, it is to be invoked via HTTP
GET, which means we can try it out in the browser.

👉 Do that now - select the function
<http://localhost:4004/simple/outOfStockProducts>.

In contrast to every other request you've made, each of which has been
fulfilled by the CAP framework itself with built-in handling for all CRUD style
operations, this "out-of-band" call is something we're going to have to provide
an implementation for ourselves, as we can see from the error message returned:

```json
{
  "error": {
    "message": "Service \"Simple\" has no handler for \"outOfStockProducts\".",
    "code": "501",
    "@Common.numericSeverity": 4
  }
}
```

## Provide an implementation for the function

👉 Create a new file `srv/services.js`, with the following content:

```javascript
const cds = require('@sap/cds')

class Simple extends cds.ApplicationService {
  init() {
    const { Products } = this.entities('workshop')
    this.on('outOfStockProducts', async () => {
      return await SELECT.from(Products).where({ stock: 0 })
    })
    return super.init()
  }
}
module.exports = { Simple }
```

While digging into this JavaScript is beyond the scope of this CDS modelling
introduction workshop, there are a few points worth highlighting:

- The name of the file matches the name of the service definition file (save
  for the extension), according to the [convention over configuration
  here](https://cap.cloud.sap/docs/node.js/core-services#in-sibling-js-files-next-to-cds-sources)
- The name of the class is `Simple`, matching the `Simple` service name in the
  CDS file
- The handler for the unbound function is defined in the [on
  phase](https://cap.cloud.sap/docs/node.js/core-services#srv-on-request)
