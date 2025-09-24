<a id="web-services-api-http-methods"></a>

# Available HTTP Methods

The primary or most commonly-used HTTP methods are POST, GET, PUT, PATCH, and DELETE. These methods correspond to create, read, update, and delete (or CRUD) operations, respectively. There are a number of other methods, too, but they are utilized less frequently.

Below is a table summarizing HTTP methods available in Oro API and their return values in combination with the resource URIs:

| HTTP Method   | CRUD operation   | Entire Collection (e.g. /users)                                                                    | Specific Item (e.g. /users/{id})                                                    |
|---------------|------------------|----------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------|
| GET           | Read             | 200 (OK), list of entities.<br/>Use pagination, sorting and filtering<br/>to navigate big lists.   | 200 (OK), single entity.<br/><br/>404 (Not Found), if ID not found or invalid.      |
| POST          | Create           | 201 (Created), Response contains<br/>response similar to **GET** /user/{id}<br/>containing new ID. | **not applicable**                                                                  |
| PATCH         | Update           | [Batch API](batch-api.md#web-services-api-batch-api)                                               | 200 (OK) or 204 (No Content).<br/><br/>404 (Not Found), if ID not found or invalid. |
| DELETE        | Delete           | 204 (No Content).<br/><br/>400(Bad Request) if no filter is specified.                             | 204 (No Content).<br/><br/>404 (Not Found), if ID not found or invalid.             |
| PUT           | Create/Replace   | **not implemented**                                                                                | **not implemented**                                                                 |

Also, the HTTP methods can be classified by the *idempotent* and *safe* properties.

The *safe* methods are the HTTP methods that do not modify resources. For instance, using GET or HEAD on a resource URL,
should NEVER change the resource.

An *idempotent* HTTP method is an HTTP method that can be called many times without different outcomes. It would not
matter if the method is called only once, or ten times over. The result should be the same.
For more details, please see <a href="https://tools.ietf.org/html/rfc7231#section-4.2" target="_blank">RFC 7231: Common Method Properties</a>.

Below is a table summarizing HTTP methods by its idempotency and safety:

| HTTP Method   | Idempotent   | Safe   |
|---------------|--------------|--------|
| OPTIONS       | yes          | yes    |
| GET           | yes          | yes    |
| HEAD          | yes          | yes    |
| PUT           | yes          | no     |
| POST          | no           | no     |
| DELETE        | yes          | no     |
| PATCH         | no           | no     |

#### NOTE
Business Tip

Researching <a href="https://oroinc.com/b2b-ecommerce/b2b-ecommerce-comparison" target="_blank">B2B eCommerce platforms</a>? Explore our platform comparison guide to help you make an informed decision.

<a id="web-services-api-http-methods-get"></a>

## GET

The HTTP GET method is used to *read* (or retrieve) a representation of a resource. In case of success (or non-error), GET returns a representation in JSON and an HTTP response status code of 200 (OK). In an error case, it most often returns a 404 (NOT FOUND) or 400 (BAD REQUEST).

#### NOTE
According to the design of the HTTP specification, GET requests are used only to read data and not change it.
So, they are considered safe. That is they can be called without risk of data modification or corruption—calling it once has the same effect as calling it 10 times.

<a id="web-services-api-http-methods-post"></a>

## POST

The POST method is most often utilized to *create* new resources. In particular, it is used to create subordinate
resources. That is subordinate to some other (e.g. parent) resource. In other words, when creating a new resource,
POST to the parent and the service takes care of associating the new resource with the parent, assigning an
ID (new resource URI), etc.

On successful creation, HTTP response code 201 is returned.

#### NOTE
POST is not a safe operation. Making two identical POST requests will most likely result in two resources containing the same information but with different identifiers.

#### NOTE
It is possible to create both primary and related API resources via a single API request. For details, see the
[Creating and Updating Related Resources with Primary API Resource](create-update-related-resources.md#web-services-api-create-update-related-resources) section.

#### NOTE
For some resources, you can create a resource (if it did not already exist) or update it (if it does) via a single API request.
For details, see the [Creating Resource or Updating Existing Resource via Single API Request](upsert-operation.md#web-services-api-upsert-operation) section.

<a id="web-services-api-http-methods-patch"></a>

## PATCH

PATCH is used to *modify* resources. The PATCH request only needs to contain the changes to the resource,
not the complete resource.

In other words, the body should contain a set of instructions describing how a resource currently residing on the
server should be modified to produce a new version.

#### NOTE
For details, see the [Creating and Updating Related Resources with Primary API Resource](create-update-related-resources.md#web-services-api-create-update-related-resources) section.

#### NOTE
For some resources, you can create a resource (if it did not already exist) or update it (if it does) via a single API request.
For details, see the [Creating Resource or Updating Existing Resource via Single API Request](upsert-operation.md#web-services-api-upsert-operation) section.

<a id="web-services-api-http-methods-delete"></a>

## DELETE

DELETE is quite easy to understand. It is used to *delete* a resource identified by filters or ID.

On successful deletion, the HTTP response status code 204 (No Content) returns with no response body.

#### IMPORTANT
If you DELETE a resource, it is removed. Repeatedly calling DELETE on that resource will often return a 404 (NOT FOUND) status code since it was already removed and, therefore, is no longer findable.

<!-- Frontend -->
<!-- fa-bars = fa-navicon -->
<!-- Ic Tiles is used as Set As Default in saved views, and as tiles in display layout options -->
<!-- IcPencil refers to Rename in Commerce and Inline Editing in CRM -->
<!-- Check mark in the square. -->
<!-- SortDesc is also used as drop-down arrow -->
