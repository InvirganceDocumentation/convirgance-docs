# Database Operations

Database operations provide a unified way to work with your database while maintaining data consistency and performance. Rather than dealing with low-level JDBC code, you can work with intuitive concepts like atomic operations (where everything either succeeds or fails together), batch processing for better performance, and simple querying that returns easy-to-use JSON objects. This gives you the control of direct SQL with the safety of managed transactions.

## The DBMS

The Convirgance DBMS streamlines database operations by abstracting complex implementation details into a simple, robust interface. By handling the heavy lifting of connection management and transaction safety, developers can focus on their core application logic by simply providing a data `source` and using the straightforward `query` and `update` operations.

### Parameter Binding.

Parameter binding in Convirgance is a secure method for incorporating values into SQL queries using named parameters. Each parameter is prefixed with a colon (:) in the SQL template and matched with corresponding values in a JSONObject. Using named parameters keeps query logic flexible by avoiding the traditional `?` approach.

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

## Querying Data

### Example

Here we simply retreive all the customers and print out their information.

```java
Query query = new Query("SELECT * FROM CUSTOMER");
DBMS database = new DBMS(source);
Iterable<JSONObject> results = database.query(query);

for (JSONObject record : results)
{
    System.out.println(record);
}
```

## Inserting Data

### Example

In this example we insert a single `JSONObject`. You can see the Convirgance named bindings in use through the `template` string.

```java
String template = "INSERT INTO customer VALUES (:id, :name, :email)";
String example = String example = "{ \"id\": 1, \"name\": \"John\", \"email\": \"john@email.com\" }";

JSONObject customer = new JSONObject(example);

DBMS dbms = new DBMS(source);
Query insert = new Query(template, customer);
QueryOperation operation = new QueryOperation(insert);

dbms.update(operation);
```

### BatchOperation

The `BatchOperation` provides a simple way to insert a large amount of objects.

```java
String template = "INSERT INTO customer (id, name, age) VALUES (:id, :name, :age)";

List<JSONObject> records = List.of(
    new JSONObject().put("id", 1).put("name", "Alice").put("age", 30),
    new JSONObject().put("id", 2).put("name", "Bob").put("age", 25)
);

DBMS database = new DBMS(source);
Query query = new Query(template);

BatchOperation batch = new BatchOperation(query, records);
batch.setCommit(50);

database.execute(batch);
```

## Transactions: Inserts and Queries

#### A brief overview

Below is a example with pseudo methods, showcasing that `TransactionOperation` executes queries sequentially. Any number of Queries can be provided.

```java
Query truncate = createTruncateStatement();
Query resequence = createResetSequenceStatement();
Query data = createDefaultDataInsert();

transaction = new TransactionOperation(truncate, resequence, batch);
```

### Bulk Insert and Query

Here is an example utilizing `TransactionOperation`. Which executes each operation sequentially, if at any point an issue occurs all changes are rolled back.

```java
DBMS dbms = new DBMS(source);

// Creates the insert statement for JSONObjects.
Query query = getInsertQuery();
Query reset = new Query("TRUNCATE table SETTINGS");

TransactionOperation transaction;
QueryOperation truncate = new QueryOperation(reset);

// Reading in JSONObjects from a file 'source'.
InputCursor<JSONObject> items = input.read(file);
BatchOperation batch = new BatchOperation(query, items);

transaction = new TransactionOperation(truncate, batch);
dbms.update(transaction);
```

## Best Practices

- Use `TransactionOperation` for critical workflows that involve multiple steps to ensure consistency.
- Leverage `BatchOperation` for large-scale data processing to optimize performance.
  - Using interval commits to avoid overflowing the transaction buffer
- Always validate query parameters to prevent SQL injection and ensure data integrity.

## Further Reading

- [Java Documentation](https://docs.invirgance.com/javadocs/convirgance/latest/com/invirgance/convirgance/dbms/package-summary.html)
