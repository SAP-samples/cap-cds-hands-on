# 14 - Declarative contraints and assertions

TAKEN OUT FROM 12

Make the `Percentage` type (from ex 12) better with an assert range:

```cds
type Percentage : Integer @assert.range: [
  1,
  100
];
```

- the `Percentage` type definition is annotated with a [range
  assertion](https://cap.cloud.sap/docs/guides/providing-services#assert-range),
  a feature of CAP's multifaceted [input
  validation](https://cap.cloud.sap/docs/guides/providing-services#input-validation)
  where we can yet again express intent (we're expecting a percentage value)
  and let the framework do the rest

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
