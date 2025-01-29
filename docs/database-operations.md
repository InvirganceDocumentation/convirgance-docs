# Database Operations

Database operations provide a unified way to work with your database while maintaining data consistency and performance. Rather than dealing with low-level JDBC code, you can work with intuitive concepts like atomic operations (where everything either succeeds or fails together), batch processing for better performance, and simple querying that returns easy-to-use JSON objects. This gives you the control of direct SQL with the safety of managed transactions.

## The DBMS

The Convirgance DBMS streamlines database operations by abstracting complex implementation details into a simple, robust interface. By handling the heavy lifting of connection management and transaction safety, developers can focus on their core application logic by simply providing a data `source` and using the straightforward `query` and `update` operations.

### Parameter Binding.

Convirgance's parameter binding system utilizes named bindings, allowing fearless updates furthermore templates can be created allowing you to save time. The below example uses the named bindings from the query agaisnt the customer(record).

```java
DBMS dbms = new DBMS(source);
String template = "insert into CUSTOMER values (:CUSTOMER_ID, :ZIP, :NAME, :ADDRESS, :CITY, :STATE, :PHONE, :EMAIL)";
Query query = new Query(template);

JSONObject record = createNewCustomer();

QueryOperation operation = new QueryOperation(query, record);
dbms.update(operation);
```

## Querying Data

### Example Usage

```java
Query query = new Query("SELECT * FROM CUSTOMER");
DBMS database;
Iterable<JSONObject> results

database = new DBMS(source);
results; = database.query(query);

for (JSONObject record : results)
{
    System.out.println(record);
}
```

## Inserting Data

```java
// Connect to database
DBMS dbms = new DBMS(source);
String template = "INSERT INTO customer VALUES (:id, :name, :email)";
Query insert;
QueryOperation operation;

JSONObject customer = new JSONObject();
customer.put("id", 1);
customer.put("name", "John");
customer.put("email", "john@email.com");

insert = new Query(template, customer);
operation = new QueryOperation(insert);

dbms.update(operation);
```

### BatchOperation

```java
Query query;
List<JSONObject> records;
BatchOperation batch;
DBMS database;

Query query = new Query("INSERT INTO CUSTOMER (id, name, age) VALUES (?, ?, ?)");

records = List.of(
    new JSONObject().put("id", 1).put("name", "Alice").put("age", 30),
    new JSONObject().put("id", 2).put("name", "Bob").put("age", 25)
);

batch = new BatchOperation(query, records);
batch.setCommit(50);

database = new DBMS(source);
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

```java
DBMS dbms = new DBMS(source);

// Creates the insert statement for JSONObjects
Query query = getInsertQuery();
Query reset = new Query("truncate table SETTINGS");

TransactionOperation transaction;
QueryOperation truncate = new QueryOperation(reset);
BatchOperation batch;

batch = new BatchOperation(query, input.read(source));

transaction = new TransactionOperation(truncate, batch);

dbms.update(transaction);
```

## Best Practices

- Use `TransactionOperation` for critical workflows that involve multiple steps to ensure consistency.
- Leverage `BatchOperation` for large-scale data processing to optimize performance.
  - Using interval commits to avoid overflowing the transaction buffer
- Always validate query parameters to prevent SQL injection and ensure data integrity.
