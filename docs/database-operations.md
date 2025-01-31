# Database Operations

Database operations provide a unified way to work with your database while maintaining data consistency and performance. Rather than dealing with low-level JDBC code, you can work with intuitive concepts like atomic operations (where everything either succeeds or fails together), batch processing for better performance, and simple querying that returns easy-to-use JSON objects. This gives you the control of direct SQL with the safety of managed transactions.

## The DBMS

The Convirgance DBMS streamlines database operations by abstracting complex implementation details into a simple, robust interface. By handling the heavy lifting of connection management and transaction safety, developers can focus on their core application logic by simply providing a `DataSource` and using the straightforward `query` and `update` operations.

## Parameter Binding

Parameter binding in Convirgance uses named parameters (prefixed with `:`) to securely integrate values into SQL queries. Values are bound using a `JSONObject`, eliminating the need to count and align traditional ? placeholders. For example, `:userId` in your query would match with the `userId` field in your binding object.

```java
DBMS dbms = new DBMS(source);
String template = "SELECT * FROM customer " +
                 "WHERE DISCOUNT_CODE = :membershipType " +
                 "AND STATE = :state";

JSONObject bindings = new JSONObject();
bindings.put("membershipType", "G");
bindings.put("state", "CA");

Query query = new Query(template, bindings);
Iterable<JSONObject> results = dbms.query(query);
```

You can also setup a config like this to bind a large amount of records.

```java
String template = "INSERT INTO customer (id, name, age) VALUES (:id, :name, :age)";

DBMS database = new DBMS(source);
Query query = new Query(template);

Iterable<JSONObject> records = JSONInput().read(someExampleFile);

BatchOperation batch = new BatchOperation(query, records);
```

## Querying Data

Querying data is pretty straight forward, you create your SQL query and execute it with the `DBMS` like below. And a Iterable of `JSONObject`s will be returned containing the results.

### Example

Here we simply retreive all the customers and print out their information.

```java
Query query = new Query("SELECT * FROM customer");
DBMS database = new DBMS(source);
Iterable<JSONObject> results = database.query(query);

for (JSONObject record : results)
{
    System.out.println(record);
}
```

## Transaction Operations

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

Here is an example utilizing `TransactionOperation` along with `BatchOperation`.

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

- [Java Documentation](https://docs.invirgance.com/javadocs/convirgance/latest/com/invirgance/convirgance/dbms/package-summary.html)
