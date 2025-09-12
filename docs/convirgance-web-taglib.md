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

- Iterator
- If
- Set

## Database Query