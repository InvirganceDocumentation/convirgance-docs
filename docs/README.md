# Convirgance

Convirgance is a modern, streamlined approach to database access. Unlike traditional ORM frameworks that map your database into Java objects, Convirgance gives you direct SQL control while returning results as a stream of `Map` objects. These objects can be manipulated, filtered, and transformed using common operations, making it an excellent drop in tool for querying and managing data.

## Why Convirgance?

Traditional ORMs force you to map your database to Java objects, adding complexity and overhead. Convirgance takes a different approach:

- **Direct SQL Control**: Write the SQL you want, get back the data you need.
- **Stream Processing**: Handle large datasets efficiently without loading everything into memory.
- **Format Freedom**: Read from a database, output to CSV, or transform JSON - it's all the same to Convirgance.
- **SQL-Like Operations**: Our filters and transformers provide concepts like WHERE clauses and GROUP BY, allowing you to work with data in a familiar way.

## Simple Example

Here are a few examples showcasing the simplicity convirgance offers:

### Database to CSV

In this example we are taking the query `results` and writing out the contents to a CSV. All of that in only 7 LoC (lines of code)!

```java
// Query your database
DBMS database = new DBMS(source);
Query query = new Query("select name, devices, pets from CUSTOMER");
Iterable<JSONObject> results = database.query(query);

File file = new File("./example.csv");
FileTarget target = new FileTarget(file);

// Output to CSV
CSVOutput output = new CSVOutput();
output.write(target, results);

/*
Contents of 'examples.csv'

name, devices, pets
John, 3, 1
Bob, 1, 2
Kyle, 1, 10
*/
```

### Filtering Database results

Deduplicating the results from a join query between two tables.

```java
DBMS database = new DBMS(source);
Query query = new Query("SELECT * FROM \"orders\" o\n"
                + "JOIN order_line l ON l.order_id = o.order_id "
                + "order by o.order_id, l.line_id");

Iterable<JSONObject> results = dbms.query(query);

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

## Community and Support

We're here to help:

<div style="display: flex; align-items: center; gap: 8px; margin-bottom: 16px">
 <img src="./images/github.png" width="24" height="24" style="display: flex; align-items: center; justify-content: center;">
 <div>
     <a href="https://github.com/InvirganceOpenSource/convirgance">Convirgance</a>
     <span>- Report bugs or request features</span>
 </div>
</div>

<div style="display: flex; align-items: center; gap: 8px; margin-bottom: 16px">
  <span style="display: flex; align-items: center; justify-content: center;font-size:20px; width: 24px; height: 24px">📑</span>
  <div>
    <a href="./#/contact.md">Contact</a>
    <span>- Get in touch with the team</span>
  </div>
</div>
<div style="display: flex; align-items: center; gap: 8px; margin-bottom: 16px">
  <span style="display: flex; align-items: center; justify-content: center;font-size:20px; width: 24px; height: 24px">📚</span>
  <div>
    <a href="https://docs.invirgance.com/javadocs/convirgance/latest/com/invirgance/convirgance/package-summary.html">Java Documentation</a>
    <span>- Have a look behind the scenes</span>
  </div>
</div>

## License

Convirgance is available under the MIT License. See [License](https://raw.githubusercontent.com/InvirganceOpenSource/convirgance/refs/heads/main/LICENSE.md) for more details.
