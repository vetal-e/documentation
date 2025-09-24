<a id="web-api"></a>

# API Developer Guide

This section describes the Web API development framework for the application data. It provides the ability to define API in the YAML configuration files regardless of standards or formats. Out-of-the-box, the framework opens REST API that conforms the <a href="http://jsonapi.org/format/" target="_blank">JSON:API specification</a> and enables CRUD operations support for the application ORM entities.

The Web API development framework is implemented by OroApiBundle and based on the following components:

* <a href="https://github.com/oroinc/platform/tree/master/src/Oro/Component/ChainProcessor" target="_blank">ChainProcessor</a> — Organizes data processing flow.
* <a href="https://github.com/oroinc/platform/tree/master/src/Oro/Component/EntitySerializer" target="_blank">EntitySerializer</a> — Provides fast access to entities data.
* <a href="https://github.com/symfony/form" target="_blank">Symfony Form</a> — Provides a flexible way to map request data to the entity object.

<a href="https://github.com/FriendsOfSymfony/FOSRestBundle" target="_blank">FOSRestBundle</a> and <a href="https://github.com/nelmio/NelmioApiDocBundle" target="_blank">NelmioApiDocBundle</a> are also used for REST API.

#### NOTE
The main format for REST API is described at <a href="http://jsonapi.org/" target="_blank">JSON:API</a>. Please make sure that you are familiar with it before creating REST API for your entities.

The auto-generated documentation and sandbox for REST API is available in the /api/doc, e.g., <a href="http://demo.orocrm.com/api/doc" target="_blank">http://demo.orocrm.com/api/doc</a>.

By default, only custom entities, dictionaries, and enumerations are accessible through the back-office API. For how to make other entities available via the API, see [Configuration Reference](configuration.md#web-api-configuration).

* [CLI Commands](commands.md)
* [Configure Stateless Security Firewalls](security.md)
* [Configure Feature Depended Firewall Authenticators](firewall-authenticators.md)
* [General Configuration](configuration-general.md)
* [Configuration Reference](configuration.md)
* [Configuration Extras](configuration-extra.md)
* [Configuration Extensions](configuration-extensions.md)
* [Forms and Validators Configuration](forms.md)
* [Documenting API Resources](documentation.md)
* [Actions](actions.md)
* [Request Type](request-type.md)
* [Processors](processors.md)
* [Headers](headers.md)
* [Filters](filters.md)
* [Post Processors](post-processors.md)
* [How to](how-to.md)
* [CORS Configuration](cors.md)
* [CORS Configuration for Published OpenAPI Specifications](cors-open-api.md)
* [Testing REST API](testing.md)
* [Storefront REST API](storefront.md)
* [Storefront Routes](storefront-routes.md)
* [Batch API](batch-api.md)

<!-- Frontend -->
