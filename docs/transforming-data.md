# Data Transformations

Transformations let you modify your data as it flows through your application. Think of them as a pipeline where each step can clean, enrich, or reshape your data. Common uses include converting data types (like turning strings into numbers), grouping related records together (similar to SQL GROUP BY), or adding computed fields. This allows you to adapt data from one format or structure to another without loading everything into memory at once.

| Name                       | Description                                                                                                  |
| -------------------------- | ------------------------------------------------------------------------------------------------------------ |
| CoerceStringsTransformer   | Transforms objects into their actual counterparts, evaluating key values into their real data type.          |
| IdentityTransformer        | A Interface used when creating transformers.                                                                 |
| InsertKeyTransformer       | For replacing existing key:value pairs.                                                                      |
| SortedGroupByTransformer   | Used when working with data thats already sorted, grouping similar values on a provided JSON field criteria. |
| Transformer                | The heart of the transformer library, implement this when creating your own.                                 |
| UnsortedGroupByTransformer | The opposite of SortedGroupBy, works on unsorted data and returns the sorted + grouped version.              |

### Usage Example

The following example showcases joining two table, using the `SortedGroupByTransformer` to group the data into a more compact from.

```java
String[] fields = new String[]
{
    "ORDER_ID", "RECIPIENT", "TOTAL", "ITEMS"
};
SortedGroupByTransformer sorter = new SortedGroupByTransformer(fields, "lines");

Query query = new Query("SELECT * FROM \"orders\" o\n"
                + "JOIN order_line l ON l.order_id = o.order_id "
                + "order by o.order_id, l.line_id");

Iterable<JSONObject> results = dbms.query(query);

/*
Results would look like something similiar to this, not very useful. But it looks like the data is sorted.

  {"ORDER_ID":1,"TOTAL":54.12,"ITEMS":3,"RECIPIENT":"bob","LINE_ID":1,"PRODUCT":"Fish tank","PRICE":30.00,"QUANTITY":1}
  {"ORDER_ID":1,"TOTAL":54.12,"ITEMS":3,"RECIPIENT":"bob","LINE_ID":2,"PRODUCT":"Fish food","PRICE":4.00,"QUANTITY":3}
  {"ORDER_ID":1,"TOTAL":54.12,"ITEMS":3,"RECIPIENT":"bob","LINE_ID":3,"PRODUCT":"Fish filter","PRICE":12.12,"QUANTITY":1}
*/

Iterable<JSONObject> customerData = sorter.transform(results)

/*
Using the transformer would return the following. Much more concise, and notably a much smaller footprint
  {
    "RECIPIENT": "bob",
    "TOTAL": 54.12,
    "ORDER_ID": 1,
    "ITEMS": 3,
    "lines": [
      {
        "LINE_ID": 1,
        "PRODUCT": "Fish tank",
        "PRICE": 30,
        "QUANTITY": 1
      },
      {
        "LINE_ID": 2,
        "PRODUCT": "Fish food",
        "PRICE": 4,
        "QUANTITY": 3
      },
      {
        "LINE_ID": 3,
        "PRODUCT": "Fish filter",
        "PRICE": 12.12,
        "QUANTITY": 1
      }
    ]
  }
*/
```

## Best Practices

- Use `CoerceStringsTransformer` to standardize data types before processing.
- Leverage `SortedGroupByTransformer` for efficient grouping when working with pre-sorted data.
- Apply `UnsortedGroupByTransformer` when sorting is not guaranteed but grouping is required.
- Combine transformers to build complex transformation pipelines.

## Further Reading

- [Java Documentation](https://docs.invirgance.com/javadocs/convirgance/latest/com/invirgance/convirgance/transform/package-summary.html)
