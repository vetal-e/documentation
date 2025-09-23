<a id="web-api-actions"></a>

# Actions

The action is a set of processors that handle a request.

Each action has two required elements:

- **context** -  An object that stores the input and output data and shares data between processors.
- **main processor** - The main entry point for an action. This class is responsible for creating the context and executing all worker processors.

For more details about these elements, see the [Creating a New Action]() section.

The following table shows all actions provided out-of-the-box:

| Action Name                                            | Description                                                                                                                                                                                                                                            |
|--------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| [get](#get-action)                                     | Returns an entity by its identifier.                                                                                                                                                                                                                   |
| [get_list](#get-list-action)                           | Returns a list of entities.                                                                                                                                                                                                                            |
| [delete](#delete-action)                               | Deletes an entity by its identifier.                                                                                                                                                                                                                   |
| [delete_list](#delete-list-action)                     | Deletes a list of entities.                                                                                                                                                                                                                            |
| [create](#create-action)                               | Creates a new entity.                                                                                                                                                                                                                                  |
| [update](#update-action)                               | Updates an existing entity.                                                                                                                                                                                                                            |
| [update_list](#update-list-action)                     | Updates a list of entities of the same type.                                                                                                                                                                                                           |
| [get_subresource](#get-subresource-action)             | Returns a list of related entities represented by a relationship.                                                                                                                                                                                      |
| [update_subresource](#update-subresource-action)       | Updates an entity or entities (depending on the association type) connected to<br/>an entity to which the sub-resource belongs. This action does not have the default<br/>implementation, additional processors should be added for each sub-resource  |
| [add_subresource](#add-subresource-action)             | Adds an entity or entities (depending on the association type) connected to<br/>an entity to which the sub-resource belongs. This action does not have a default<br/>implementation, additional processors should be added for each sub-resource.      |
| [delete_subresource](#delete-subresource-action)       | Deletes an entity or entities (depending on the association type) connected to<br/>an entity to which the sub-resource belongs. This action does not have the default<br/>implementation, additional processors should be added for each sub-resource. |
| [get_relationship](#get-relationship-action)           | Returns relationship data.                                                                                                                                                                                                                             |
| [update_relationship](#update-relationship-action)     | Updates the “to-one” relationship and completely replaces all members of the<br/>“to-many” relationship.                                                                                                                                               |
| [add_relationship](#add-relationship-action)           | Adds one or several entities to a relationship. This action is applicable only to<br/>“to-many” relationships.                                                                                                                                         |
| [delete_relationship](#delete-relationship-action)     | Deletes one or several entities from a relationship. This action is applicable<br/>only to “to-many” relationships.                                                                                                                                    |
| [options](#options-action)                             | Returns communication options for a resource.                                                                                                                                                                                                          |
| [customize_loaded_data](#customize-loaded-data-action) | Makes modifications to the data loaded by the **get**, **get_list** and<br/>**get_subresource** actions.                                                                                                                                               |
| [customize_form_data](#customize-form-data-action)     | Makes modifications to and validation of the submitted data, and entities to be<br/>persisted into the database by the **create**, **update**, **update_relationship**,<br/>**add_relationship** and **delete_relationship** actions.                  |
| [get_config](#get-config-action)                       | Returns the configuration of an entity.                                                                                                                                                                                                                |
| [get_metadata](#get-metadata-action)                   | Returns metadata of an entity.                                                                                                                                                                                                                         |
| [normalize_value](#normalize-value-action)             | Converts a value to a requested data type.                                                                                                                                                                                                             |
| [collect_resources](#collect-resources-action)         | Returns a list of all resources accessible through the API.                                                                                                                                                                                            |
| [collect_subresources](#collect-subresources-action)   | Returns a list of all sub-resources accessible through the API for a given entity type.                                                                                                                                                                |
| [not_allowed](#not-allowed-action)                     | Builds a response for the case when a request does not match any public action, e.g.,<br/>when the HTTP method is not supported for the REST API request.                                                                                              |
| [unhandled_error](#unhandled-error-action)             | Builds a response for the case when an unexpected error happens before any public<br/>action is started.                                                                                                                                               |
| [batch_update](#batch-update-action)                   | Used by the **update_list** action to update/create a set of entities of the same type.                                                                                                                                                                |
| [batch_update_item](#batch-update-item-action)         | Used by the **batch_update** action to update/create an entity that is part of<br/>a batch operation.                                                                                                                                                  |

See the [Processors](processors.md#web-api-processors) section for more details on creating a processor.

You can use the [oro:api:debug](commands.md#oroapidebug) command to display the list of all available actions and processors.

<a id="web-api-actions-public-actions"></a>

## Public Actions

<a id="get-action"></a>

### get Action

This action is intended to retrieve an entity by its identifier. For more details, see the <a href="http://jsonapi.org/format/#fetching" target="_blank">Fetching Data</a> section of the JSON:API specification.

The route name for REST API: `oro_rest_api_item`.

The URL template for REST API: `/api/{entity}/{id}`.

The HTTP method for REST API: `GET`.

The context class: <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Processor/Get/GetContext.php" target="_blank">GetContext</a>. Also see [Context](#context-class) class for more details.

The main processor class: <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Processor/GetProcessor.php" target="_blank">GetProcessor</a>.

Existing worker processors: <a href="https://github.com/oroinc/platform/tree/master/src/Oro/Bundle/ApiBundle/Resources/config/processors.get.yml" target="_blank">processors.get.yml</a>, <a href="https://github.com/oroinc/platform/tree/master/src/Oro/Bundle/ApiBundle/Resources/config/processors.shared.yml" target="_blank">processors.shared.yml</a>. Run `php bin/console oro:api:debug get` to display the list of processors.

This action has the following processor groups:

| Group Name          | Responsibility of Processors                                                       | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
|---------------------|------------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| initialize          | Context initialization.                                                            | Processors from this group are executed during the generation of the API documentation.                                                                                                                                                                                                                                                                                                                                                                                |
| resource_check      | Checking whether the requested resource type is accessible via API.                | –                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| normalize_input     | Preparing the input data to be used by processors from the next groups.            | –                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| security_check      | Checking whether access to the requested resource type is granted.                 | To add a new processor to this group, include it to the **security_check** group of actions that execute this action. For example, compare with the **security_check** group of the [create](#create-action) or [update](#update-action) actions.                                                                                                                                                                                                                      |
| build_query         | Building a query required to load the data.                                        | –                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| load_data           | Loading data.                                                                      | Use the [customize_loaded_data](#customize-loaded-data-action) action to modify loaded data.                                                                                                                                                                                                                                                                                                                                                                           |
| data_security_check | Checking whether access to the loaded data is granted.                             | Use the same rules as for the **security_check** group to add a new processor to this group.                                                                                                                                                                                                                                                                                                                                                                           |
| normalize_data      | Converting the loaded data into an array.                                          | In most cases, the processors from this group are skipped because the <a href="https://github.com/oroinc/platform/tree/master/src/Oro/Component/EntitySerializer" target="_blank">EntitySerializer</a> loads most entities and returns already normalized data. For details, see <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Processor/Shared/LoadEntityByEntitySerializer.php" target="_blank">LoadEntityByEntitySerializer</a>. |
| finalize            | Final validation of the loaded data and addition of the required response headers. | –                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| normalize_result    | Building the action result.                                                        | The processors from this group are executed even if a processor from previous groups throws an exception. For implementation details, see <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Processor/NormalizeResultActionProcessor.php" target="_blank">NormalizeResultActionProcessor</a>.                                                                                                                                           |

The following diagram shows the main data flow for this action:

![Data flow for the get action](img/backend/api/get.png)

See the `handleGet` method of <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Request/RequestActionHandler.php" target="_blank">RequestActionHandler</a> as an example.

<a id="get-list-action"></a>

### get_list Action

This action retrieves a list of entities. For more details, see the <a href="http://jsonapi.org/format/#fetching" target="_blank">Fetching Data</a> section of the JSON:API specification.

The route name for REST API: `oro_rest_api_list`.

The URL template for REST API: `/api/{entity}`.

The HTTP method for REST API: `GET`.

The context class: <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Processor/GetList/GetListContext.php" target="_blank">GetListContext</a>. Also see [Context](#context-class) class for more details.

The main processor class: <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Processor/GetListProcessor.php" target="_blank">GetListProcessor</a>.

Existing worker processors: <a href="https://github.com/oroinc/platform/tree/master/src/Oro/Bundle/ApiBundle/Resources/config/processors.get_list.yml" target="_blank">processors.get_list.yml</a>, <a href="https://github.com/oroinc/platform/tree/master/src/Oro/Bundle/ApiBundle/Resources/config/processors.shared.yml" target="_blank">processors.shared.yml</a>. Run `php bin/console oro:api:debug get_list` to list the processors.

This action has the following processor groups:

| Group Name          | Responsibility of Processors                                              | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
|---------------------|---------------------------------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| initialize          | Context initialization.                                                   | Processors from this group are executed during the generation of the API documentation.                                                                                                                                                                                                                                                                                                                                                                                   |
| resource_check      | Checking whether the requested resource type is accessible via API.       | –                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| normalize_input     | Preparing the input data to be used by processors from the next groups.   | –                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| security_check      | Checking whether access to the requested resource type is granted.        | –                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| build_query         | Building a query required to load data.                                   | –                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| load_data           | Loading data.                                                             | Use the [customize_loaded_data](#customize-loaded-data-action) action to modify loaded data.                                                                                                                                                                                                                                                                                                                                                                              |
| data_security_check | Checking whether access to the loaded data is granted.                    | –                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| normalize_data      | Converting the loaded data into an array.                                 | In most cases, the processors from this group are skipped because the <a href="https://github.com/oroinc/platform/tree/master/src/Oro/Component/EntitySerializer" target="_blank">EntitySerializer</a> loads most entities and returns already normalized data. For details see <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Processor/Shared/LoadEntitiesByEntitySerializer.php" target="_blank">LoadEntitiesByEntitySerializer</a>. |
| finalize            | Final validation of the loaded data and adding required response headers. | –                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| normalize_result    | Building the action result.                                               | The processors from this group are executed even if a processor from one of the previous groups throws an exception. For implementation details, see <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Processor/NormalizeResultActionProcessor.php" target="_blank">NormalizeResultActionProcessor</a>.                                                                                                                                   |

The following diagram shows the main data flow for this action:

![Data flow for get_list action](img/backend/api/get_list.png)

See the `handleGetList` method of <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Request/RequestActionHandler.php" target="_blank">RequestActionHandler</a> as an example.

<a id="delete-action"></a>

### delete Action

This action deletes an entity by its identifier. For more details, see the <a href="http://jsonapi.org/format/#crud-deleting" target="_blank">Deleting Resources</a> section of the JSON:API specification.

The route name for REST API: `oro_rest_api_item`.

The URL template for REST API: `/api/{entity}/{id}`.

The HTTP method for REST API: `DELETE`.

The context class: <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Processor/Delete/DeleteContext.php" target="_blank">DeleteContext</a>. Also see [Context](#context-class) class for more details.

The main processor class: <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Processor/DeleteProcessor.php" target="_blank">DeleteProcessor</a>.

Existing worker processors: <a href="https://github.com/oroinc/platform/tree/master/src/Oro/Bundle/ApiBundle/Resources/config/processors.delete.yml" target="_blank">processors.delete.yml</a>, <a href="https://github.com/oroinc/platform/tree/master/src/Oro/Bundle/ApiBundle/Resources/config/processors.shared.yml" target="_blank">processors.shared.yml</a>. Run `php bin/console oro:api:debug delete` to list the processors.

This action has the following processor groups:

| Group Name          | Responsibility of Processors                                            | Description                                                                                                                                                                                                                                                                                                                    |
|---------------------|-------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| initialize          | Context initialization.                                                 | Processors from this group are executed during the generation of the API documentation.                                                                                                                                                                                                                                        |
| resource_check      | Checking whether the requested resource type is accessible via API.     | –                                                                                                                                                                                                                                                                                                                              |
| normalize_input     | Preparing the input data to be used by processors from the next groups. | –                                                                                                                                                                                                                                                                                                                              |
| security_check      | Checking whether access to the requested resource type is granted.      | –                                                                                                                                                                                                                                                                                                                              |
| load_data           | Loading an entity to be deleted.                                        | –                                                                                                                                                                                                                                                                                                                              |
| data_security_check | Checking whether access to the loaded data is granted.                  | –                                                                                                                                                                                                                                                                                                                              |
| delete_data         | Deleting an entity.                                                     | –                                                                                                                                                                                                                                                                                                                              |
| finalize            | Adding required response headers.                                       | –                                                                                                                                                                                                                                                                                                                              |
| normalize_result    | Building the action result.                                             | The processors from this group are executed even if any processor from previous groups throws an exception. For implementation details, see <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Processor/NormalizeResultActionProcessor.php" target="_blank">NormalizeResultActionProcessor</a>. |

The following diagram shows the main data flow for this action:

![Data flow for delete action](img/backend/api/delete.png)

See the `handleDelete` method of <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Request/RequestActionHandler.php" target="_blank">RequestActionHandler</a> as an example.

<a id="delete-list-action"></a>

### delete_list Action

This action deletes a list of entities.

The entities list is built based on input filters. You must specify at least one filter, otherwise, you will get an error.

By default, the maximum number of entities that can be deleted by one request is 100, see the `max_delete_entities` option in [General Configuration](configuration-general.md#web-api-configuration-general). This limit minimizes the impact on the server. You can change this limit for an entity in Resources/config/oro/api.yml but test it carefully as a higher limit can significantly impact the server. See an example of changing the default limit in the [How To](how-to.md#max-number-of-entities-to-be-deleted) topic.

The route name for REST API: `oro_rest_api_list`.

The URL template for REST API: `/api/{entity}`.

The HTTP method for REST API: `DELETE`.

The context class: <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Processor/DeleteList/DeleteListContext.php" target="_blank">DeleteListContext</a>. See the [Context](#context-class) class for more details.

The main processor class: <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Processor/DeleteListProcessor.php" target="_blank">DeleteListProcessor</a>.

Existing worker processors: <a href="https://github.com/oroinc/platform/tree/master/src/Oro/Bundle/ApiBundle/Resources/config/processors.delete_list.yml" target="_blank">processors.delete_list.yml</a>, <a href="https://github.com/oroinc/platform/tree/master/src/Oro/Bundle/ApiBundle/Resources/config/processors.shared.yml" target="_blank">processors.shared.yml</a>. Run `php bin/console oro:api:debug delete_list` to display the list of processors.

This action has the following processor groups:

| Group Name          | Responsibility of Processors                                            | Description                                                                                                                                                                                                                                                                                                                  |
|---------------------|-------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| initialize          | Context initialization.                                                 | Processors from this group are executed during the generation of the API documentation.                                                                                                                                                                                                                                      |
| resource_check      | Checking whether the requested resource type is accessible via API.     | –                                                                                                                                                                                                                                                                                                                            |
| normalize_input     | Preparing the input data to be used by processors from the next groups. | –                                                                                                                                                                                                                                                                                                                            |
| security_check      | Checking whether access to the requested resource type is granted.      | –                                                                                                                                                                                                                                                                                                                            |
| build_query         | Building a query that loads a list of entities to be deleted.           | –                                                                                                                                                                                                                                                                                                                            |
| load_data           | Loading the list of entities to be deleted.                             | –                                                                                                                                                                                                                                                                                                                            |
| data_security_check | Checking whether access to the loaded data is granted.                  | –                                                                                                                                                                                                                                                                                                                            |
| delete_data         | Deleting the list of entities.                                          | –                                                                                                                                                                                                                                                                                                                            |
| finalize            | Adding required response headers.                                       | –                                                                                                                                                                                                                                                                                                                            |
| normalize_result    | Building the action result.                                             | The processors from this group are executed even if a processor from previous groups throws an exception. For implementation details, see <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Processor/NormalizeResultActionProcessor.php" target="_blank">NormalizeResultActionProcessor</a>. |

The following diagram shows the main data flow for this action:

![Data flow for delete_list action.](img/backend/api/get_list.png)

See the `handleDeleteList` method of <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Request/RequestActionHandler.php" target="_blank">RequestActionHandler</a> as an example.

<a id="create-action"></a>

### create Action

This action creates a new entity. For more details, see the <a href="http://jsonapi.org/format/#crud-creating" target="_blank">Creating Resources</a> section of the JSON:API specification.

The route name for REST API: `oro_rest_api_list`.

The URL template for REST API: `/api/{entity}`.

The HTTP method for REST API: `POST`.

The context class: <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Processor/Create/CreateContext.php" target="_blank">CreateContext</a>. See [Context](#context-class) class for more details.

The main processor class: <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Processor/CreateProcessor.php" target="_blank">CreateProcessor</a>.

Existing worker processors: <a href="https://github.com/oroinc/platform/tree/master/src/Oro/Bundle/ApiBundle/Resources/config/processors.create.yml" target="_blank">processors.create.yml</a>, <a href="https://github.com/oroinc/platform/tree/master/src/Oro/Bundle/ApiBundle/Resources/config/processors.shared.yml" target="_blank">processors.shared.yml</a>. Run `php bin/console oro:api:debug create` to display the list of processors.

This action has the following processor groups:

| Group Name          | Responsibility of Processors                                                     | Description                                                                                                                                                                                                                                                                                                                                                                                  |
|---------------------|----------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| initialize          | Context initialization.                                                          | Processors from this group are executed during the generation of the API documentation.                                                                                                                                                                                                                                                                                                      |
| resource_check      | Checking whether the requested resource type is accessible via API.              | –                                                                                                                                                                                                                                                                                                                                                                                            |
| normalize_input     | Preparing input data to be used by processors from the next groups.              | –                                                                                                                                                                                                                                                                                                                                                                                            |
| security_check      | Checking whether access to the requested resource type is granted.               | If you add your own security processor in the **security_check** group of the [get](#get-action) action, add it to this group as well. It is required because the **VIEW** permission is checked here: the created entity should be returned in the response, and the **security_check** group of the [get](#get-action) action is disabled by **oro_api.load_normalized_entity** processor. |
| load_data           | Creating a new entity object.                                                    | –                                                                                                                                                                                                                                                                                                                                                                                            |
| data_security_check | Checking whether access to the loaded data is granted.                           | Use the same rules as for the **security_check** group to add a new processor to this group.                                                                                                                                                                                                                                                                                                 |
| transform_data      | Building a Symfony Form and using it to transform and validate the request data. | –                                                                                                                                                                                                                                                                                                                                                                                            |
| save_data           | Persisting an entity.                                                            | Use the [customize_form_data](#customize-form-data-action) action to modify and validate submitted data and entities to be persisted into the database.                                                                                                                                                                                                                                      |
| normalize_data      | Converting created entity into array.                                            | –                                                                                                                                                                                                                                                                                                                                                                                            |
| finalize            | Adding required response headers.                                                | –                                                                                                                                                                                                                                                                                                                                                                                            |
| normalize_result    | Building the action result.                                                      | The processors from this group are executed even if a processor from previous groups throws an exception. For implementation details, see <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Processor/NormalizeResultActionProcessor.php" target="_blank">NormalizeResultActionProcessor</a>. Processors from this group are not executed for 4.              |

The following diagram shows the main data flow for this action:

![Data flow for the create action](img/backend/api/create.png)

See the `handleCreate` method of <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Request/RequestActionHandler.php" target="_blank">RequestActionHandler</a> as an example.

<a id="update-action"></a>

### update Action

This action updates an entity. For more details, see the <a href="http://jsonapi.org/format/#crud-updating" target="_blank">Updating Resources</a> section of the JSON:API specification.

The route name for REST API: `oro_rest_api_item`.

The URL template for REST API: `/api/{entity}/{id}`.

The HTTP method for REST API: `PATCH`.

The context class: <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Processor/Update/UpdateContext.php" target="_blank">UpdateContext</a>. Also see [Context](#context-class) class for more details.

The main processor class: <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Processor/UpdateProcessor.php" target="_blank">UpdateProcessor</a>.

Existing worker processors: <a href="https://github.com/oroinc/platform/tree/master/src/Oro/Bundle/ApiBundle/Resources/config/processors.update.yml" target="_blank">processors.update.yml</a>, <a href="https://github.com/oroinc/platform/tree/master/src/Oro/Bundle/ApiBundle/Resources/config/processors.shared.yml" target="_blank">processors.shared.yml</a>. Run `php bin/console oro:api:debug update` to display the list of processors.

This action has the following processor groups:

| Group Name          | Responsibility of Processors                                                     | Description                                                                                                                                                                                                                                                                                                                                                                               |
|---------------------|----------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| initialize          | Context initialization.                                                          | Processors from this group are executed during the generation of the API documentation.                                                                                                                                                                                                                                                                                                   |
| resource_check      | Checking whether the requested resource type is accessible via API.              | –                                                                                                                                                                                                                                                                                                                                                                                         |
| normalize_input     | Preparing the input data to be used by processors from the next groups.          | –                                                                                                                                                                                                                                                                                                                                                                                         |
| security_check      | Checking whether access to the requested resource is granted.                    | When you add a new processor to the **security_check** group of the [get](#get-action) action, add it to this group as well. This is necessary because the **VIEW** permission is checked here: the updated entity should be returned in the response, and the **security_check** group of the [get](#get-action) action is disabled by the **oro_api.load_normalized_entity** processor. |
| load_data           | Loading an entity object to be updated.                                          | –                                                                                                                                                                                                                                                                                                                                                                                         |
| data_security_check | Checking whether access to the loaded data is granted.                           | Use the same rules as for the **security_check** group to add a new processor to this group.                                                                                                                                                                                                                                                                                              |
| transform_data      | Building a Symfony Form and using it to transform and validate the request data. | –                                                                                                                                                                                                                                                                                                                                                                                         |
| save_data           | Persisting an entity.                                                            | Use the [customize_form_data](#customize-form-data-action) action to modify and validate submitted data, and entities to be persisted into the database.                                                                                                                                                                                                                                  |
| normalize_data      | Converting updated entity into an array.                                         | –                                                                                                                                                                                                                                                                                                                                                                                         |
| finalize            | Adding the required response headers.                                            | –                                                                                                                                                                                                                                                                                                                                                                                         |
| normalize_result    | Building the action result.                                                      | The processors from this group are executed even if a processor from previous groups throws an exception. For implementation details, see <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Processor/NormalizeResultActionProcessor.php" target="_blank">NormalizeResultActionProcessor</a>.                                                              |

The following diagram shows the main data flow for this action:

![Data flow for update action](img/backend/api/update.png)

See the `handleUpdate` method of <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Request/RequestActionHandler.php" target="_blank">RequestActionHandler</a> as an example.

<a id="update-list-action"></a>

### update_list Action

This action is intended to create or update the list of entities of the same type.

The action works as an asynchronous operation. The result of this action is the initial status of the created
asynchronous operation and the `Content-Location` response header that contains an URL of API resource
of this operation.

The action is disabled by default.
See [Batch API documentation](batch-api.md#web-api-batch-api-enable) for details on enabling it for an API resource.

The route name for REST API: `oro_rest_api_list`.

The URL template for REST API: `/api/{entity}`.

The HTTP method for REST API: `PATCH`.

The context class: <a href="https://github.com/oroinc/platform/tree/master/src/Oro/Bundle/ApiBundle/Processor/UpdateList/UpdateListContext.php" target="_blank">UpdateListContext</a>. See [Context](#context-class) class for more details.

The main processor class: <a href="https://github.com/oroinc/platform/tree/master/src/Oro/Bundle/ApiBundle/Processor/UpdateListProcessor.php" target="_blank">UpdateListProcessor</a>.

Existing worker processors: <a href="https://github.com/oroinc/platform/tree/master/src/Oro/Bundle/ApiBundle/Resources/config/processors.update_list.yml" target="_blank">processors.update_list.yml</a>, <a href="https://github.com/oroinc/platform/tree/master/src/Oro/Bundle/ApiBundle/Resources/config/processors.shared.yml" target="_blank">processors.shared.yml</a>. Run `php bin/console oro:api:debug update_list` to display the list of processors.

This action has the following processor groups:

| Group Name       | Responsibility of Processors                                            | Description                                                                                                                                                                                                                                                                                                                                                                               |
|------------------|-------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| initialize       | Context initialization                                                  | Processors from this group are executed during the generation of the API documentation.                                                                                                                                                                                                                                                                                                   |
| resource_check   | Checking whether the requested resource type is accessible via API.     | –                                                                                                                                                                                                                                                                                                                                                                                         |
| normalize_input  | Preparing the input data to be used by processors from the next groups. | –                                                                                                                                                                                                                                                                                                                                                                                         |
| security_check   | Checking whether access to the requested resource is granted.           | When you add a new processor to the **security_check** group of the [get](#get-action) action, add it to this group as well. This is necessary because the **VIEW** permission is checked here: the updated entity should be returned in the response, and the **security_check** group of the [get](#get-action) action is disabled by the **oro_api.load_normalized_entity** processor. |
| load_data        | Loading a request data to the storage.                                  | –                                                                                                                                                                                                                                                                                                                                                                                         |
| save_data        | Creating an asynchronous batch operation.                               | –                                                                                                                                                                                                                                                                                                                                                                                         |
| finalize         | Adding the required response headers.                                   | –                                                                                                                                                                                                                                                                                                                                                                                         |
| normalize_result | Building the action result.                                             | The processors from this group are executed even if a processor from previous groups throws an exception. For implementation details, see <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Processor/NormalizeResultActionProcessor.php" target="_blank">NormalizeResultActionProcessor</a>.                                                              |

The following diagram shows the main data flow for this action:

![Data flow for update_list action](img/backend/api/update_list.png)

See the `handleUpdateList` method of <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Request/RequestActionHandler.php" target="_blank">RequestActionHandler</a> as an example.

<a id="get-subresource-action"></a>

### get_subresource Action

This action retrieves an entity (for the “to-one” relationship) or a list of entities (for the “to-many” relationship) connected to the entity by a given association. For more details, see the <a href="http://jsonapi.org/format/#fetching-resources" target="_blank">Fetching Resources</a> section of the JSON:API specification.

The route name for REST API: `oro_rest_api_subresource`.

The URL template for REST API: `/api/{entity}/{id}/{association}`.

The HTTP method for REST API: `GET`.

The context class: <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Processor/Subresource/GetSubresource/GetSubresourceContext.php" target="_blank">GetSubresourceContext</a>. Also see [SubresourceContext](#subresourcecontext-class) class for more details.

The main processor class: <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Processor/Subresource/GetSubresourceProcessor.php" target="_blank">GetSubresourceProcessor</a>.

Existing worker processors: <a href="https://github.com/oroinc/platform/tree/master/src/Oro/Bundle/ApiBundle/Resources/config/processors.get_subresource.yml" target="_blank">processors.get_subresource.yml</a>,  <a href="https://github.com/oroinc/platform/tree/master/src/Oro/Bundle/ApiBundle/Resources/config/processors.shared.yml" target="_blank">processors.shared.yml</a>. Run `php bin/console oro:api:debug get_subresource` to display the list of processors.

This action has the following processor groups:

| Group Name          | Responsibility of Processors                                                 | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
|---------------------|------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| initialize          | Context initialization.                                                      | Processors from this group are executed during the generation of the API documentation.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| resource_check      | Checking whether the requested resource type is accessible via API.          | –                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| normalize_input     | Preparing the input data to be used by processors from the next groups.      | –                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| security_check      | Checking whether access to the requested resource type is granted.           | –                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| build_query         | Building a query to use to load data.                                        | –                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| load_data           | Loading data.                                                                | Use the [customize_loaded_data](#customize-loaded-data-action) action to modify loaded data.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| data_security_check | Checking whether access to the loaded data is granted.                       | –                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| normalize_data      | Converting the loaded data into an array.                                    | In most cases, the processors from this group are skipped because the <a href="https://github.com/oroinc/platform/tree/master/src/Oro/Component/EntitySerializer" target="_blank">EntitySerializer</a> loads most entities and returns already normalized data. For details see <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Processor/Shared/LoadEntityByEntitySerializer.php" target="_blank">LoadEntityByEntitySerializer</a> and <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Processor/Shared/LoadEntitiesByEntitySerializer.php" target="_blank">LoadEntitiesByEntitySerializer</a>. |
| finalize            | Final validation of the loaded data and adding the required response headers | –                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| normalize_result    | Building the action result.                                                  | The processors from this group are executed even if a processor from previous groups throws an exception. For implementation details, see <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Processor/NormalizeResultActionProcessor.php" target="_blank">NormalizeResultActionProcessor</a>.                                                                                                                                                                                                                                                                                                                                       |

The following diagram shows the main data flow for this action:

![Data flow for get_subresource action](img/backend/api/get_subresource.png)

See the `handleGetSubresource` method of <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Request/RequestActionHandler.php" target="_blank">RequestActionHandler</a> as an example.

<a id="update-subresource-action"></a>

### update_subresource Action

Updates an entity or entities (depending on the association type) connected to an entity to which the sub-resource belongs. As this action does not have a default implementation, additional processors should be added, at least a processor that will build a form builder for your sub-resource. Take a look at <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Processor/Subresource/ChangeSubresource/BuildFormBuilder.php" target="_blank">BuildFormBuilder</a> and <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Processor/Subresource/ChangeSubresource/BuildCollectionFormBuilder.php" target="_blank">BuildCollectionFormBuilder</a> as examples of such processors. Use the [request_target_class](configuration.md#subresources-config) configuration option when request data are different from response data. Use the [request_documentation_action](configuration.md#subresources-config) configuration option when the request data documentation is different from the response data documentation.

The route name for REST API: `oro_rest_api_subresource`.

The URL template for REST API: `/api/{entity}/{id}/{association}`.

The HTTP method for REST API: `PATCH`.

The context class: <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Processor/Subresource/ChangeSubresourceContext.php" target="_blank">ChangeSubresourceContext</a>. See the [SubresourceContext](#subresourcecontext-class) class for more details.

The main processor class: <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Processor/Subresource/ChangeSubresourceProcessor.php" target="_blank">ChangeSubresourceProcessor</a>.

Existing worker processors: <a href="https://github.com/oroinc/platform/tree/master/src/Oro/Bundle/ApiBundle/Resources/config/processors.change_subresource.yml" target="_blank">processors.change_subresource.yml</a>, <a href="https://github.com/oroinc/platform/tree/master/src/Oro/Bundle/ApiBundle/Resources/config/processors.shared.yml" target="_blank">processors.shared.yml</a>. Run `php bin/console oro:api:debug update_subresource` to display the list of processors.

This action has the following processor groups:

| Group Name          | Responsibility of Processors                                                     | Description                                                                                                                                                                                                                                                                                                                  |
|---------------------|----------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| initialize          | Context initialization.                                                          | Processors from this group are executed during the generation of the API documentation.                                                                                                                                                                                                                                      |
| resource_check      | Checking whether the requested resource type is accessible via API.              | –                                                                                                                                                                                                                                                                                                                            |
| normalize_input     | Preparing the input data to be used by processors from the next groups.          | –                                                                                                                                                                                                                                                                                                                            |
| security_check      | Checking whether access to the requested resource type is granted.               | –                                                                                                                                                                                                                                                                                                                            |
| load_data           | Loading an entity object to be updated.                                          | –                                                                                                                                                                                                                                                                                                                            |
| data_security_check | Checking whether access to the loaded data is granted.                           | –                                                                                                                                                                                                                                                                                                                            |
| transform_data      | Building a Symfony Form and using it to transform and validate the request data. | –                                                                                                                                                                                                                                                                                                                            |
| save_data           | Persisting an entity.                                                            | Use the [customize_form_data](#customize-form-data-action) action to modify and validate submitted data, and entities to be persisted into the database.                                                                                                                                                                     |
| normalize_data      | Converting the resulting entity into an array.                                   | –                                                                                                                                                                                                                                                                                                                            |
| finalize            | Final validation of the loaded data. Adding the required response headers.       | –                                                                                                                                                                                                                                                                                                                            |
| normalize_result    | Building the action result.                                                      | The processors from this group are executed even if a processor from previous groups throws an exception. For implementation details, see <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Processor/NormalizeResultActionProcessor.php" target="_blank">NormalizeResultActionProcessor</a>. |

The following diagram shows the main data flow for this action:

![Data flow for update_subresource action](img/backend/api/update_subresource.png)

See the `handleUpdateSubresource` method of <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Request/RequestActionHandler.php" target="_blank">RequestActionHandler</a> as an example.

Below is an example of registering a processor to build a form builder:

```yaml
acme.api.items.build_form_builder:
    class: Oro\Bundle\ApiBundle\Processor\Subresource\ChangeSubresource\BuildFormBuilder
    arguments:
        - '@oro_api.form_helper'
    tags:
        - { name: oro.api.processor, action: update_subresource, group: transform_data, association: items, parentClass: Acme\Bundle\DemoBundle\Entity\ShoppingList, priority: 100 }
```

<a id="add-subresource-action"></a>

### add_subresource Action

Adds an entity or entities (depending on the association type) connected to an entity to which the sub-resource belongs. As this action does not have a default implementation, additional processors should be added, at least a processor that will build a form builder for your sub-resource. Take a look at <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Processor/Subresource/ChangeSubresource/BuildFormBuilder.php" target="_blank">BuildFormBuilder</a> and <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Processor/Subresource/ChangeSubresource/BuildCollectionFormBuilder.php" target="_blank">BuildCollectionFormBuilder</a> as examples of such processors. Use the [request_target_class](configuration.md#subresources-config) configuration option when request data are different from response data. Use the [request_documentation_action](configuration.md#subresources-config) configuration option when the request data documentation is different from the response data documentation.

The route name for REST API: `oro_rest_api_subresource`.

The URL template for REST API: `/api/{entity}/{id}/{association}`.

The HTTP method for REST API: `POST`.

The context class: <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Processor/Subresource/ChangeSubresourceContext.php" target="_blank">ChangeSubresourceContext</a>. See the [SubresourceContext](#subresourcecontext-class) class for more details.

The main processor class: <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Processor/Subresource/ChangeSubresourceProcessor.php" target="_blank">ChangeSubresourceProcessor</a>.

Existing worker processors: <a href="https://github.com/oroinc/platform/tree/master/src/Oro/Bundle/ApiBundle/Resources/config/processors.change_subresource.yml" target="_blank">processors.change_subresource.yml</a>, <a href="https://github.com/oroinc/platform/tree/master/src/Oro/Bundle/ApiBundle/Resources/config/processors.shared.yml" target="_blank">processors.shared.yml</a>. Run `php bin/console oro:api:debug add_subresource` to display the list of processors.

This action has the following processor groups:

| Group Name          | Responsibility of Processors                                                     | Description                                                                                                                                                                                                                                                                                                                  |
|---------------------|----------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| initialize          | Context initialization.                                                          | Processors from this group are executed during the generation of the API documentation.                                                                                                                                                                                                                                      |
| resource_check      | Checking whether the requested resource type is accessible via API.              | –                                                                                                                                                                                                                                                                                                                            |
| normalize_input     | Preparing the input data to be used by processors from the next groups.          | –                                                                                                                                                                                                                                                                                                                            |
| security_check      | Checking whether access to the requested resource type is granted.               | –                                                                                                                                                                                                                                                                                                                            |
| load_data           | Loading an entity object to be updated.                                          | –                                                                                                                                                                                                                                                                                                                            |
| data_security_check | Checking whether access to the loaded data is granted.                           | –                                                                                                                                                                                                                                                                                                                            |
| transform_data      | Building a Symfony Form and using it to transform and validate the request data. | –                                                                                                                                                                                                                                                                                                                            |
| save_data           | Persisting an entity.                                                            | Use the [customize_form_data](#customize-form-data-action) action to modify and validate submitted data, and entities to be persisted into the database.                                                                                                                                                                     |
| normalize_data      | Converting the resulting entity into an array.                                   | –                                                                                                                                                                                                                                                                                                                            |
| finalize            | Adding the required response headers.                                            | –                                                                                                                                                                                                                                                                                                                            |
| normalize_result    | Building the action result.                                                      | The processors from this group are executed even if a processor from previous groups throws an exception. For implementation details, see <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Processor/NormalizeResultActionProcessor.php" target="_blank">NormalizeResultActionProcessor</a>. |

The following diagram shows the main data flow for this action:

![Data flow for add_subresource actions](img/backend/api/add_subresource.png)

See the `handleAddSubresource` method of <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Request/RequestActionHandler.php" target="_blank">RequestActionHandler</a> as an example.

Below is an example of registering a processor to build a form builder:

```yaml
acme.api.items.build_form_builder:
    class: Oro\Bundle\ApiBundle\Processor\Subresource\ChangeSubresource\BuildFormBuilder
    arguments:
        - '@oro_api.form_helper'
    tags:
        - { name: oro.api.processor, action: add_subresource, group: transform_data, association: items, parentClass: Acme\Bundle\DemoBundle\Entity\ShoppingList, priority: 100 }
```

<a id="delete-subresource-action"></a>

### delete_subresource Action

Deletes an entity or entities (depending on the association type) connected to an entity to which the sub-resource belongs. As this action does not have a default implementation, additional processors should be added, at least a processor that will build a form builder for your sub-resource. Take a look at BuildFormBuilder and BuildCollectionFormBuilder as examples of such processors. Use the [request_target_class](configuration.md#subresources-config) configuration option when request data are different from response data. Use the [request_documentation_action](configuration.md#subresources-config) configuration option when the request data documentation is different from the response data documentation.

The route name for REST API: `oro_rest_api_subresource`.

The URL template for REST API: `/api/{entity}/{id}/{association}`.

The HTTP method for REST API: `DELETE`.

The context class: <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Processor/Subresource/ChangeSubresourceContext.php" target="_blank">ChangeSubresourceContext</a>. See the [SubresourceContext](#subresourcecontext-class) class for more details.

The main processor class: <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Processor/Subresource/ChangeSubresourceProcessor.php" target="_blank">ChangeSubresourceProcessor</a>.

Existing worker processors: <a href="https://github.com/oroinc/platform/tree/master/src/Oro/Bundle/ApiBundle/Resources/config/processors.change_subresource.yml" target="_blank">processors.change_subresource.yml</a>, <a href="https://github.com/oroinc/platform/tree/master/src/Oro/Bundle/ApiBundle/Resources/config/processors.shared.yml" target="_blank">processors.shared.yml</a>. Run `php bin/console oro:api:debug delete_subresource` to display the list of processors.

This action has the following processor groups:

| Group Name          | Responsibility of Processors                                                     | Description                                                                                                                                                                                                                                                                                                                  |
|---------------------|----------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| initialize          | Context initialization.                                                          | Processors from this group are executed during the generation of the API documentation.                                                                                                                                                                                                                                      |
| resource_check      | Checking whether the requested resource type is accessible via API.              | –                                                                                                                                                                                                                                                                                                                            |
| normalize_input     | Preparing the input data to be used by processors from the next groups.          | –                                                                                                                                                                                                                                                                                                                            |
| security_check      | Checking whether access to the requested resource type is granted.               | –                                                                                                                                                                                                                                                                                                                            |
| load_data           | Loading an entity object to be updated.                                          | –                                                                                                                                                                                                                                                                                                                            |
| data_security_check | Checking whether access to the loaded data is granted.                           | –                                                                                                                                                                                                                                                                                                                            |
| transform_data      | Building a Symfony Form and using it to transform and validate the request data. | –                                                                                                                                                                                                                                                                                                                            |
| save_data           | Persisting an entity.                                                            | Use the [customize_form_data](#customize-form-data-action) action to modify and validate submitted data, and entities to be persisted into the database.                                                                                                                                                                     |
| normalize_data      | Converting the resulting entity into an array.                                   | –                                                                                                                                                                                                                                                                                                                            |
| finalize            | Adding the required response headers.                                            | –                                                                                                                                                                                                                                                                                                                            |
| normalize_result    | Building the action result.                                                      | The processors from this group are executed even if a processor from previous groups throws an exception. For implementation details, see <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Processor/NormalizeResultActionProcessor.php" target="_blank">NormalizeResultActionProcessor</a>. |

The following diagram shows the main data flow for this action:

![Data flow for delete_subresource action](img/backend/api/delete_subresource.png)

See the `handleDeleteSubresource` method of <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Request/RequestActionHandler.php" target="_blank">RequestActionHandler</a> as an example.

Below is an example of registering a processor to build a form builder:

```yaml
acme.api.items.build_form_builder:
    class: Oro\Bundle\ApiBundle\Processor\Subresource\ChangeSubresource\BuildFormBuilder
    arguments:
        - '@oro_api.form_helper'
    tags:
        - { name: oro.api.processor, action: delete_subresource, group: transform_data, association: items, parentClass: Acme\Bundle\DemoBundle\Entity\ShoppingList, priority: 100 }
```

<a id="get-relationship-action"></a>

### get_relationship Action

This action retrieves an entity identifier (for the “to-one” relationship) or a list of entities’ identifiers (for the “to-many” relationship) connected to the entity by a given association. For more details, see the <a href="http://jsonapi.org/format/#fetching-relationships" target="_blank">Fetching Relationships</a> section of the JSON:API specification.

The route name for REST API: `oro_rest_api_relationship`.

The URL template for REST API: `/api/{entity}/{id}/relationships/{association}`.

The HTTP method for REST API: `GET`.

The context class: <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Processor/Subresource/GetRelationship/GetRelationshipContext.php" target="_blank">GetRelationshipContext</a>. Also see [SubresourceContext](#subresourcecontext-class) class for more details.

The main processor class: <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Processor/Subresource/GetRelationshipProcessor.php" target="_blank">GetRelationshipProcessor</a>.

Existing worker processors: <a href="https://github.com/oroinc/platform/tree/master/src/Oro/Bundle/ApiBundle/Resources/config/processors.get_relationship.yml" target="_blank">processors.get_relationship.yml</a>, <a href="https://github.com/oroinc/platform/tree/master/src/Oro/Bundle/ApiBundle/Resources/config/processors.shared.yml" target="_blank">processors.shared.yml</a>. Run `php bin/console oro:api:debug get_relationship` to display the list of processors.

This action has the following processor groups:

| Group Name          | Responsibility of Processors                                                  | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
|---------------------|-------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| initialize          | Context initialization.                                                       | Processors from this group are executed during the generation of the API documentation.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| resource_check      | Checking whether the requested resource type is accessible via API.           | –                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| normalize_input     | Preparing the input data to be used by processors from the next groups.       | –                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| security_check      | Checking whether access to the requested resource type is granted.            | –                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| build_query         | Building a query to use to load data.                                         | –                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| load_data           | Loading data.                                                                 | –                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| data_security_check | Checking whether access to the loaded data is granted.                        | –                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| normalize_data      | Converting loaded data into an array.                                         | In most cases, the processors from this group are skipped because the <a href="https://github.com/oroinc/platform/tree/master/src/Oro/Component/EntitySerializer" target="_blank">EntitySerializer</a> loads most entities and returns already normalized data. For details see <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Processor/Shared/LoadEntityByEntitySerializer.php" target="_blank">LoadEntityByEntitySerializer</a> and <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Processor/Shared/LoadEntitiesByEntitySerializer.php" target="_blank">LoadEntitiesByEntitySerializer</a>. |
| finalize            | Final validation of the loaded data and adding the required response headers. | –                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| normalize_result    | Building the action result.                                                   | The processors from this group are executed even if a processor from previous groups throws an exception. For implementation details, see <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Processor/NormalizeResultActionProcessor.php" target="_blank">NormalizeResultActionProcessor</a>.                                                                                                                                                                                                                                                                                                                                       |

The following diagram shows the main data flow for this action:

![Data flow for get_relationship flow](img/backend/api/get_relationship.png)

See the `handleGetRelationship` method of <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Request/RequestActionHandler.php" target="_blank">RequestActionHandler</a> as an example.

<a id="update-relationship-action"></a>

### update_relationship Action

This action changes an entity (for the “to-one” relationship) or completely replaces all entities (for the “to-many” relationship) connected to a given entity by a given association. For more details, see the <a href="http://jsonapi.org/format/#crud-updating-relationships" target="_blank">Updating Relationships</a> section of the JSON:API specification.

The route name for REST API: `oro_rest_api_relationship`.

The URL template for REST API: `/api/{entity}/{id}/relationships/{association}`.

The HTTP method for REST API: `PATCH`.

The context class: <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Processor/Subresource/UpdateRelationship/UpdateRelationshipContext.php" target="_blank">UpdateRelationshipContext</a>. Also see [SubresourceContext](#subresourcecontext-class) class for more details.

The main processor class: <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Processor/Subresource/UpdateRelationshipProcessor.php" target="_blank">UpdateRelationshipProcessor</a>.

Existing worker processors: <a href="https://github.com/oroinc/platform/tree/master/src/Oro/Bundle/ApiBundle/Resources/config/processors.update_relationship.yml" target="_blank">processors.update_relationship.yml</a>, <a href="https://github.com/oroinc/platform/tree/master/src/Oro/Bundle/ApiBundle/Resources/config/processors.shared.yml" target="_blank">processors.shared.yml</a>. Run `php bin/console oro:api:debug update_relationship` to display the list of processors.

This action has the following processor groups:

| Group Name          | Responsibility of Processors                                                     | Description                                                                                                                                                                                                                                                                                                                  |
|---------------------|----------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| initialize          | Context initialization.                                                          | Processors from this group are executed during the generation of the API documentation.                                                                                                                                                                                                                                      |
| resource_check      | Checking whether the requested resource type is accessible via API.              | –                                                                                                                                                                                                                                                                                                                            |
| normalize_input     | Preparing the input data to be used by processors from the next groups.          | –                                                                                                                                                                                                                                                                                                                            |
| security_check      | Checking whether access to the requested resource type is granted.               | –                                                                                                                                                                                                                                                                                                                            |
| load_data           | Loading an entity object to be updated.                                          | –                                                                                                                                                                                                                                                                                                                            |
| data_security_check | Checking whether access to the loaded data is granted.                           | –                                                                                                                                                                                                                                                                                                                            |
| transform_data      | Building a Symfony Form and using it to transform and validate the request data. | –                                                                                                                                                                                                                                                                                                                            |
| save_data           | Persisting an entity.                                                            | –                                                                                                                                                                                                                                                                                                                            |
| normalize_data      | Converting the resulting relationship into an array.                             | –                                                                                                                                                                                                                                                                                                                            |
| finalize            | Adding the required response headers.                                            | –                                                                                                                                                                                                                                                                                                                            |
| normalize_result    | Building the action result.                                                      | The processors from this group are executed even if a processor from previous groups throws an exception. For implementation details, see <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Processor/NormalizeResultActionProcessor.php" target="_blank">NormalizeResultActionProcessor</a>. |

See the `handleUpdateRelationship` method of <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Request/RequestActionHandler.php" target="_blank">RequestActionHandler</a> as an example.

<a id="add-relationship-action"></a>

### add_relationship Action

This action adds one or several entities to a “to-many” relationship. For more details, see the <a href="http://jsonapi.org/format/#crud-updating-relationships" target="_blank">Updating Relationships</a> section of the JSON:API specification.

The route name for REST API: `oro_rest_api_relationship`.

The URL template for REST API: `/api/{entity}/{id}/relationships/{association}`.

The HTTP method for REST API: `POST`.

The context class: <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Processor/Subresource/AddRelationship/AddRelationshipContext.php" target="_blank">AddRelationshipContext</a>. See the [SubresourceContext](#subresourcecontext-class) class for more details.

The main processor class: <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Processor/Subresource/AddRelationshipProcessor.php" target="_blank">AddRelationshipProcessor</a>

Existing worker processors: <a href="https://github.com/oroinc/platform/tree/master/src/Oro/Bundle/ApiBundle/Resources/config/processors.add_relationship.yml" target="_blank">processors.add_relationship.yml</a>, <a href="https://github.com/oroinc/platform/tree/master/src/Oro/Bundle/ApiBundle/Resources/config/processors.shared.yml" target="_blank">processors.shared.yml</a> or run `php bin/console oro:api:debug add_relationship`.

This action has the following processor groups:

| Group Name          | Responsibility of Processors                                                     | Description                                                                                                                                                                                                                                                                                                                  |
|---------------------|----------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| initialize          | Context initialization.                                                          | Processors from this group are executed during the generation of the API documentation.                                                                                                                                                                                                                                      |
| resource_check      | Checking whether the requested resource type is accessible via API.              | –                                                                                                                                                                                                                                                                                                                            |
| normalize_input     | Preparing the input data to be used by processors from the next groups.          | –                                                                                                                                                                                                                                                                                                                            |
| security_check      | Checking whether access to the requested resource type is granted.               | –                                                                                                                                                                                                                                                                                                                            |
| load_data           | Loading an entity object to be updated.                                          | –                                                                                                                                                                                                                                                                                                                            |
| data_security_check | Checking whether access to the loaded data is granted.                           | –                                                                                                                                                                                                                                                                                                                            |
| transform_data      | Building a Symfony Form and using it to transform and validate the request data. | –                                                                                                                                                                                                                                                                                                                            |
| save_data           | Persisting an entity.                                                            | –                                                                                                                                                                                                                                                                                                                            |
| normalize_data      | Converting the resulting relationship into an array.                             | –                                                                                                                                                                                                                                                                                                                            |
| finalize            | Adding the required response headers.                                            | –                                                                                                                                                                                                                                                                                                                            |
| normalize_result    | Building the action result.                                                      | The processors from this group are executed even if a processor from previous groups throws an exception. For implementation details, see <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Processor/NormalizeResultActionProcessor.php" target="_blank">NormalizeResultActionProcessor</a>. |

The following diagram shows the main data flow for this action:

![Data flow for add_relationship action](img/backend/api/add_relationship.png)

See the `handleAddRelationship` method of <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Request/RequestActionHandler.php" target="_blank">RequestActionHandler</a> as an example.

<a id="delete-relationship-action"></a>

### delete_relationship Action

This action removes one or several entities from a “to-many” relationship. For more details, see the <a href="http://jsonapi.org/format/#crud-updating-relationships" target="_blank">Updating Relationships</a> section of the JSON:API specification.

The route name for REST API: `oro_rest_api_relationship`.

The URL template for REST API: `/api/{entity}/{id}/relationships/{association}`.

The HTTP method for REST API: `POST`.

The context class: <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Processor/Subresource/AddRelationship/AddRelationshipContext.php" target="_blank">AddRelationshipContext</a>. See the [SubresourceContext](#subresourcecontext-class) class for more details.

The main processor class:  <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Processor/Subresource/AddRelationshipProcessor.php" target="_blank">AddRelationshipProcessor</a>.

Existing worker processors: <a href="https://github.com/oroinc/platform/tree/master/src/Oro/Bundle/ApiBundle/Resources/config/processors.delete_relationship.yml" target="_blank">processors.delete_relationship.yml</a>, <a href="https://github.com/oroinc/platform/tree/master/src/Oro/Bundle/ApiBundle/Resources/config/processors.shared.yml" target="_blank">processors.shared.yml</a>. Run `php bin/console oro:api:debug delete_relationship` to display the list of processors.

This action has the following processor groups:

| Group Name          | Responsibility of Processors                                                     | Description                                                                                                                                                                                                                                                                                                                  |
|---------------------|----------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| initialize          | Context initialization.                                                          | Processors from this group are executed during the generation of the API documentation.                                                                                                                                                                                                                                      |
| resource_check      | Checking whether the requested resource type is accessible via API.              | –                                                                                                                                                                                                                                                                                                                            |
| normalize_input     | Preparing the input data to be used by processors from the next groups.          | –                                                                                                                                                                                                                                                                                                                            |
| security_check      | Checking whether access to the requested resource type is granted.               | –                                                                                                                                                                                                                                                                                                                            |
| load_data           | Loading an entity object to be updated.                                          | –                                                                                                                                                                                                                                                                                                                            |
| data_security_check | Checking whether access to the loaded data is granted.                           | –                                                                                                                                                                                                                                                                                                                            |
| transform_data      | Building a Symfony Form and using it to transform and validate the request data. | –                                                                                                                                                                                                                                                                                                                            |
| save_data           | Persisting an entity.                                                            | –                                                                                                                                                                                                                                                                                                                            |
| normalize_data      | Converting the resulting relationship into an array.                             | –                                                                                                                                                                                                                                                                                                                            |
| finalize            | Adding the required response headers.                                            | –                                                                                                                                                                                                                                                                                                                            |
| normalize_result    | Building the action result.                                                      | The processors from this group are executed even if a processor from previous groups throws an exception. For implementation details, see <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Processor/NormalizeResultActionProcessor.php" target="_blank">NormalizeResultActionProcessor</a>. |

The following diagram shows the main data flow for this action:

![Data flow for delete_relationship action](img/backend/api/delete_relationship.png)

See the `handleDeleteRelationship` method of <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Request/RequestActionHandler.php" target="_blank">RequestActionHandler</a> as an example.

<a id="options-action"></a>

### options Action

This action retrieves the communication options for a resource. For more details, see the <a href="https://www.w3.org/Protocols/rfc2616/rfc2616-sec9.html#sec9.2" target="_blank">OPTIONS</a> section of the HTTP specification.

This action is also intended <a href="https://www.w3.org/TR/cors/#resource-preflight-requests" target="_blank">CORS preflight requests</a> for REST API. For more details, see the [CORS Configuration](cors.md#api-cors-config) section.

The HTTP method for REST API: `OPTIONS`.

The context class: <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Processor/Options/OptionsContext.php" target="_blank">OptionsContext</a>.

The main processor class: <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Processor/OptionsProcessor.php" target="_blank">OptionsProcessor</a>.

Existing worker processors: <a href="https://github.com/oroinc/platform/tree/master/src/Oro/Bundle/ApiBundle/Resources/config/processors.options.yml" target="_blank">processors.options.yml</a>, <a href="https://github.com/oroinc/platform/tree/master/src/Oro/Bundle/ApiBundle/Resources/config/processors.shared.yml" target="_blank">processors.shared.yml</a>. Run php `bin/console oro:api:debug options` to list the processors.

This action has the following processor groups:

| Group Name       | Responsibility of Processors                                                                              | Description                                                                                                                                                                                                                                                                                                                  |
|------------------|-----------------------------------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| initialize       | Context initialization.                                                                                   | Processors from this group are executed during the generation of the API documentation.                                                                                                                                                                                                                                      |
| resource_check   | Checking whether the requested resource type is accessible via API and validating the request parameters. | –                                                                                                                                                                                                                                                                                                                            |
| normalize_result | Building the action result.                                                                               | The processors from this group are executed even if a processor from previous groups throws an exception. For implementation details, see <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Processor/NormalizeResultActionProcessor.php" target="_blank">NormalizeResultActionProcessor</a>. |

See the `handleOptionsItem`, `handleOptionsList`, `handleOptionsSubresource` and `handleOptionsRelationship` methods of <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Request/RequestActionHandler.php" target="_blank">RequestActionHandler</a> as examples.

<a id="web-api-auxiliary-actions"></a>

## Auxiliary Actions

<a id="customize-loaded-data-action"></a>

### customize_loaded_data Action

This action makes modifications to the data loaded by the [get](#get-action), [get_list](#get-list-action) and [get_subresource](#get-subresource-action) actions.

The context class: <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Processor/CustomizeLoadedData/CustomizeLoadedDataContext.php" target="_blank">CustomizeLoadedDataContext</a>.

The main processor class: <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Processor/CustomizeLoadedDataProcessor.php" target="_blank">CustomizeLoadedDataProcessor</a>.

An example of a processor used to modify the loaded data is <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Processor/CustomizeLoadedData/ComputePrimaryField.php" target="_blank">ComputePrimaryField</a> or [Add a Computed Field](how-to.md#add-computed-field). Run `php bin/console oro:api:debug customize_loaded_data` to display other processors registered in this action.

The `collection` tag attribute can be used for processors of this action to process all primary entities in [get_list](#get-list-action) or [get_subresource](#get-subresource-action) actions or all entities in `to-many` associations for [get](#get-action), [get_list](#get-list-action) or [get_subresource](#get-subresource-action) actions. An example of a case when using this attribute can be helpful if you want to execute one SQL query for all entities in a collection to get additional data instead of executing a separate SQL query for each entity in a collection. The default value the `collection` tag attribute is `false`. An example of a processor that should be executed to a whole collection:

```yaml
services:
    acme.api.process_my_collection:
        class: Acme\Bundle\DemoBundle\Api\Processor\ProcessMyCollection
        tags:
            - { name: oro.api.processor, action: customize_loaded_data, collection: true, class: Acme\Bundle\DemoBundle\Entity\MyEntity }
```

#### IMPORTANT
The collection elements are an associative array, and processors responsible for customizing the collection must keep keys in this array without changes.

#### NOTE
All processors for this action have the `identifier_only` tag attribute set to `false`. It means that such processors are not executed when loading relationships. If your processor should be executed when loading the relationships, set the `identifier_only` tag attribute to `true`. If your processor should be executed when loading relationships, primary, and included entities, set the `identifier_only` tag attribute to `null`. For example:

```yaml
services:
    acme.api.compute_my_field:
        class: Acme\Bundle\DemoBundle\Api\Processor\ComputeMyField
        tags:
            - { name: oro.api.processor, action: customize_loaded_data, identifier_only: true, class: Acme\Bundle\DemoBundle\Entity\MyEntity }
```

#### NOTE
The `identifier_only` tag attribute is not supported if the `collection` tag attribute equals `true`. All processors intended to modify collections are executed when loading primary entities and entities in to-many associations, even if only the identifier field is requested.

#### NOTE
The <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Request/ValueTransformer.php" target="_blank">ValueTransformer</a> can be used in `customize_loaded_data` processors to convert a value to a format suitable for the API response.

<a id="customize-form-data-action"></a>

### customize_form_data Action

Makes modification and validation of submitted data, and entities to be persisted in the database by
the [create](#create-action), [update](#update-action), [update_relationship](#update_relationship-action),
[add_relationship](#add_relationship-action) and [delete_relationship](#delete_relationship-action) actions.

The context class: <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Processor/CustomizeFormData/CustomizeFormDataContext.php" target="_blank">CustomizeFormDataContext</a>.

The main processor class: <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Processor/CustomizeFormDataProcessor.php" target="_blank">CustomizeFormDataProcessor</a>.

This action is executed when the following <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Processor/CustomizeFormData/CustomizeFormDataContext.php" target="_blank">Events</a> are dispatched:

| Event Name                 | Description                                                                                                                                                                                                                                                                                                         |
|----------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| pre_submit                 | This event is dispatched at the beginning of the Form::submit() method.                                                                                                                                                                                                                                             |
| submit                     | This event is dispatched after the Form::submit() method has submitted and mapped the children, and after reverse transformation to normalized representation.                                                                                                                                                      |
| post_submit                | This event is dispatched at the very end of the Form::submit().                                                                                                                                                                                                                                                     |
| pre_validate               | This event is dispatched at the end of the form submitting process, just before data validation. It can be used to final form data correcting after all listeners, except data validation listener, are executed and all relationships between submitted data are set.                                              |
| post_validate              | This event is dispatched at the end of the form submitting process, just after data validation. It can be used to finalize the form after all listeners, including the data validation listener, are executed. E.g. it can be used to correct form validation result.                                               |
| pre_flush_data             | This event is dispatched after the database transaction is open but before data are flushed into the database. Do not call EntityManager::persist() and EntityManager::remove() during handling of this event, use addAdditionalEntity() and addAdditionalEntityToRemove() methods of the context instead.          |
| post_flush_data            | This event is dispatched after data are successfully flushed into the database but before the database transaction is committed.                                                                                                                                                                                    |
| post_save_data             | This event is dispatched after data are successfully flushed into the database, and the database transaction is committed. It can be used to perform some not crucial operations after data are saved into the database. It means that failure of these operations will not roll back data saved into the database. |
| rollback_validated_request | This event is dispatched before the database transaction is rolled back for API requests with a meta validate operation flag.                                                                                                                                                                                       |

#### NOTE
All these events use the same context, so it can be used to share data between events.

#### NOTE
When a request contains [included entities](../../api/create-update-related-resources.md#web-services-api-create-update-related-resources), these events are dispatched for the included entities first and then for the primary entity. Each entity has its own context. Use the **getSharedData()** method of the [Context](#context-class) class to share data between events for different entities.

An example of a processor used to modify the submitted data is <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Processor/CustomizeFormData/MapPrimaryField.php" target="_blank">MapPrimaryField</a>. You can also run `php bin/console oro:api:debug customize_form_data` to display other processors registered in this action.

<a id="get-config-action"></a>

### get_config Action

This action retrieves a configuration of an entity.

The context class: <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Processor/GetConfig/ConfigContext.php" target="_blank">ConfigContext</a>.

The main processor class: <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Processor/GetConfig/ConfigProcessor.php" target="_blank">ConfigProcessor</a>.

Existing worker processors: <a href="https://github.com/oroinc/platform/tree/master/src/Oro/Bundle/ApiBundle/Resources/config/processors.get_config.yml" target="_blank">processors.get_config.yml</a>. Run `php bin/console oro:api:debug get_config` to see the list of processors.

Additionally, <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Provider/ConfigProvider.php" target="_blank">ConfigProvider</a> was created to make usage of this action as easy as possible.

Example:

```php
/** @var ConfigProvider $configProvider */
$configProvider = $container->get('oro_api.config_provider');
$config = $configProvider->getConfig($entityClassName, $version, $requestType, $configExtras);
```

<a id="get-metadata-action"></a>

### get_metadata Action

This action retrieves the metadata of an entity.

The context class: <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Processor/GetMetadata/MetadataContext.php" target="_blank">MetadataContext</a>.

The main processor class: <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Processor/MetadataProcessor.php" target="_blank">MetadataProcessor</a>.

Existing worker processors: <a href="https://github.com/oroinc/platform/tree/master/src/Oro/Bundle/ApiBundle/Resources/config/processors.get_metadata.yml" target="_blank">processors.get_metadata.yml</a>. Run `php bin/console oro:api:debug get_metadata` to see the list of processors.

Additionally, <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Provider/MetadataProvider.php" target="_blank">MetadataProvider</a> was created to make usage of this action as easy as possible.

Example:

```php
/** @var MetadataProvider $metadataProvider */
$metadataProvider = $container->get('oro_api.metadata_provider');
$metadata = $metadataProvider->getMetadata($entityClassName, $version, $requestType, $entityConfig, $metadataExtras);
```

<a id="normalize-value-action"></a>

### normalize_value Action

This action converts an input value to a requested data type.

The context class: <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Processor/NormalizeValue/NormalizeValueContext.php" target="_blank">NormalizeValueContext</a>.

The main processor class: <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Processor/NormalizeValueProcessor.php" target="_blank">NormalizeValueProcessor</a>.

Existing worker processors: <a href="https://github.com/oroinc/platform/tree/master/src/Oro/Bundle/ApiBundle/Resources/config/processors.normalize_value.yml" target="_blank">processors.normalize_value.yml</a>. Run `php bin/console oro:api:debug normalize_value` to see the list of processors.

Additionally, <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Request/ValueNormalizer.php" target="_blank">ValueNormalizer</a> and <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Util/ValueNormalizerUtil.php" target="_blank">ValueNormalizerUtil</a> were created to make usage of this action as easy as possible.

Example:

```php
/** @var ValueNormalizer $valueNormalizer */
$valueNormalizer = $container->get('oro_api.metadata_provider');
$normalizedValue = $valueNormalizer->normalizeValue($value, $dataType, $requestType);
```

#### NOTE
The <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Request/ValueNormalizer.php" target="_blank">ValueNormalizer</a> is intended to process input values only. In case you need to convert a value for the API response, use <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Request/ValueTransformer.php" target="_blank">ValueTransformer</a>.

<a id="collect-resource-action"></a>

### collect_resources Action

This action gets a list of all resources accessible via the API.

The context class: <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Processor/CollectResources/CollectResourcesContext.php" target="_blank">CollectResourcesContext</a>.

The main processor class: <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Processor/CollectResourcesProcessor.php" target="_blank">CollectResourcesProcessor</a>.

Existing worker processors:<a href="https://github.com/oroinc/platform/tree/master/src/Oro/Bundle/ApiBundle/Resources/config/processors.collect_resources.yml" target="_blank">processors.collect_resources.yml</a>. Run `php bin/console oro:api:debug collect_resources` to see the list of processors.

Additionally, <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Provider/ResourcesProvider.php" target="_blank">ResourcesProvider</a> was created to make it as easy as possible to use this action.

Example:

```php
/** @var ResourcesProvider $resourcesProvider */
$resourcesProvider = $container->get('oro_api.resources_provider');
// get all API resources
// (all resources are configured to be used in API, including not accessible resources)
$resources = $resourcesProvider->getResources($version, $requestType);
// get class names of all API resources accessible through API
$accessibleResources = $resourcesProvider->getAccessibleResources($version, $requestType);
// check whether an entity is configured to be used in API
$isKnown = $resourcesProvider->isResourceKnown($entityClass, $version, $requestType);
// check whether an entity is accessible through API
$isAccessible = $resourcesProvider->isResourceAccessible($entityClass, $version, $requestType);
// check whether an entity is accessible as an association in API
$isAccessibleAsAssociation = $resourcesProvider->isResourceAccessibleAsAssociation($entityClass, $version, $requestType);
// check whether an entity does not have an identifier field
$isResourceWithoutIdentifier = $resourcesProvider->isResourceWithoutIdentifier($entityClass, $version, $requestType);
```

<a id="collect-subresource-action"></a>

### collect_subresources Action

This action retrieves a list of all sub-resources accessible via the API for a given entity type.

The context class: <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Processor/CollectSubresources/CollectSubresourcesContext.php" target="_blank">CollectSubresourcesContext</a>.

The main processor class: <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Processor/CollectSubresourcesProcessor.php" target="_blank">CollectSubresourcesProcessor</a>.

Existing worker processors: <a href="https://github.com/oroinc/platform/tree/master/src/Oro/Bundle/ApiBundle/Resources/config/processors.collect_subresources.yml" target="_blank">processors.collect_subresources.yml</a>. Run `php bin/console oro:api:debug collect_subresources` to see the list of processors.

Additionally, <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Provider/SubresourcesProvider.php" target="_blank">SubresourcesProvider</a> was created to make usage of this action as easy as possible.

Example:

```php
/** @var SubresourcesProvider $subresourcesProvider */
$subresourcesProvider = $container->get('oro_api.subresources_provider');
// get all sub-resources for a given entity
$entitySubresources = $subresourcesProvider->getSubresources($entityClass, $version, $requestType);
```

<a id="not-allowed-action"></a>

### not_allowed Action

This action builds a response for the case when a request does not match any public action. An example of such a case can be a REST API request with an unsupported HTTP method.

This action does not have its own context and processor classes. It can work with any context class based on [Context](#context-class) class, and any public action processor can process it. The processor to be used depends on the request attributes.

Run `php bin/console oro:api:debug not_allowed` to list the processors.

This action has the following processor groups:

| Group Name       | Responsibility of Processors                                                | Description                                                                                                                                                                                                                                                                                                                  |
|------------------|-----------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| initialize       | Context initialization.                                                     | –                                                                                                                                                                                                                                                                                                                            |
| build_response   | Building the action response body, if the current request type requires it. | –                                                                                                                                                                                                                                                                                                                            |
| normalize_result | Building the action result.                                                 | The processors from this group are executed even if a processor from previous groups throws an exception. For implementation details, see <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Processor/NormalizeResultActionProcessor.php" target="_blank">NormalizeResultActionProcessor</a>. |

See the `handleNotAllowedItem`, `handleNotAllowedList`, `handleNotAllowedSubresource` and `handleNotAllowedRelationship` methods of <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Request/RequestActionHandler.php" target="_blank">RequestActionHandler</a> as examples.

<a id="unhandled-error-action"></a>

### unhandled_error Action

This action builds a response for the case when an unexpected error happens before any public action is started.

The context class: this action does not have its own context class, and it uses the [Context](#context-class) class.

The main processor class: <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Processor/UnhandledErrorProcessor.php" target="_blank">UnhandledErrorProcessor</a>.

Run `php bin/console oro:api:debug unhandled_error` to list the processors.

This action has the following processor groups:

| Group Name       | Responsibility of Processors   | Description                                                                                                                                                                                                                                                                                                                  |
|------------------|--------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| initialize       | Context initialization.        | –                                                                                                                                                                                                                                                                                                                            |
| normalize_result | Building the action result.    | The processors from this group are executed even if a processor from previous groups throws an exception. For implementation details, see <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Processor/NormalizeResultActionProcessor.php" target="_blank">NormalizeResultActionProcessor</a>. |

See the `handleUnhandledError` method of <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Request/RequestActionHandler.php" target="_blank">RequestActionHandler</a> as an example.

<a id="batch-update-action"></a>

### batch_update Action

This action updates or creates a set of entities of the same type that are part of an asynchronous
batch operation. It is triggered by the [update_list](#update-list-action) action.

The context class: <a href="https://github.com/oroinc/platform/tree/master/src/Oro/Bundle/ApiBundle/Batch/Processor/Update/BatchUpdateContext.php" target="_blank">BatchUpdateContext</a>.

The main processor class: <a href="https://github.com/oroinc/platform/tree/master/src/Oro/Bundle/ApiBundle/Batch/Processor/BatchUpdateProcessor.php" target="_blank">BatchUpdateProcessor</a>.

Existing worker processors:<a href="https://github.com/oroinc/platform/tree/master/src/Oro/Bundle/ApiBundle/Resources/config/processors.batch_update.yml" target="_blank">processors.batch_update.yml</a>. Run `php bin/console oro:api:debug batch_update` to see the list of processors.

This action has the following processor groups:

| Group Name       | Responsibility of Processors          | Description                                                                                                                                                                                                                                                                                                                              |
|------------------|---------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| initialize       | Context initialization.               | –                                                                                                                                                                                                                                                                                                                                        |
| finalize         | Adding the required response headers. | –                                                                                                                                                                                                                                                                                                                                        |
| save_data        | Persisting entities.                  | –                                                                                                                                                                                                                                                                                                                                        |
| save_errors      | Persisting found errors.              | –                                                                                                                                                                                                                                                                                                                                        |
| normalize_result | Building the action result.           | The processors from this group are executed even if a processor from previous groups throws an exception. For implementation details, see <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Processor/ByStepNormalizeResultActionProcessor.php" target="_blank">ByStepNormalizeResultActionProcessor</a>. |

The following diagram shows the main data flow for this action:

![Data flow for batch_update action](img/backend/api/batch_update.png)

See <a href="https://github.com/oroinc/platform/tree/master/src/Oro/Bundle/ApiBundle/Batch/Handler/BatchUpdateHandler.php" target="_blank">BatchUpdateHandler</a> as an example.

<a id="batch-update-item-action"></a>

### batch_update_item Action

This action creates or updates an entity that is part of an asynchronous batch operation.
It is used by the [batch_update](#batch-update-action) action.

The context class: <a href="https://github.com/oroinc/platform/tree/master/src/Oro/Bundle/ApiBundle/Batch/Processor/UpdateItem/BatchUpdateItemContext.php" target="_blank">BatchUpdateItemContext</a>.

The main processor class: <a href="https://github.com/oroinc/platform/tree/master/src/Oro/Bundle/ApiBundle/Batch/Processor/BatchUpdateItemProcessor.php" target="_blank">BatchUpdateItemProcessor</a>.

Existing worker processors:<a href="https://github.com/oroinc/platform/tree/master/src/Oro/Bundle/ApiBundle/Resources/config/processors.batch_update_item.yml" target="_blank">processors.batch_update_item.yml</a>. Run `php bin/console oro:api:debug batch_update_item` to see the list of processors.

This action has the following processor groups:

| Group Name       | Responsibility of Processors                | Description                                                                                                                                                                                                                                                                                                                              |
|------------------|---------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| initialize       | Context initialization.                     | –                                                                                                                                                                                                                                                                                                                                        |
| transform_data   | Converts the request data to entity object. | –                                                                                                                                                                                                                                                                                                                                        |
| normalize_result | Building the action result.                 | The processors from this group are executed even if a processor from previous groups throws an exception. For implementation details, see <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Processor/ByStepNormalizeResultActionProcessor.php" target="_blank">ByStepNormalizeResultActionProcessor</a>. |

The following diagram shows the main data flow for this action:

![Data flow for batch_update_item action](img/backend/api/batch_update_item.png)

See <a href="https://github.com/oroinc/platform/tree/master/src/Oro/Bundle/ApiBundle/Batch/Handler/BatchUpdateItem.php" target="_blank">BatchUpdateItem</a> as an example.

<a id="web-api-context-class"></a>

## Context Class

The <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Processor/Context.php" target="_blank">Context</a> class is used as a superclass for the context classes of CRUD actions such as [get](#get-action), [get_list](#get-list-action), [create](#create-action), [update](#update-action), [delete](#delete-action), and [delete_list](#delete-list-action).

General methods:

- **getClassName()** - Retrieves the fully-qualified class name of an entity.
- **setClassName(className)** - Sets fully-qualified class name of an entity.
- **getRequestHeaders()** - Retrieves the request headers.
- **setRequestHeaders(parameterBag)** - Sets an object to use for accessing the request headers.
- **getResponseHeaders()** - Retrieves the response headers.
- **setResponseHeaders(parameterBag)** - Sets an object to use for accessing accessing the response headers.
- **getResponseStatusCode()** - Retrieves the response status code.
- **setResponseStatusCode(statusCode)** - Sets the response status code.
- **isSuccessResponse()** - Indicates whether a result document represents a success response.
- **getResponseDocumentBuilder()** - Retrieves the response document builder.
- **setResponseDocumentBuilder(documentBuilder)** - Sets the response document builder.
- **getFilters()** -  Retrieves a <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Filter/FilterCollection.php" target="_blank">list of filters</a> to set additional restrictions to a query used to retrieve the entity data.
- **getFilterValues()** - Retrieves a collection of the <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Filter/FilterValue.php" target="_blank">FilterValue</a> objects that contains all incoming filters.
- **setFilterValues(accessor)** - Sets an <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Filter/FilterValueAccessorInterface.php" target="_blank">object</a> to use for accessing the incoming filters.
- **isMainRequest()** - Indicates whether the current action processes a main API request or is executed as part of another action.
- **setMainRequest(main)** - Sets a flag that indicates the current action processes a main API request or is executed as part of another action.
- **isCorsRequest()** - Indicates whether the current request is <a href="https://www.w3.org/TR/cors/" target="_blank">CORS</a> request.
- **setCorsRequest(cors)** - Sets a flag indicates whether the current request is <a href="https://www.w3.org/TR/cors/" target="_blank">CORS</a> request.
- **isHateoasEnabled()** - Indicates whether <a href="https://restfulapi.net/hateoas/" target="_blank">HATEOAS</a> is enabled.
- **setHateoas(flag)** - Sets a flag indicates whether <a href="https://restfulapi.net/hateoas/" target="_blank">HATEOAS</a> is enabled.
- **hasQuery()** - Checks whether a query used to get the result data exists.
- **getQuery()** - Retrieves a query used to get result data.
- **setQuery(query)** - Sets a query used to get result data.
- **getCriteria()** - Retrieves the <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Collection/Criteria.php" target="_blank">Criteria</a> object that sets additional restrictions to a query used to retrieve the entity data.
- **setCriteria(criteria)** - Sets the <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Collection/Criteria.php" target="_blank">Criteria</a> object that sets additional restrictions to a query used to retrieve the result data.
- **getAllEntities(primaryOnly = false)** - Gets all entities, primary and included ones, that are processing by an action.
- **hasErrors()** - Checks whether any error happened when processing an action.
- **getErrors()** - Retrieves all <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Model/Error.php" target="_blank">errors</a> that occurred during the processing of an action.
- **addError(error)** - Registers an <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Model/Error.php" target="_blank">errors</a>.
- **resetErrors()** - Removes all errors.
- **isSoftErrorsHandling()** - Retrieves a value that indicates whether to stop the further processing or throw an exception in case of error.
- **setSoftErrorsHandling(softErrorsHandling)** - Sets a value that indicates whether to stop the further processing or throw an exception in case of error.
- **setProcessed(operationName)** - Marks work as done. In most cases, this method is unnecessary because it is easy to determine when work is already complete by checking the state of a context. However, if a processor performs complex work, it might be required to mark it as already done directly.
- **clearProcessed(operationName)** - Marks work as not yet done.
- **isProcessed(operationName)** - Checks whether work is already done.
- **getSharedData()** - Retrieves an object used to share data between a primary action and actions executed as part of this action. Also, this object can be used to share data among different kinds of child actions.
- **setSharedData(parameterBag)** - Sets an object used to share data.
- **getNormalizationContext()** - Gets a context for response data normalization.
- **getInfoRecords()** - Gets a list of records containing additional information about collections, e.g., the “has_more” flag in such a record indicates whether a collection has more records than was requested.
- **setInfoRecords(infoRecords)** - Sets a list of records containing additional information about collections, e.g., the “has_more” flag in such a record indicates whether a collection has more records than was requested.
- **addInfoRecord(key, value)** - Adds a record that contains an additional information about collections.
- **addAssociationInfoRecords(propertyPath, infoRecords)** - Adds records that contain an additional information about a collection valued association.
- **getNotResolvedIdentifiers()** - Gets all not resolved identifiers.
- **addNotResolvedIdentifier(path, identifier)** - Adds an identifier that cannot be resolved.
- **removeNotResolvedIdentifier(path)** - Removes an identifier that cannot be resolved.

Entity configuration-related methods:

- **getConfigExtras()** - Retrieves a list of <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Config/Extra/ConfigExtraInterface.php" target="_blank">requests for configuration data</a>.
- **setConfigExtras(extras)** - Sets a list of requests for configuration data.
- **hasConfigExtra(extraName)** - Checks whether some configuration data is requested.
- **getConfigExtra(extraName)** - Retrieves a request for configuration data by its name.
- **addConfigExtra(extra)** - Adds a request for some configuration data.
- **removeConfigExtra(extraName)** - Removes a request for some configuration data.
- **getConfigSections()** - Retrieves names of all requested <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Config/Extra/ConfigExtraSectionInterface.php" target="_blank">configuration sections</a>.
- **hasConfig()** - Checks whether a configuration of an entity exists.
- **getConfig()** - Retrieves a <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Config/EntityDefinitionConfig.php" target="_blank">configuration of an entity</a>.
- **setConfig(config)** - Sets a custom configuration of an entity. This method can be used to override the default configuration of an entity completely.
- **hasConfigOfFilters(initialize)** - Checks whether an entity has a configuration of filters.
- **getConfigOfFilters()** - Retrieves a <a href="https://github.com/oroinc/platform/blob/3.1/src/Oro/Bundle/ApiBundle/Config/FiltersConfig.php" target="_blank">configuration of filters</a> for an entity.
- **setConfigOfFilters(config)** - Sets a custom configuration of filters. This method can be used to override the default configuration of filters completely.
- **hasConfigOfSorters(initialize)** - Checks whether an entity has a configuration of sorters.
- **getConfigOfSorters()** - Retrieves a <a href="https://github.com/oroinc/platform/blob/3.1/src/Oro/Bundle/ApiBundle/Config/SortersConfig.php" target="_blank">configuration of sorters</a> for an entity.
- **setConfigOfSorters(config)** - Sets a custom configuration of sorters. This method can be used to override the default configuration of sorters completely.
- **hasConfigOf(configSection, initialize)** - Checks whether a configuration of the given section exists.
- **getConfigOf(configSection)** - Retrieves a configuration from the given section.
- **setConfigOf(configSection, config)** - Sets a configuration for the given section. This method can be used to override the default configuration for the given section completely.

Entity metadata-related methods:

- **hasIdentifierFields()** - Checks whether metadata of an entity has at least one identifier field.
- **getMetadataExtras()** - Retrieves a list of <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Metadata/Extra/MetadataExtraInterface.php" target="_blank">requests for additional metadata info</a>.
- **setMetadataExtras(extras)** - Sets a list of requests for additional metadata info.
- **hasMetadataExtra()** - Checks whether some additional metadata info is requested.
- **addMetadataExtra(extra)** - Adds a request for some additional metadata info.
- **removeMetadataExtra(extraName)** - Removes a request for some additional metadata info.
- **hasMetadata()** - Checks whether metadata of an entity exists.
- **getMetadata()** - Retrieves <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Metadata/EntityMetadata.php" target="_blank">metadata</a> of an entity.
- **setMetadata(metadata)** - Sets metadata of an entity. This method can be used to override the default metadata of an entity completely.

<a id="web-api-subresourcecontext-class"></a>

## SubresourceContext Class

The <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Processor/Subresource/GetSubresource/GetSubresourceContext.php" target="_blank">SubresourceContext</a> class is used as a superclass for the context classes of sub-resources related actions such as [get_subresource](#get-subresource-action), [get_relationship](#get-relationship-action), [update_relationship](#update-relationship-action), [add_relationship](#add-relationship-action) and
[delete_relationship](#delete-relationship-action). This class provides methods to work with parent entities in addition to the [Context](#context-class) class.

General methods:

- **getParentClassName()** - Retrieves the fully-qualified class name of the parent entity.
- **setParentClassName(className)** - Sets fully-qualified class name of the parent entity.
- **getParentId()** - Retrieves an identifier of the parent entity.
- **setParentId(parentId)** - Sets an identifier of the parent entity.
- **getAssociationName()** - Retrieves an association name that represents a relationship.
- **setAssociationName(associationName)** - Sets an association name that represents a relationship.
- **isCollection()** - Indicates an association that represents the “to-many” or “to-one” relationship.
- **setIsCollection(value)** - Sets a flag that indicates whether an association represents “to-many” or “to-one” relationship.
- **hasParentEntity()** - Checks whether the parent entity exists in the context.
- **getParentEntity()** - Retrieves the parent entity object.
- **setParentEntity(parentEntity)** - Sets the parent entity object.

Parent entity configuration-related methods:

- **getParentConfigExtras()** - Retrieves a <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Processor/Subresource/SubresourceContext.php" target="_blank">list of requests for configuration data</a> for the parent entity.
- **setParentConfigExtras(extras)** - Sets a list of requests for configuration data for the parent entity.
- **hasParentConfig()** - Checks whether a configuration of the parent entity exists.
- **getParentConfig()** - Retrieves a <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Config/EntityDefinitionConfig.php" target="_blank">configuration of the parent entity</a>
- **setParentConfig(config)** - Sets a custom configuration of the parent entity. This method can be used to completely override the default configuration of the parent entity.

Parent entity metadata-related methods:

- **getParentMetadataExtras()** - Retrieves a list of <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Metadata/Extra/MetadataExtraInterface.php" target="_blank">requests for additional metadata info</a> for the parent entity.
- **setParentMetadataExtras(extras)** - Sets a list of requests for additional metadata info for the parent entity.
- **hasParentMetadata()** - Checks whether metadata of the parent entity exists.
- **getParentMetadata()** - Retrieves <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Metadata/EntityMetadata.php" target="_blank">metadata of the parent entity</a>.
- **setParentMetadata(metadata)** - Sets metadata of the parent entity. This method can be used to override the default metadata of the parent entity completely.

<a id="web-api-actions-create"></a>

## Creating a New Action

To create a new action, to create two classes:

- **context** - This class represents a context in the scope of which an action is executed. An instance of this class is used to store the input and output data and share data between processors. This class must extend <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Processor/ApiContext.php" target="_blank">ApiContext</a>. Depending on your needs, you can use another classes derived from <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Processor/ApiContext.php" target="_blank">ApiContext</a>, for example <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Processor/Context.php" target="_blank">Context</a>, <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Processor/SingleItemContext.php" target="_blank">SingleItemContext</a> or <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Processor/ListContext.php" target="_blank">ListContext</a>.
- **main processor** - This class is the main entry point for an action and is responsible for creating an instance of the context class and executing all worker processors. This class must extend <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Component/ChainProcessor/ActionProcessor.php" target="_blank">ActionProcessor</a> and implement the `createContextObject` method. Depending on your needs, you can use other classes derived from <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Component/ChainProcessor/ActionProcessor.php" target="_blank">ActionProcessor</a>, for example, <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Processor/NormalizeResultActionProcessor.php" target="_blank">NormalizeResultActionProcessor</a>.

```php
namespace Acme\Bundle\DemoBundle\Api\Processor;

use Oro\Bundle\ApiBundle\Processor\ApiContext;

class MyActionContext extends ApiContext
{
}
```

```php
namespace Acme\Bundle\DemoBundle\Api\Processor;

use Oro\Component\ChainProcessor\ActionProcessor;

class MyActionProcessor extends ActionProcessor
{
    #[\Override]
    protected function createContextObject(): MyActionContext
    {
        return new MyActionContext();
    }
}
```

Additionally, you need to register your processor in the dependency injection container:

```yaml
acme.my_action.processor:
    class: Acme\Bundle\DemoBundle\Api\Processor\MyActionProcessor
    public: false
    arguments:
        - '@oro_api.processor_bag'
        - my_action # the name of an action
```

If you need to create groups for your action, register them in the ApiBundle configuration. To do this, add Resources/config/oro/app.yml to your bundle, for example:

```yaml
oro_api:
    actions:
        my_action:
            processing_groups:
                initialize:
                    priority: -10
                load_data:
                    priority: -20
                finalize:
                    priority: -30
```

#### NOTE
The `priority` attribute is used to control the order in which groups of processors are executed. The higher the priority, the earlier a group of processors is executed. The default value is 0. The possible range is from -254 to 252. For details on processor creation, see the [Processors](processors.md#web-api-processors) section.

<!-- Frontend -->
