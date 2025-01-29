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

- Ensure proper escaping of special characters (e.g., commas, quotes) in CSV files.
- Validate input data to match format-specific requirements.
- Optimize performance for writing large files by using buffered output streams.
- Ensure robust error handling to deal with malformed files.
- Optimize performance for large files by using streaming approaches where applicable.

### Example: Properties File

#### Example Input

Here is the data for our example properties file.

```sh
# Example 'pet_type.properties' file
dog=true
cat=false
```

#### Example Output

Example output from the `PropertiesInput`'s read as it processes the stream of the example file, printing out each record. Further clarification, the example implementation reads the example files `stream` into an `Iterable` of `JSONObject`s, a for-each loop is used to print each `JSONObject`.

```json
{"dog":"true"}
{"cat":"false"}
```

### Example: PropertiesInput Implementation

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

### Example: PropertiesOutput Implementation

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

       // Keep in mind this example is lossy, any values with new lines would not be read properly.
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

##### DB Data

| blending_mode | accuracy | model         |
| ------------- | -------- | ------------- |
| lighten       | 0.75     | photoshop-cs6 |
| darken        | 0.82     | gimp-2.10     |
| difference    | 0.79     | krita-5.0     |

##### Pulling the first record into the local `properties` file

```java
byte[] bytes;

PropertiesInput input = new PropertiesInput();
PropertiesOutput output = new PropertiesOutput();
ByteArraySource byteSource;
File example = new File("./user.properties")

FileTarget target = new FileTarget(example);

DBMS dbms = new DBMS(source);
Query query = new Query("select blending_mode, accuracy, model from SETTINGS limit 1");

Iterable<JSONObject> results = dbms.query(query);
output.write(target, results);
```

#### File Contents

```sh
blending_mode.0=lighten
accuracy.0=0.75
model.0=photoshop-cs6
```

Want to learn more about inserting data? Documentation covering that is found [here](./database-operations.md#inserting-data)

## More File Examples

The following examples are for files with native support.

### Delimited Files

Delimited files provide a way to organize data seperated by a specific character.

```java
JSONArray results = new JSONArray("[{\"name\":\"John\", \"devices\":3}]");

String[] fields = new String[]{"name", "devices"};

// This will delimit the content with '?' using only the provided headers.
DelimitedOutput output = new DelimitedOutput(fields, "?");
ByteArrayTarget target = new ByteArrayTarget();

// Write out the results with only the three fields.
output.write(target, results);

/*
The files output would look something like:

  John?3
*/
```

### JSON

<!-- TODO fix this  -->

```java
// filename being the path to some json file
FileSource source = new FileSource(new File(filename));
Iterable<JSONObject> input = new JSONInput().read(source);

ByteArrayTarget target = new ByteArrayTarget();

output.write(target, audience);
```

### CSV

Writing out to CSV from JSON is pretty simple, fields names are converted into headers and any values found matching the field names are added below.

```java
JSONArray example = new JSONArray("[{\"name\":\"John\", \"devices\":3}]");
ByteArrayTarget target = new ByteArrayTarget();
CSVOutput output = new CSVOutput();

try(OutputCursor cursor = output.write(target))
{
    cursor.write(example);
    cursor.close();
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

- [Java Documentation](https://docs.invirgance.com/javadocs/convirgance/latest/com/invirgance/convirgance/source/package-summary.html)
