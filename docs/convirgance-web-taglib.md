# Tag Library: Convirgance (Web Services)

While the Java Standard Tag Library (JSTL) is very effective at developing
templated pages, numerous shortcomings were found in trying to implement
Hypermedia Driven Applications (HDA) using the library.T

To improve this situation, Convirgance (Web Services) provides a replacement
taglib better suited to Convirgance and HDA design. 

## Usage

The tag library can be easily imported into a JSP page with the following tag:

```jsp
<%@taglib uri="convirgance:web" prefix="virge" %>
```

Note that the prefix used in these examples will be `virge`. This can be changed
by updating the `prefix` attribute in the tag above.

## Functions

One of the critical failures of JSTL is its approach to outputing data into the
template. No attempt is made to differentiate between HTML, Javascript, and URL
Parameters. This leads to numerous vectors for injection attacks, even when
correctly using the `<c:out>` tag. 

Convirgance solves this problem by providing functions for formatting values
based on the output language. The `${virge:html(param.value)}` function attempts to
format data as HTML.

The `${virge:javascript(param.value)}` function outputs strings properly quoted 
for Javascript/JSON, numbers as numbers, and JSON structures such as objects and
arrays as Javascript objects and arrays.

Finally, you can use `${virge:urlparam(param.value)}` when constructing a URL to
ensure that values are properly escaped in the URL.

The tag library additionally provides `${virge:first(list)}` and `${virge:last(list)}`
to retrieve the first or last records of an iterable such as a service call.

## JSON

JSON tags allow JSON objects to be created and manipulated within a JSP page. All
tags have an optional `var` attribute to assign the object to, and a `scope`
variable to set the scope of the `var`. 

However, these tags can be embedded into other tags, allowing the creation of
deep hierarchies created at runtime.

| Tag            | Contents                |
|----------------|-------------------------|
| virge:json     | A JSON string to decode |
| virge:object   | Use the `virge:key` tag as children to set the key/value pairs of the `JSONObject` |
| virge:array    | Use the `virge:value` tag to add entries to the `JSONArray` |

These tags can be incredibly useful in a number of ways. Here are just a few
suggestions:

- Description of a form layout which can then be looped through to generate form fields
- Construction of a `JSONObject` from parameter data serialized into a Javascript variable
- Parameters to another tag like `virge:query`


## Service Call

The `virge:service` call allows you to easily access backend services from the
JSP page. Note that this tag is intended to allow access to data and must route
to a `SelectService` or equivalent. The request to the service is equivalent to
a `GET` request to the service.

Example of a service call to retrieve a single customer:

```xml
<virge:service var="customers" path="/api/customers/${param.api}" />
<virge:set var="customer" value="${virge:first(customers)}" />
```

The `virge:parameter` tag can be used to pass query parameters to the service. You
can use the `default` attribute to control what gets passed to the service rather
than passing a `null` value.

Example of a service call to retrieve a filtered customer list:

```xml
<virge:service var="customers" path="/api/customers">
    <virge:parameter name="zipcode" value="${param.zipcode}" default="" />
    <virge:parameter name="state" value="${param.state}" default="" />
    <virge:parameter name="discountCode" value="${param.discountCode}" default="" />
</virge:service>
```

## Control Flow

The Convirgance (Web Services) tag library provides control flow tools designed
to work more cleanly with the `Iterable`, `JSONObject`, and `JSONArray` objects
that the platform is based upon. 

In addition, key functionality from the Java Standard Tag Library (JSTL) has been
reimplemented to avoid having to include multiple tag libraries.

All tags have a `var` attribute to assign the object to, and a `scope`
variable to set the scope of the `var`. In the case of `virge:iterate`, the variable
specified by `var` will contain the current object for that loop.

| Tag              | Description                                                |
|------------------|------------------------------------------------------------|
| `virge:iterate`  | Iterates over the values passed to the `items` attribute.  |
| `virge:if`       | Evaluates the `test` attribute as a boolean. Contents of the tag are only rendered if `test` is true.  |
| `virge:set`      | Sets the value passed to the `value` attribute as the value of the variable specified by `var` |


## Database Query

While it is generally not recommended to make direct database queries from web page
templates, the Convirgance (Web Services) tag library provides the `virge:query`
tag to accomplish direct queries.

The JNDI path of the database connection must be passed in the `jndi` attribute. A
binding object can also be optionally passed to the `binding` attribute. Typically
this object would be constructed with one of the JSON tags above.

The contents of tag is the query that you want to execute. Values from the binding
object can be referenced by using the named parameter `:key` syntax.

Note that JSP EL syntax is explicity disabled in the query. To prevent SQL injection
attacks, you must use a binding object to pass in parameters.

**Example:**

```html
<virge:object var="binding">
    <virge:key name="zipcode" value="${param.zipcode}" default="" />
    <virge:key name="state" value="${param.state}" default="" />
    <virge:key name="discountCode" value="${param.discountCode}" default="" />
</virge:object>

<virge:query var="customers" jndi="jdbc/sample" binding="${binding}">
select * from APP.CUSTOMER
where (:zipcode = '' or ZIP = :zipcode)
and (:state = '' or STATE = :state)
and (:discountCode = '' or DISCOUNT_CODE = :discountCode)
</virge:query>
```