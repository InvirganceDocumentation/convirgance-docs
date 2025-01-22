# File Formats

Convirgance treats all data sources equally, whether they're CSV files, JSON documents, or database tables. You can read from one format and write to another without changing your business logic. Need to take a CSV file and output JSON? Or query a database and save to CSV? The same filtering and transformation rules work regardless of your input or output format, making it easy to work with data in whatever form it arrives or needs to be delivered.

## Supported Formats

| Format        | Description                                             | Read/Write |
| ------------- | ------------------------------------------------------- | ---------- |
| CSV           | Comma-separated values, widely used for tabular data.   | Read/Write |
| JSON          | JavaScript Object Notation, common for structured data. | Read/Write |
| SQL           | Structured Query Language, for relational databases.    | Read/Write |
| Excel (TBD)   | Microsoft Excel spreadsheets (.xls, .xlsx).             | Read/Write |
| Parquet (TBD) | Columnar storage file format for big data applications. | Read/Write |

## Extending the Input Interface

Convirgance provides an extensible input interface to allow developers to integrate support for new data formats. By implementing the `Input` interface, developers can define custom readers for various formats.

### Example: CSVInput Implementation

```java
/**
 * An implementation of the Input interface to handle CSV files.
 */
public class CSVInput implements Input<JSONObject>
{
    @Override
    public JSONObject read(String filePath) throws IOException
    {
        JSONObject row;
        JSONObject result = new JSONObject();
        String[] values;
        String[] headers;
        String line;

        try (BufferedReader buffer = new BufferedReader(new FileReader(filePath)))
        {
            line = null;
            headers = null;

            while ((line = buffer.readLine()) != null)
            {
                values = line.split(",");
                if (headers == null)
                {
                    headers = values;
                }
                else
                {
                    row = new JSONObject();

                    for (int i = 0; i < headers.length; i++)
                    {
                        row.put(headers[i], values[i]);
                    }

                    result.append("rows", row);
                }
            }
        }

        return result;
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

### Example: CSVOutput Implementation

```java
/**
 * An implementation of the Output interface to handle CSV file writing.
 */
public class CSVOutput implements Output<JSONObject>
{
    @Override
    public void write(String filePath, JSONObject data) throws IOException
    {
        JSONObject firstRow;
        JSONObject row;
        String line;
        String headers;
        JSONArray rows;

        try (BufferedWriter writer = new BufferedWriter(new FileWriter(filePath)))
        {
            rows = data.getJSONArray("rows");

            // Write headers
            if (rows.length() > 0)
            {
                firstRow = rows.getJSONObject(0);
                headers = String.join(",", firstRow.keySet());
                writer.write(headers);
                writer.newLine();
            }

            // Write data rows
            for (int i = 0; i < rows.length(); i++)
            {
                row = rows.getJSONObject(i);

                line = row.keySet().stream()
                    .map(key -> row.optString(key, ""))
                    .collect(Collectors.joining(","));

                writer.write(line);
                writer.newLine();
            }
        }
    }
}
```

### Steps to Add Support for Other Output Formats

1. **Implement the Output Interface**: Create a class implementing `Output<T>`, where `T` is the input type for writing data (e.g., `JSONObject`).
2. **Handle File Writing Logic**: Write the logic to convert the input data structure into the desired file format.

### Best Practices

- Ensure proper escaping of special characters (e.g., commas, quotes) in CSV files.
- Validate input data to match format-specific requirements.
- Optimize performance for writing large files by using buffered output streams.

## Format-Specific Examples

### Delimited Files

```java
import com.convirgance.Convirgance;

public class Demo
{
    private static DataSource source;

    public static void main(String[] args)
    {
        DBMS database = new DBMS(source);
        Iterable<JSONObject> results;
        String[] fields;

        // This will delimit the content with '?' using only the provided headers
        DelimitedOutput input = new DelimitedOutput(fields, "?");
        ByteArrayTarget target = new ByteArrayTarget();

        results = database.query(new Query("select * from CUSTOMER"));
        fields = new String[]{"Names", "Device Count", "Dependents"};

        // Write out the results with only the three fields.
        output.write(target, audience);
    }
}
```

### JSON

```java
import com.convirgance.Convirgance;

public class Demo
{
    private static DataSource source;

    public static void main(String[] args)
    {
        source = new FileSource(new File(filename));
        input = new JSONInput().read(source);

        ByteArrayTarget target = new ByteArrayTarget();

        // Write out the results with only the three fields.
        output.write(target, audience);
    }
}
```

### CSV

```java
import com.convirgance.Convirgance;

public class Demo
{
    public static void main(String[] args)
    {
        ByteArrayTarget target = new ByteArrayTarget();
        CSVOutput output = new CSVOutput();

        try(OutputCursor cursor = output.write(target))
        {
            cursor.write(new JSONArray("[{\"name\":\"John\",\"quote\":\"His favorite quote is \\\"Hello World\\\"\"},{\"name\":\"Alice\",\"quote\":\"She said \\\"Hi\\\"\"}]"));
        }
    }
}
```

### JBIN

```java
import com.convirgance.Convirgance;

public class Demo
{
    private static DataSource source;

    public static void main(String[] args)
    {
        DBMS database = new DBMS(source);
        Iterable<JSONObject> results = database.query(new Query("select * from CUSTOMER"));
        ByteArrayTarget target = new ByteArrayTarget();
        JBINInput input = new JBINInput();
        JBINOutput output = new JBINOutput();

        try(OutputCursor cursor = output.write(target))
        {
            cursor.write(results);
        }
    }
}
```
