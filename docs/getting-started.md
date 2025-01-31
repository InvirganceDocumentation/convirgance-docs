# Getting Started with Convirgance

Welcome to the Convirgance documentation! In this section we'll go over getting started, first installing and then covering a few simple examples.

As you read through the documentation, I encourage you to build along and explore how different parts of the library interact with each other. It may seem daunting at first, but this is only because it's new. Convirgance is crazy simple-just give it a shot.

## Installation

Using Maven add the following to your `pom` file:

```xml
<dependency>
    <groupId>com.invirgance</groupId>
    <artifactId>convirgance</artifactId>
    <version>1.0.0</version>
</dependency>
```

Or use the CLI:

```sh
mvn dependency:get -Dartifact=com.invirgance:convirgance:1.0.0
```

## Quick Start

Here's a basic example of using Convirgance to query a database and filter the results:

Here are the filters we want to apply to our data.

```java
GreaterThanFilter devices = new GreaterThanFilter("devices", 1);
LessThanFilter pets = new LessThanFilter("pets", 3);
```

Next we setup the target for our data to be written to.

```java
// `pdc' stands for pipe delimited values.
File file = new File("./example.pdv");
FileTarget target = new FileTarget(file);
```

Now we setup the output writer. Most writers can be configured quite a bit, take a peek at their docs if you get the chance.

```java
DelimitedOutput output = new DelimitedOutput();
```

Here is the database and the query collecting our results. But, keep in mind you could also use input from other places too.

```java
/*
Other possible inputs:
ex: (DelimimtedInput(), JSONInput()...)
Iterable<JSONObject> rows = JSONInput().read(someFile);
*/
DBMS database = new DBMS(source);
Query example = new Query("select pets, name, devices, account_type from CUSTOMER");
Iterable<JSONObject> rows = database.query(example);
```

And now that we have our data we can filter, transform as needed.

```java
/*
Lets say pet count was a varchar(string) instead of an integer, we can use the following transformer to coerce the data so the filter works correctly.
*/
CoerceStringsTransformer parser = new CoerceStringsTransformer();

Iterable<JSONObject> parsed = parser.transform(rows);
Iterable<JSONObject> filtered = pets.transform(parsed);
Iterable<JSONObject> result = devices.transform(filtered);

output.write(target, result);

/*
Output file would look like:

  name, devices, account_type, pets,
  John, 3, gold, 0
  Bob, 1, bronze, 1
  Jane, 2, gold, 2
*/
```

## Inputs and Outputs

After transforming your data, Convirgance supports various input and output options you can also implement support for other file-types by using our [interfaces](/file-formats?id=example-properties-file).

### Delimited:

When working with delimited files you can select which keys to keep, and if needed set the character to delimit with.

```java
DBMS dbms = new DBMS(source);
Query query = new Query("select * from CUSTOMER");
Iterable<JSONObject> results = dbms.query(query);

// Notice that the query is using *, we can narrow down the output by setting the headers to use.
String wanted = new String[]{"names", "devices", "pets"};
DelimitedOutput output = new DelimitedOutput(wanted);

File file = new File("./example.pdv");
FileTarget target = new FileTarget(file);

output.write(target, results);
```

### JSON:

JSONObjects are created based on the coloum names returned from the database query.

```java
DBMS dbms = new DBMS(source);
Query query = new Query("select name, pets, devices from CUSTOMER");

Iterable<JSONObject> results = dbms.query(query);

File file = new File("./example.json");
FileTarget target = new FileTarget(file);

new JSONOutput().write(target, results);
/*
Contents of example.json:

  {name: "John", pets: "2", devices: 2}
*/
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

File file = new File("./example.csv");
FileTarget target = new FileTarget(file);

// The values to include in the CSV.
String wanted = new String[]{"name","devices","house"};

new CSVOutput(wanted).write(target, results);
/*
Contents of example.csv:

  name, devices, house
  John, 2,
*/
```

### JBIN:

JBIN a Convirgance file-type, is used to convert JSON into a binary encoded format. Useful for high-throughput scenarios.

```java
DBMS dbms = new DBMS(source);
Query query = new Query("select * from CUSTOMER");

Iterable<JSONObject> results = dbms.query(query);

File file = new File("./example.csv");
FileTarget target = new FileTarget(file);

new JBINOutput().write(target, results);
```

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
