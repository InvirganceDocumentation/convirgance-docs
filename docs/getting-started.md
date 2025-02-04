# Getting Started with Convirgance

Welcome to the Convirgance documentation! In this section we'll go over getting started, first installing and then covering a few simple examples.

As you read through the documentation, I encourage you to build along and explore how different parts of the library interact with each other. It may seem daunting at first, but this is only because it's new. Convirgance is crazy simple-just give it a shot.

## Installation

Add the following dependency to your Maven `pom.xml` file:

```xml
<dependency>
    <groupId>com.invirgance</groupId>
    <artifactId>convirgance</artifactId>
    <version>1.0.0</version>
</dependency>
```

## Quick Start

To start, here is a example of querying some database and writing out the resulting data in JSON format:

```java
// Query the database
DBMS database = new DBMS(source);
Query query = new Query("select name, devices, pets from CUSTOMER");
Iterable<JSONObject> results = database.query(query);

// Specify the target file
FileTarget target = new FileTarget("example.json");

// Write the stream to a JSON file
new JSONOutput().write(target, results);
```

## Inputs and Outputs

After transforming your data, Convirgance supports various input and output options you can also implement support for other file-types by using our [interfaces](/file-formats?id=example-properties-file).

### Delimited:

Lets reuse the previous JSON file we created from the last example. We will take the current JSON records and turn them into a delimited format. You should note that when working with delimited files you can select which keys to keep, and if needed set the character to delimit with.

```java
// Reading in the JSON file from the previous example
FileSource example = new FileSource("example.json");
Iterable<JSONObject> records = JSONInput().read(example);

// Notice that the query is using *, we can narrow down the output by setting the headers to use.
String wanted = new String[]{"names", "devices", "pets"};

// Here we supply the headers we want, along with the character to delimit with.
DelimitedOutput output = new DelimitedOutput(wanted, '?');

File file = new File("example.qdv");
FileTarget target = new FileTarget(file);

output.write(target, results);
```

### JSON:

JSONObjects are created based on the coloum names returned from the database query.

```java
DBMS dbms = new DBMS(source);
Query query = new Query("select name, pets, devices from CUSTOMER");

Iterable<JSONObject> results = dbms.query(query);

File file = new File("example.json");
FileTarget target = new FileTarget(file);

new JSONOutput().write(target, results);

```

The contents of `example.json` would look like below:

```json
{ "name": "John", "pets": "2", "devices": 2 }
```

And here is an example inserting all the JSON records from a file back into the database. You can see named binds and `BatchOperation` in use here, you will learn more about these in later chapters.

```java
DBMS database = new DBMS(source);

String template = "INSERT INTO customer (id, name, age) VALUES (:id, :name, :age)";
Query query = new Query(template);

FileSource example = new FileSource(jsonFile);
Iterable<JSONObject> records = JSONInput().read(example);

BatchOperation batch = new BatchOperation(query, records);

database.execute(batch);
```

### CSV:

CSV known as comma seperated values. When writing, the header names can be provided, otherwise the JSON keys will be used. The same idea applies when reading in a CSV file.

```java
DBMS dbms = new DBMS(source);
Query query = new Query("select name, pets, devices from CUSTOMER");

Iterable<JSONObject> results = dbms.query(query);

File file = new File("example.csv");
FileTarget target = new FileTarget(file);

// The values to include in the CSV.
String wanted = new String[]{ "name", "devices", "house" };

new CSVOutput(wanted).write(target, results);
```

Here is what the exported data of `example.csv` would look like:

| name | devices | house |
| ---- | ------- | ----- |
| John | 2       |       |
| ...  | ...     | ...   |

## Quick Links

Ready to dive deeper? Here's what you need to know:

1. [Core Concepts](core-concepts.md) - The concepts underpinning Convirgance architecture
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
