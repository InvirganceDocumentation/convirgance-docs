
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
FileSource json = new FileSource("data.json");
Iterable<JSONObject> stream = new JSONInput().read(json);

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

The basic pattern of transformations in Convirgance are to pass an `Iterable` in
and get a new `Iterable` out. e.g.

```java
FileSource json = new FileSource("data.json");
Iterable<JSONObject> stream = new JSONInput().read(json);

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

Apply SQL-like conditions to any data source. Combine filters with AND/OR operations just like you would in a WHERE clause.

## Database Operations

Handle database interactions with confidence. Atomic operations ensure your transactions succeed or fail cleanly, while batch processing keeps things fast. Streamlining any interaction that you make with any database, all without the overhead from your typical ORM.

## Data Formats

Convirgance will handle reading and writing from, JSON, JBIN, CSV, Delimited (Pipe, Comma, ...). And, if support doesn't exist we have you covered with our easily extendible I/O interfaces.


