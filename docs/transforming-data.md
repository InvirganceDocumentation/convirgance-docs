# Data Transformations

Transformations let you modify your data as it flows through your application. Think of them as a pipeline where each step can clean, enrich, or reshape your data. Common uses include converting data types (like turning strings into numbers), grouping related records together (similar to SQL GROUP BY), or adding computed fields. This allows you to adapt data from one format or structure to another without loading everything into memory at once.

## Transformers

| Name                       | Description                                                                                                  |
| -------------------------- | ------------------------------------------------------------------------------------------------------------ |
| CoerceStringsTransformer   | Transforms objects into their actual counterparts, evaluating key values into their real data type.          |
| IdentityTransformer        | A Interface used when creating transformers.                                                                 |
| InsertKeyTransformer       | For replacing existing key:value pairs.                                                                      |
| SortedGroupByTransformer   | Used when working with data thats already sorted, grouping similar values on a provided JSON field criteria. |
| Transformer                | The heart of the transformer library, implement this when creating your own.                                 |
| UnsortedGroupByTransformer | The opposite of SortedGroupBy, works on unsorted data and returns the sorted + grouped version.              |

### Examples

Below are two examples, one goes over the `SorterGroupByTransfromer` and the other demonstrates how to use the `Transformer` interface to create an anonymous class.

#### SorterGroupByTransfromer Example

The following example showcases joining two tables, and using the `SortedGroupByTransformer` to group the data into a more compact from.

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
The transformer would return the following. In comparison to what the database returned this output is much more concise. Also notice that the duplicate fields were removed.
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

#### Interface Example

You can create anonymous classes using the `Transform` interface to quickly create your own transformers.

```java
Transformer transformer = new Transformer() {
    @Override
    public Iterator<JSONObject> transform(Iterator<JSONObject> iterator) {
        return new Iterator<JSONObject>() {
            @Override
            public boolean hasNext() {
                return iterator.hasNext();
            }

            @Override
            public JSONObject next() {
                JSONObject obj = iterator.next();
                // Example transformation: Add timestamp
                obj.put("timestamp", System.currentTimeMillis());
                return obj;
            }
        };
    }
};

List<JSONObject> data = Arrays.asList(new JSONObject(), new JSONObject());
Iterable<JSONObject> transformed = transformer.transform(data);
```

## Best Practices

- Use `CoerceStringsTransformer` to standardize data types before processing.
- Leverage `SortedGroupByTransformer` for efficient grouping when working with pre-sorted data.
- Apply `UnsortedGroupByTransformer` when sorting is not guaranteed but grouping is required.
- Combine transformers to build complex transformation pipelines.

## Further Reading

<div style="display: flex; align-items: center; gap: 8px; margin-bottom: 16px">
  <span style="display: flex; align-items: center; justify-content: center;font-size:20px; width: 24px; height: 24px">📚</span>
  <a href="https://docs.invirgance.com/javadocs/convirgance/latest/com/invirgance/convirgance/transform/package-summary.html">Java Documentation: Transformers</a>
</div>
