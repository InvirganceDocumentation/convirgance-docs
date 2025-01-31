# File Formats

Convirgance treats all data sources equally, whether they're CSV files, JSON documents, or database tables. You can read from one format and write to another without changing your business logic. Need to take a CSV file and output JSON? Or query a database and save to CSV? The same filtering and transformation rules work regardless of your input or output format, making it easy to work with data in whatever form it arrives or needs to be delivered.

## Supported Formats

| Format         | Description                                    | Read/Write | Extensions     |
| -------------- | ---------------------------------------------- | ---------- | -------------- |
| CSV            | Comma-separated values for tabular data        | Read/Write | `.csv`         |
| Pipe Delimited | Column data separated by pipes                 | Read/Write | `.psv`, `.txt` |
| Tab Delimited  | Column data separated by tabs                  | Read/Write | `.tsv`, `.txt` |
| Delimited      | Custom delimiter-separated data                | Read/Write | `.txt`         |
| JSON           | JavaScript Object Notation for structured data | Read/Write | `.json`        |
| JBIN           | Binary JSON format                             | Read/Write | `.jbin`        |

Each format supports full read and write operations with configurable parsing options.

## The Input/Output Interfaces

Convirgance provides an extensible input interface to allow you to integrate support for new data formats by defining custom readers and writers.

### Best Practices

- Ensure you're following the specifications of the file, created by [IETF](https://www.ietf.org/).
- Ensure robust error handling to deal with malformed files.
- Optimize performance for large files by using streaming approaches where applicable.

### Example: Properties File

Lets go over adding support for the `.properties` file-type.

#### PropertiesInput Implementation

Here is the basic implementation of `Input` for our `.properties` file.

```java
// Reads the data from a .properties file and returns a stream of JSONObjects.
public class PropertiesInput implements Input<JSONObject>
{
    @Override
    public InputCursor<JSONObject> read(Source source)
    {
        return new PropertiesInputCursor(source);
    }

    private class PropertiesInputCursor implements InputCursor<JSONObject>
    {
        BufferedReader reader;

        // public PropertiesInputCursor(Source source)...

        @Override
        public CloseableIterator<JSONObject> iterator()
        {
            return new CloseableIterator<JSONObject>(){
                String nextLine;

                {
                    try
                    {
                        nextLine = reader.readLine();
                    }
                    catch (IOException ex)
                    {
                        throw new RuntimeException(ex);
                    }
                }

                // @Override
                // public boolean hasNext()...

                @Override
                public JSONObject next()
                {
                    String[] property = nextLine.split("=", 2);
                    JSONObject obj = new JSONObject();

                    try
                    {
                        nextLine = reader.readLine();
                    }
                    catch (IOException ex)
                    {
                        throw new RuntimeException(ex);
                    }


                    obj.put(property[0], property[1]);
                    return obj;
                }

                // @Override
                // public void close() throws IOException...
            };
        }
    }
}
```

#### PropertiesOutput Implementation

Here is the basic implementation of `Output` for our `.properties` file.

```java
/**
 * This is used to write out JSONObjects to a .properties files.
 */
public class PropertiesOutput implements Output
{
    //  @Override
    //  public OutputCursor write(Target target)...

    //  @Override
    //  public String getContentType()...

   private class PropertiesOutputCursorWriter implements OutputCursor
   {
       private final Target target;
       private PrintWriter out;

       public PropertiesOutputCursorWriter(Target target)
       {
           this.target = target;
           this.out = new PrintWriter(target.getOutputStream(), false);
       }

       // Note: this example is lossy, any values with new lines would not be read properly.
       @Override
       public void write(JSONObject record)
       {
           Object value;

           for (String key : record.keySet())
           {
               value = record.get(key);

               out.print(key);
               out.print("=");
               out.print(value.toString().replace("\n", ""));
               out.print("\n");
           }
       }

       //  @Override
       //  public void close()...
   }
}
```

### Using the Properties Implementation

#### Reading properties values from a Database

In the following example we are going to use our new `.properties` implementation to write the results of a query to a file.

Database Data:

| blending_mode | accuracy | model         |
| ------------- | -------- | ------------- |
| lighten       | 0.75     | photoshop-cs6 |

```java
File example = new File("./user.properties")
FileTarget target = new FileTarget(example);

DBMS dbms = new DBMS(source);
Query query = new Query("select blending_mode, accuracy, model from SETTINGS limit 1");
Iterable<JSONObject> results = dbms.query(query);

PropertiesOutput output = new PropertiesOutput();
output.write(target, results);

/*
# user.properties would contain the following info (based on the data stored in the database)

blending_mode=lighten
accuracy=0.75
model=photoshop-cs6
*/
```

## Reading and Writing

Below are some short examples covering reading and writing with different outputs, they all follow the same pattern.

### Delimited Files

Delimited files provide a way to organize data seperated by a specific character.

```java
JSONArray results = new JSONArray("[{\"name\":\"John\", \"devices\":3}]");

// 'qdc' file type being question mark delimited values.
File file = new File("./test.qdv");
FileTarget target = new FileTarget(file);

// This will delimit the content with '?' using only the provided headers.
String[] fields = new String[]{"name", "devices"};
DelimitedOutput output = new DelimitedOutput(fields, '?');

// Write out the results with only the three fields.
output.write(target, results);

/*
The files output would look something like:
  name?devices
  John?3
*/
```

### JSON

#### Reading

The below examples reads in from some example file and prints out the `toString()` representation of the record.

```java
FileSource source = new FileSource(new File("./example.json"));
Iterable<JSONObject> input = new JSONInput().read(source);

for(JSONObject record : input){
  System.out.println(record);
}

/*
  {"name": "John", "devices": 3}
*/
```

#### Writing

```java
DBMS database = new DBMS(source);
Query query = new Query("SELECT * FROM customer");

Iterable<JSONObject> results = database.query(query);

File file = new File("./test.json");
FileTarget target = new FileTarget(file);

JSONOutput out = new JSONOutput();
out.write(target, results);

/*
  {"name": "John", "devices": 3},
  {"name": "Bob", "devices": 1},
*/
```

### CSV

Writing out to CSV from JSON is pretty simple, fields names are converted into headers and any values found matching the field names are added below.

```java
JSONArray example = new JSONArray("[{\"name\":\"John\", \"devices\":3}]");

File file = new File("./test.csv");
FileTarget target = new FileTarget(file);

CSVOutput output = new CSVOutput();
try(OutputCursor cursor = output.write(target))
{
    cursor.write(example);
}

/*
Output would look like:

  name, devices
  John, 3
*/
```

### JBIN

`JBIN` is useful in high throughput situations. Here is an example of writing a `JSONObject` into `JBIN`.

```java
JSONArray example = new JSONArray("[{\"name\":\"John\", \"devices\":3}]");
ByteArrayTarget target = new ByteArrayTarget();

JBINOutput output = new JBINOutput();

try(OutputCursor cursor = output.write(target))
{
    cursor.write(results.iterator());
    cursor.close();
}
```

## Further Reading

<div style="display: flex; align-items: center; gap: 8px; margin-bottom: 16px">
  <span style="display: flex; align-items: center; justify-content: center;font-size:20px; width: 24px; height: 24px">📚</span>
  <a href="https://docs.invirgance.com/javadocs/convirgance/latest/com/invirgance/convirgance/source/package-summary.html">Java Documentation: File Formats</a>
</div>
