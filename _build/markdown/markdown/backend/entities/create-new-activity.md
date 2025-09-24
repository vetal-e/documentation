<a id="backend-make-entity-activities"></a>

# Turn an Entity into an Activity

To create an activity from your new entity, make the entity extended and include it in the activity group.

To make the entity extended, implement the ExtendEntityInterface using the ExtendEntityTrait. The class must also implement <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ActivityBundle/Model/ActivityInterface.php" target="_blank">ActivityInterface</a>.

Here is an example:

#### NOTE
src/Acme/Bundle/DemoBundle/Entity/Sms.php
```php
use Oro\Bundle\ActivityBundle\Model\ActivityInterface;
use Oro\Bundle\ActivityBundle\Model\ExtendActivity;
use Oro\Bundle\EntityExtendBundle\Entity\ExtendEntityInterface;
use Oro\Bundle\EntityExtendBundle\Entity\ExtendEntityTrait;

class Sms implements
    ActivityInterface,
    ExtendEntityInterface
{
    use ExtendActivity;
    use ExtendEntityTrait;
    // ...
}
```

Use this class as the superclass for your entity. To include the entity in the activity group, use the ORO entity configuration, for example:

```php
#[Config(
    defaultValues: [
        'grouping' => ['groups' => ['activity']],
    ]
)]
class Sms implements
    ActivityInterface,
    ExtendEntityInterface
{
    // ...
}
```

Your entity is now recognized as the activity entity. To make sure that the activity is displayed correctly, you need to configure its UI.

<a id="backend-make-entity-activities-working-with-activity-associations"></a>

## Working with Activity Associations

Activity associations are represented by [multiple many-to-many](extend-entities/multi-target-associations.md#book-entities-extended-entities-multi-target-associations-types) associations.
It is quite a complex type of associations, and to help work with activities, use the <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ActivityBundle/Manager/ActivityManager.php" target="_blank">ActivityManager</a> class.

This class provides the following functionality:

* Check whether a specific type of entity has any activity associations.
* Check whether a specific type of entity can be associated with a specific activity.
* Get a list of entity types of all activity entities.
* Get the list of fields responsible for storing activity associations for a specific type of activity entity.
* Get a query builder that can be used for fetching a list of entities associated with a specific activity.
* Get a list of fields responsible for storing activity associations for a specific type of entity.
* Get a query builder that can be used to fetch a list of activity entities associated with a specific entity.
* Get an array that contains info about all activity associations for a specific type of entity.
* Get an array that contains info about all activity actions for a specific type of entity.
* Add a filter by a specific entity to a query builder that is used to get a list of activities.
* Associate an entity with an activity entity.
* Remove an association between an entity and an activity entity.

<a id="backend-entity-activities-configure-ui"></a>

## Configure UI for the Activity Entity

Before using the new activity entity within OroPlatform, you need to:

* [Configure UI for Activity List Section]()
* [Configure UI for an Activity Button]()

Take a look at <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ActivityBundle/Resources/config/oro/entity_config.yml" target="_blank">all configuration options</a> for the activity scope before reading further.

### Configure UI for Activity List Section

First, create a new action in your controller and a TWIG template responsible for rendering the list of your activities.

Keep in mind that:

- The controller action must accept two parameters: $entityClass and $entityId.
- The entity class name can be encoded to avoid routing collisions. That is why you need to use the oro_entity.routing_helper service to get the entity by its class name and id.
- In the following example, the activity-sms-grid datagrid is used to render the list of activities. This grid is defined in the *datagrids.yml* file:

```php
    /**
     * This action is used to render the list of sms associated with the given entity
     * on the view page of this entity
     */
    #[Route(
        path: '/activity/view/{entityClass}/{entityId}',
        name: 'acme_demo_sms_activity_view',
        requirements: ['entityClass' => '\w+', 'entityId' => '\d+']
    )]
    #[AclAncestor('acme_demo_sms_view')]
    public function activityAction(string $entityClass, int $entityId): Response
    {
        return $this->render(
            '@AcmeDemo/Sms/activity.html.twig',
            [
                'entity' => $this->container->get(EntityRoutingHelper::class)->getEntity($entityClass, $entityId),
            ]
        );
    }
```

```html
{% import '@OroDataGrid/macros.html.twig' as dataGrid %}

<div class="widget-content">
    {{ dataGrid.renderGrid('activity-sms-grid', {
        entityClass: oro_class_name(entity, true),
        entityId: entity.id
    }) }}
</div>
```

```yaml
datagrids:
    activity-sms-grid:
        extends: acme-demo-sms-grid-base
```

Now, you need to bind the controller to your activity entity. Use ORO entity configuration, for example:

```php
#[Config(
    defaultValues: [
        'grouping' => ['groups' => ['activity']],
        'activity' => [
            'route' => 'acme_demo_sms_activity_view',
            'acl' => 'acme_demo_sms_view',
        ]
    ]
)]
class Sms implements
    ActivityInterface,
    ExtendEntityInterface
{
    // ...
}
```

Please note that the example above contains the route attribute to specify the controller path and the acl attribute to set ACL restrictions.

### Configure UI for an Activity Button

To add an activity button to the view page of the entity with the assigned activity:

1. Create two TWIG templates responsible for rendering the button and the link in the dropdown menu. Please note that you should provide both templates because an action can be rendered either as a button or a link depending on the number of actions, UI theme, device (desktop/mobile), etc.

Here is an example of TWIG templates:

```html
{% import '@OroUI/macros.html.twig' as UI %}

{{ UI.clientButton({
    'dataUrl': path(
        'acme_demo_sms_create', {
            entityClass: oro_class_name(entity, true),
            entityId: entity.id
        }),
    'aCss': 'no-hash',
    'iCss': 'fa-question',
    'dataId': entity.id,
    'label': 'acme.demo.sms.action.add'|trans,
    'widget': {
        'type': 'dialog',
        'multiple': false,
        'refresh-widget-alias': 'activity-list-widget',
        'options': {
            'alias': 'sms-dialog',
            'dialogOptions': {
                'title': 'acme.demo.sms.action.add'|trans,
                'allowMaximize': true,
                'allowMinimize': true,
                'dblclick': 'maximize',
                'maximizedHeightDecreaseBy': 'minimize-bar',
                'width': 1000,
                'minWidth': 'expanded'
            }
        }
    }
}) }}
```

```html
{% import '@OroUI/macros.html.twig' as UI %}

{{ UI.clientLink({
    'dataUrl': path(
        'acme_demo_sms_create', {
            entityClass: oro_class_name(entity, true),
            entityId: entity.id
        }),
    'aCss': 'dropdown-item no-hash',
    'iCss': 'fa-question',
    'dataId': entity.id,
    'label': 'acme.demo.sms.action.add'|trans,
    'widget': {
        'type': 'dialog',
        'multiple': false,
        'refresh-widget-alias': 'activity-list-widget',
        'options': {
            'alias': 'sms-dialog',
            'dialogOptions': {
                'title': 'acme.demo.sms.action.add'|trans,
                'allowMaximize': true,
                'allowMinimize': true,
                'dblclick': 'maximize',
                'maximizedHeightDecreaseBy': 'minimize-bar',
                'width': 1000,
                'minWidth': 'expanded'
            }
        }
    }
}) }}
```

1. Register these templates in *placeholders.yml*, for example:

```yaml
placeholders:
    items:
        acme_demo_add_sms_button:
            template: '@@AcmeDemo/Sms/activityButton.html.twig'
            acl: acme_demo_sms_create

        acme_demo_add_sms_link:
            template: '@@AcmeDemo/Sms/activityLink.html.twig'
            acl: acme_demo_sms_create
```

1. Bind the items declared in *placeholders.yml* to the activity entity using the action_button_widget and action_link_widget attributes, for example:

```php
#[Config(
    defaultValues: [
        'grouping' => ['groups' => ['activity']],
        'activity' => [
            'route' => 'acme_demo_sms_activity_view',
            'acl' => 'acme_demo_sms_view',
            'action_button_widget' => 'acme_demo_add_sms_button',
            'action_link_widget' => 'acme_demo_add_sms_link'
        ]
    ]
)]
class Sms implements
    ActivityInterface,
    ExtendEntityInterface
{
    // ...
}
```

The following screenshot is an example of new activity from the view page:

![Make an entity an activity](img/bundles/ActivityListBundle/activities-create-new-activity.png)

<a id="backend-entity-activities-configure-custom-grid"></a>

## Configure Custom Grid for Activity Context Dialog

If you want to define a context grid for an entity (e.g., Document) in the activity context dialog, add the context option in the entity class #[Config] attribute, for example:

```php
#[Config(
    defaultValues: [
        'grid' => [
            'default' => 'acme-demo-document-grid-select',
            'context' => 'document-for-context-grid'
        ],
    ]
)]
```

This option is used to recognize the grid for the entity with a higher priority than the default option.
If these options (context or default) are not defined for an entity, the grid does not appear in the context dialog.

Example configure custom grid for activity context dialog:

![Configure custom grid for activity context dialog](img/bundles/ActivityListBundle/activities-activity-list-add-context.png)

<a id="backend-entity-activities-enable-context-column"></a>

## Enable Contexts Column in Activity Entity Grids

You can add a column for any activity entity grid that includes all context entities.

Have a look at the following example of sms configuration in *datagrids.yml*:

```yaml
datagrids:
    acme-demo-sms-grid-base:
        options:
            contexts:
                enabled: true          # default `false`
                column_name: contexts  # optional, column identifier, default is `contexts`
                entity_name: ~         # optional, set the FQCN of the grid base entity if auto detection fails
```

This configuration creates a column named contexts and tries to detect the activity class name automatically. If, for some reason, it fails, you can specify an FQCN in the entity_name option.

If you wish to configure the column, add a section with the name specified in the column_name option:

```yaml
datagrids:
    acme-demo-sms-grid-base:
        columns:
            contexts:                                 # the column name defined in options
                label: acme.demo.sms.contexts.label   # optional, default `oro.activity.contexts.column.label`
                renderable: true                      # optional, default `true`
```

![Enable contexts column in activity entity grids](img/bundles/ActivityListBundle/activities-enable-context-column.png)

The column type is twig (unchangeable), so you can also specify template.

The default one is <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ActivityBundle/Resources/views/Grid/Column/contexts.html.twig" target="_blank">@OroActivity/Grid/Column/contexts.html.twig</a>.

```twig
{% for item in value %}
    {% apply spaceless %}
        <span class="cell-context-item">
            <span class="context-icon {{ item.icon }}" aria-hidden="true"></span>
            {% if item.link %}
                <a href="{{ item.link }}" class="context-label" title="{{ item.title }}">{{ item.title }}</a>
            {% else %}
                <span class="context-label" title="{{ item.title }}">{{ item.title }}</span>
            {% endif %}
        </span>
    {% endapply %}
{% endfor %}
```

<a id="backend-create-new-activity-displayed-widget"></a>

## Add a New Entity to be Displayed within a Widget

To add a new entity to be displayed within a widget, register a service that implements **ActivityListProviderInterface** and tag it as **oro_activity_list.provider**. A working example of this is available in EmailBundle or CalendarBundle, for example:

```yaml
services:
    acme_demo_sms.activity_list.provider:
        class: Acme\Bundle\DemoBundle\Provider\SmsActivityListProvider
        arguments:
            - "@oro_entity.doctrine_helper"
            - "@oro_security.owner.entity_owner_accessor.link"
            - "@oro_activity.association_helper"
            - "@oro_comment.association_helper"
        tags:
            - { name: oro_activity_list.provider, class: Acme\Bundle\DemoBundle\Entity\Sms, priority: 10 }
```

This will add your provider class into providers (**ActivityListChainProvider**) that will be invoked to fetch data ordering by priority (added in service definition). Priority is helpful for future implementations or overriding existing providers in third-party bundles.

Each activity entity has its own row template for the UI component. Although you can place it anywhere, make sure its path is returned in the Provider via the getTemplate() method. For instance:

```php
namespace Acme\Bundle\DemoBundle\Provider;

use Acme\Bundle\DemoBundle\Entity\Sms;
use Oro\Bundle\ActivityBundle\Tools\ActivityAssociationHelper;
use Oro\Bundle\ActivityListBundle\Entity\ActivityList;
use Oro\Bundle\ActivityListBundle\Entity\ActivityOwner;
use Oro\Bundle\ActivityListBundle\Model\ActivityListDateProviderInterface;
use Oro\Bundle\ActivityListBundle\Model\ActivityListProviderInterface;
use Oro\Bundle\CommentBundle\Model\CommentProviderInterface;
use Oro\Bundle\CommentBundle\Tools\CommentAssociationHelper;
use Oro\Bundle\EntityBundle\ORM\DoctrineHelper;
use Oro\Component\DependencyInjection\ServiceLink;

/**
 * Provides a way to use Sms entity in an activity list.
 */
class SmsActivityListProvider implements
    ActivityListProviderInterface,
    CommentProviderInterface,
    ActivityListDateProviderInterface
{
    /** @var DoctrineHelper */
    protected $doctrineHelper;

    /** @var ServiceLink */
    protected $entityOwnerAccessorLink;

    /** @var ActivityAssociationHelper */
    protected $activityAssociationHelper;

    /** @var CommentAssociationHelper */
    protected $commentAssociationHelper;

    public function __construct(
        DoctrineHelper $doctrineHelper,
        ServiceLink $entityOwnerAccessorLink,
        ActivityAssociationHelper $activityAssociationHelper,
        CommentAssociationHelper $commentAssociationHelper
    ) {
        $this->doctrineHelper            = $doctrineHelper;
        $this->entityOwnerAccessorLink   = $entityOwnerAccessorLink;
        $this->activityAssociationHelper = $activityAssociationHelper;
        $this->commentAssociationHelper  = $commentAssociationHelper;
    }

    #[\Override]
    public function isApplicableTarget($entityClass, $accessible = true)
    {
        return $this->activityAssociationHelper->isActivityAssociationEnabled(
            $entityClass,
            Sms::class,
            $accessible
        );
    }

    /**
     * @param Sms $entity
     */
    #[\Override]
    public function getSubject($entity)
    {
        return substr(trim($entity->getMessage()), 0, 20);
    }

    #[\Override]
    public function getDescription($entity)
    {
        return null;
    }

    /**
     * @param Sms $entity
     */
    #[\Override]
    public function getOwner($entity)
    {
        return $entity->getOwner();
    }

    /**
     * @param Sms $entity
     */
    #[\Override]
    public function getCreatedAt($entity)
    {
        return $entity->getCreatedAt();
    }

    /**
     * @param Sms $entity
     */
    #[\Override]
    public function getUpdatedAt($entity)
    {
        return $entity->getUpdatedAt();
    }

    #[\Override]
    public function getData(ActivityList $activityList)
    {
        /** @var SMS $sms */
        $sms =  $this->doctrineHelper
            ->getEntityManager($activityList->getRelatedActivityClass())
            ->getRepository($activityList->getRelatedActivityClass())
            ->find($activityList->getRelatedActivityId());

        return [
            'fromContact' => $sms->getFromContact(),
            'toContact' => $sms->getToContact()
        ];
    }

    /**
     * @param Sms $entity
     */
    #[\Override]
    public function getOrganization($entity)
    {
        return $entity->getOrganization();
    }

    #[\Override]
    public function getTemplate()
    {
        return '@AcmeDemo/Sms/js/activityItemTemplate.html.twig';
    }

    #[\Override]
    public function getRoutes($entity)
    {
        return [
            'itemView'   => 'acme_demo_sms_widget_info',
            'itemEdit'   => 'acme_demo_sms_update',
            'itemDelete' => 'acme_demo_api_delete_sms'
        ];
    }

    #[\Override]
    public function getActivityId($entity)
    {
        return $this->doctrineHelper->getSingleEntityIdentifier($entity);
    }

    #[\Override]
    public function isApplicable($entity)
    {
        if (\is_object($entity)) {
            return $entity instanceof Sms;
        }

        return $entity === Sms::class;
    }

    /**
     * @param Sms $entity
     */
    #[\Override]
    public function getTargetEntities($entity)
    {
        return $entity->getActivityTargets();
    }

    #[\Override]
    public function isCommentsEnabled($entityClass)
    {
        return $this->commentAssociationHelper->isCommentAssociationEnabled($entityClass);
    }

    /**
     * @param Sms $entity
     */
    #[\Override]
    public function getActivityOwners($entity, ActivityList $activityList)
    {
        $organization = $this->getOrganization($entity);
        $owner = $this->entityOwnerAccessorLink->getService()->getOwner($entity);

        if (!$organization || !$owner) {
            return [];
        }

        $activityOwner = new ActivityOwner();
        $activityOwner->setActivity($activityList);
        $activityOwner->setOrganization($organization);
        $activityOwner->setUser($owner);

        return [$activityOwner];
    }

    #[\Override]
    public function isActivityListApplicable(ActivityList $activityList): bool
    {
        return true;
    }
}
```

Here is an example of TWIG templates:

```none
{% extends '@OroActivityList/ActivityList/js/activityItemTemplate.html.twig' %}
{% import '@OroActivity/macros.html.twig' as AC %}

{% set entityClass = 'Acme\\Bundle\\DemoBundle\\Entity\\Sms' %}
{% set entityName = oro_entity_config_value(entityClass, 'label')|trans %}

{% block activityDetails %}
    {{ entityName }}
    <% var template = (verb == 'create')
        ? {{ 'acme.demo.sms.sms_created_by'|trans|json_encode|raw }}
        : {{ 'acme.demo.sms.sms_changed_by'|trans|json_encode|raw }};
    %>
    <%= _.template(template, { interpolate: /\{\{(.+?)\}\}/g })({
        user: owner_url ? '<a class="user" href="' + owner_url + '">' +  _.escape(owner) + '</a>' :  '<span class="user">' + _.escape(owner) + '</span>',
        date: '<i class="date">' + createdAt + '</i>',
        editor: editor_url ? '<a class="user" href="' + editor_url + '">' +  _.escape(editor) + '</a>' : _.escape(editor),
        editor_date: '<i class="date">' + updatedAt + '</i>'
    }) %>
{% endblock %}

{% block activityActions %}
    {% import '@OroActivity/macros.html.twig' as AC %}

    {% set action %}
        <% if (editable) { %>
        {{ AC.activity_context_link() }}
        <% } %>
    {% endset %}
    {% set actions = [action] %}

    {% set action %}
        <a href="<%- routing.generate('acme_demo_sms_view', {'id': relatedActivityId}) %>"
           class="dropdown-item"
           title="{{ 'acme.demo.sms.sms_view'|trans({'{{ entity }}': entityName}) }}"><span
                    class="fa-eye hide-text" aria-hidden="true">{{ 'acme.demo.sms.sms_view'|trans({'{{ entity }}': entityName}) }}</span>
            {{ 'acme.demo.sms.sms_view'|trans({'{{ entity }}': entityName}) }}
        </a>
    {% endset %}
    {% set actions = actions|merge([action]) %}

    {% set action %}
        <% if (editable) { %>
        <a href="#" class="dropdown-item action item-edit-button"
           title="{{ 'acme.demo.sms.sms_update'|trans({'{{ entity }}': entityName}) }}"
           data-action-extra-options="{{ {dialogOptions: {width: 1000}}|json_encode }}">
            <span class="fa-pencil-square-o hide-text">{{ 'acme.demo.sms.sms_update'|trans({'{{ entity }}': entityName}) }}</span>
            {{ 'acme.demo.sms.sms_update'|trans({'{{ entity }}': entityName}) }}
        </a>
        <% } %>
    {% endset %}
    {% set actions = actions|merge([action]) %}

    {% set action %}
        <% if (removable) { %>
        <a href="#" class="dropdown-item action item-remove-button"
           title="{{ 'acme.demo.sms.sms_delete'|trans({'{{ entity }}': entityName}) }}">
            <span class="fa-trash-o hide-text" aria-hidden="true">{{ 'acme.demo.sms.sms_delete'|trans({'{{ entity }}': entityName}) }}</span>
            {{ 'acme.demo.sms.sms_delete'|trans({'{{ entity }}': entityName}) }}
        </a>
        <% } %>
    {% endset %}
    {% set actions = actions|merge([action]) %}

    {{ parent() }}
{% endblock %}
```

Method getRoutes() returns an array of route names. You need to implement a functionality to display information on the specified routers.

![New entity to be displayed within a widget](img/bundles/ActivityListBundle/activities-activity-list.png)

### View an Activity in the Activity list

Create a view action in your controller and a TWIG template.

```php
    #[Route(
        path: '/widget/info/{id}',
        name: 'acme_demo_sms_widget_info',
        requirements: ['id' => '\d+'],
        options: ['expose' => true]
    )]
    #[Template('@AcmeDemo/Sms/widget/info.html.twig')]
    #[AclAncestor('acme_demo_sms_view')]
    public function infoAction(Request $request, Sms $entity): array
    {
        $targetEntity = $this->getTargetEntity($request);
        $renderContexts = null !== $targetEntity;

        return [
            'entity' => $entity,
            'target' => $targetEntity,
            'renderContexts' => $renderContexts,
        ];
    }

    /**
     * Get target entity
     *
     * @param Request $request
     *
     * @return object|null
     */
    protected function getTargetEntity(Request $request)
    {
        $entityRoutingHelper = $this->container->get(EntityRoutingHelper::class);
        $targetEntityClass = $entityRoutingHelper->getEntityClassName($request, 'targetActivityClass');
        $targetEntityId = $entityRoutingHelper->getEntityId($request, 'targetActivityId');
        if (!$targetEntityClass || !$targetEntityId) {
            return null;
        }

        return $entityRoutingHelper->getEntity($targetEntityClass, $targetEntityId);
    }
```

```html
{% import '@OroUI/macros.html.twig' as UI %}
{% import '@OroActivity/macros.html.twig' as AC %}

<div class="widget-content form-horizontal box-content row-fluid">
    <div class="responsive-block">
        {# Display contexts targets in the activity list item view #}
        {% if renderContexts is defined and renderContexts %}
            <div class="activity-context-activity-list">
                {{ AC.activity_contexts(entity, target, true) }}
            </div>
        {% endif %}

        {% set realFromContact = entity.fromContact %}

        {{ UI.renderProperty('acme.demo.sms.sms_created_by'|trans, entity.getCreatedAt() | oro_format_datetime ) }}
        {{ UI.renderProperty('acme.demo.sms.from_contact.label'|trans, realFromContact ) }}
        {{ UI.renderProperty('acme.demo.sms.to_contact.label'|trans, entity.toContact ) }}
    </div>
    <div class="responsive-block">
        <h5>{{'acme.demo.sms.message.label'| trans }}</h5>
        <pre>{{ entity.message}}</pre>
    </div>
</div>
```

### Edit an Activity in the Activity List

Define the update action in your controller.

```php
    /**
     * Edit Sms form
     */
    #[Route(
        path: '/update/{id}',
        name: 'acme_demo_sms_update',
        requirements: ['id' => '\d+'],
        options: ['expose' => true]
    )]
    #[Template('@AcmeDemo/Sms/update.html.twig')]
    #[Acl(id: 'acme_demo_sms_update', type: 'entity', class: 'Acme\Bundle\DemoBundle\Entity\Sms', permission: 'EDIT')]
    public function updateAction(Sms $entity, Request $request): array|RedirectResponse
    {
        $updateMessage = $this->container->get(TranslatorInterface::class)->trans(
            'acme.demo.controller.sms.saved.message'
        );

        return $this->update($entity, $request, $updateMessage);
    }
```

### Delete an Activity in the Activity List

Example of implementing the deletion of an activity from an activity list on a page.

Create a controller. Here is an example of the controller:

```php
namespace Acme\Bundle\DemoBundle\Controller\Api\Rest;

use Acme\Bundle\DemoBundle\Entity\Sms;
use Nelmio\ApiDocBundle\Annotation\ApiDoc;
use Oro\Bundle\SecurityBundle\Attribute\Acl;
use Oro\Bundle\SoapBundle\Controller\Api\FormAwareInterface;
use Oro\Bundle\SoapBundle\Controller\Api\Rest\RestController;
use Oro\Bundle\SoapBundle\Entity\Manager\ApiEntityManager;
use Symfony\Component\HttpFoundation\Response;

/**
 * REST API controller for Sms entity.
 */
class SmsController extends RestController
{
    /**
     * REST DELETE
     *
     * @param int $id
     *
     * @ApiDoc(
     *      description="Delete sms",
     *      resource=true
     * )
     * @return Response
     */
    #[Acl(id: 'acme_demo_sms_delete', type: 'entity', permission: 'DELETE', class: Sms::class)]
    public function deleteAction(int $id)
    {
        return $this->handleDeleteRequest($id);
    }

    /**
     * Get entity Manager
     *
     * @return ApiEntityManager
     */
    #[\Override]
    public function getManager()
    {
        return $this->container->get('acme_demo_sms.manager.api');
    }

    /**
     * @return FormAwareInterface
     */
    #[\Override]
    public function getFormHandler()
    {
        return $this->container->get('acme_demo_sms.form.handler.sms_api');
```

Register the created controller.

```yaml
acme_demo_api_delete_sms:
    path: '/api/rest/{version}/smses/{id}.{_format}'
    methods: [DELETE]
    defaults:
        _controller: 'Acme\Bundle\DemoBundle\Controller\Api\Rest\SmsController::deleteAction'
        _format: json
        version: latest
    requirements:
        id: \d+
        _format: json|html
        version: latest|v1
    options:
        expose: true
```

```yaml
services:
    _defaults:
        public: true

    AcmeDemoBundleRestApiController:
        namespace: Acme\Bundle\DemoBundle\Controller\Api\Rest\
        resource: '../../Controller/Api/Rest/*Controller.php'
        calls:
            - [setContainer, ['@service_container']]
```

```yaml
services:
    acme_demo_sms.form.handler.sms_api:
        class: Acme\Bundle\DemoBundle\Form\Handler\SmsApiHandler
        public: true
        arguments:
            - '@form.factory'
            - '@request_stack'
            - '@doctrine.orm.entity_manager'
        tags:
            - { name: oro_form.form.handler, alias: acme_demo_sms.form.handler.sms_api }

    acme_demo_sms.manager.api:
        class: Oro\Bundle\SoapBundle\Entity\Manager\ApiEntityManager
        public: true
        parent: oro_soap.manager.entity_manager.abstract
        arguments:
            - 'Acme\Bundle\DemoBundle\Entity\Sms'
            - '@doctrine.orm.entity_manager'
```

This API handler is the implementation of REST API.

```php
namespace Acme\Bundle\DemoBundle\Form\Handler;

use Acme\Bundle\DemoBundle\Entity\Sms;
use Acme\Bundle\DemoBundle\Form\Type\SmsApiType;
use Doctrine\Persistence\ObjectManager;
use Oro\Bundle\FormBundle\Form\Handler\RequestHandlerTrait;
use Oro\Bundle\SoapBundle\Controller\Api\FormAwareInterface;
use Symfony\Component\Form\FormFactory;
use Symfony\Component\HttpFoundation\RequestStack;

/**
 *  This API handler is the implementation of REST API.
 */
class SmsApiHandler implements FormAwareInterface
{
    use RequestHandlerTrait;

    /**
     * @var FormFactory
     */
    protected $formFactory;

    /**
     * @var RequestStack
     */
    protected $requestStack;

    /**
     * @var ObjectManager
     */
    protected $manager;

    public function __construct(FormFactory $formFactory, RequestStack $requestStack, ObjectManager $manager)
    {
        $this->formFactory = $formFactory;
        $this->requestStack = $requestStack;
        $this->manager = $manager;
    }

    /**
     * Process form
     *
     * @param  Sms $entity
     * @return bool True on successful processing, false otherwise
     */
    public function process(Sms $entity)
    {
        $form = $this->getForm();
        $form->setData($entity);

        $request = $this->requestStack->getCurrentRequest();

        if (\in_array($request->getMethod(), ['POST', 'PUT'], true)) {
            $this->submitPostPutRequest($form, $request);
            if ($form->isValid()) {
                $this->onSuccess($entity);

                return true;
            }
        }

        return false;
    }

    #[\Override]
    public function getForm()
    {
        return $this->formFactory->createNamed('', SmsApiType::class);
    }

    /**
     * "Success" form handler
     */
    protected function onSuccess(Sms $entity)
    {
        $this->manager->persist($entity);
        $this->manager->flush();
    }
}
```

Create a form type to add the createdAt field.

```php
namespace Acme\Bundle\DemoBundle\Form\Type;

use Acme\Bundle\DemoBundle\Entity\Sms;
use Oro\Bundle\FormBundle\Form\Type\OroDateTimeType;
use Oro\Bundle\SoapBundle\Form\EventListener\PatchSubscriber;
use Symfony\Component\Form\FormBuilderInterface;
use Symfony\Component\OptionsResolver\OptionsResolver;

/**
 * Form type for old REST API to add createdAt field
 */
class SmsApiType extends SmsType
{
    #[\Override]
    public function buildForm(FormBuilderInterface $builder, array $options): void
    {
        parent::buildForm($builder, $options);

        $builder->add(
            'createdAt',
            OroDateTimeType::class,
            [
                'required' => false,
            ]
        );

        $builder->addEventSubscriber(new PatchSubscriber());
    }

    #[\Override]
    public function configureOptions(OptionsResolver $resolver): void
    {
        $resolver->setDefaults(
            [
                'data_class' => Sms::class,
                'csrf_protection' => false
            ]
        );
    }

    public function getName()
    {
        return $this->getBlockPrefix();
    }

    #[\Override]
    public function getBlockPrefix()
    {
        return 'sms';
    }
}
```

<!-- Frontend -->
