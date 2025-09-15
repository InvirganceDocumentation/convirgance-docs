# Configuration: Convirgance (Web Services)

Convirgance defaults to obtaining database connections and other configuration
from the environment in which the application is deployed. For example, this 
means that code looks for a JNDI reference to access the desired database.

In some circumstances you may want the configuration to be included within
the assembled `.war` file. Particualrly when you are using Convirgance (Boot)
to run the applicaiton from an executable `.jar` file. 

In those cases, you can include an `application.properties` file to configure
database connectivity.

Example:

```properties
jdbc.database.username=sa
jdbc.database.password=
jdbc.database.url=jdbc:h2:mem:clinic;DB_CLOSE_DELAY=-1;DB_CLOSE_ON_EXIT=false
jdbc.database.jndi=jdbc/petclinic
jdbc.init.sql.schema=/sql/init/schema.sql
jdbc.init.sql.data=/sql/init/data.sql
```


## Dependencies

If you want to configure your web application's database with an `application.properties` 
file, you will need to add an initializer to your `src/main/webapp/WEB-INF/web.xml` file:

```xml
<listener>
    <description>ServletContextListener</description>
    <listener-class>com.invirgance.convirgance.web.servlet.ApplicationInitializer</listener-class>
</listener>
```

This will trigger the parsing of the `application.properties` on startup. 

You will also need to add [Convirgance (JDBC)](convirgance-jdbc.md) to your dependencies to handle the download
of database drivers. Open your `pom.xml` file and add the following dependency:

```xml
<dependency>
    <groupId>com.invirgance</groupId>
    <artifactId>convirgance-jdbc</artifactId>
    <version>0.3.0</version>
</dependency>
```

## Properties

Create the file as `src/main/resources/application.properties`. You can then
add the following properties to configure database access:

| Property                 | Required |Description                                       |
|--------------------------|----------|--------------------------------------------------|
| jdbc.database.username   | Yes      | The username to use when connecting to the db    |
| jdbc.database.password   | Yes      | The password to use when connecting to the db    |
| jdbc.database.url        | Yes      | The JDBC url to the db. a Driver will be automatically identified based on this URL. |
| jdbc.database.jndi       | Yes      | The JNDI path you wish to register. This path is what you will use in your services to access the database connection. |
| jdbc.init.sql.schema     | No       | A script to run at startup for initializing the database schema |
| jdbc.init.sql.data       | No       | A script to run at startup for loading data into the database schema |


## Sections

##### [Previous: Hypermedia](convirgance-web-hypermedia.md)

##### [Next: Community and Support](contact.md)