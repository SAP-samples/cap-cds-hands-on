# 07 - Link entities together with associations

In this exercise we'll learn how to use associations in CDL to relate entities together.

## Review what we have and what's missing

Right now in our `db/schema.cds` file we have a couple of entities, `Products`
and `Suppliers`:

```cds
entity Products : cuid, managed {
  name  : String;
  stock : Integer;
  price : Price;
}

entity Suppliers : cuid, managed {
  company : String;
}
```

Logically there is a relationship between these two entities, as demonstrated
in the original
[Northbreeze](https://developer-challenge.cfapps.eu10.hana.ondemand.com/odata/v4/northbreeze)
service.

[Each product comes from a certain supplier](https://developer-challenge.cfapps.eu10.hana.ondemand.com/odata/v4/northbreeze/Products?$top=5&$select=ProductName&$expand=Supplier($select=CompanyName)):

```json
{
  "@odata.context": "$metadata#Products(ProductName,ProductID,Supplier(CompanyName,SupplierID))",
  "value": [
    {
      "ProductName": "Chai",
      "ProductID": 1,
      "Supplier": {
        "SupplierID": 1,
        "CompanyName": "Exotic Liquids"
      }
    },
    {
      "ProductName": "Chang",
      "ProductID": 2,
      "Supplier": {
        "SupplierID": 1,
        "CompanyName": "Exotic Liquids"
      }
    },
    {
      "ProductName": "Aniseed Syrup",
      "ProductID": 3,
      "Supplier": {
        "SupplierID": 1,
        "CompanyName": "Exotic Liquids"
      }
    },
    {
      "ProductName": "Chef Anton's Cajun Seasoning",
      "ProductID": 4,
      "Supplier": {
        "SupplierID": 2,
        "CompanyName": "New Orleans Cajun Delights"
      }
    },
    {
      "ProductName": "Chef Anton's Gumbo Mix",
      "ProductID": 5,
      "Supplier": {
        "SupplierID": 2,
        "CompanyName": "New Orleans Cajun Delights"
      }
    }
  ]
}
```

Also, [some suppliers have more than one product available](https://developer-challenge.cfapps.eu10.hana.ondemand.com/odata/v4/northbreeze/Suppliers?$top=3&$select=CompanyName&$expand=Products($select=ProductName)):

```json
{
  "@odata.context": "$metadata#Suppliers(CompanyName,SupplierID,Products(ProductName,ProductID))",
  "value": [
    {
      "CompanyName": "Exotic Liquids",
      "SupplierID": 1,
      "Products": [
        {
          "ProductID": 1,
          "ProductName": "Chai"
        },
        {
          "ProductID": 2,
          "ProductName": "Chang"
        },
        {
          "ProductID": 3,
          "ProductName": "Aniseed Syrup"
        }
      ]
    },
    {
      "CompanyName": "New Orleans Cajun Delights",
      "SupplierID": 2,
      "Products": [
        {
          "ProductID": 4,
          "ProductName": "Chef Anton's Cajun Seasoning"
        },
        {
          "ProductID": 5,
          "ProductName": "Chef Anton's Gumbo Mix"
        },
        {
          "ProductID": 65,
          "ProductName": "Louisiana Fiery Hot Pepper Sauce"
        },
        {
          "ProductID": 66,
          "ProductName": "Louisiana Hot Spiced Okra"
        }
      ]
    },
    {
      "CompanyName": "Grandma Kelly's Homestead",
      "SupplierID": 3,
      "Products": [
        {
          "ProductID": 6,
          "ProductName": "Grandma's Boysenberry Spread"
        },
        {
          "ProductID": 7,
          "ProductName": "Uncle Bob's Organic Dried Pears"
        },
        {
          "ProductID": 8,
          "ProductName": "Northwoods Cranberry Sauce"
        }
      ]
    }
  ]
}
```

What we really need in our model is a similar relationship, one that goes both ways:

- `Products` -> `Suppliers`
- `Suppliers` -> `Products`

## Use an association to link a product to a supplier

In CDS modelling, there is the concept of
[associations](https://cap.cloud.sap/docs/cds/cdl#associations) whose purpose
is to describe relationships between entities.

Our first task, to declare a `Products` -> `Suppliers` relationship, can be
achieved with a so-called "managed to-one association". What does that mean?

The "to-one" part is half of the classic [one-to-one](https://en.wikipedia.org/wiki/One-to-one_(data_model)) relationship:

```text
+-----+  1:1  +-----+
|  A  |<----->|  B  |
+-----+       +-----+
```

Why only half of it? Well, the one-to-one model is "_a type of cardinality that
refers to the relationship between two entities A and B in which one element of
A may only be linked to one element of B, **and vice versa**._"

