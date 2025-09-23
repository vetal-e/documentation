<a id="backend-entity-activities"></a>

# Entity Activities

## Enable Activity Association Using Migrations

Usually, an administrator provides a predefined set of associations between the activity entity and other entities. If necessary, you can also create this association type using ref:migrations <backend-entities-migrations>.

The following example illustrates how to do it:

```php
<?php

namespace Acme\Bundle\DemoBundle\Migrations\Schema\v1_10;

use Doctrine\DBAL\Schema\Schema;
use Oro\Bundle\ActivityBundle\Migration\Extension\ActivityExtensionAwareInterface;
use Oro\Bundle\ActivityBundle\Migration\Extension\ActivityExtensionAwareTrait;
use Oro\Bundle\MigrationBundle\Migration\Migration;
use Oro\Bundle\MigrationBundle\Migration\QueryBag;

class AcmeDemoBundle implements Migration, ActivityExtensionAwareInterface
{
    use ActivityExtensionAwareTrait;

    #[\Override]
    public function up(Schema $schema, QueryBag $queries): void
    {
        $this->activityExtension->addActivityAssociation($schema, 'oro_email', 'acme_demo_document', true);
        $this->activityExtension->addActivityAssociation($schema, 'acme_demo_sms', 'acme_demo_document', true);
        $this->activityExtension->addActivityAssociation($schema, 'orocrm_call', 'acme_demo_priority');
    }
}
```

<a id="backend-entity-activities-add-widget-column"></a>

## Add a Widget to the Entity View Page

OroActivityListBundle adds a widget to the entity view page. The widget displays a list of activities related to the entity record in chronological order. It also enables widget configuration in the system configuration section.

You can add the following activities to other entities:

- Call
- Task
- Email
- Calendar event
- Notes

Visualization of the Activity list is defined as a widget block. It shows activities related to the entity record currently being viewed in a single list, with the ability to filter it by activity type (*multiselect*) and date (*daterange* filter).

Each activity row shows basic information: the activity type, who and when created and updated it. You can also access the full activity record with the help of the “expand” action. By default, it displays 10 records sorted by the Update date in descending order. You can change the limitation and sorting in the UI [via system configuration](#bundle-docs-platform-activity-list-bundle-configuration).

The widget is currently displayed in the *Activities* placeholder block on the view page of an entity.

The following screenshot is an example of a widget on a contact page:

![An example of a widget](img/bundles/ActivityListBundle/activities-widget-example.png)

<a id="backend-entity-activities-show-widget-on-specific-page"></a>

## Show Widget and Its Button on a Specific Page

Set entity annotation to show a widget and its button on specific pages.

The widget can be displayed on the view and/or update pages. The list of allowed values is available in `\Oro\Bundle\ActivityBundle\EntityConfig\ActivityScope`, for example:

```php
#[Config(
    defaultValues: [
        'activity' => ['show_on_page' => \Oro\Bundle\ActivityBundle\EntityConfig\ActivityScope::UPDATE_PAGE]
    ]
)]
class Priority implements
    ExtendEntityInterface
{
    //    ...
}
```

The following screenshot is an example of a widget on a priority update page:

![An example showing a widget-specific page](img/bundles/ActivityListBundle/activities-widget-specific-page.png)

<a id="bundle-docs-platform-activity-list-bundle-configuration"></a>

## Change Sorting and Limitation in Configuration

You can change sorting and limitation in the UI under **System > Configuration > Display Settings > Activity Lists**.

| Option                    | Description                                                                                                        |
|---------------------------|--------------------------------------------------------------------------------------------------------------------|
| Sort By Field             | Sorts activity records by the date when they were created or by the date when they were updated for the last time. |
| Sort Direction            | Sorts records in the ascending or descending direction.                                                            |
| Items Per Page By Default | Sets how many records appear on one page of the Activity section grids.                                            |
![Activity list global configuration](img/bundles/ActivityListBundle/activity-lists-configuration.png)

<a id="bundle-docs-platform-activity-list-bundle-permissions"></a>

## Configure Permissions

Each activity entity must contain a provider (for example, *EmailActivityListProvider*) with the implemented *ActivityListProviderInterface* interface. The *ActivityList::getActivityOwners* method returns one or many ActivityOwner entities connected to their activity list entity.

<a id="bundle-docs-platform-activity-list-bundle-filter"></a>

## Filter Activities in Segments

ActivityListBundle extends OroSegmentBundle with the Activity filter type.

This filter can be used to filter records if they:

* have an activity with a value in the field (e.g., a Contact who has an activity “Email” where the subject of the email contains the text “Re:”)
* do not have an activity with a value in the field (e.g., Contact who does not have activity “Email” where the subject of the email contains text “Meeting”)

If you select only one activity type in the filter, you can filter based on any field of the activity.

If you select more than one activity type in the filter, you can filter based on the fields “updatedAt” and “createdAt” of the selected activities.

![Activity widget in segment's filters](img/bundles/ActivityListBundle/activity-in-segment-filters.png)
<!-- Need to test the example before publishing -->
<!-- Add Inheritance of Activity Lists to the Target Entity -->
<!-- ------------------------------------------------------ -->
<!-- You can add inheritance of activity lists to the target entity from some related inheritance target entities. -->
<!-- It means that in target entities, you can see all activity list from the general entity and related entities. -->
<!-- To enable this option, configure the target entity to identify all inheritance target entities: use migration extension to add all necessary configurations to the entity config. -->
<!-- The following is an example of the migration to enable the display of contact activity lists in the appropriate account: -->
<!-- .. code-block:: none -->
<!-- class InheritanceActivityTargets implements Migration, ActivityListExtensionAwareInterface
{
    /** @var ActivityListExtension */
    protected $activityListExtension;
    #[\Override]
    public function setActivityListExtension(ActivityListExtension $activityListExtension)
    {
        $this->activityListExtension = $activityListExtension;
    }
    #[\Override]
    public function up(Schema $schema, QueryBag $queries)
    {
        $activityListExtension->addInheritanceTargets($schema, 'orocrm_account', 'orocrm_contact', ['accounts']);
    }
} -->
<!-- Method parameters:
.. * addInheritanceTargets(Schema $schema, $targetTableName, $inheritanceTableName, $path)
.. * string $targetTableName - Target entity table name
.. * string $inheritanceTableName - Inheritance entity table name
.. * string[] $path - Path of relations to target entity -->
<!-- Frontend -->
