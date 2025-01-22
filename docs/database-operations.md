# Database Operations

Database operations provide a unified way to work with your database while maintaining data consistency and performance. Rather than dealing with low-level JDBC code, you can work with intuitive concepts like atomic operations (where everything either succeeds or fails together), batch processing for better performance, and simple querying that returns easy-to-use JSON objects. This gives you the control of direct SQL with the safety of managed transactions.

## Core Interfaces and Classes

- **AtomicOperation**: Represents a single, encapsulated database operation. This interface ensures that the operation can be executed safely within a managed transaction.
- **Query**: Encapsulates a SQL query for execution against a database. Provides a way to parameterize and reuse queries.
- **QueryOperation**: Executes a single query as an atomic operation. Parameters are bound to ensure safe execution.
- **TransactionOperation**: Groups multiple atomic operations into a single transaction. Guarantees that either all operations succeed, or none are applied.
- **BatchOperation**: Facilitates bulk operations by batching multiple records into a single query for improved performance.

## AtomicOperation Interface

The `AtomicOperation` interface defines a standard way to encapsulate database operations, ensuring proper lifecycle management and transaction safety.

### Key Methods

- `void execute(Connection connection)`: Executes the operation using the provided database connection.

## Example: Custom AtomicOperation

A custom implementation of the `AtomicOperation` interface that logs each record in a table to an external system.

```java
/**
 * A custom implementation of the AtomicOperation interface that logs each record 
 * in a table to an external system.
 */
public class LoggingOperation implements AtomicOperation {
    private Query query; // The query to fetch the records.
    private Logger logger; // The logging system to use.
    
    /**
     * Constructor to initialize the query and logger.
     *
     * @param query The query to execute.
     * @param logger The logger for recording entries.
     */
    public LoggingOperation(Query query, Logger logger) {
        this.query = query;
        this.logger = logger;
    }
    
    /**
     * Executes the logging operation within a transaction.
     *
     * @param connection The database connection to use.
     * @throws SQLException If a database error occurs.
     */
    @Override
    public void execute(Connection connection) throws SQLException {
        JSONObject record;
        ResultSetMetaData metaData;
        
        try (PreparedStatement statement = connection.prepareStatement(query.toString());
             ResultSet resultSet = statement.executeQuery()) {
            
            while (resultSet.next()) {
                record = new JSONObject();
                metaData = resultSet.getMetaData();
                
                for (int i = 1; i <= metaData.getColumnCount(); i++) {
                    record.put(metaData.getColumnName(i), resultSet.getObject(i));
                }
                
                logger.log(record.toString());
            }
        }
    }
}
```

## TransactionOperation

The `TransactionOperation` class allows multiple `AtomicOperation` instances to be executed as a single transaction, ensuring atomicity.

### Key Features

- Groups multiple operations for execution within a transaction.
- Handles rollback in case of failure, maintaining database consistency.

### Usage Example

```java
public void transactionOperation() {
    DBMS database = new DBMS(source);
    
    QueryOperation insertCustomer = new QueryOperation(
        new Query("INSERT INTO CUSTOMER (id, name) VALUES (1, 'Alice')")
    );
    QueryOperation updateAccount = new QueryOperation(
        new Query("UPDATE ACCOUNT SET balance = balance - 100 WHERE id = 1")
    );
    
    TransactionOperation transaction = new TransactionOperation(
        List.of(insertCustomer, updateAccount)
    );
    database.execute(transaction);
}
```

## BatchOperation

The `BatchOperation` class is designed for executing bulk queries efficiently. It reduces overhead by batching multiple records into a single operation.

### Key Features

- Executes multiple inserts, updates, or deletes in a single query.
- Supports setting a commit size for better control over batch transactions.

### Usage Example

```java
public void batchOperation() {
    Query query = new Query("INSERT INTO CUSTOMER (id, name, age) VALUES (?, ?, ?)");
    List<JSONObject> records = List.of(
        new JSONObject().put("id", 1).put("name", "Alice").put("age", 30),
        new JSONObject().put("id", 2).put("name", "Bob").put("age", 25)
    );
    
    BatchOperation batch = new BatchOperation(query, records);
    batch.setCommit(50); // Commit after every 50 records
    
    DBMS dbms = new DBMS(source);
    dbms.execute(batch);
}
```

## Query Class

The `Query` class encapsulates a SQL query for execution. It allows parameterized queries and ensures safe binding of values.

### Example Usage

```java
public void queryOperation() {
    Query query = new Query("SELECT * FROM CUSTOMER WHERE id = ?");
    query.setParameter(1, 12345);
    
    DBMS dbms = new DBMS(source);
    Iterable<JSONObject> results = dbms.query(query);
    
    for (JSONObject record : results) {
        System.out.println(record);
    }
}
```

## Best Practices

- Use `TransactionOperation` for critical workflows that involve multiple steps to ensure consistency.
- Leverage `BatchOperation` for large-scale data processing to optimize performance.
- Always validate query parameters to prevent SQL injection and ensure data integrity.