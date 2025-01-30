# Convirgance

Convirgance is a modern, streamlined approach to database access. Unlike traditional ORM frameworks that map your database into Java objects, Convirgance gives you direct SQL control while returning results as a stream of `Map` objects. These objects can be manipulated, filtered, and transformed using common operations, making it an excellent drop in tool for querying and managing data.

## Why Convirgance?

Traditional ORMs force you to map your database to Java objects, adding complexity and overhead. Convirgance takes a different approach:

- **Direct SQL Control**: Write the SQL you want, get back the data you need
- **Stream Processing**: Handle large datasets efficiently without loading everything into memory
- **Format Freedom**: Read from a database, output to CSV, or transform JSON - it's all the same to Convirgance
- **SQL-Like Operations**: Use familiar concepts like WHERE clauses and GROUP BY, even on non-SQL data sources

## Simple Example

Here's how simple it is to query a database and transform the results:

```java
// Query your database
DBMS database = new DBMS(source);
Query query = new Query("select * from CUSTOMER");
Iterable<JSONObject> results = database.query(query);

for(JSONObject record : results.iterator())
{
  System.out.println(record.toString(4));
}

// Output to CSV
CSVOutput output = new CSVOutput();
output.write(target, results);
```

```java
/*
  The contents of `results`
  {"ORDER_ID":1,"TOTAL":54.12,"ITEMS":3,"RECIPIENT":"bob","LINE_ID":1,"PRODUCT":"Fish tank","PRICE":30.00,"QUANTITY":1}
  {"ORDER_ID":1,"TOTAL":54.12,"ITEMS":3,"RECIPIENT":"bob","LINE_ID":2,"PRODUCT":"Fish food","PRICE":4.00,"QUANTITY":3}
*/
String[] fields = new String[]
{
    "ORDER_ID", "RECIPIENT", "TOTAL", "ITEMS"
};
SortedGroupByTransformer sorter = new SortedGroupByTransformer(fields, "lines");
Iterable<JSONObject> customerData = sorter.transform(results)

/*
The transformer would return the following. In comparison to what the database returned this output is much more concise. Also notice that the duplicate fields were removed.
  {
    "RECIPIENT": "bob",
    "TOTAL": 42,
    "ORDER_ID": 1,
    "ITEMS": 2,
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
      }
    ]
  }
*/
```

## Getting Started

Ready to dive in? Here's what you need to know:

1. [Getting Started Guide](getting-started.md) - Installation and examples
2. [Database Operations](database-operations.md) - Working with databases efficiently
3. [Filtering Data](filtering-data.md) - SQL-like operations for any data source
4. [Transforming Data](transforming-data.md) - Reshape and enrich your data
5. [File Formats](file-formats.md) - Working with CSV, JSON, and more

## Core Concepts and Goals

### Database Operations

Handle database interactions with confidence. Atomic operations ensure your transactions succeed or fail cleanly, while batch processing keeps things fast. Streamlining any interaction that you make with any database, all without the overhead from your typical ORM.

### Data Flexibility

Using our pluggable transformers and filters you can manipulate your data in whatever way is needed. Further more our library will handle all your file type needs, JSON, CSV, Delimimted. And, if support doesn't exist we have you covered with our easily extendible interfaces.

#### Filters

Apply SQL-like conditions to any data source. Combine filters with AND/OR operations just like you would in a WHERE clause.

#### Transformations

Build data pipelines that clean and reshape your data. Convert types, group records, or compute new fields - all without loading everything into memory.

#### File Formats

Work with data in any format. Read from CSV, transform in memory, write to JSON - all using the same consistent API.

## Community and Support

We're here to help:

<div style="display: flex; align-items: center; gap: 8px; margin-bottom: 16px">
 <img src="/images/github.png" width="24" height="24" style="display: flex; align-items: center; justify-content: center;">
 <div>
     <a href="https://github.com/InvirganceOpenSource/convirgance">Convirgance</a>
     <span>- Report bugs or request features</span>
 </div>
</div>

<div style="display: flex; align-items: center; gap: 8px; margin-bottom: 16px">
  <span style="display: flex; align-items: center; justify-content: center;font-size:20px; width: 24px; height: 24px">📑</span>
  <div>
    <a href="/#/contact.md">Contact</a>
    <span>- Get in touch with the team</span>
  </div>
</div>

## License

Convirgance is available under the MIT License. See [License](https://raw.githubusercontent.com/InvirganceOpenSource/convirgance/refs/heads/main/LICENSE.md) for more details.
