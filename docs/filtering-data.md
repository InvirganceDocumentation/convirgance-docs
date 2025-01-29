# Filters

Filters provide SQL-like operations for working with your data stream. You can use familiar concepts like equals, greater than, less than, and combine them with AND/OR operations - just like you would in a SQL WHERE clause. This gives you the power of SQL filtering even when working with non-SQL data sources. For example, you can easily filter records where age > 18 AND status = 'active', or find all users who are either admins OR moderators.

# Core Interfaces and Classes

| Symbol   | Description                                         |
| -------- | --------------------------------------------------- |
| `Filter` | Base interface for filter evaluation and conditions |
| `&&`     | Combines filters with AND logic                     |
| `\|\|`   | Combines filters with OR logic                      |
| `!=`     | Inverts filter result                               |
| `==`     | Coercive equality comparison                        |
| `===`    | Strict equality comparison                          |
| `>`      | Greater than comparison                             |
| `>=`     | Greater than or equal comparison                    |
| `<`      | Less than comparison                                |
| `<=`     | Less than or equal comparison                       |

## Filter Interface

The `Filter` interface extends `Transformer`, allowing filters to work within transformation pipelines.

```java
File file = new File("./clientData.json");
FileSource source = new FileSource(file);
Iterator<JSONObject> records = new JSONInput().read(source);

String key = "name";
String find = "Smith";

Filter nameFilter = new Filter()
{
    @Override
    public boolean test(JSONObject record)
    {
        return record.get(key).contains(find);
    }
};

Iterator<JSONObject> filtered = nameFilter.transform(records);
```

### Examples

The below example filters data from some `file` source. Compararing it to the database.

```java
DBMS database = new DBMS(source);

String search = "Select last_update FROM customer";

ComparatorFilter dateFilter = new ComparatorFilter()
{
    @Override
    public boolean test(JSONObject record)
    {
        Date otherDate = new SimpleDateFormat("yyyy-MM-dd").parse(record.get(getKey()));
        Date currentDate = new SimpleDateFormat("yyyy-MM-dd").parse(getValue());

        return currentDate.after(otherDate);
    }
};

Iterable<JSONObject> old = database.query(new Query(search));

JSONArray updatedItems = new JSONArray();

dateFilter.setKey("last_update");
dateFilter.setValue("2007-01-01");

for(JSONObject oldRecord : old)
{
    if(dateFilter.test(oldRecord))
    {
        updatedItems.put(oldRecord);
        continue;
    }
}


```

## Logical Filters

### AndFilter

Combines multiple filters, requiring all of them to evaluate the record as true.

```java
Filter filter = new AndFilter(
    new GreaterThanFilter("age", 18),
    new EqualsFilter("status", "active")
);

Iterable<JSONObject> filtered = filter.transform(sourceData);
```

### OrFilter

Combines multiple filters, requiring at least one to evaluate the record as true.

```java
Filter filter = new OrFilter(
    new EqualsFilter("role", "admin"),
    new EqualsFilter("role", "moderator")
);

Iterable<JSONObject> filtered = filter.transform(sourceData);
```

### NotFilter

Inverts the result of another filter.

```java
Filter filter = new NotFilter(new EqualsFilter("status", "inactive"));

Iterable<JSONObject> filtered = filter.transform(sourceData);
```

## Comparator-Based Filters

### EqualsFilter

Filters records where a specified key's value is equal to a given value.

```java
Filter filter = new EqualsFilter("country", "USA");

Iterable<JSONObject> filtered = filter.transform(sourceData);
```

### GreaterThanFilter

Filters records where a specified key's value is greater than a given value.

```java
Filter filter = new GreaterThanFilter("age", 30);

Iterable<JSONObject> filtered = filter.transform(sourceData);
```

### GreaterThanOrEqualFilter

Filters records where a specified key's value is greater than or equal to a given value.

```java
Filter filter = new GreaterThanOrEqualFilter("score", 85);

Iterable<JSONObject> filtered = filter.transform(sourceData);
```

### LessThanFilter

Filters records where a specified key's value is less than a given value.

```java
Filter filter = new LessThanFilter("price", 100);

Iterable<JSONObject> filtered = filter.transform(sourceData);
```

### LessThanOrEqualFilter

Filters records where a specified key's value is less than or equal to a given value.

```java
Filter filter = new LessThanOrEqualFilter("quantity", 50);

Iterable<JSONObject> filtered = filter.transform(sourceData);
```

## Custom Comparisons

The `CoerciveComparator` allows for comparisons between values of different types, supporting nulls, numbers, and other comparables. It is used internally by `ComparatorFilter` and its subclasses for flexible and robust evaluations.

## Best Practices

- Combine filters with `AndFilter` and `OrFilter` to define complex filtering criteria.
- Use `NotFilter` to exclude records based on specific conditions.
- When working with numeric or comparable fields, leverage `ComparatorFilter` subclasses like `GreaterThanFilter` for concise evaluations.

## Further Reading

- [Java Documentation](https://docs.invirgance.com/javadocs/convirgance/latest/com/invirgance/convirgance/transform/filter/package-summary.html)
