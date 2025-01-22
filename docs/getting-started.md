# Getting Started with Convirgance

Welcome to the Convirgance documentation! This section will guide you through the basics.

## What is Convirgance?

Convirgance is a modern, streamlined approach to database access. Unlike traditional ORM frameworks that map your database into Java objects, Convirgance gives you direct SQL control while returning results as a stream of `Map` objects. These objects can be manipulated, filtered, and transformed using common operations, making it an excellent drop in tool for querying and managing data.

## Goals

- **Simplified DB interaction**: streamlined way to interact with your database without the overhead of an ORM
- **Flexibility**: pluggable transformations, filters, and outputs
- **Portability**: Output your data to multiple formats such as CSV, JSON or JBIN

## Installation

To get started, you need to install Convirgance. Here's how:

### Using Maven:

```bash
mvn install $PLACEHOLDER_CORDINATES$
```

## Quick Start

Here's a basic example of using Convirgance to query a database:

```java
import com.convirgance.Convirgance;

public class Demo
{
    private static DataSource source;

    public static void main(String[] args)
    {
        DBMS database = new DBMS(source);
        CoerceStringsTransformer parser = new CoerceStringsTransformer();

        DelimitedInput input = new DelimitedInput();
        DelimitedOutput output = new DelimitedOutput();
        ByteArrayTarget target = new ByteArrayTarget();

        Iterable<JSONObject> rows = database.query(new Query("select * from CUSTOMER"));
        GreaterThanFilter devices = new GreaterThanFilter("Devices", 10);
        LessThanFilter zip = new LessThanFilter("ZIP", 20000);

        // Used to convert Zip Codes stored as a string into their actual type (int).
        Iterable<JSONObject> parsed = parser.transform(rows);
        Iterable<JSONObject> filtered = zip.transform(parsed);
        Iterable<JSONObject> result = devices.transform(filtered);

        // Execute the query and write the results
        output.write(target, audience);
    }
}
```

This code will query the database for all active users, filtering the results to customers with more than 10 smart devices, additionally the CoerceTransformer converts data into their actual types. Then writes out the results to the Target.

## Outputs

After transforming your data, Convirgance supports various output options. You can export your results as:

### Delimited Files:

```java
import com.convirgance.Convirgance;

public class Demo {
    private static DataSource source;

    public static void main(String[] args) {
        DBMS dbms = new DBMS(source);
        Iterable<JSONObject> results = dbms.query(new Query("select * from CUSTOMER"));

        // Assuming results contains these three fields.
        DelimitedOutput input = new DelimitedOutput(new String[]{"Names", "Device Count", "Dependents"});

        ByteArrayTarget target = new ByteArrayTarget();

        // Write out the results with only the three fields.
        output.write(target, audience);
    }
}
```

### JSON Files:

```java
import com.convirgance.Convirgance;

public class Demo {
    private static DataSource source;

    public static void main(String[] args) {
        DBMS dbms = new DBMS(source);
        Iterable<JSONObject> results = dbms.query(new Query("select * from CUSTOMER"));
        ByteArrayTarget target = new ByteArrayTarget();

        // Writes out all of the results on the fly.
        new JSONOutput().write(out, array);
    }
}
```

### CSV:

```java
import com.convirgance.Convirgance;

public class Demo {
    private static DataSource source;

    public static void main(String[] args) {
        DBMS dbms = new DBMS(source);
        Iterable<JSONObject> results = dbms.query(new Query("select * from CUSTOMER"));
        ByteArrayTarget target = new ByteArrayTarget();

        // The headers we would like to be included in the CSV.
        String wanted = new String[]{"CUSTOMER_ID"};

        // Writes out all of the results on the fly.
        new CSVOutput(wanted).write(target, results);
    }
}
```

## Community and Support

Have questions? Feel free to reach out through:

- [GitHub Repository](https://github.com/)
- [Twitter](https://X.com/)
