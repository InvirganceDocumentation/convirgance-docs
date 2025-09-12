# Hypermedia Driven Applications (HDA)

The [Hypermedia Driven Application (HDA)](https://htmx.org/essays/hypermedia-driven-applications/) 
architecture is an alternate approach to building web applications that relies more
heavily on the backend for application delivery. It combines the 
simplicity & flexibility of traditional Multi-Page Applications (MPAs) with the 
better user experience of Single-Page Applications (SPAs).

Convirgance (Web Services) provides backend support for HDA applications via
the `HypermediaService` implementation. Combined with the `RoutedService`, complex
hierarchies of hypermedia interfaces can be exposed.

## Basic Structure

In its simplest form, a Hypermedia resource renders a JSP page for a REST 
resource. 

We'll use the example of a `Customer` resource. We'll need a JSON service for the
backend, a JSP page to render the data, and a `HypermediaService` to tie it all
together.

`src/main/webapp/api/customer.xml` - Backend Service:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<SelectService>
    <parameters>
        <list>
            <PathVariable>
                <path>/api/customer/{customerId}</path>
            </PathVariable>
        <list>
    </parameters>
    <binding>
        <QueryBinding>
            <sql>
            <![CDATA[
SELECT * from CUSTOMER
WHERE id = :customerId
]]>
            </sql>
        </QueryBinding>
    </binding>
    <output>
        <JSONOutput />
    </output>
</SelectService>
```

`src/main/webapp/customer.jsp` - Customer Page:

```html
<%@page contentType="text/html" pageEncoding="UTF-8"%>
<%@taglib uri="convirgance:web" prefix="virge" %>
<!doctype html>

<%-- Read the customer information from the backend service -->
<virge:service var="customers" path="/api/customer/${param.customerId}" />
<html>
    <head>
        <title>Customer</title>
    <head>
    <body>
        <virge:iterate var="customer" items="${customers}">
            <div>Customer Id: ${virge:html(customer.id)}</div>
            <div>Name: ${virge:html(customer.name)}</div>
            <div>Address: ${virge:html(customer.address)}</div>
            <div>State: ${virge:html(customer.state)}</div>
            <div>Zipcode: ${virge:html(customer.zipcode)}</div>
        </virge:iterate>
    <body>
</html>
```

`src/main/webapp/view/customer.xml` - Hypermedia Service:

```xml
<HypermediaService>
    <parameters>
        <list>
            <PathVariable>
                <path>/view/customer/{customerId}</path>
            </PathVariable>
        </list>
    </parameters>
    <page>/customer.jsp</page>
</HypermediaService>
```

With this setup, visiting the `/view/customer/1` path in your browser would render
the `customer.jsp` page for `customerId` of `1`. Though in its current form, this
is not much different than just accessing the url `/customer.jsp?customerId=1`. 

The real power of the Hypermedia service is when we start using verbs.

## Understanding Verbs

REST services typically provide one retreival interface in terms of a `GET`
request with `POST`, `PUT`, and `DELETE` taking actions upon the data. This is
a good design as a backend API. 

This approach creates a problem for hypermedia interfaces. For example, you need
an interface to edit a customer in addition to the service to commit the edit.

Further complicating the matter is that HTML only supports `GET` and `POST` in
its hypermedia controls. (e.g. `form`, `a`, etc.) 

Verbs are a solution to this problem. For example, if we want to edit the customer
resource at `/customer/1`, we can create a verb called `edit` which would then be
accessible at `/customer/1/edit`. The verb can be configured with a page template on
`GET` and a backend service to call on `POST`. This provides a clean, symmetrical 
interface with no special URL needing to be configured on the HTML `form` element.

The verb will redirect back to the non-verb path after a `POST`
request, ensuring that the updated data can be reviewed. e.g.  `/customer/1/edit`
redirects back to `/customer/1` unless an alternate redirect path is supplied.

Here is an update to the example Hypermedia Service with the `edit` verb added: 

```xml
<HypermediaService>
    <parameters>
        <list>
            <PathVariable>
                <path>/view/customer/{customerId}</path>
            </PathVariable>
        </list>
    </parameters>
    <page>/customer.jsp</page>
    <verbs>
        <list>
            <verb>
                <name>edit</name>
                <page>/customer_edit.jsp</page>
                <service>/api/customer/{id}</service>
                <method>PUT</method>
            </verb>
        </list>
    </verbs>
</HypermediaService>
```

Note how we have configured a PUT request to the `/api/customer` service to
handle the `POST` submitted to the verb. The verb automatically translates the 
form data post into a JSON stream before calling the backend service. This makes
the call transparent to the backend.

Now we need to update the backend service so it can handle the PUT.

## REST Service backend 

While we could create a separate backend service for each verb, generally we
want to pair our Hypermedia views with a REST service backend. We can use 
`RESTService` to provide separate logic based upon the HTTP method.

To see how this works, we'll update our `src/main/webapp/api/customer.xml` example
so that it supports separate logic for `GET` and `PUT` requests:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<RESTService>
    <GET>
        <SelectService>
            <parameters>
                <list>
                    <PathVariable>
                        <path>/api/customer/{customerId}</path>
                    </PathVariable>
                <list>
            </parameters>
            <binding>
                <QueryBinding>
                    <sql>
                    <![CDATA[
SELECT * from CUSTOMER
WHERE id = :customerId
]]>
                    </sql>
                </QueryBinding>
            </binding>
            <output>
                <JSONOutput />
            </output>
        </SelectService>
    </GET>
    <PUT>
        <InsertService>
            <parameters>
                <list>
                    <PathVariable>
                        <path>/api/customer/{customerId}</path>
                    </PathVariable>
                </list>
            </parameters>
            <injectParameters>customerId</injectParameters>
            <origin>
                <RequestBodyOrigin />
            </origin>
            <input>
                <JSONInput />
            </input>
            <consumer>
                <QueryConsumer>
                    <jndiName>jdbc/customerdb</jndiName>
                    <sql>
                        <![CDATA[
    UPDATE CUSTOMER SET 
        NAME = :name,
        ADDRESS = :address,
        STATE = :state,
        ZIPCODE = :zipcode
    WHERE ID = :customerId
]]>
                    </sql>
                </QueryConsumer>
            </consumer>
        </InsertService>
    </PUT>
</RESTService>
```

Now that we understand how verbs work and how REST services work, we only need
to look at how the `customer_edit.jsp` page might be implemented.

## HTML Forms

Because the `HypermediaService` is exposing a clean REST interface, the `edit`
interface can be little more than a `<form method="POST">` tag with various
`<input>` controls. 

No `action` attribute is required to change the URL as the data can be submitted
directly back to the same URL. Since the identifier is embedded in the URL and
injected by the backend service, there's no need to use a hidden input to capture 
that identifier. 

The result is a clean form that is easy to understand and update:

```html
<%@page contentType="text/html" pageEncoding="UTF-8"%>
<%@taglib uri="convirgance:web" prefix="virge" %>
<!doctype html>

<%-- Read the customer information from the backend service -->
<virge:service var="customers" path="/api/customer/${param.customerId}" />
<html>
    <head>
        <title>Edit Customer</title>
    <head>
    <body>
        <virge:iterate var="customer" items="${customers}">
            <form method="POST">
                <div>
                    Name: 
                    <input name="name" value="${virge:html(customer.name)}">
                </div>
                <div>
                    Address: 
                    <input name="address" value="${virge:html(customer.address)}">
                </div>
                <div>
                    State: 
                    <input name="state" value="${virge:html(customer.state)}">
                </div>
                <div>
                    Zipcode: 
                    <input name="zipcode" value="${virge:html(customer.zipcode)}">
                </div>
            </form>
        </virge:iterate>
    <body>
</html>
```
