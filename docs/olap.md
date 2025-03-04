<style>body {text-align: left}</style>

# Online Analytical Processing (OLAP)

As businesses grow and expand, they gain useful analytical data about 
their own customers and the market itself. Online Analytical Processing, or OLAP, 
provides a means of sythesizing this data into useful metrics to provide business intelligence insights.

Convirgance (OLAP) provides an easy and intuitive
way to build OLAP tools around Star Schemas to produce SQL queries and perform 
data analysis to aid businesses intelligence, without the need for expensive
outsourcing.

A star schema is a type of multidimensional database design commonly used in data warehousing, 
where a central fact table (containing quantitative data) is connected to multiple
dimension tables (describing the context of the data). This structure simplifies analytical queries and accelerates 
information retrieval from databases.

## Scenario 
Suppose you are working on an online retail analytics and you have the following
star schema model with ORDERS as the central fact table, and PRODUCTS, 
SALESPERSONS, CUSTOMERS, and DAYS as dimensions:

 ![alt text](images/starschema.svg)


The eventual goal of this exercise is to create SQL queries from OLAP structure.
Read along to better understand how Convirgance (OLAP) offers such functionality.

## Installation

Add the following dependency to your Maven `pom.xml` file:

```xml
<dependency>
    <groupId>com.invirgance</groupId>
    <artifactId>convirgance-olap</artifactId>
    <version>0.1.0</version>
</dependency>
```


## Representing Database Structure


<!--
| Class          | Function                                                                                          |
| -------------- | ------------------------------------------------------------------------------------------------- |
| `Database`     | Database structure representation; contains table objects.                   |
| `Table`        | For Table object representation. Contains a primary key and list of foreign keys, which are columns shared between a source and target table.      |
| `ForeignKey`   | Captures the connection between a source table, a column on the source table, and a target table. |
| `SQLGenerator` | Support for creating and outputting SQL queries for working with OLAP.                   |

-->

You can use Convirgance (OLAP) to first capture the database structure with the five tables:

```java
Database stardb = new Database("StarDB");

Table orders = new Table("ORDERS", "order_id"); // "order_id" is the primary key for ORDERS table
Table products = new Table("PRODUCTS", "product_id");
Table salespersons = new Table("SALESPERSONS", "salesperson_id");
Table customers = new Table("CUSTOMERS", "customer_id");
Table days = new Table("DAYS", "day_id");

stardb.addTable(orders);
stardb.addTable(products);
stardb.addTable(salespersons);
stardb.addTable(customers);
stardb.addTable(days);
```

You can then capture the relationship between the ORDERS table and the rest of 
the tables by adding the foreign keys. The ORDERS table will
become the table following the `FROM` command in SQL queries.

``` java
orders.addForeignKey("product_id", products); 
orders.addForeignKey("salesperson_id", salespersons);
orders.addForeignKey("customer_id", customers);
orders.addForeignKey("day_id", days);
```

## Generating SQL Queries


Now that we have our database representation, we can use the SQLGenerator to 
output OLAP queries. Here is how it works:

<!-- 
table just represents that a table exists. 
an object in star schema. 
on top of that we put dimensions/metrics

todo: 
1. just say we create a structure for the star schema
2. once we have the structure, we can specify the dimensions and metrics
3. create a super simple star schema
    star diagram (e.g. 4 dimensions, 1 central table)

Explain with the example the schema, dimensions, facts (metrics)
Fact table at the centre is the table containing all the facts/metrics.
Only have a few records (4-10).
-->


This java code...

```java
generator.addTable(orders);
generator.addSelect("product_name", products);
generator.addSelect("salesperson_name", salespersons);
generator.addAggregate("sum", "cost_total", orders, "cost_dollars");

generator.getSQL(); // Get the SQL!
```
...generates the following SQL query:

```SQL
select
    PRODUCTS.product_name,
    SALESPERSONS.salesperson_name,
    sum(ORDERS.cost_dollars) as cost_total,
from ORDERS
join PRODUCTS on PRODUCTS.product_id = ORDERS.product_id
join SALESPERSONS on SALESPERSONS.salesperson_id = ORDERS.salesperson_id
group by
    PRODUCTS.product_name,
    SALESPERSONS.salesperson_name
```

1. Adding ORDERS table to the generator first sets it as the central fact table
and handles the joins between ORDERS and other tables.

2. The `addSelect(String Column, Table table)` method lets us specify which column to
select from table. It relies on previously specified primary and foreign keys to
handle the joins. 

3. The `addAggregate(String function, String column, Table table, String alias)` method
allows you to specify an aggregate metric with a SQL function, which is generated over a quantitative
measure in the central table. You can optionally specify the alias for the metric by passing
the fourth parameter to the method.



<!--
1.  The `addSelect(String column, Table table)` lets us define a select from a table
2.  We capture this select in a `Column` inner class that represents the column and table
3.  We add the table to a list of tables we need to create joins between
4.  The addTable(Table table) method is public because OLAP queries are centered around a Fact table. The Fact table must be added first as the “From” table so that all joins fan out from it. Even if we never select any data from the Fact table itself.
5.  The getSQL() method loops through the select list to generate the columns to select and generates the first table in the list as the “from” table. It then calls generateJoins(from) and generateGroupBy().
<!-- TODO nit: 4 and 5 are a little on the long side 
6.  generateJoins(from) matches the ForeignKeys in the “from” table to tables in our list of selects and generates a join for each one
7.  generateGroupBy() loops through the selected columns again and generates a group by list of all columns that are not aggregates. i.e. They are dimension columns.

    - Wait… what are aggregates?

8.  Aggregates are the “fact” or “metric” columns that we want to roll up using a function like “sum()” or “avg()”
<!-- TODO nit 4, 5, 6, 8: use backticks when referring to the From table also with functions/objects table
9.  We can call `addAggregate(String function, String column, Table table)` to add the aggregate to our select list
10. We use an inner class called Aggregate to capture this select. Aggregate uses the inheritance pattern in OOP to “be” a Column while additionally capturing the function and overriding the getSQL() method.

    - Because our Aggregate “is” a Column, we don’t need to change any of our select logic or manage it separately.

    <!-- TODO nit: use single quotes when referring to something abstract ex nuclear fission reactors 'existed' 2 billion years ago but only because of the earths enviroment and very specific conditions. -->


- [detailed documentation for classes used above](https://docs.invirgance.com/javadocs/convirgance-olap/latest/com/invirgance/convirgance/olap/sql/package-summary.html)



## Spring Method

The above example still requires a lot of code to produce a single query.
We should have a config file. Thanks to the inherent properties of Java Objects, we can just pull a Spring configuration file off the shelf!
We simply load the Spring context, request the database, and pull whichever tables we need by calling `database.getTable(String name)`.

<!-- TODO maybe add an example for this 

use the example above, create the spring config file based off that.

-->

## OLAP Interface

To bridge the database representation with an OLAP interface, we need additional layer of tooling
to represent Dimensions, Metrics (aka Facts), and Measures (just aggregated Metrics). These act as building blocks to comprise
the Star Schema, from which we then generate reports.

| Class             | Function                                                                     |
| ----------------- | ---------------------------------------------------------------------------- |
| `Dimension`       | Provides support for qualitative/contextual descriptions of data.            |
| `Measure`         | Provides support for aggregated quantitative values of data.                 |
| `Metric`          | Provides support for quantitative values of data.                            |
| `ReportGenerator` | Provides support for the SQL query generation from constructed star schemas. |
| `Star`            | Provides support for the central star schema.                                |

- [detailed documentation for the classes above](https://docs.invirgance.com/javadocs/convirgance-olap/latest/com/invirgance/convirgance/olap/package-summary.html)

1. The `Star` plays the role of a central container, and it tracks the fact table at the center.
   Around the fact table are Dimensions. Inside the fact table are Metrics.
   Metrics will be aggregated in queries, so these are represented by Measures.

2. `Dimension` and `Metric` are simple, they simply contain a column
   name and a Table reference.

3. `Measure` also contains a column name, but no table. Instead, it wraps the
   associated with it Metric along with a function. The function is just the name of the
   SQL function we want to apple, such as "avg" or "sum".

4. Finally, the `ReportGenerator` allows to create queries within OLAP context.
   As an OLAP user, you no longer need to think about the underlying query structure. Instead,
   you simply specify Dimensions and Measures, and the OLAP layer takes care of the rest by pushing
   the the parameters down to the SQLGenerator.

  <!-- TODO nit: order list as items appear in the table or reoder the table -->

Here too, you can add the Star schema to the Spring configuration file just like we did with the Database above.
You can then lookup the Star from the configuration file and query immediately:

Here's how we can generate the same SQL Query as above by using the Star configured in Spring:

<!-- fill in the story with code actually constructing a star.
There is a test case that constructs a star that you can use. -->

```java
Star star = getStar(); // Load the Star from the Spring file
ReportGenerator generator = new ReportGenerator(star);

generator.addDimension(star.getDimension("Franchise Name"));
generator.addDimension(star.getDimension("Store Name"));
generator.addMeasure(star.getMeasure("Units Sold"));

generator.getSQL(); // Generate the SQL!
```

<!--As a next step, you can use [Convirgance-WEB](https://github.com/InvirganceOpenSource/convirgance-web) to create web services around this OLAP
structure. -->

##### [Previous: File Formats](./file-formats)

##### [Support and Contacts](./contact)

##### [Back to start?](./?id=convirgance)

<!-- TODO links to docs just copy-paste the format from another readme -->
