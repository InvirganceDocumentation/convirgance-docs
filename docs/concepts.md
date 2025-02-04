
# Core Concepts

Today's systems collect more logging, handle more transations, report on more 
analytics, and define more complex relationships than ever before. This growth in
data sizes strains the classic model of object mapping to its breaking point. 

Convirgance embraces modern data by building upon unix streams to implement a
Data Flow pattern. Bytes are translated into records which can be easily 
transformed and then serialized back into a unix stream of bytes once processed.

## Records

The core unit of data in Convirgance is a record. Records are represented by
`JSONObject` which is an implementation of the Java Collections `Map` interface.
The implementation is designed to easily parse and print to JSON making debugging 
incredibly easy and complex test cases a breeze. 

Data is stored in `JSONObject` in key/value pairs. This aligns with both the 
JSON specification and the requirements of the `Map` interface.

Implementing the `Map` interface additionally makes each record compatible with
any Java APIs or features that support `Map`. For example, JSP pages can utilize
native JSTL syntax on a `JSONObject` when rendering.

## Streams

Java enhances the idea of unix streams (represented by `InputStream` and 
`OutputStream`) with the concept of `Iterator`. An iterator is a stream of data types
a level above a byte stream. Each entry is a Java object of some sort. 

Convirgance hooks into this feature of the langauge by implementing its streams
of records as `Iterator<JSONObject>`. 

The only issue with Iterators in Java is that they don't have full language
support. The `Iterator` object is already in process of streaming the data by
the time it exists. Thus Java introduced the concept of `Iterable` to provide
a mechanism for delivering streams on demand. Convirgance supports this approach
by using `Iterable<JSONObject>` as its primary interface for setting up streams.

Thus it becomes easy to use features like the enhanced `for` loop:

```java
FileSource file = new FileSource("data.json");
Iterable<JSONObject> stream = new JSONInput().read(file);

for(JSONObject record : stream)
{
    // Pretty prints each record as JSON
    System.out.println(record.toString(4));
}
```

A critical point to understand about iterating streams is that they do not
load the data into memory. The data is transformed into records as it is read
from the underlying byte stream. Which means that as long as the record itself can
fit into memory, Convirgance can handle an unlimited number of records.

## Transformations

The basic pattern of transformations in Convirgance is to pass an `Iterable` into 
a transformer and get a new `Iterable` back out. For example:

```java
FileSource file = new FileSource("data.csv");
Iterable<JSONObject> stream = new CSVInput().read(file);

// Transform the stream by parsing string values into numbers and booleans
iterable = new CoerceStringsTransformer().transform(iterable);
```

Each tranformation places a processing step on records as they pass through the
stream. Critically, there is only one record in the stream at a time. That record
steps through each transformation and then is released once all operations have
been completed on it.

This approach is important because it minimizes memory usage, aligns data with
the young GC collector, and maximizes the use of the CPUs L1 and L2 caches. For
data requiring a large number of operations, performance can improve by orders
of magnitude.  

It should be noted that the `Iterable` in / `Iterable` out approach means that
transformers can manipuate the data in any manner they choose. They can perform
simply updates to a record as it goes by, thus leaving the record count intact.
Or they group data, aggregate data, or even filter data out of the stream. 

## Filters

Filters are an extension to transformers that specifically reject records
based upon a "predicate". A predicate is a condition that must be met for the
record to be kept. 

Convirgance supports many of the types of [filters](filtering-data.md) you would 
expect in a SQL engine. Including equals, greater than, less than, etc. Filters
based on boolean logic can be combined to create and/or/not logic.

`Filter` also implements the `java.util.Predicate` class to be compatible with
Java functional programming techniques. Implementing a filter is accomplished
in a very similar manner and can be done with either a class, anonymous inner
class, or lambda arrow funtion. For example:

```java
// Find Bob
Filter bob = new Filter() {
    public boolean test(JSONObject record)
    {
        String name = record.getString("name");

        return (name != null && name.contains("Bob"));
    }
};

// Find Rob
Filter rob = (JSONObject record) -> {
    return !record.isNull("name") && record.getString("name").contains("Rob");
};

// Find Bob or Rob
Filter or = new OrFilter(bob, rob);

// Find everyone except Bob and Rob
Filter not = new NotFilter(or);
```


## Database Operations

Handle database interactions with confidence. Atomic operations ensure your transactions succeed or fail cleanly, while batch processing keeps things fast. Streamlining any interaction that you make with any database, all without the overhead from your typical ORM.

## Data Formats

Convirgance will handle reading and writing from, JSON, JBIN, CSV, Delimited (Pipe, Comma, ...). And, if support doesn't exist we have you covered with our easily extendible I/O interfaces.


