<a id="web-api-how-to"></a>

# How to

## Turn on API for an Entity

By default, API for entities is disabled. To turn on API for an entity, add the entity to Resources/config/oro/api.yml in your bundle:

```yaml
api:
    entities:
        Acme\Bundle\DemoBundle\Entity\SomeEntity: ~
```

<a id="turn-on-api-for-entity-disabled-in-entity-yml"></a>

## Turn on API for an Entity Disabled in Resources/config/oro/entity.yml

The `exclusions` section of the Resources/config/oro/entity.yml configuration file is used to make an entity or a field inaccessible for a user. The entities and fields from this section are inaccessible via the API as well. However, it is possible to override this rule for the API. To do this, use the `exclude` option in Resources/config/oro/api.yml.

Let us consider the case when you have the following Resources/config/oro/entity.yml:

```yaml
oro_entity:
    exclusions:
        - { entity: Acme\Bundle\DemoBundle\Entity\SomeEntity1 }
        - { entity: Acme\Bundle\DemoBundle\Entity\SomeEntity2, field: field1 }
```

To override these rules in the API, add the following lines to the Resources/config/oro/api.yml:

```yaml
api:
    entities:
        Acme\Bundle\DemoBundle\Entity\SomeEntity1:
            exclude: false # override exclude rule from entity.yml
        Acme\Bundle\DemoBundle\Entity\SomeEntity2:
            fields:
                field1:
                    exclude: false # override exclude rule from entity.yml
```

<a id="advanced-operators-for-string-filter"></a>

## Enable Advanced Operators for String Filter

By performance reasons the following operators are disabled out of the box:

* `~` (`contains`) - uses `LIKE %text%` to check that a field value contains the text
* `!~` (`not_contains`) - uses `NOT LIKE %text%` to check that a field value does not contain the text
* `^` (`starts_with`) - uses `LIKE text%` to check that a field value starts with the text
* `!^` (`not_starts_with`) - uses `NOT LIKE text%` to check that a field value does not start with the text
* `$` (`ends_with`) - uses `LIKE %text` to check that a field value ends with the text
* `!$` (`not_ends_with`) - uses `NOT LIKE %text` to check that a field value does not end with the text
* N/A (`empty`) - uses logical OR operator to check that a field value is empty or `null` or to check that a field value is not empty and not `null`

To enable these operators, use `operators` option for filters in Resources/config/oro/api.yml, e.g.:

```yaml
api:
    entities:
        Acme\Bundle\DemoBundle\Entity\SomeEntity1:
            filters:
                fields:
                    field1:
                        operators: ['=', '!=', '*', '!*', '~', '!~', '^', '!^', '$', '!$', 'empty']
```

<a id="case-insensitive-string-filter"></a>

## Enable Case-insensitive String Filter

Depending on the <a href="https://en.wikipedia.org/wiki/Collation" target="_blank">collation</a> settings of your database the case-insensitive filtering may be already enforced to be used on the database level. For example, if you are using PostgreSQL all collations are case-sensitive by default. But if the collation of your database or a particular field is not case-insensitive and you need to enable the case-insensitive filtering for this field, you can use `case_insensitive` option for a filter in Resources/config/oro/api.yml, e.g.:

```yaml
api:
    entities:
        Acme\Bundle\DemoBundle\Entity\SomeEntity1:
            filters:
                fields:
                    field1:
                        options:
                            case_insensitive: true
```

#### NOTE
Please note that the `LOWER` function will be used in this case and it can impact performance if there is no <a href="https://use-the-index-luke.com/sql/where-clause/functions/case-insensitive-search" target="_blank">proper index</a>.

Also sometimes data in the database are already converted to lowercase or uppercase, in this case you can use `value_transformer` option to convert the filter value to before it will be passed to the database query, e.g.:

```yaml
api:
    entities:
        Acme\Bundle\DemoBundle\Entity\SomeEntity1:
            filters:
                fields:
                    field1:
                        options:
                            value_transformer: strtoupper # convert the filter value to uppercase
```

<a id="change-acl-for-action"></a>

## Change an ACL Resource for an Action

By default, the following permissions are used to restrict access to an entity in a scope of the specific action (see the [actions topic](actions.md#web-api-actions) for more details on each action):

| Action              | Permission      |
|---------------------|-----------------|
| get                 | VIEW            |
| get_list            | VIEW            |
| delete              | DELETE          |
| delete_list         | DELETE          |
| create              | CREATE and VIEW |
| update              | EDIT and VIEW   |
| get_subresource     | VIEW            |
| get_relationship    | VIEW            |
| update_relationship | EDIT and VIEW   |
| add_relationship    | EDIT and VIEW   |
| delete_relationship | EDIT and VIEW   |

If you want to change permission or disable access checks for some action, you can use the `acl_resource` option of the `actions` configuration section.

For example, to change permissions for the `delete` action, add the following lines to the Resources/config/oro/api.yml in your bundle:

```yaml
api:
    entities:
        Acme\Bundle\DemoBundle\Entity\SomeEntity1:
            actions:
                delete:
                    acl_resource: access_entity_view
```

If there is the `access_entity_view` ACL resource:

```yaml
access_entity_view:
    type: entity
    class: Acme\Bundle\DemoBundle\Entity\SomeEntity1
    permission: VIEW
```

As a result, the `VIEW` permission will be used instead of the `DELETE` permission.

<a id="disable-access-check-for-action"></a>

## Disable Access Checks for an Action

You can disable access checks for some action by setting `null` as a value for the `acl_resource` option in Resources/config/oro/api.yml:

```yaml
api:
    entities:
        Acme\Bundle\DemoBundle\Entity\SomeEntity1:
            actions:
                get_list:
                    acl_resource: ~
```

<a id="disable-entity-action"></a>

## Disable an Entity Action

When you add an entity to the API, all the actions will be available by default.

If an action should be inaccessible, disable it in Resources/config/oro/api.yml:

```yaml
api:
    entities:
        Acme\Bundle\DemoBundle\Entity\SomeEntity1:
            actions:
                delete:
                    exclude: true
```

You can use the short syntax:

```yaml
api:
    entities:
        Acme\Bundle\DemoBundle\Entity\SomeEntity1:
            actions:
                delete: false
```

<a id="max-number-of-entities-to-be-deleted"></a>

## Change the Maximum Number of Entities that Can Be Deleted by One Request

By default, the `delete_list` action can delete not more than 100 entities, see the `max_delete_entities` option in [General Configuration](configuration-general.md#web-api-configuration-general). This limit is set by the <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Processor/DeleteList/SetDeleteLimit.php" target="_blank">SetDeleteLimit</a> processor.

If your want to use another limit, set it using the `max_results` option in Resources/config/oro/api.yml:

```yaml
api:
    entities:
        Acme\Bundle\DemoBundle\Entity\SomeEntity1:
            actions:
                delete_list:
                    max_results: 200
```

You can remove the limit at all. To do this, set `-1` as a value for the `max_results` option:

```yaml
api:
    entities:
        Acme\Bundle\DemoBundle\Entity\SomeEntity1:
            actions:
                delete_list:
                    max_results: -1
```

<a id="configure-nested-object"></a>

## Configure a Nested Object

Sometimes it is required to group several fields and expose them as a nested object in the API. For example,
consider the case when an entity has two fields `intervalNumber` and `intervalUnit` but you need to expose them
in API as `number` and `unit` properties of `interval` field. To achieve it, use one of the following
configurations:

```yaml
api:
    entities:
        Oro\Bundle\ReminderBundle\Entity\Reminder:
            fields:
                interval:
                    data_type: nestedObject
                    form_options:
                        data_class: Oro\Bundle\ReminderBundle\Model\ReminderInterval
                        by_reference: false
                    fields:
                        number:
                            property_path: intervalNumber
                        unit:
                            property_path: intervalUnit
                intervalNumber:
                    exclude: true
                intervalUnit:
                    exclude: true
```

Please note that in this case an entity, in this example *Oro\\Bundle\\ReminderBundle\\Entity\\Reminder*, should have
`setInterval(ReminderInterval $interval)` method. This method is called by [create](actions.md#create-action)
and [update](actions.md#update-action) actions to set the nested object. If an entity does not have this method,
the following alternative configuration with the `inherit_data` form option can be used:

```yaml
api:
    entities:
        Oro\Bundle\ReminderBundle\Entity\Reminder:
            fields:
                interval:
                    data_type: nestedObject
                    form_options:
                        inherit_data: true
                    fields:
                        number:
                            property_path: intervalNumber
                        unit:
                            property_path: intervalUnit
                intervalNumber:
                    exclude: true
                intervalUnit:
                    exclude: true
```

To make a nested object read-only the `mapped` form option can be used, for example:

```yaml
api:
    entities:
        Oro\Bundle\ReminderBundle\Entity\Reminder:
            fields:
                interval:
                    data_type: nestedObject
                    form_options:
                        inherit_data: true
                        mapped: true
                    fields:
                        number:
                            property_path: intervalNumber
                        unit:
                            property_path: intervalUnit
                intervalNumber:
                    exclude: true
                intervalUnit:
                    exclude: true
```

Here is an example how the nested objects looks in JSON:API:

```json
{
  "data": {
    "type": "reminders",
    "id": "1",
    "attributes": {
      "interval": {
        "number": 2,
        "unit": "H"
      }
    }
  }
}
```

<a id="configure-nested-association"></a>

## Configure a Nested Association

Sometimes a relationship with a group of entities is implemented as two fields, “entityClass” and “entityId”,
rather than a [multi-target associations](../entities/extend-entities/multi-target-associations.md#book-entities-extended-entities-multi-target-associations).
But in the API these fields should be represented as a regular relationship. To achieve this, a special data type
named `nestedAssociation` was implemented. For example, let us suppose that an entity has two fields
`sourceEntityClass` and `sourceEntityId` and you need to expose them in API as `source` relationship.
To achieve this, use the following configuration:

```yaml
api:
    entities:
        Oro\Bundle\OrderBundle\Entity\Order:
            fields:
                source:
                    data_type: nestedAssociation
                    fields:
                        __class__:
                            property_path: sourceEntityClass
                        id:
                            property_path: sourceEntityId
```

Here is an example how the nested association looks in JSON:API:

```json
{
    "data": {
        "type": "orders",
        "id": "1",
        "relationships": {
            "source": {
                "type": "contacts",
                "id": 123
            }
        }
    }
}
```

#### NOTE
Please note that fields used in a nested association, in this example `sourceEntityClass` and `sourceEntityId`, are automatically excluded from the result and you do not need to mark them with `exclude` option. Moreover, they will be excluded even if you mark them with `exclude: false` in a configuration file.

<a id="extended-many-to-one-association"></a>

## Configure an Extended Many-To-One Association

For information about this type of associations,
see the [multi-target associations](../entities/extend-entities/multi-target-associations.md#book-entities-extended-entities-multi-target-associations) topic.

Depending on the current entity configuration, each association resource (e.g. attachment) can be assigned to one of the resources (e.g. user, account, contact) that support such associations.

By default, there is no possibility to retrieve targets of such associations. To make targets available for retrieving, enable this in Resources/config/oro/api.yml:

```yaml
api:
    entities:
        Oro\Bundle\AttachmentBundle\Entity\Attachment:
            fields:
                target:
                    data_type: association:manyToOne
```

After applying the configuration, the `targets` relationship becomes available for the [get_list](actions.md#get-list-action), [get](actions.md#get-action), [create](actions.md#create-action) and [update](actions.md#update-action) actions. Also the `targets` relationship becomes also available as a subresource and thus, it is possible to perform the [get_subresource](actions.md#get-subresource-action), [get_relationship](actions.md#get-relationship-action), [update_relationship](actions.md#update-relationship-action), [add_relationship](actions.md#add-relationship-action) and [delete_relationship](actions.md#delete-relationship-action) actions.

The `data_type` parameter has format: `association:relationType:associationKind`, where

- `relationType` part should have ‘manyToOne’ value for extended Many-To-One association;
- `associationKind` is the optional part that represents the kind of the association.

<a id="extended-many-to-many-association"></a>

## Configure an Extended Many-To-Many Association

For information about this type of associations,
see the [multi-target associations](../entities/extend-entities/multi-target-associations.md#book-entities-extended-entities-multi-target-associations) topic.

Depending on the current entity configuration, each association resource (e.g. call) can be assigned to several resources (e.g. user, account, contact) that support such associations.

By default, there is no possibility to retrieve targets of such associations. To make targets available for retrieving, enable this in Resources/config/oro/api.yml, for instance:

```yaml
api:
    entities:
        Oro\Bundle\CallBundle\Entity\Call:
            fields:
                targets:
                    data_type: association:manyToMany
```

After applying the configuration, the `targets` relationship becomes available in scope of the [get_list](actions.md#get-list-action), [get](actions.md#get-action) , [create](actions.md#create-action) and [update](actions.md#update-action) actions. The `targets` relationship also becomes  available as a subresource and thus, it is possible to perform [get_subresource](actions.md#get-subresource-action), [get_relationship](actions.md#get-relationship-action), [update_relationship](actions.md#update-relationship-action), [add_relationship](actions.md#add-relationship-action) and [delete_relationship](actions.md#delete-relationship-action) actions.

The `data_type` parameter has format: `association:relationType:associationKind`, where

- `relationType` part should have ‘manyToMany’ value for extended Many-To-Many association;
- `associationKind` is the optional part that represents the kind of the association.

<a id="extended-multiple-many-to-one-association"></a>

## Configure an Extended Multiple Many-To-One Association

For information about this type of associations,
see the [multi-target associations](../entities/extend-entities/multi-target-associations.md#book-entities-extended-entities-multi-target-associations) topic.

Depending on the current entity configuration, each association resource (e.g. call) can be assigned to several resources (e.g. user, account, contact) that support such associations. However, in case of multiple many-to-one association, a resource can be associated with only one other resource of each type. For example, a call can be associated only with one user, one account, etc.

By default, there is no possibility to retrieve targets of such associations. To make targets available for retrieving, enable this in Resources/config/oro/api.yml, for instance:

```yaml
api:
    entities:
        Oro\Bundle\CallBundle\Entity\Call:
            fields:
                targets:
                    data_type: association:multipleManyToOne
```

After applying the configuration, the `targets` relationship becomes available in scope of [get_list](actions.md#get-list-action), [get](actions.md#get-action), [create](actions.md#create-action) and [update](actions.md#update-action) actions. The `targets` relationship also becomes  available as a subresource and thus, it is possible to perform [get_subresource](actions.md#get-subresource-action), [get_relationship](actions.md#get-relationship-action), [update_relationship](actions.md#update-relationship-action), [add_relationship](actions.md#add-relationship-action) and [delete_relationship](actions.md#delete-relationship-action) actions.

The `data_type` parameter has format: `association:relationType:associationKind`, where

- `relationType` part should have ‘multipleManyToOne’ value for extended Multiple Many-To-One association;
- `associationKind` is the optional part that represents the kind of the association.

<a id="configure-unidirectional-association"></a>

## Configure an Unidirectional Association

To add an ORM association that is the inverse side of an unidirectional association to the API, use a special `unidirectionalAssociation` data type. Its full definition is `unidirectionalAssociation:targetAssociationName`, where `targetAssociationName` is the name of the owning side association. To specify the entity that contains the owning side association, use the `target_class` option.

To illustrate the configuration of an unidirectional association, consider two entities, `Product` and `Category`. The `Product` entity has the `category` association that is an unidirectional many-to-one association to the `Category` entity. To add products to the `Category` API resource, use the following Resources/config/oro/api.yml:

```yaml
api:
    entities:
        Acme\Bundle\DemoBundle\Entity\Category:
            fields:
                products:
                    data_type: unidirectionalAssociation:category
                    target_class: Acme\Bundle\DemoBundle\Entity\Product
```

#### NOTE
Only one-to-one, many-to-one and many-to-many unidirectional associations are supported.

#### NOTE
This data type is not supported for models that replace ORM entities.

<a id="add-custom-controller"></a>

## Add a Custom Controller

By default, all REST API resources are handled by <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Controller/RestApiController.php" target="_blank">RestApiController</a> that handles [get_list](actions.md#get-list-action), [get](actions.md#get-action), [delete](actions.md#delete-action), [delete_list](actions.md#delete-list-action), [create](actions.md#create-action), [update](actions.md#update-action), [get_subresource](actions.md#get-subresource-action), [get_relationship](actions.md#get-relationship-action), [update_relationship](actions.md#update-relationship-action), [add_relationship](actions.md#add-relationship-action) and [delete_relationship](actions.md#delete-relationship-action) actions.

If this controller cannot handle the implementation of your REST API resources, you can register a custom controller. Please note that this is not recommended and should be used only in very special cases. Having a custom controller implies that many processes should be implemented from scratch, including:

> * extracting and validation of the input data
> * building and formatting the output document
> * error handling
> * loading data from the database
> * saving data to the database
> * implementing relationships with other API resources
> * documenting such API resources
> * implementing OPTIONS HTTP method for such API resources

If you know about these disadvantages and still want to proceed registering a custom controller, perform the following steps:

> 1. Create a controller.
> 2. Register the created controller using the Resources/config/oro/routing.yml configuration file.

Here is an example of the controller:

```php
namespace Acme\Bundle\DemoBundle\Controller\Api;

use Nelmio\ApiDocBundle\Annotation\ApiDoc;
use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\HttpFoundation\Response;

class MyResourceController extends AbstractController
{
    /**
     * Retrieve a specific record.
     *
     * @param Request $request
     *
     * @ApiDoc(
     *     resource=true,
     *     description="Get a resource",
     *     views={"rest_json_api"},
     *     section="myresources",
     *     requirements={
     *          {
     *              "name"="id",
     *              "dataType"="integer",
     *              "requirement"="\d+",
     *              "description"="The 'id' requirement description."
     *          }
     *     },
     *     filters={
     *          {
     *              "name"="aFilter",
     *              "dataType"="string",
     *              "requirement"=".+",
     *              "description"="The 'aFilter' filter description."
     *          }
     *     },
     *     output={
     *          "class"="Your\Namespace\Class",
     *          "fields"={
     *              {
     *                  "name"="aField",
     *                  "dataType"="string",
     *                  "description"="The 'aField' field description."
     *              }
     *          }
     *     },
     *     statusCodes={
     *          200="Returned when successful",
     *          500="Returned when an unexpected error occurs"
     *     }
     * )
     *
     * @return Response
     */
    public function getAction(Request $request): Response
    {
        // add an implementation here
    }
}
```

An example of the Resources/config/oro/routing.yml configuration file:

```yaml
acme_api_get_my_resource:
    path: '%oro_api.rest.prefix%myresources/{id}'
    methods: [GET]
    defaults:
        _controller: AcmeDemoBundle:Api\MyResource:get
    options:
        group: rest_api
```

For the information on the `ApiDoc` annotation, see <a href="https://symfony.com/bundles/NelmioApiDocBundle/current/index.html" target="_blank">the Symfony documentation</a>. To learn about all possible properties of the `fields` option, see <a href="https://github.com/nelmio/NelmioApiDocBundle/blob/2.x/Formatter/AbstractFormatter.php" target="_blank">AbstractFormatter class in NelmioApiDocBundle</a>. Please note that the `fields` option can be used inside the `input` and `output` options.

#### NOTE
Please note that the `all` view name can be used in the `ApiDoc` annotation to add the API resource into all views.

Use the [oro:api:doc:cache:clear](commands.md#oroapidoccacheclear-command) command to apply changes in the `ApiDoc` annotation to [API Sandbox](../../api/sandbox.md#web-services-api-sandbox).

<a id="add-custom-route"></a>

## Add a Custom Route

As described in [Add a Custom Controller](), <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Controller/RestApiController.php" target="_blank">RestApiController</a> handles all registered REST API resources, and in most cases you do not need to change this.
But sometimes you need to change the default mapping between URI and an action of this controller for some
REST API resources.
For example, imagine that URI of the REST API resource for the registered user’s profile is `/api/userprofile`. If you take a look at <a href="https://github.com/oroinc/platform/tree/master/src/Oro/Bundle/ApiBundle/Resources/config/oro/routing.yml" target="_blank">routing.yml</a>, you will see that this URI is matched by the `/api/{entity}` pattern, but the action that handles this
pattern works with a list of entities, not with a single entity. The challenge is to map `/api/userprofile` to the `Oro\Bundle\ApiBundle\Controller\RestApiController::itemAction` action that works with a single entity and to remove handling of
`/api/userprofile/{id}`. This can be achieved using own route definition with the `override_path` option.

Use [oro:api:doc:cache:clear](commands.md#oroapidoccacheclear) command to apply changes in `ApiDoc` annotation to [API Sandbox](../../api/sandbox.md#web-services-api-sandbox).

Here is an example of the Resources/config/oro/routing.yml configuration file:

```yaml
acme_rest_api_user_profile:
    path: '%oro_api.rest.prefix%userprofile'
    controller: Oro\Bundle\ApiBundle\Controller\RestApiController::itemAction
    defaults:
        entity: userprofile
    options:
        group: rest_api
        override_path: '%oro_api.rest.prefix%userprofile/{id}'
```

<a id="configure-upsert-operation"></a>

## Configure Upsert Operation

By default, [the upsert operation](../../api/upsert-operation.md#web-services-api-upsert-operation) is enabled for API resources based on
ORM entities that have no auto-generated identifier or have unique database indexes.

If the upsert operation is enabled for an API resource by default, but you need to disable it, use
the “upsert” configuration option in Resources/config/oro/api.yml :

```yaml
api:
    entities:
        Acme\Bundle\DemoBundle\Entity\SomeEntity:
            upsert: false
```

or

```yaml
api:
    entities:
        Acme\Bundle\DemoBundle\Entity\SomeEntity:
            upsert:
                disable: true
```

If the upsert operation is disabled for an API resource by default, it can be enabled in Resources/config/oro/api.yml
by specifying the field(s) that can be used to identify a resource:

```yaml
api:
    entities:
        Acme\Bundle\DemoBundle\Entity\SomeEntity:
            upsert:
                add: [['field1'], ['field2', 'field3']]
```

If you need to disable identification of a resource by some field(s), you can use “remove” option:

```yaml
api:
    entities:
        Acme\Bundle\DemoBundle\Entity\SomeEntity:
            upsert:
                remove: [['field4'], ['field5', 'field6']]
```

Also the identification field(s) can be completely replaced:

```yaml
api:
    entities:
        Acme\Bundle\DemoBundle\Entity\SomeEntity:
            upsert:
                replace: [['field7'], ['field8', 'field9']]
```

To check which entities support the upsert operation,
the [oro:api:dump](commands.md#oroapidump-command) command with `--upsert` option can be used:

```none
php bin/console oro:api:dump --upsert
```

<a id="configure-validate-operation"></a>

## Configure Validate Operation

By default, [the validate operation](../../api/validate-operation.md#web-services-api-validate-operation) is disabled for API resources.

If the validate operation is disabled for an API resource by default, but you need to enable it, use
the “enable_validation” configuration option in Resources/config/oro/api.yml :

```yaml
api:
    entities:
        Acme\Bundle\DemoBundle\Entity\SomeEntity:
            enable_validation: true
```

#### NOTE
Please be aware that when validation requests are processed, the `rollback_validated_request` event is dispatched instead of `post_flush_data` and `post_save_data`. Additionally, make sure that no extra actions, such as data indexing or sending emails, are performed when using validation requests. These extra actions can significantly impact the application’s performance. For example, if data is indexed but not stored in the database, or if an email is sent for a record that was not created or updated in the database.

To check which entities support the validate operation, use
the [oro:api:dump](commands.md#oroapidump-command) command with `--validate` option:

```none
php bin/console oro:api:dump --validate
```

<a id="using-a-non-primary-key-to-identify-an-entity"></a>

## Using a Non-Primary Key to Identify an Entity

By default, a primary key is used to identify ORM entities in API. If you need another field as an identifier, specify it using the `identifier_field_names` option.

For example, let your entity has the `id` field that is the primary key and the `uuid` field that contains a unique value for each entity. To use the `uuid` field to identify the entity, add the following details to Resources/config/oro/api.yml:

```yaml
api:
    entities:
        Acme\Bundle\DemoBundle\Entity\SomeEntity:
            identifier_field_names: ['uuid']
```

You can also exclude the `id` field (primary key) if you do not want to expose it via API:

```yaml
api:
    entities:
        Acme\Bundle\DemoBundle\Entity\SomeEntity:
            identifier_field_names: ['uuid']
            fields:
                id:
                    exclude: true
```

The default description for all resource identifiers is “The unique identifier of a resource.”. If you need another description, specify it using the `identifier_description` option, e.g.:

```yaml
api:
    entities:
        Acme\Bundle\DemoBundle\Entity\SomeEntity:
            identifier_field_names: ['uuid']
            identifier_description: 'The unique identifier of a resource. It is a UUID value.'
```

<a id="api-for-entity-wo-id"></a>

## Enable API for an Entity Without Identifier

Sometimes, it is required to create API resource that does not have an identifier. An example of such API resources can be resources for registering a new account or logging in a user.

The following steps describe how to create such API resources:

1. Create a PHP class to represent the API resource. Usually, such classes are named as models and located in the `Api/Model` directory. For example:
   > ```php
   > namespace Acme\Bundle\DemoBundle\Api\Model;

   > class Account
   > {
   >     private ?string $name;

   >     public function getName(): ?string
   >     {
   >         return $this->name;
   >     }

   >     public function setName(?string $name): void
   >     {
   >         $this->name = $name;
   >     }
   > }
   > ```
2. Describe the model in the Resources/config/oro/api.yml configuration file in your bundle, e.g.:
   > ```yaml
   > api:
   >     entity_aliases:
   >         Acme\Bundle\DemoBundle\Api\Model\Account:
   >             alias: registeraccount
   >             plural_alias: registeraccount
   >     entities:
   >         Acme\Bundle\DemoBundle\Api\Model\Account:
   >             fields:
   >                 name:
   >                     data_type: string
   >                     description: The user name
   >                     form_options:
   >                         constraints:
   >                             - NotBlank: ~
   >             actions:
   >                 create:
   >                     description: Register a new account
   >                 get: false
   >                 update: false
   >                 delete: false
   > ```
3. Register a route in the Resources/config/oro/routing.yml configuration file in your bundle using the `Oro\Bundle\ApiBundle\Controller\RestApiController::itemWithoutIdAction` as a controller, e.g.,:
   > ```yaml
   > acme_rest_api_register_account:
   >     path: '%oro_api.rest.prefix%registeraccount'
   >     controller: Oro\Bundle\ApiBundle\Controller\RestApiController::itemWithoutIdAction
   >     defaults:
   >         entity: registeraccount
   >     options:
   >         group: rest_api
   > ```
4. Create a processor to handle data, e.g.:
   > ```php
   > namespace Acme\Bundle\DemoBundle\Api\Processor;

   > use Acme\Bundle\DemoBundle\Api\Model\Account;
   > use Oro\Component\ChainProcessor\ContextInterface;
   > use Oro\Component\ChainProcessor\ProcessorInterface;

   > class RegisterAccount implements ProcessorInterface
   > {
   >     #[\Override]
   >     public function process(ContextInterface $context): void
   >     {
   >         /** @var Account $account */
   >         $account = $context->getResult();

   >         // implement registration of a new account
   >     }
   > }
   > ```
5. Register a processor in the dependency injection container, e.g.:
   > ```yaml
   > services:
   >    acme.api.register_account:
   >        class: Acme\Bundle\DemoBundle\Api\Processor\RegisterAccount
   >        tags:
   >            - { name: oro.api.processor, action: create, group: save_data, class: Acme\Bundle\DemoBundle\Api\Model\Account }
   > ```

<a id="enable-custom-api"></a>

## Enable Custom API

Before you begin, ensure that you are familiar with [The Request Type](request-type.md#api-request-type).

Let us consider a case when you need API for integration with some ERP system. In this case,
to simplify the development and to avoid unnecessary API calls, your API resources should have the same identifiers as the ERP system. The easiest way to achieve this is to create the `erpId` field for each entity and map this field as the identifier of API resource via the [identifier_field_names](#using-a-non-primary-key-to-identify-an-entity) configuration option. But the drawback of this
approach is that you have to change existing API, and as the result, it may lead to failure of existing API clients.
To avoid this, you can keep existing API unchanged and create a new type of API that will have all features
of existing API and will have modifications specific for this new integration as well.

To do this, you need to perform the following:

1. Decide how the API clients should inform server that they need to work with a new type of API. The simplest way is to use a custom HTTP header. If a client sends this header, it will work with new API, if it does not it will work with already existing API. Lets assume that we will use `X-Integration-Type` header to switch API types. If this header is sent and its value is `ERP` the new API will be used; otherwise, the already existing API will be used.
2. Decide which name of the request type you will use for the new API. Lets assume it will be `erp`.
3. Decide which name of API configuration files you will use to add modifications specific for the new API. Let’s assume, it will be `api_erp.yml`.
4. Add the new type of API to ApiBundle and configure API Sandbox via Resources/config/oro/app.yml configuration file in your bundle:
   > ```yaml
   > oro_api:
   >    # add API type for ERP integration
   >    config_files:
   >        erp:
   >            # load API configuration for ERP integration from two types of files, api_erp.yml and api.yml
   >            # the first file has higher priority and any configuration in this file will override
   >            # configuration from the second one
   >            file_name: [api_erp.yml, api.yml]
   >            # use this configuration only if ERP integration API is requested
   >            request_type: ['erp']

   >    # configure API Sandbox
   >    api_doc_views:
   >        erp_rest_json_api:
   >            label: ERP Integration
   >            underlying_view: rest_json_api
   >            headers:
   >                X-Integration-Type: ERP
   >            request_type: ['rest', 'json_api', 'erp']
   > ```
5. Create a processor that will check the request header and add `erp` request type to the execution context of processors:
   > ```php
   > namespace Acme\Bundle\DemoBundle\Api\Processor;

   > use Oro\Bundle\ApiBundle\Processor\Context;
   > use Oro\Component\ChainProcessor\ContextInterface;
   > use Oro\Component\ChainProcessor\ProcessorInterface;

   > class CheckErpRequestType implements ProcessorInterface
   > {
   >     private const REQUEST_HEADER_NAME = 'X-Integration-Type';
   >     private const REQUEST_HEADER_VALUE = 'ERP';
   >     private const REQUEST_TYPE = 'erp';

   >     #[\Override]
   >     public function process(ContextInterface $context): void
   >     {
   >         /** @var Context $context */
   >         $requestType = $context->getRequestType();
   >         if (!$requestType->contains(self::REQUEST_TYPE)
   >             && self::REQUEST_HEADER_VALUE === $context->getRequestHeaders()->get(self::REQUEST_HEADER_NAME)
   >         ) {
   >             $requestType->add(self::REQUEST_TYPE);
   >         }
   >     }
   > }
   > ```
6. Register this processor in the dependency injection container in the Resources/config/services.yml file:
   > ```yaml
   > acme.api.erp.check_erp_request_type:
   >     class: Acme\Bundle\DemoBundle\Api\Processor\CheckErpRequestType
   >     tags:
   >         - { name: oro.api.processor, action: get, group: initialize, priority: 250 }
   >         - { name: oro.api.processor, action: get_list, group: initialize, priority: 250 }
   >         - { name: oro.api.processor, action: delete, group: initialize, priority: 250 }
   >         - { name: oro.api.processor, action: delete_list, group: initialize, priority: 250 }
   >         - { name: oro.api.processor, action: create, group: initialize, priority: 250 }
   >         - { name: oro.api.processor, action: update, group: initialize, priority: 250 }
   >         - { name: oro.api.processor, action: update_list, group: initialize, priority: 250 }
   >         - { name: oro.api.processor, action: get_subresource, group: initialize, priority: 250 }
   >         - { name: oro.api.processor, action: delete_subresource, group: initialize, priority: 250 }
   >         - { name: oro.api.processor, action: add_subresource, group: initialize, priority: 250 }
   >         - { name: oro.api.processor, action: update_subresource, group: initialize, priority: 250 }
   >         - { name: oro.api.processor, action: get_relationship, group: initialize, priority: 250 }
   >         - { name: oro.api.processor, action: delete_relationship, group: initialize, priority: 250 }
   >         - { name: oro.api.processor, action: add_relationship, group: initialize, priority: 250 }
   >         - { name: oro.api.processor, action: update_relationship, group: initialize, priority: 250 }
   >         - { name: oro.api.processor, action: not_allowed, group: initialize, priority: 250 }
   >         - { name: oro.api.processor, action: unhandled_error, group: initialize, priority: 250 }
   >         - { name: oro.api.processor, action: options, group: initialize, priority: 250 }
   > ```
7. Add a localizable name and description for the new API type to Resources/translations/messages.en.yml that will be used by OpenAPI management UI:
   > ```yaml
   > oro:
   >     api:
   >         open_api:
   >             views:
   >                 erp_rest_json_api:
   >                     label: JSON:API (ERP Integration)
   >                     description: Backoffice REST API that is used by ERP integration.
   > ```
8. Execute the `cache:clear` command to apply the changes and the `oro:api:doc:cache:clear` command to build API Sandbox.

That is all. Now, you can open [API Sandbox](../../api/sandbox.md#web-services-api-sandbox) and check that it has the `ERP Integration` link at the top. Click on this link and try to perform any API request.

To configure the new API, use the Resources/config/oro/api_erp.yml configuration file.

All API processors related to the new API should be registered with the `requestType: erp` attribute
for the `oro.api.processor` tag, e.g.:

```yaml
acme.api.erp.do_something:
    class: Acme\Bundle\DemoBundle\Api\Processor\DoSomething
    tags:
        - { name: oro.api.processor, action: get, group: initialize, requestType: erp, priority: -10 }
```

For more details about the configuration and processors, see [Configuration Reference](configuration.md#web-api-configuration), [Actions](actions.md#web-api-actions) and [Processors](processors.md#web-api-processors).

<a id="add-id-to-api-resource"></a>

## Add a Predefined Identifier to API Resource

Imagine that you want to provide an API resource for the current authenticated user. There are several ways to do this:

* [Add a Custom Route]()
* [Add a Custom Controller]()
* create a model inherited from an User entity and expose it as a separate API resource
* reserve some word, e.g. **mine**, as an predefined identifier of the current authenticated user

The last approach is simplest to implement and more preferred in the most cases, because it gives a possibility
to use such identifier in a resource path, filters and request data.

To implement this approach, you need to perform the following:

1. Create a class that implements <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Request/EntityIdResolverInterface.php" target="_blank">EntityIdResolverInterface</a>, e.g.:
   > ```php
   > namespace Oro\Bundle\UserBundle\Api;

   > use Oro\Bundle\ApiBundle\Request\EntityIdResolverInterface;
   > use Oro\Bundle\SecurityBundle\Authentication\TokenAccessorInterface;
   > use Oro\Bundle\UserBundle\Entity\User;

   > /**
   >  * Resolves "mine" identifier for User entity.
   >  * This identifier can be used to identify the current authenticated user.
   >  */
   > class MineUserEntityIdResolver implements EntityIdResolverInterface
   > {
   >     private TokenAccessorInterface $tokenAccessor;

   >     public function __construct(TokenAccessorInterface $tokenAccessor)
   >     {
   >         $this->tokenAccessor = $tokenAccessor;
   >     }

   >     #[\Override]
   >     public function getDescription(): string
   >     {
   >         return <<<MARKDOWN
   > **mine** can be used to identify the current authenticated user.
   > MARKDOWN;
   >     }

   >     #[\Override]
   >     public function resolve(): mixed
   >     {
   >         $user = $this->tokenAccessor->getUser();

   >         return $user instanceof User
   >             ? $user->getId()
   >             : null;
   >     }
   > }
   > ```
2. Register this class as a service and tag it with `oro.api.entity_id_resolver`, e.g.:
   > ```yaml
   > oro_user.api.mine_user_entity_id_resolver:
   >      class: Oro\Bundle\UserBundle\Api\MineUserEntityIdResolver
   >      arguments:
   >          - '@oro_security.token_accessor'
   >      tags:
   >          - { name: oro.api.entity_id_resolver, id: mine, class: Oro\Bundle\UserBundle\Entity\User }
   > ```

If a predefined identifier should be available only for a specific request type, use the [requestType](request-type.md#api-request-type) attribute of the tag, e.g.:

```yaml
tags:
        - { name: oro.api.entity_id_resolver, id: mine, requestType: json_api, class: Oro\Bundle\UserBundle\Entity\User }
```

<a id="add-computed-field"></a>

## Add a Computed Field

Sometimes, it is required to add to the API a field that does not exist in an entity for which API is created.
In this case, such field should be added to the API via [Resources/config/oro/api.yml](configuration.md#web-api-fields-config) and
the [customize_loaded_data](actions.md#customize-loaded-data-action) action should be used to set a value
of this field.

For example, imagine that a “price” field need to be added to a product API. The following steps show how to do this:

1. Add the “price” field to the product API via Resources/config/oro/api.yml
   > ```yaml
   > api:
   >     entities:
   >         Acme\Bundle\DemoBundle\Entity\Product:
   >             fields:
   >                 price:
   >                     data_type: money
   >                     property_path: _
   > ```
2. Create a processor for `customize_loaded_data` action that will set a value for the “price” field
   > ```php
   > namespace Acme\Bundle\DemoBundle\Api\Processor;

   > use Oro\Bundle\ApiBundle\Processor\CustomizeLoadedData\CustomizeLoadedDataContext;
   > use Oro\Component\ChainProcessor\ContextInterface;
   > use Oro\Component\ChainProcessor\ProcessorInterface;

   > class ComputeProductPriceField implements ProcessorInterface
   > {
   >     #[\Override]
   >     public function process(ContextInterface $context): void
   >     {
   >         /** @var CustomizeLoadedDataContext $context */
   >         $data = $context->getData();

   >         $priceFieldName = $context->getResultFieldName('price');
   >         if (!$context->isFieldRequested($priceFieldName, $data)) {
   >             return;
   >         }

   >         $productIdFieldName = $context->getResultFieldName('id');
   >         if (!$productIdFieldName || empty($data[$productIdFieldName])) {
   >             return;
   >         }

   >         $data[$priceFieldName] = $this->loadProductPrice((int)$data[$productIdFieldName]);
   >         $context->setData($data);
   >     }

   >     /**
   >      * @param int $productIdFieldName
   >      * @return null|float
   >      */
   >     private function loadProductPrice(int $productIdFieldName): ?float
   >     {
   >         // load the product price in this method
   >     }
   > }
   > ```
3. Register the processor in the dependency injection container
   > ```yaml
   > services:
   >     acme.api.compute_product_price_field:
   >         class: Acme\Bundle\DemoBundle\Api\Processor\ComputeProductPriceField
   >         tags:
   >             - { name: oro.api.processor, action: customize_loaded_data, class: Acme\Bundle\DemoBundle\Entity\Product }
   > ```

## Add an Association with a Custom Query

Let’s use the following schema of entities to illustrate how to use a custom query for an association in API:

- Account entity

```php
#[ORM\OneToMany(mappedBy: 'account', targetEntity: 'AccountContactLink')]
private $contactLinks;
```

- Contact entity

```php
#[ORM\OneToMany(mappedBy: 'contact', targetEntity: 'AccountContactLink')]
private $accountLinks;
```

- AccountContactLink entity

```php
#[ORM\ManyToOne(targetEntity: 'Account', inversedBy: 'contactLinks')]
private $account;

#[ORM\ManyToOne(targetEntity: 'Contact', inversedBy: 'accountLinks')]
private $contact;

#[ORM\Column(type: 'boolean', nullable: false, options: ['default' => true])]
private $enabled = true;
```

This schema represents a many-to-many association between the Account and Contact entities but with an additional attribute for each associated record (e.g., attribute `enabled` in the example above).

To elaborate illustration further, let’s add `contacts` relationship to the Account API resource that will contain only enabled contacts. To achieve this:

- Add the `contacts` field via Resources/config/oro/api.yml

```yaml
api:
    entities:
        Acme\Bundle\DemoBundle\Entity\Account:
        fields:
            contacts:
                target_class: Acme\Bundle\DemoBundle\Entity\Contact
                target_type: to-many
                property_path: _
```

- Add a processor to register ORM query that should be used to get enabled contacts for the [get](actions.md#get-action) and [get_list](actions.md#get-list-action) actions

  **Note:** Aliases `e` and `r` are reserved and both must exist in the query. The alias `e` must correspond to the owning entity of the association. The alias `r` must correspond to the target entity of the association.

```php
namespace Acme\Bundle\DemoBundle\Api\Processor;

use Acme\Bundle\DemoBundle\Entity\Account;
use Oro\Bundle\ApiBundle\Processor\GetConfig\ConfigContext;
use Oro\Bundle\ApiBundle\Util\DoctrineHelper;
use Oro\Component\ChainProcessor\ContextInterface;
use Oro\Component\ChainProcessor\ProcessorInterface;

/**
 * Adds a query for "contacts" association of Account entity.
 */
class SetAccountContactsAssociationQuery implements ProcessorInterface
{
    private DoctrineHelper $doctrineHelper;

    public function __construct(DoctrineHelper $doctrineHelper)
    {
        $this->doctrineHelper = $doctrineHelper;
    }

    #[\Override]
    public function process(ContextInterface $context): void
    {
        /** @var ConfigContext $context */

        $definition = $context->getResult();
        $contactsField = $definition->getField('contacts');
        if (null !== $contactsField
            && !$contactsField->isExcluded()
            && null === $contactsField->getAssociationQuery()
        ) {
            $contactsField->setAssociationQuery(
                $this->doctrineHelper
                    ->createQueryBuilder(Account::class, 'e')
                    ->innerJoin('e.contactLinks', 'links')
                    ->innerJoin('links.contact', 'r')
                    ->where('links.enabled = :contacts_enabled')
                    ->setParameter('contacts_enabled', true)
            );
        }
    }
}
```

```yaml
services:
    acme.api.set_account_contacts_association_query:
        class: Acme\Bundle\DemoBundle\Api\Processor\SetAccountContactsAssociationQuery
        arguments:
            - '@oro_api.doctrine_helper'
        tags:
            - { name: oro.api.processor, action: get_config, extra: '!identifier_fields_only', class: Acme\Bundle\DemoBundle\Entity\Account, priority: -35 }
```

- If the query added on the previous step is not applicable to load subresources, add a processor to register ORM query that should be used to get enabled contacts for the [get_subresource](actions.md#get-subresource-action) and [get_relationship](actions.md#get-relationship-action) actions
  > ```php
  > namespace Acme\Bundle\DemoBundle\Api\Processor;

  > use Acme\Bundle\DemoBundle\Entity\Contact;
  > use Acme\Bundle\DemoBundle\Entity\AccountContactLink;
  > use Oro\Bundle\ApiBundle\Collection\Join;
  > use Oro\Bundle\ApiBundle\Processor\Subresource\Shared\AddParentEntityIdToQuery;
  > use Oro\Bundle\ApiBundle\Processor\Subresource\SubresourceContext;
  > use Oro\Bundle\ApiBundle\Util\DoctrineHelper;
  > use Oro\Component\ChainProcessor\ContextInterface;
  > use Oro\Component\ChainProcessor\ProcessorInterface;

  > /**
  >  * Builds ORM QueryBuilder object that will be used to get a list of contacts
  >  * for Account entity for "get_relationship" and "get_subresource" actions.
  >  */
  > class BuildAccountContactsSubresourceQuery implements ProcessorInterface
  > {
  >     private DoctrineHelper $doctrineHelper;

  >     public function __construct(DoctrineHelper $doctrineHelper)
  >     {
  >         $this->doctrineHelper = $doctrineHelper;
  >     }

  >     #[\Override]
  >     public function process(ContextInterface $context): void
  >     {
  >         /** @var SubresourceContext $context */

  >         if ($context->hasQuery()) {
  >             // a query is already built
  >             return;
  >         }

  >         $query = $this->doctrineHelper
  >             ->createQueryBuilder(Contact::class, 'e')
  >             ->innerJoin(AccountContactLink::class, 'links', Join::WITH, 'links.contact = e')
  >             ->where('links.account = :' . AddParentEntityIdToQuery::PARENT_ENTITY_ID_QUERY_PARAM_NAME)
  >             ->setParameter(AddParentEntityIdToQuery::PARENT_ENTITY_ID_QUERY_PARAM_NAME, $context->getParentId());

  >         $context->setQuery($query);
  >     }
  > }
  > ```

  > ```yaml
  > services:
  >     acme.api.build_account_contacts_subresource_query:
  >         class: Acme\Bundle\DemoBundle\Api\Processor\BuildAccountContactsSubresourceQuery
  >         arguments:
  >             - '@oro_api.doctrine_helper'
  >         tags:
  >             - { name: oro.api.processor, action: get_subresource, group: build_query, association: contacts, parentClass: Acme\Bundle\DemoBundle\Entity\Account, priority: -90 }
  >             - { name: oro.api.processor, action: get_relationship, group: build_query, association: contacts, parentClass: Acme\Bundle\DemoBundle\Entity\Account, priority: -90 }
  > ```

<a id="disable-hateoas"></a>

## Disable HATEOAS

It is not possible to disable <a href="https://restfulapi.net/hateoas/" target="_blank">HATEOAS</a> via a configuration.
But you can send API request with `noHateoas` value in [X-Include header](headers.md#existing-x-include-keys) to exclude HATEOAS links from a response of a particular request.

<a id="validate-virtual-fields"></a>

## Validate Virtual Fields

There are cases when an API resource contains virtual fields; these are fields that do not exist in an entity.

Like with regular fields, values of these fields need to be validated during the [create](actions.md#create-action) and [update](actions.md#update-action) actions.

In this case, you can use an API processor for the `post_submit` event of the [customize_form_data](actions.md#customize-form-data-action) action because common Symfony Forms validators are not applicable.

For example, the following API processor validates that a value of a virtual field called `label` should not be blank for
a new `Acme\Bundle\DemoBundle\Entity\SomeEntity` entity:

```php
namespace Acme\Bundle\DemoBundle\Api\Processor;

use Oro\Bundle\ApiBundle\Form\FormUtil;
use Oro\Bundle\ApiBundle\Processor\CustomizeFormData\CustomizeFormDataContext;
use Oro\Bundle\ApiBundle\Request\ApiAction;
use Oro\Component\ChainProcessor\ContextInterface;
use Oro\Component\ChainProcessor\ProcessorInterface;
use Symfony\Component\Validator\Constraints\NotBlank;

class ValidateLabelField implements ProcessorInterface
{
    #[\Override]
    public function process(ContextInterface $context): void
    {
        /** @var CustomizeFormDataContext $context */

        $form = $context->findFormField('label');
        if (null === $form) {
            return;
        }

        if ($context->getParentAction() === ApiAction::CREATE && !$form->isSubmitted()) {
            FormUtil::addFormConstraintViolation($form, new NotBlank());
        }

        if ($form->isSubmitted() && (null === $form->getData() || '' === $form->getData())) {
            FormUtil::addFormConstraintViolation($form, new NotBlank());
        }
    }
}
```

```yaml
services:
    acme.api.validate_label_field:
        class: Acme\Bundle\DemoBundle\Api\Processor\ValidateLabelField
        tags:
            - { name: oro.api.processor, action: customize_form_data, event: post_submit, class: Acme\Bundle\DemoBundle\Entity\SomeEntity }
```

<!-- Frontend -->
