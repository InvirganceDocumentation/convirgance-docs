<style>body {text-align: left}</style>
# OLAP

Convirgance-OLAP, built on top of Convirgance package, further provides an easy and intuitive
way to build fast OLAP tools for interactive multidimnesional data analysis .

With some additional web configuration, this package gives you an opportunity 
to quickly build in-house analytic systems to support your 
business intelligence without the need for expensive outsourcing.

Read on to better understand how Convirgance-OLAP offers such functionality. 


## Representing Database Structure

To provide the base for building OLAP tools, Convirgance-OLAP uses the 
following classes to first captures the core structure of the database 
we want to model:

| Class                      | Function                                                                                    |
| -------------------------- | --------------------------------------------------------------------------------------------|
| ```Database```             | Provides support for Database structure representation; contains table objects.             |
| ```ForeignKey```           | Captures the connection between a source table, a column on the source table, and a target table. |
| ```SQLGenerator```         | Provides support for creating and outputting SQL queries for working with OLAP.             |
| ```Table```                | Provides support for Table representation. Contains a primary key and list of foreign keys. |


Note how the ```Table``` class only contains the primary and foreign keys, but no other 
columns. This approach prevents us from the costly ORM-like representation and only
captures the essential structure at this stage. 
The necessary columns are specified later within the SQLGenerator.


With the relationship defined by the Table, ForeignKey, and Database classes,
the SQLGenerator can now output OLAP queries for us. Here is how it works:

1. The ```addSelect(String column, Table table)``` lets us define a select from a table
2. We capture this select in a ```Column``` inner class that represents the column and table
3. We add the table to a list of tables we need to create joins between
4. The addTable(Table table) method is public because OLAP queries are centered around a Fact table. The Fact table must be added first as the “From” table so that all joins fan out from it. Even if we never select any data from the Fact table itself.
5. The getSQL() method loops through the select list to generate the columns to select and generates the first table in the list as the “from” table. It then calls generateJoins(from) and generateGroupBy().
6. generateJoins(from) matches the ForeignKeys in the “from” table to tables in our list of selects and generates a join for each one
7. generateGroupBy() loops through the selected columns again and generates a group by list of all columns that are not aggregates. i.e. They are dimension columns. 
    
    * Wait… what are aggregates?

8. Aggregates are the “fact” or “metric” columns that we want to roll up using a function like “sum()” or “avg()”
9. We can call addAggregate(String function, String column, Table table) to add the aggregate to our select list
10. We use an inner class called Aggregate to capture this select. Aggregate uses the inheritance pattern in OOP to “be” a Column while additionally capturing the function and overriding the getSQL() method.
    
    * Because our Aggregate “is” a Column, we don’t need to change any of our select logic or manage it separately.








##### [Previous: File Formats](./file-formats)
##### [Support and Contacts](./contact) 
##### [Back to start?](./?id=convirgance)


