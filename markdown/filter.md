# Filters

Filters provide SQL-like operations for working with your data stream. You can use familiar concepts like equals, greater than, less than, and combine them with AND/OR operations - just like you would in a SQL WHERE clause. This gives you the power of SQL filtering even when working with non-SQL data sources. For example, you can easily filter records where age > 18 AND status = 'active', or find all users who are either admins OR moderators.

## Core Interfaces and Classes

- **Filter**: The base interface for defining filters that evaluate each record in a collection and return only those that meet specified conditions.
- **AndFilter**: Combines multiple filters, requiring all to evaluate the record as true.
- **OrFilter**: Combines multiple filters, requiring at least one to evaluate the record as true.
- **NotFilter**: Inverts the result of another filter.
- **ComparatorFilter**: Abstract class for filters that use a coercive comparator to evaluate key-value conditions.
- **EqualsFilter**: Filters records where a specified key's value is equal to a given value.
- **GreaterThanFilter**: Filters records where a specified key's value is greater than a given value.
- **GreaterThanOrEqualFilter**: Filters records where a specified key's value is greater than or equal to a given value.
- **LessThanFilter**: Filters records where a specified key's value is less than a given value.
- **LessThanOrEqualFilter**: Filters records where a specified key's value is less than or equal to a given value.

## Filter Interface

The `Filter` interface extends `Transformer`, allowing filters to work within transformation pipelines.

### Key Methods

- `Iterator<JSONObject> transform(Iterator<JSONObject> iterator)`: Applies the filter condition to each record in the iterator, returning only those that pass the filter. This method is inherited from the `Transformer` interface.
- `boolean test(JSONObject record)`: Tests if the given record meets the filter condition. If the record passes the condition, it returns `true`, otherwise `false`.

```java
// Custom Filter that checks if a field contains a specific substring
public class ContainsSubstringFilter implements Filter {
    private String field;
    private String substring;
    
    // Constructor to initialize the field and substring to check for
    public ContainsSubstringFilter(String field, String substring) {
        this.field = field;
        this.substring = substring;
    }
    
    // Implementing the 'test' method from Filter interface
    @Override
    public boolean test(JSONObject record) {
        // Check if the field exists and is a string
        if (record.has(field) && record.get(field) instanceof String) {
            String value = record.getString(field);
            return value.contains(substring);
        }
        
        return false;
    }
    
    // Implementing the 'transform' method from Transformer interface
    @Override
    public Iterator<JSONObject> transform(Iterator<JSONObject> iterator) {
        return new Iterator<JSONObject>() {
            @Override
            public boolean hasNext() {
                while (iterator.hasNext()) {
                    JSONObject next = iterator.next();
                    // Skip any records that don't pass the filter
                    if (test(next)) return true;
                }
                
                return false;
            }
            
            @Override
            public JSONObject next() {
                return iterator.next();
            }
        };
    }
}

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