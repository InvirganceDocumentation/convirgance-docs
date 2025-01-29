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

## Extending the Input Interface

Convirgance provides an extensible input interface to allow developers to integrate support for new data formats. By implementing the `Input` interface, developers can define custom readers for various formats.

### Example: Customer Properties File Implementation

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

### Steps to Add Support for Other Formats

1. **Implement the Input Interface**: Create a class implementing `Input<T>`, where `T` is the desired output type, such as `JSONObject`.
2. **Handle File Reading Logic**: Write the logic to parse the specific file format and convert the data into the desired structure.

### Best Practices

- Ensure robust error handling to deal with malformed files.
- Optimize performance for large files by using streaming approaches where applicable.

## Extending the Output Interface

Convirgance also allows developers to extend its output interface for supporting additional file formats. By implementing the `Output` interface, you can define custom writers to generate files in new formats.

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

### Steps to Add Support for Other Output Formats

1. **Implement the Output Interface**: Create a class implementing `Output`.
2. **Handle File Writing Logic**: Write the logic to convert the input data structure into the desired file format.

### Best Practices

- Ensure proper escaping of special characters (e.g., commas, quotes) in CSV files.
- Validate input data to match format-specific requirements.
- Optimize performance for writing large files by using buffered output streams.

### Scenarios

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

FileTarget target = new FileTarget(new File("./user.properties"));

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

You can find an example covering inserting data [here](./database-operations.md#transactions-inserting-and-querying-data)

## Format-Specific Examples

### Delimited Files

```java
Iterable<JSONObject> results =  new JSONArray("[{\"name\":\"John\"}]");

// Notice that our source data is missing fields, these will be supplemented and assigned to null.
String[] fields = new String[]{"name", "Device Count", "Dependents"};

// This will delimit the content with '?' using only the provided headers.
DelimitedOutput input = new DelimitedOutput(fields, "?");
ByteArrayTarget target = new ByteArrayTarget();

// Write out the results with only the three fields.
output.write(target, audience);
```

### JSON

```java
// filename being the path to some json file
source = new FileSource(new File(filename));
input = new JSONInput().read(source);

ByteArrayTarget target = new ByteArrayTarget();

output.write(target, audience);
```

### CSV

```java
ByteArrayTarget target = new ByteArrayTarget();
CSVOutput output = new CSVOutput();

try(OutputCursor cursor = output.write(target))
{
    cursor.write(new JSONArray("[{\"name\":\"John\"}]"));
    cursor.close();
}
```

### JBIN

```java
Iterable<JSONObject> results =  new JSONArray("[{\"name\":\"John\"}]");
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
