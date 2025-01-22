# Convirgance

Convirgance gives you the power of SQL with the flexibility of modern data processing. It's designed for developers who want direct control over their database operations without the overhead of traditional ORMs.

## Why Convirgance?

Traditional ORMs force you to map your database to Java objects, adding complexity and overhead. Convirgance takes a different approach:

- **Direct SQL Control**: Write the SQL you want, get back the data you need
- **Stream Processing**: Handle large datasets efficiently without loading everything into memory
- **Format Freedom**: Read from a database, output to CSV, or transform JSON - it's all the same to Convirgance
- **SQL-Like Operations**: Use familiar concepts like WHERE clauses and GROUP BY, even on non-SQL data sources

## Quick Example

Here's how simple it is to query a database and transform the results:

```java
// Query your database
DBMS dbms = new DBMS(source);
Iterable<JSONObject> results = dbms.query(new Query("select * from CUSTOMER"));

// Filter customers (like a SQL WHERE clause)
Filter activeHighValueCustomers = new AndFilter(
    new GreaterThanFilter("purchases", 1000),
    new EqualsFilter("status", "active")
);

// Transform the data (like SQL GROUP BY)
Transformer groupByRegion = new SortedGroupByTransformer(
    new String[]{"region"},
    "customers"
);

// Output to CSV
CSVOutput output = new CSVOutput();
output.write(target, groupByRegion.transform(activeHighValueCustomers.transform(results)));
```

## Getting Started

Ready to dive in? Here's what you need to know:

1. [Getting Started Guide](getting-started.md) - Installation and your first Convirgance application
2. [Database Operations](database-operations.md) - Working with databases efficiently
3. [Filtering Data](filtering-data.md) - SQL-like operations for any data source
4. [Transforming Data](transforming-data.md) - Reshape and enrich your data
5. [File Formats](file-formats.md) - Working with CSV, JSON, and more

## Core Concepts

### Database Operations

Handle database interactions with confidence. Atomic operations ensure your transactions succeed or fail cleanly, while batch processing keeps things fast.

### Filters

Apply SQL-like conditions to any data source. Combine filters with AND/OR operations just like you would in a WHERE clause.

### Transformations

Build data pipelines that clean and reshape your data. Convert types, group records, or compute new fields - all without loading everything into memory.

### File Formats

Work with data in any format. Read from CSV, transform in memory, write to JSON - all using the same consistent API.

## Installation

Using Maven:

```xml
<dependency>
    <groupId>com.convirgance</groupId>
    <artifactId>convirgance-core</artifactId>
    <version>$VERSION$</version>
</dependency>
```

## Community and Support

We're here to help:

- [GitHub Issues](https://github.com/convirgance/issues) - Report bugs or request features
- [Documentation](https://docs.convirgance.com) - Detailed guides and API reference
- [Contact](contact.md) - Get in touch with the team

## Contributing

We welcome contributions! Check out our [Contributing Guide](CONTRIBUTING.md) to get started.

## License

Convirgance is available under the Apache 2.0 License. See [LICENSE](LICENSE) for details.
