<a id="api-request-type"></a>

# Request Type

A request type is a crucial concept of the ApiBundle.

A set of processors handles each API request, and each processor pulls its weight to reach the
requested result. Some processors analyze and validate the request data, while others update the database
and prepare the correct response. If you need to process different types of
You need to have a different set of processors for API requests, such as REST API and REST API that conform to the <a href="http://jsonapi.org/format/" target="_blank">JSON:API specification</a>. Some may work for all request types, and others only for specific ones.
The concept of the request type in ApiBundle reflects all mentioned above and
enables you to configure shared and specific processors easily.

Take a look at the <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Request/RequestType.php" target="_blank">RequestType</a> class. It was designed to contain different aspects
of a request; the combination of these aspects represents a specific request type.
For instance, if this class contains both `rest` and `json_api`, it can be interpreted as a request type for REST API that conforms to the JSON:API specification. If you add the `my_erp` aspect to this request type, it means that it represents REST API specially designed for the integration with the “My ERP” system and is based on the JSON:API specifications. If, for example, you have two types of REST API, one that conforms to the JSON:API specification and another one that conforms to the GraphQL specification, then the RequestType object may contain `rest` and `json_api` for JSON:API requests and `rest` and `graphql` for GraphQL requests.

This combination of aspects enables you to configure different sets of processors for each request type.
See examples of configuring processors for different aspects and their combinations in the
[Processor Conditions](processors.md#processor-conditions) topic.

<!-- Frontend -->
