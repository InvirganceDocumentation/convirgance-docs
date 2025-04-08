# Convirgance (JDBC)

Convirgance (JDBC) is a AIO database library, bridging the gaps across the variety of SQL servers libraries in the Java ecosystem. Further, through the use of lazy loading by using this library you won't bloat your application size with external SQL libraries. Its hassle-free query builders let you focus on business logic rather than SQL syntax, while providing universal support for any SQL driver library and offering stored connection management.

## Installation

> ![WARNING](images/warning.svg) **<font color="#AA9900">WARNING:</font>**
> Convirgance (JDBC) is in pre-release and may be subject to change

## Installation

Add the following dependency to your Maven `pom.xml` file:

```xml
<dependency>
    <groupId>com.invirgance</groupId>
    <artifactId>convirgance-jdbc</artifactId>
    <version>0.1.0</version>
</dependency>
```

## AutomaticDrivers

Like previously mentioned drivers can be lazy loaded.

```java
AutomaticDriver postgres = AutomaticDrivers.getDriverByName("PostgreSQL");
```

And thankfully its just as simple to add your own lazy-loaded drivers. All you need to do is provide the artifact id, connection prefix and a datasource.

```java
String name = "PostgreSQLAlternative";
AutomaticDrivers.AutomaticDriverBuilder builder;
AutomaticDrivers drivers = new AutomaticDrivers();
AutomaticDriver descriptor = drivers.getDriverByName(name)

builder = drivers.createDriver(name)
        .artifact(artifacts.toArray(new String[artifacts.size()]))
        .prefix(prefixes.toArray(new String[prefixes.size()]))
        .datasource("com.invirgance.virge.jdbc.DriverDataSource");

descriptor = builder.build();
descriptor.save()
```

## Stored Connections

Stored connections can be used to create reusable pre-configured database connections.

```java

AutomaticDriver postgres = AutomaticDrivers.getDriverByName("PostgreSQL");
StoredConnection inventory = postgres.createConnection("InventoryDB")
    .driver()
        .url("jdbc:postgresql://localhost:5432/inventory")
        .username("postgres")
    .build();
inventory.save();

AutomaticDriver mysql = AutomaticDrivers.getDriverByName("MySQL");
StoredConnection customers = mysql.createConnection("CustomersDB")
                            .datasource()
                                .property("serverTimezone", "UTC")
                                .property("useSSL", true)
                                .property("connectTimeout", 5000)
                            .done()
                            .driver()
                                .url("jdbc:mysql://localhost:3306/customers")
                                .username("root")
                                .password("password")
                            .build();
customers.save();
```

### Example SQLStatement

The SQLStatement implementations simplify the process of creating queries based off of existing Table objects. You can also create aliases, filter data. Pretty much anything you can think of.

```java
DatabaseSchemaLayout layout = getLayout();
Table table = layout.getCurrentSchema().getTable("CUSTOMER");

SQLStatement statement = table
                        .select()
                        .column(table.getColumn("name"))
                        .column(table.getColumn("email"), "contact_email")
                        .from(table, "c")
                        .where()
                          .equals(table.getColumn("status"), "active")
                          .and()
                            .greaterThan(table.getColumn("last_order_id"), 8)
                          .end()
                        .done()
                        .order(table.getColumn("name"));

```

## Further Reading

<!-- TODO add public doc link -->
<div style="display: flex; align-items: center; gap: 8px; margin-bottom: 16px">
  <span style="display: flex; align-items: center; justify-content: center;font-size:20px; width: 24px; height: 24px">📚</span>
  <a href="https://docs.invirgance.com/javadocs/convirgance/latest/com/invirgance/convirgance/dbms/package-summary.html">TODO JavaDocs: Convirgance (JDBC)</a>
</div>

## Sections

##### [Previous: OLAP](./olap?id=online-analytical-processing-olap)

##### [Back to start?](./?id=convirgance)
