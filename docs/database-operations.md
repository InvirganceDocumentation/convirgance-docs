# Database Operations

Convirgance provides a unified method of working with SQL database management
systems (DBMS) while maintaining both [ACID compliance](https://en.wikipedia.org/wiki/ACID)
and industry-leading performance.

Rather than dealing with low-level JDBC code, you can work with intuitive
concepts like atomic operations (where everything either succeeds or fails together),
batch processing for better performance, and simple querying that returns
easy-to-use JSON objects. This gives you the control of direct SQL with
the safety of managed transactions.

## Database Connections

Database connectivity is dependent on the modern `javax.sql.DataSource` approach
to database connections. Database-specific `DataSource` implementations can be
initialized and configured with server information and credentials. Once this is
done, the `DataSource` becomes a source for obtaining connections.

Most modern systems provide a method of configuring `DataSource` objects for use
by the application. Common approaches include:

- JNDI registration in Application Servers (Glassfish, Tomcat, JBoss, etc.)
- Spring Boot [Data Access](https://docs.spring.io/spring-boot/how-to/data-access.html)
- Connection Pool configuration (e.g. [Apache DBCP](https://commons.apache.org/proper/commons-dbcp/))
- Manual initialization of database-specific `DataSource` implementation

Consult the documentation for your server or framework for more on how
to configure and obtain a `DataSource` instance.

## DBMS API

Central to database access is the `DBMS` object. Simply pass your `DataSource`
to the constructor and you are ready to work with the database.

```java
DataSource source = ...;
DBMS dbms = new DBMS(source);
```

Now we are ready to use the DBMS APIs for querying and inserting/updating data.

## Querying

SQL queries in Convirgance are wrapped by the `Query` object. This object is
responsible for parsing the SQL to detect dynamic bind variables, then allowing
the values to be set.

<!-- TODO The wording here seems odd, maybe 'allowing values to be bound later on'  -->

The prepared `Query` is then passed to the `DBMS` to obtain an `Iterable<JSONObject>`
stream that is compatible with Convirgance's APIs. Here is an example of a
basic, un-parameterized query:

```java
Query query = new Query("select * from CUSTOMER");

for(JSONObject record : dbms.query(query))
{
    // Pretty print the database record
    System.out.println(record.toString(4));
}
```

### Parameter Binding

Convirgance uses named parameters (prefixed with `:`) to
safely bind values into SQL queries. Values can be set directly on `Query` or
bound in bulk using a `JSONObject`.

The use of named variables eliminates the need to count and align traditional
`?` placeholders. For example, `:userId` in your query would match with the
`userId` field in your binding object. Multiple occurrences of `:userId` in the
SQL would all bind to the same value.

```java
Query query = new Query("SELECT * FROM customer " +
                        "WHERE DISCOUNT_CODE = :membershipType " +
                        "AND STATE = :state");

// Set the bind variables
query.setBinding("membershipType", "G");
query.setBinding("state", "CA");

for(JSONObject record : dbms.query(query))
{
    // Pretty print the database record
    System.out.println(record.toString(4));
}
```

## Transactions

The `TransactionOperation` represents operations such as updates, inserts or deleting. Multiple operations can be queued and then executed sequentially. However if an error occurs during one of the operations, all changes will be rolled back.

### Atomic Operations

Atomic operations follow the principle of atomicity - making changes as small, indivisible units. This approach provides precise control and better error handling. This concept is used throughout the Convirgance DBMS, allowing changes to be rolled-back if an issue occurs. An interface is available [here](https://docs.invirgance.com/javadocs/convirgance/latest/com/invirgance/convirgance/dbms/AtomicOperation.html) if needed.

### Query Operation

At a high level, `QueryOperation` serves as a transaction wrapper around Query. Its main role is to ensure database operations are executed atomically (as a single unit), making them safer and more reliable.

Can you guess what class `QueryOperation` implements? (Check the documentation link at the bottom for the answer)

### Inserting Data

#### Example

In this example we insert a single `JSONObject`. You can see the Convirgance named bindings in use through the `template` string.

```java
String template = "INSERT INTO customer VALUES (:id, :name, :email)";
String example = "{ \"id\": 1, \"name\": \"John\", \"email\": \"john@email.com\" }";

JSONObject customer = new JSONObject(example);

DBMS dbms = new DBMS(source);
Query insert = new Query(template, customer);
QueryOperation operation = new QueryOperation(insert);

dbms.update(operation);
```

### BatchOperation

`BatchOperation` is like `QueryOperation` but designed for efficiently executing the same query multiple times with different values. Here we see it in use to bind the values from a file containing multiple JSONObjects.

```java
String template = "INSERT INTO customer (id, name, age) VALUES (:id, :name, :age)";

DBMS database = new DBMS(source);
Query query = new Query(template);

/*
  The contents of 'jsonFile'. Notice that the field names match with VALUES

  {
    id: 1,
    name: "Bob",
    age: 55
  },
  {
    id: 2,
    name: "Potato",
    age: 35
  },
*/
FileSource example = new FileSource(jsonFile);
Iterable<JSONObject> records = JSONInput().read(example);


BatchOperation batch = new BatchOperation(query, records);

database.execute(batch);
```

### Transactions: Inserts and Queries

#### Example

Below is a example with pseudo methods, showcasing `TransactionOperation` executing queries sequentially. If an error occured during one of these operations the database would be rolled back to its previous state before the `TransactionOperation` ran.

```java
// First: Get the query to truncate/drop the table
Query truncate = createTruncateStatement();

// Second: Get resequence statement
Query resequence = createResetSequenceStatement();

// Third: Get the bulk query to insert seed data.
Query data = createDefaultDataInsert();

transaction = new TransactionOperation(truncate, resequence, batch);
```

### Bulk Insert and Query

Here is an example utilizing `TransactionOperation` along with `BatchOperation`. If an error occurs during the bulk insert or while truncating the operation will cancel and the database will rollback to its previous state.

```java
DBMS dbms = new DBMS(source);

// Creates the insert statement for JSONObjects.
Query query = getInsertQuery();
Query reset = new Query("TRUNCATE table customers");

TransactionOperation transaction;
QueryOperation truncate = new QueryOperation(reset);

// Reading in JSONObjects from a file 'source'.
Iterable<JSONObject> items = new JSONInput().read(file);
BatchOperation batch = new BatchOperation(query, items);

transaction = new TransactionOperation(truncate, batch);
dbms.update(transaction);
```

## Best Practices

- Use `TransactionOperation` for multiple operations and to ensure atomicity incase issues arise.
- Leverage `BatchOperation` for large-scale operations to optimize performance.
- Using interval commits to avoid overflowing the transaction buffer
- Utilize named bindings as they ensure the correct JSONObject values will be used.

## Further Reading

<div style="display: flex; align-items: center; gap: 8px; margin-bottom: 16px">
  <span style="display: flex; align-items: center; justify-content: center;font-size:20px; width: 24px; height: 24px">📚</span>
  <a href="https://docs.invirgance.com/javadocs/convirgance/latest/com/invirgance/convirgance/dbms/package-summary.html">Java Documentation: DBMS</a>
</div>
