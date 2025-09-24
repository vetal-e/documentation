<a id="web-api-testing"></a>

# Testing REST API

To ensure that your REST API resources work properly, cover them with <a href="https://oroinc.com/doc/orocrm/current/book/functional-tests" target="_blank">functional tests</a>. To simplify the creation of the functional test for REST API resources that conforms to <a href="http://jsonapi.org/format/" target="_blank">JSON:API specification</a>, the <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Tests/Functional/RestJsonApiTestCase.php" target="_blank">RestJsonApiTestCase</a> test case was created. The following table contains the list of the most valuable methods in this class:

| Method                                 | Description                                                                                                                                                                                                                                                                                                                                                                                |
|----------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| request                                | Sends a REST API request.                                                                                                                                                                                                                                                                                                                                                                  |
| options                                | Sends the OPTIONS request. See [options](actions.md#options-action) action.                                                                                                                                                                                                                                                                                                                |
| get                                    | Sends the GET request for a single entity. See [get](actions.md#get-action) action.                                                                                                                                                                                                                                                                                                        |
| cget                                   | Sends the GET request for a list of entities. See [get_list](actions.md#get-list-action) action.                                                                                                                                                                                                                                                                                           |
| post                                   | Sends the POST request for an entity resource. See [create](actions.md#create-action) action. If the second parameter is a file name, the file should be located in the `requests` directory next to the PHP file that contains the test.                                                                                                                                                  |
| patch                                  | Sends the PATCH request for a single entity. See [update](actions.md#update-action) action. If the second parameter is a file name, the file should be located in the `requests` directory next to the PHP file that contains the test.                                                                                                                                                    |
| cpatch                                 | Sends PATCH request for a list of entities. See [update_list](actions.md#update-list-action) action. If the second parameter is a file name, the file should be located in the `requests` directory near to PHP file containing the test.                                                                                                                                                  |
| delete                                 | Sends the DELETE request for a single entity. See [delete](actions.md#delete-action) action.                                                                                                                                                                                                                                                                                               |
| cdelete                                | Sends the DELETE request for a list of entities. See [delete_list](actions.md#delete-list-action) action.                                                                                                                                                                                                                                                                                  |
| getSubresource                         | Sends the GET request for a sub-resource of a single entity. See [get_subresource](actions.md#get-subresource-action) action.                                                                                                                                                                                                                                                              |
| postSubresource                        | Sends the POST request for a sub-resource of a single entity. See [add_relationship](actions.md#add-relationship-action) action.                                                                                                                                                                                                                                                           |
| patchSubresource                       | Sends the PATCH request for a sub-resource of a single entity. See [update_relationship](actions.md#update-action) action.                                                                                                                                                                                                                                                                 |
| deleteSubresource                      | Sends the DELETE request for a sub-resource of a single entity. See [delete_relationship](actions.md#delete-relationship-action) action.                                                                                                                                                                                                                                                   |
| getRelationship                        | Sends the GET request for a relationship of a single entity. See [get_relationship](actions.md#get-relationship-action) action.                                                                                                                                                                                                                                                            |
| postRelationship                       | Sends the POST request for a relationship of a single entity. See [add_relationship](actions.md#add-relationship-action) action.                                                                                                                                                                                                                                                           |
| patchRelationship                      | Sends the PATCH request for a relationship of a single entity. See [update_relationship](actions.md#update-relationship-action) action.                                                                                                                                                                                                                                                    |
| deleteRelationship                     | Sends the DELETE request for a relationship of a single entity. See [delete_relationship](actions.md#delete-relationship-action) action.                                                                                                                                                                                                                                                   |
| assertResponseContains                 | Asserts that the response content contains the given data. If the first parameter is a file name, the file should be located in the `responses` directory next to the PHP file that contains the test.                                                                                                                                                                                     |
| assertResponseCount                    | Asserts that the response contains the given number of data items.                                                                                                                                                                                                                                                                                                                         |
| assertResponseNotEmpty                 | Asserts that the response data are not empty.                                                                                                                                                                                                                                                                                                                                              |
| assertResponseNotHasAttributes         | Asserts that the response content does not contain the given attributes.                                                                                                                                                                                                                                                                                                                   |
| assertResponseNotHasRelationships      | Asserts that the response content does not contain the given relationships.                                                                                                                                                                                                                                                                                                                |
| assertResponseValidationError          | Asserts that the response content contains one validation error and it is the given error.                                                                                                                                                                                                                                                                                                 |
| assertResponseContainsValidationError  | Asserts that the response content contains the given validation error.                                                                                                                                                                                                                                                                                                                     |
| assertResponseValidationErrors         | Asserts that the response content contains the given validation errors and only them.                                                                                                                                                                                                                                                                                                      |
| assertResponseContainsValidationErrors | Asserts that the response content contains the given validation errors.                                                                                                                                                                                                                                                                                                                    |
| assertAllowResponseHeader              | Asserts `Allow` response header equals to the expected value.                                                                                                                                                                                                                                                                                                                              |
| assertMethodNotAllowedResponse         | Asserts response status code equals to 405 (Method Not Allowed) and `Allow` response header equals to the expected value.                                                                                                                                                                                                                                                                  |
| dumpYmlTemplate                        | Saves a response content to a YAML file. If the first parameter is a file name, the file is saved into the responses directory next to the PHP file that contains the test.                                                                                                                                                                                                                |
| getResourceId                          | Extracts the JSON:API resource identifier from the response. For details, see <a href="http://jsonapi.org/format/" target="_blank">JSON:API specification</a>.                                                                                                                                                                                                                             |
| getNewResourceIdFromIncludedSection    | Extracts the JSON:API resource identifier from the `included` section of the response. For details, see [Create and Update Related Resources Together with a Primary API Resource](../../api/create-update-related-resources.md#web-services-api-create-update-related-resources).                                                                                                         |
| getRequestData                         | Converts the given request to an array that can be sent to the server. The request can be a path to a file containing the request data or an array with the request data. If the request is a file name, the file should be located in the `requests` directory next to the PHP file that contains the test.                                                                               |
| getResponseData                        | Converts the given response to an array that can be used to compare it with a response received from the server. The given response can be a path to a file that contains the response data or an array with the response data. If the response is a file name, the file should be located in the `responses` directory next to the PHP file that contains the test.                       |
| getResponseErrors                      | Extracts the list of errors from the JSON:API response. For details, see <a href="http://jsonapi.org/format/" target="_blank">JSON:API specification</a>.                                                                                                                                                                                                                                  |
| updateResponseContent                  | Replaces all values in the given expected response content with the corresponding value from the actual response content when the key of an element is equal to the given key and the value of this element is equal to the given placeholder. If the first parameter is a file name, the file should be located in the `responses` directory next to the PHP file that contains the test. |
| getApiBaseUrl                          | Returns the base URL for all REST API requests, e.g. `http://localhost/api`.                                                                                                                                                                                                                                                                                                               |
| appendEntityConfig                     | Appends a configuration of an entity. This method is helpful when you create a general functionality and need to test it for different configurations without creating a test entity for each of them. Please note that the configuration is restored after each test, and thus, you do not need to do it manually.                                                                        |

#### NOTE
By default, HATEOAS is disabled in functional tests, although it is enabled by default in production and API Sandbox. It was done to avoid cluttering up the tests with HATEOAS links. In case you want to enable HATEOAS for your test, use HTTP_HATEOAS server parameter, e.g. `$this->cget(['entity' => 'products'], [], ['HTTP_HATEOAS' => true])`.

<a id="api-batch-api"></a>

## Testing Batch API

To simplify the creation of the functional test for REST Batch API resources that conforms to <a href="http://jsonapi.org/format/" target="_blank">JSON:API specification</a>,
the <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Tests/Functional/RestJsonApiUpdateListTestCase.php" target="_blank">RestJsonApiUpdateListTestCase</a> test case was created. The following table contains the list of the most valuable
methods in this class:

| Method            | Description                                                                                                                                                                             |
|-------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| processUpdateList | Sends PATCH request for a list of entities (see [update_list](actions.md#update-list-action) action) and executes all message queue processors required to process Batch API operation. |

Example of a functional test for Batch API:

```php
public function testCreateEntities()
{
    $this->processUpdateList(
        Lead::class,
        [
            'data' => [
                [
                    'type'       => 'leads',
                    'attributes' => ['name' => 'New Lead 1']
                ]
            ]
        ]
    );

    $response = $this->cget(['entity' => 'leads'], ['fields[leads]' => 'name']);
    $responseContent = $this->updateResponseContent(
        [
            'data' => [
                [
                    'type'       => 'leads',
                    'id'         => '<toString(@lead1->id)>',
                    'attributes' => ['name' => 'Existing Lead 1']
                ],
                [
                    'type'       => 'leads',
                    'id'         => 'new',
                    'attributes' => ['name' => 'New Lead 1']
                ]
            ]
        ],
        $response
    );
    $this->assertResponseContains($responseContent, $response);
}
```

To simplify the creation of the functional test for synchronous REST Batch API resources that conforms to <a href="http://jsonapi.org/format/" target="_blank">JSON:API specification</a>, use the <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Tests/Functional/RestJsonApiSyncUpdateListTestCase.php" target="_blank">RestJsonApiSyncUpdateListTestCase</a> test case.

Example of a functional test for synchronous Batch API:

```php
public function testCreateEntities()
{
    $response = $this->cpatch(
        ['entity' => 'leads'],
        [
            'data' => [
                [
                    'type'       => 'leads',
                    'attributes' => ['name' => 'New Lead 1']
                ]
            ]
        ],
        ['HTTP_X-Mode' => 'sync']
    );

    $responseContent = $this->updateResponseContent(
        [
            'data' => [
                [
                    'type'       => 'leads',
                    'id'         => 'new',
                    'attributes' => ['name' => 'New Lead 1']
                ]
            ]
        ],
        $response
    );
    $this->assertResponseContains($responseContent, $response);
}
```

<a id="api-load-fixtures"></a>

## Load Fixtures

You can use [Doctrine and Alice fixtures](../automated-tests/index.md#automated-test):

```php
class InventoryLevelTest extends RestJsonApiTestCase
{
    protected function setUp()
    {
        parent::setUp();
        $this->loadFixtures([@OroInventoryBundle/Tests/Functional/DataFixtures/inventory_level.yml']);
    }
```

Fixture file:

```yaml
dependencies:
  - Oro\Bundle\WarehouseBundle\Tests\Functional\DataFixtures\LoadWarehouseAndInventoryLevels

Oro\Bundle\InventoryBundle\Entity\InventoryLevel:
  warehouse_inventory_level.warehouse.1.product_unit_precision.product-1.primary_unit:
    warehouse: '@warehouse.1'
    productUnitPrecision: '@product-1->primaryUnitPrecision'
    quantity: 10
```

You can use the `dependencies` section if a fixture depends on another Doctrine or Alice fixtures. References are shared between Alice and Doctrine fixtures.

<a id="api-alice-references"></a>

## Alice References

You can use references in Alice fixtures:

```php
@product-1
```

Use methods of properties with references:

```php
@product-2->createdAt->format("Y-m-d\TH:i:s\Z")
```

<a id="yaml-templates"></a>

## YAML Templates

A YAML template is a regular YAML file. The only difference is that you can use references and fakers in values. They will be processed by Alice and replaced with the appropriate real values. For details, see the <a href="https://github.com/nelmio/alice/blob/master/doc/relations-handling.md#references" target="_blank">Alice documentation</a>.

<a id="assert-expectations"></a>

## Assert the Expectations

Assert the expected response by using YAML templates.

A YAML template:

```yaml
data:
    -
        type: inventorylevels
        id: '@warehouse_inventory_level.warehouse.1.product_unit_precision.product-1.liter->id'
        attributes:
            quantity: '10'
        relationships:
            product:
                data:
                    type: products
                    id: '@product-1->id'
            productUnitPrecision:
                data:
                    type: productunitprecisions
                    id: '@product_unit_precision.product-1.liter->id'
            warehouse:
                data:
                    type: warehouses
                    id: '@warehouse.1->id'
```

In php test:

```php
public function testGetList()
{
    $response = $this->cget(
        ['entity' => 'inventorylevels'],
        [
            'include' => 'product,productUnitPrecision',
            'filter' => [
                'product.sku' => '@product-1->sku',
            ]
        ]
    );

    $this->assertResponseContains('cget_filter_by_product.yml', $response);
}
```

<a id="yaml-templates-for-request-body"></a>

## YAML Templates for a Request Body

You can use an array with references for a request body:

```php
public function testUpdateEntity()
{
    $response = $this->patch(
        ['entity' => 'inventorylevels', 'product.sku' => '@product-1->sku'],
        [
            'data' => [
                'type' => 'inventorylevels',
                'id' => '<toString(@product-1->id)>',
                'attributes' => [
                    'quantity' => '17'
                ]
            ],
        ]
    );
}
```

Alternatively, you can store YAML in a `.yml` file:

```php
public function testCreateCustomer()
{
    $this->post(
        ['entity' => 'customers'],
        'create_customer.yml' // loads data from __DIR__ . '/requests/create_customer.yml'
    );
}
```

<a id="process-single-reference"></a>

## Process Single Reference

To process a single reference, for example, to compare it with another value:

```php
self::processTemplateData('@inventory_level.product_unit_precision.product-1.liter->quantity')
```

The `processTemplateData` method can process a string, an array, or a YAML file.

<a id="dump-response-into-yaml-template"></a>

## Dump the Response into a YAML Template

When you develop new tests for REST API, it may be convenient to dump responses into a YAML template:

```php
public function testGetList()
{
    $response = $this->cget(
        ['entity' => 'products'],
        ['filter' => ['sku' => '@product-1->sku']]
    );
    // dumps response content to __DIR__ . '/responses/' . 'test_cget_entity.yml'
    $this->dumpYmlTemplate('test_cget_entity.yml', $response);
}
```

#### IMPORTANT
Do not forget to check references after you dump a response for the first time: there can be collisions if references have the same IDs.

## Base Test Cases for Unit Tests

To simplify <a href="http://softwaretestingfundamentals.com/unit-testing/" target="_blank">unit testing</a> of [API processors](processors.md#web-api-processors), you can use one of the following base classes:

| Class                                                                                                                                                                                                                          | Action                                                                                                                                                                             |
|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Tests/Unit/Processor/Get/GetProcessorTestCase.php" target="_blank">GetProcessorTestCase</a>                                                   | [get](actions.md#get-action)                                                                                                                                                       |
| <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Tests/Unit/Processor/Get/GetProcessorOrmRelatedTestCase.php" target="_blank">GetProcessorOrmRelatedTestCase</a>                               | [get](actions.md#get-action)                                                                                                                                                       |
| <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Tests/Unit/Processor/GetList/GetListProcessorTestCase.php" target="_blank">GetListProcessorTestCase</a>                                       | [get_list](actions.md#get-list-action)                                                                                                                                             |
| <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Tests/Unit/Processor/GetList/GetListProcessorOrmRelatedTestCase.php" target="_blank">GetListProcessorOrmRelatedTestCase</a>                   | [get_list](actions.md#get-list-action)                                                                                                                                             |
| <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Tests/Unit/Processor/Create/CreateProcessorTestCase.php" target="_blank">CreateProcessorTestCase</a>                                          | [create](actions.md#create-action)                                                                                                                                                 |
| <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Tests/Unit/Processor/Update/UpdateProcessorTestCase.php" target="_blank">UpdateProcessorTestCase</a>                                          | [update](actions.md#update-action)                                                                                                                                                 |
| <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Tests/Unit/Processor/UpdateList/UpdateListProcessorTestCase.php" target="_blank">UpdateListProcessorTestCase</a>                              | [update_list](actions.md#update-list-action)                                                                                                                                       |
| <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Tests/Unit/Batch/Processor/Update/BatchUpdateProcessorTestCase.php" target="_blank">BatchUpdateProcessorTestCase</a>                          | [batch_update](actions.md#batch-update-action)                                                                                                                                     |
| <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Tests/Unit/Batch/Processor/UpdateItem/BatchUpdateItemProcessorTestCase.php" target="_blank">BatchUpdateItemProcessorTestCase</a>              | [batch_update_item](actions.md#batch-update-item-action)                                                                                                                           |
| <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Tests/Unit/Processor/FormProcessorTestCase.php" target="_blank">FormProcessorTestCase</a>                                                     | [create](actions.md#create-action), [update](actions.md#update-action)                                                                                                             |
| <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Tests/Unit/Processor/Delete/DeleteProcessorTestCase.php" target="_blank">DeleteProcessorTestCase</a>                                          | [delete](actions.md#delete-action)                                                                                                                                                 |
| <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Tests/Unit/Processor/DeleteList/DeleteListProcessorTestCase.php" target="_blank">DeleteListProcessorTestCase</a>                              | [delete_list](actions.md#delete-list-action)                                                                                                                                       |
| <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Tests/Unit/Processor/Subresource/GetSubresourceProcessorTestCase.php" target="_blank">GetSubresourceProcessorTestCase</a>                     | [get_subresource](actions.md#get-subresource-action), [get_relationship](actions.md#get-relationship-action)                                                                       |
| <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Tests/Unit/Processor/Subresource/GetSubresourceProcessorOrmRelatedTestCase.php" target="_blank">GetSubresourceProcessorOrmRelatedTestCase</a> | [get_subresource](actions.md#get-subresource-action), [get_relationship](actions.md#get-relationship-action)                                                                       |
| <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Tests/Unit/Processor/Subresource/ChangeSubresourceProcessorTestCase.php" target="_blank">ChangeSubresourceProcessorTestCase</a>               | [update_subresource](actions.md#update-subresource-action), [add_subresource](actions.md#add-subresource-action), [delete_subresource](actions.md#delete-subresource-action)       |
| <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Tests/Unit/Processor/Subresource/ChangeRelationshipProcessorTestCase.php" target="_blank">ChangeRelationshipProcessorTestCase</a>             | [update_relationship](actions.md#update-relationship-action), [add_relationship](actions.md#add-relationship-action), [delete_relationship](actions.md#delete-relationship-action) |
| <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Tests/Unit/Processor/CustomizeLoadedData/CustomizeLoadedDataProcessorTestCase.php" target="_blank">CustomizeLoadedDataProcessorTestCase</a>   | [customize_loaded_data](actions.md#customize-loaded-data-action)                                                                                                                   |
| <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Tests/Unit/Processor/CustomizeFormData/CustomizeFormDataProcessorTestCase.php" target="_blank">CustomizeFormDataProcessorTestCase</a>         | [customize_form_data](actions.md#customize-form-data-action)                                                                                                                       |
| <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Tests/Unit/Processor/Options/OptionsProcessorTestCase.php" target="_blank">OptionsProcessorTestCase</a>                                       | [options](actions.md#options-action)                                                                                                                                               |
| <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Tests/Unit/Processor/UnhandledError/UnhandledErrorProcessorTestCase.php" target="_blank">UnhandledErrorProcessorTestCase</a>                  | [unhandled_error](actions.md#unhandled-error-action)                                                                                                                               |
| <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Tests/Unit/Processor/GetConfig/ConfigProcessorTestCase.php" target="_blank">ConfigProcessorTestCase</a>                                       | [get_config](actions.md#get-config-action)                                                                                                                                         |
| <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Tests/Unit/Processor/GetMetadata/MetadataProcessorTestCase.php" target="_blank">MetadataProcessorTestCase</a>                                 | [get_metadata](actions.md#get-metadata-action)                                                                                                                                     |
<!-- Frontend -->
