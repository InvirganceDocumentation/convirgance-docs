
# Core Concepts

Today's systems collect more logging, handle more transations, report on more 
analytics, and define more complex relationships than ever before. This growth in
data sizes strains the classic model of object mapping to its breaking point. 

Convirgance embraces modern data by building upon unix streams. Bytes are 
translated into records which can be easily transformed and serialized back 
into a unix stream of bytes once again.

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

Build data pipelines that clean and reshape your data. Convert types, group records, or compute new fields - all without loading everything into memory.


## Filters

Apply SQL-like conditions to any data source. Combine filters with AND/OR operations just like you would in a WHERE clause.

## Database Operations

Handle database interactions with confidence. Atomic operations ensure your transactions succeed or fail cleanly, while batch processing keeps things fast. Streamlining any interaction that you make with any database, all without the overhead from your typical ORM.

## Data Formats

Convirgance will handle reading and writing from, JSON, JBIN, CSV, Delimited (Pipe, Comma, ...). And, if support doesn't exist we have you covered with our easily extendible I/O interfaces.


