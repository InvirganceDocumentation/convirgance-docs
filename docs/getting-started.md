# Getting Started with Convirgance

Welcome to the Convirgance documentation! In this section we'll go over getting started, first installing and then covering a few simple examples.

As you read through the documentation, I encourage you to build along and explore how different parts of the library interact with each other. It may seem daunting at first, but this is only because it's new. Convirgance is crazy simple–just give it a shot.

## Installation

Add the following dependency to your Maven `pom.xml` file:

```xml
<dependency>
    <groupId>com.invirgance</groupId>
    <artifactId>convirgance</artifactId>
    <version>1.3.1</version>
</dependency>
```

## Quick Start

Imagine you’re working on a web service that manages customer records. First, you need to display customer data in a structured format, so you query the database and output the results as JSON.

```java
DBMS dbms = new DBMS(source);
Query query = new Query("SELECT name, devices, pets FROM CUSTOMER");

Iterable<JSONObject> results = dbms.query(query);
FileTarget target = new FileTarget("example.json");

new JSONOutput().write(target, results);
```

This generates a JSON file like:

```json
{ "name": "John", "devices": 2, "pets": 2 }
```

Now, suppose a business user needs the same data in a structured report but prefers a delimited format. You can transform the JSON data into a custom delimited text file by specifying which fields to include and the delimiter character:

```java
FileSource example = new FileSource("example.json");
Iterable<JSONObject> records = JSONInput().read(example);

String wanted = new String[]{ "name", "devices", "pets" };
DelimitedOutput output = new DelimitedOutput(wanted, '?');

FileTarget target = new FileTarget("example.txt");
output.write(target, records);
```

For broader compatibility, a CSV export might be needed for spreadsheet applications. You can generate a CSV version with headers:

```java
FileTarget target = new FileTarget("example.csv");

String wanted = new String[]{ "name", "devices", "pets" };
new CSVOutput(wanted).write(target, results);
```

Which results in:

| name | devices | pets |
| ---- | ------- | ---- |
| John | 2       | 2    |

Later, when users submit updates, you need to process incoming JSON records and batch-insert them into the database. Using named binds and batch operations, you can efficiently handle multiple records at once:

```java
DBMS database = new DBMS(source);
String template = "INSERT INTO customer (id, name, age) VALUES (:id, :name, :age)";
Query query = new Query(template);

FileSource example = new FileSource("updates.json");
Iterable<JSONObject> records = new JSONInput().read(example);

BatchOperation batch = new BatchOperation(query, records);
database.update(batch);
```

We can see by example, Convirgance can be used to seamlessly handle data as required, producing JSON for web consumption, exporting to delimited formats for reporting, and re-importing updates for database synchronization.

## Quick Links

Ready to dive deeper? Here's what you need to know:

- [Core Concepts](core-concepts.md) - The concepts underpinning Convirgance architecture
- [Database Operations](database-operations.md) - Working with databases efficiently
- [Filtering Data](filtering-data.md) - SQL-like operations for any data source
- [Transforming Data](transforming-data.md) - Reshape and enrich your data
- [File Formats](file-formats.md) - Working with CSV, JSON, and more


## Sections

##### [Previous: Introduction](./?id=convirgance)

##### [Next: Core Concepts](./concepts?id=core-concepts-and-goals)
