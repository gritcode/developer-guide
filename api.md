[🏠](README.md) › APIs

# API Services Guide

## Methods

CRUD | Method | Path | Response (StatusCode) | Response (JSON Payload)
---- | ------ | ---- | -------- | --------
Create | POST | /todo {paypload} | 201 (created), 404 (Not Found), 409 (Conflict) if resource already exists | Object or Array of Objects
List | GET | /todo[,?fields,limit,page] | 200 (OK) | Object or Array of Objects
Read | GET | /todo/{id} | 200 (OK), single resource. 404 (Not Found), if ID not found or invalid. | Object
Update | PUT | /todo/{id} {full payload} | 200 (OK) or 204 (No Content). 404 (Not Found), if ID not found or invalid | Object
Modify | PATCH | /todo/{id} {single payload} e.g. {status = 'disabled'} | 200 (OK) or 204 (No Content). 404 (Not Found), if ID not found or invalid | Object
Delete | DELETE | /todo/ | 200 (OK). 404 (Not Found), if ID not found or invalid | Object

[Reference](http://www.restapitutorial.com/lessons/httpmethods.html)


## Response Codes
#### _We will want to redefine these for our usecase_

- **200 OK** General success status code. This is the most common code. Used to indicate success.
- **201 CREATED** Successful creation occurred (via either POST or PUT). Set the Location header to contain a link to the newly-created resource (on POST). Response body content may or may not be present.
- **204 NO CONTENT** Indicates success but nothing is in the response body, often used for DELETE and PUT operations.
- **400 BAD REQUEST** General error for when fulfilling the request would cause an invalid state. Domain validation errors, missing data, etc. are some examples.
- **401 UNAUTHORIZED** Error code response for missing or invalid authentication token.
- **403 FORBIDDEN** Error code for when the user is not authorized to perform the operation or the resource is unavailable for some reason (e.g. time constraints, etc.).
- **404 NOT FOUND** Used when the requested resource is not found, whether it doesn't exist or if there was a 401 or 403 that, for security reasons, the service wants to mask.
- **405 METHOD NOT ALLOWED** Used to indicate that the requested URL exists, but the requested HTTP method is not applicable. For example, POST /users/12345 where the API doesn't support creation of resources this way (with a provided ID). The Allow HTTP header must be set when returning a 405 to indicate the HTTP methods that are supported. In the previous case, the header would look like "Allow: GET, PUT, DELETE"
- **409 CONFLICT** Whenever a resource conflict would be caused by fulfilling the request. Duplicate entries, such as trying to create two customers with the same information, and deleting root objects when cascade-delete is not supported are a couple of examples.
- **500 INTERNAL SERVER ERROR** Never return this intentionally. The general catch-all error when the server-side throws an exception. Use this only for errors that the consumer cannot address from their end.

[Reference](http://www.restapitutorial.com/lessons/restquicktips.html)

## Route/Resource Naming

### Route Examples
To insert (create) a new customer in the system, we might use: `POST http://www.example.com/customers`

To read a customer with Customer ID# 33245: `GET http://www.example.com/customers/33245` The same URI would be used for PUT and DELETE, to update and delete, respectively.

Here are proposed URIs for products: `POST http://www.example.com/products` for creating a new product.

`GET|PUT|DELETE http://www.example.com/products/66432` for reading, updating, deleting product 66432, respectively.

Now, here is where it gets fun... What about creating a new order for a customer? One option might be: POST http://www.example.com/orders And that could work to create an order, but it's arguably outside the context of a customer.

Because we want to create an order for a customer (note the relationship), this URI perhaps is not as intuitive as it could be. It could be argued that the following URI would offer better clarity: POST http://www.example.com/customers/33245/orders Now we know we're creating an order for customer ID# 33245.

Now what would the following return?
`GET http://www.example.com/customers/33245/orders` Probably a list of orders that customer #33245 has created or owns. Note: we may choose to not support DELETE or PUT for that url since it's operating on a collection.

Now, to continue the hierarchical concept, what about the following URI?
`POST http://www.example.com/customers/33245/orders/8769/lineitems` That might add a line item to order #8769 (which is for customer #33245). Right! GET for that URI might return all the line items for that order. However, if line items don't make sense only in customer context or also make sense outside the context of a customer, we would offer a POST www.example.com/orders/8769/lineitems URI.

Along those lines, because there may be multiple URIs for a given resource, we might also offer a `GET http://www.example.com/orders/8769` URI that supports retrieving an order by number without having to know the customer number.

To go one layer deeper in the hierarchy: `GET http://www.example.com/customers/33245/orders/8769/lineitems/1` Might return only the first line item in that same order.

By now you can see how the hierarchy concept works. There aren't any hard and fast rules, only make sure the imposed structure makes sense to consumers of your services. As with everything in the craft of Software Development, naming is critical to success.

Look at some widely used APIs to get the hang of this and leverage the intuition of your teammates to refine your API resource URIs. Some example APIs are:

### Anti-Patterns

While we've discussed some examples of appropriate resource names, sometimes it's informative to see some anti-patterns. Below are some examples of poor RESTful resource URIs seen in the "wild." These are examples of what not to do!

First up, often services use a single URI to specify the service interface, using query-string parameters to specify the requested operation and/or HTTP verb. For example to update customer with ID 12345, the request for a JSON body might be: `GET http://api.example.com/services?op=update_customer&id=12345&format=json`

By now, you're above doing this. Even though the 'services' URL node is a noun, this URL is not self- descriptive as the URI hierarchy is the same for all requests. Plus, it uses GET as the HTTP verb even though we're performing an update. This is counter-intuitive and is painful (even dangerous) to use as a client.

Here's another example following the same operation of updating a customer: `GET http://api.example.com/update_customer/12345`

And its evil twin: `GET http://api.example.com/customers/12345/update`

You'll see this one a lot as you visit other developer's service suites. Note that the developer is attempting to create RESTful resource names and has made some progress. But you're better than this —able to identify the verb phrase in the URL. Notice that we don't need to use the 'update' verb phrase in the URL because we can rely on the HTTP verb to inform that operation. Just to clarify, the following resource URL is redundant: `PUT http://api.example.com/customers/12345/update`

With both PUT and 'update' in the request, we're offering to confuse our service consumers! Is 'update' the resource? So, we've spent some time beating the horse at this point. I'm certain you understand...

### Pluralization

Let's talk about the debate between the pluralizers and the "singularizers"... Haven't heard of that debate? It does exist. Essentially, it boils down to this question...

Should URI nodes in your hierarchy be named using singular or plural nouns? For example, should your URI for retrieving a representation of a customer resource look like this: `GET http://www.example.com/customer/33245 or: GET http://www.example.com/customers/33245`

There are good arguments on both sides, but the commonly-accepted practice is to always use plurals in node names to keep your API URIs consistent across all HTTP methods. The reasoning is based on the concept that customers are a collection within the service suite and the ID (e.g. 33245) refers to one of those customers in the collection.

Using this rule, an example multi-node URI using pluralization would look like (emphasis added): `GET http://www.example.com/customers/33245/orders/8769/lineitems/1` with 'customers', 'orders', and 'lineitems' URI nodes all being their plural forms.

This implies that you only really need two base URLs for each root resource. One for creation of the resource within a collection and the second for reading, updating and deleting the resource by its identifier. For example the creation case, using customers as the example, is handled by the following URL: `POST http://www.example.com/customers`

And the read, update and delete cases are handled by the following: `GET|PUT|DELETE http://www.example.com/customers/{id}`

As mentioned earlier, there may be multiple URIs for a given resource, but as a minimum full CRUD capabilities are aptly handled with two simple URIs.

You ask if there is a case where pluralization doesn't make sense. Well, yes, in fact there is. When there isn't a collection concept in play. In other words, it's acceptable to use a singularized resource name when there can only be one of the resource—it's a singleton resource. For example, if there was a single, overarching configuration resource, you might use a singularized noun to represent that: `GET|PUT|DELETE http://www.example.com/configuration`

Note the lack of a configuration ID and usage of POST verb. And say that there was only one configuration per customer, then the URL might be: `GET|PUT|DELETE http://www.example.com/customers/12345/configuration`

Again, no ID for the configuration and no POST verb usage. Although, I'm sure that in both of these cases POST usage might be argued to be valid. Well... OK.

[Route/Resource Naming](http://www.restapitutorial.com/lessons/restfulresourcenaming.html)
