# Data Transformations

Transformations let you modify your data as it flows through your application. Think of them as a pipeline where each step can clean, enrich, or reshape your data. Common uses include converting data types (like turning strings into numbers), grouping related records together (similar to SQL GROUP BY), or adding computed fields. This allows you to adapt data from one format or structure to another without loading everything into memory at once.

## Core Interfaces and Classes

- **Transformer**: The base interface for applying transformations to JSON objects lazily during iteration.
- **IdentityTransformer**: A specialized transformer that guarantees one output record per input record, allowing modifications to individual records.
- **CoerceStringsTransformer**: Converts string values into other data types, such as `Integer`, `Double`, and `Boolean`.
- **InsertKeyTransformer**: Adds or modifies a key-value pair in JSON objects.
- **SortedGroupByTransformer**: Groups JSON objects based on specified fields, assuming the data is already sorted.
- **UnsortedGroupByTransformer**: Groups JSON objects on specified fields without requiring prior sorting.

## Transformer Interface

The `Transformer` interface provides a way to lazily apply transformations to JSON objects.

### Key Method

- `Iterable<JSONObject> transform(Iterable<JSONObject> iterable)`: Lazily transforms a collection of JSON objects.

### Usage Example

```java
Transformer transformer = new CoerceStringsTransformer();
Iterable<JSONObject> transformed = transformer.transform(sourceData);

for (JSONObject record : transformed) {
    System.out.println(record);
}
```

## CoerceStringsTransformer

The `CoerceStringsTransformer` converts string values into other data types such as `Integer`, `Double`, and `Boolean`.

### Usage Example

```java
Transformer transformer = new CoerceStringsTransformer();
Iterable<JSONObject> transformed = transformer.transform(sourceData);

for (JSONObject record : transformed) {
    System.out.println(record);
}
```

## InsertKeyTransformer

The `InsertKeyTransformer` allows adding or modifying a key-value pair in JSON objects.

### Constructor
- `InsertKeyTransformer(String key, Object value)`: Initializes the transformer with the key and value to insert.

### Usage Example

```java
Transformer transformer = new InsertKeyTransformer("newKey", "newValue");
Iterable<JSONObject> transformed = transformer.transform(sourceData);

for (JSONObject record : transformed) {
    System.out.println(record);
}
```

## SortedGroupByTransformer

The `SortedGroupByTransformer` groups data based on specified fields. The data must already be sorted for accurate grouping.

### Constructor
- `SortedGroupByTransformer(String[] fields, String output)`: Groups related data on the specified fields and assigns it to the output field.

### Usage Example

```java
Transformer transformer = new SortedGroupByTransformer(
    new String[]{"field1", "field2"},
    "groupedData"
);
Iterable<JSONObject> transformed = transformer.transform(sourceData);

for (JSONObject record : transformed) {
    System.out.println(record);
}
```

## UnsortedGroupByTransformer

The `UnsortedGroupByTransformer` groups data based on specified fields without requiring the data to be sorted beforehand.

### Constructor
- `UnsortedGroupByTransformer(String[] fields, String output)`: Groups related data on the specified fields and assigns it to the output field.

### Usage Example

```java
Transformer transformer = new UnsortedGroupByTransformer(
    new String[]{"field1"},
    "groupedData"
);
Iterable<JSONObject> transformed = transformer.transform(sourceData);

for (JSONObject record : transformed) {
    System.out.println(record);
}
```

## Best Practices

- Use `CoerceStringsTransformer` to standardize data types before processing.
- Leverage `SortedGroupByTransformer` for efficient grouping when working with pre-sorted data.
- Apply `UnsortedGroupByTransformer` when sorting is not guaranteed but grouping is required.
- Combine transformers to build complex transformation pipelines.