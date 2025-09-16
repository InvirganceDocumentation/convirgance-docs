# Getting Started: Convirgance (Web Services)

Convirgance (Web Service) projects are just Jakarta EE projects capable of running
in app servers like Tomcat, Glassfish, JBoss, Websphere and others. This is recommended
for development to reduce build/test times as most components will reload on browser
refresh.

There is also a Convirgance (Boot) component that can compile Convirgance (Web Services)
projects into excutable JAR files for either testing or deployment.

## Maven Archetype

The fastest method of starting a Convirgance (Web Services) project is to run the
Maven archetype. This will create a new project with the following `web.xml` setup:

| Path       | Description                                                                                       |
|------------|---------------------------------------------------------------------------------------------------|
| /api/*     | `.xml` files under this path will be loaded as Wiring files containing a `Service` implementation |
| /view/*    | Similar to `/api/*` mapping. Intended for Hypermedia services producing HTML output.              |
| /error.jsp | Default error page for the application.                                                           | 


Additionally, the following example files are created to demonstrate the project out of
the box:

| Path                                | Description                                                                                       |
|-------------------------------------|---------------------------------------------------------------------------------------------------|
| `src/main/resources/customers.json` | List of example customers used as data for the example web service and JSP page.                  |
| `src/main/webapp/api/sample.xml`    | Example web service that filters and returns the contents of the `customers.json` file.           |
| `src/main/webapp/index.jsp`         | Sample JSP file that consumes and renders data from the `sample.xml` service.                     |

It is recommended that you delete these three sample files once you have a clear understanding
of how to add and update services.


### New Project

Run the following command to create the project:

```sh
mvn -DarchetypeGroupId=com.invirgance \
    -DarchetypeArtifactId=convirgance-web-archetype \
    -DarchetypeVersion=0.1.0 \
    org.apache.maven.plugins:maven-archetype-plugin:3.2.1:generate
```

### Build

Once your project is created, you can run the following command to compile it:

```sh
mvn clean package
```

This will produce a `.war` file and `.jar` file under the `target/` directory.

### Execute

To start the server on port `8080`, run the following command:

```sh
java -jar target/<artifact name>-<version>.jar -p 8080
```

Replace the `artifact name` and `version` with the name and version you setup
when you created the project. If you're uncertain, look in the `target/` directory
for the `.jar` file.

Open a browser and navigate to [http://localhost:8080/](http://localhost:8080/) to 
see the example application! You can directly access the backing service at
[http://localhost:8080/api/sample](http://localhost:8080/api/sample).


## Manual Configuration

Since Convirgance (Web Services) projects are just Jakarta EE applications, the
following instructions will help you setup Convirgance in your project.

### New Project

Using an IDE or your favorite Maven Archetype, create a new Jakarta EE project.
Once created, you will need to add the desired dependencies and then configure
your `web.xml`.

### Add Dependencies

At a minimum, you will need to add the `convirgance-web` dependency. Open your
`pom.xml` and add the following dependency:

```xml
<dependency>
    <groupId>com.invirgance</groupId>
    <artifactId>convirgance-web</artifactId>
    <version>0.3.0</version>
</dependency>
```

You can optionally add the `convirgance-jdbc` dependency if you're using an
`application.properties` and want the [auto download](convirgance-web-boot.md)
of JDBC drivers feature. In that case, you'll want to add the following 
dependency to your `pom.xml`:

```xml
<dependency>
    <groupId>com.invirgance</groupId>
    <artifactId>convirgance-jdbc</artifactId>
    <version>0.3.0</version>
</dependency>
```

### Configure `web.xml`

Open the `src/main/webapp/WEB-INF/web.xml` file. 

To add support for Convirgance services, add the following Servlet
reference to the `web.xml`:

```xml
<servlet>
    <servlet-name>ServicesServlet</servlet-name>
    <servlet-class>com.invirgance.convirgance.web.servlet.JakartaServicesServlet</servlet-class>
</servlet>
```

We then need to define which URL paths can contain Service `.xml` files. If you are
creating a project that provides only backend services, you can map `/` as the path.
Under most circumstances, you'll want url space for other media like HTML pages, images,
and Javascript. In those cases, it is recommended that you map `/api` instead:

```xml
<servlet-mapping>
    <servlet-name>ServicesServlet</servlet-name>
    <url-pattern>/api/*</url-pattern>
</servlet-mapping>
```

If you want to make Hypermedia services that render HTML pages according to a 
REST pathing, then it is recommended you add a `/view` path to separate your
frontend services from your backend services:

```xml
<servlet-mapping>
    <servlet-name>ServicesServlet</servlet-name>
    <url-pattern>/api/*</url-pattern>
    <url-pattern>/view/*</url-pattern>
</servlet-mapping>
```

### Boot Plugin *(Optional)*

If you want to be able to run your project as an executable JAR file, add the 
following plugin to the `<build>` plugins in your `pom.xml`:

```xml
<plugin>
    <groupId>com.invirgance</groupId>
    <artifactId>convirganceboot-maven-plugin</artifactId>
    <version>0.1.0</version>
    <executions>
        <execution>
            <goals>
                <goal>boot</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

After rebuilding, you will then be able to start your project by executing the 
JAR file. To start the server on port `8080`, run the following command:

```sh
java -jar target/<artifact name>-<version>.jar -p 8080
```

Replace the `artifact name` and `version` with the name and version you setup
when you created the project. If you're uncertain, look in the `target/` directory
for the `.jar` file.


<div class="sections-prev-next">

##### [&laquo; Previous: Web Services](convirgance-web.md?id=convirgance-web)

##### [Next: Service Components &raquo;](convirgance-web-services.md)

</div>