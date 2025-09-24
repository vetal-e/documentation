<a id="data-grids"></a>

# Datagrids

Creating a basic datagrid to display the data of all tasks involves three steps:

1. [Configure the datagrid](#cookbook-entities-grid-config)
2. [Create a controller and template](#cookbook-entities-grid-controller)
3. [Add a link to the grid to the application menu](#cookbook-entities-grid-navigation)

<a id="cookbook-entities-grid-config"></a>

## Configure the Grid

The backend datagrid is configured in the `/config/oro/datagrids.yml` file, while the frontend datagrid is configured in the `/views/layouts/<theme>/config/datagrids.yml` file within the configuration directory of your bundle, and is divided into the sections below.

### Datasource

The `source` option is used to configure a Doctrine query builder that is used to fetch the data to be displayed in the grid:

```yaml
datagrids:
    acme-demo-question-grid-base:
        source:
            type: orm
            query:
                select:
                    - e.id
                    - e.subject
                    - e.description
                    - e.dueDate
                    - 'priority.label as priorityTitle'
                from:
                    -
                        table: Acme\Bundle\DemoBundle\Entity\Question
                        alias: e
                join:
                    left:
                        - { join: e.priority, alias: priority }
```

### Displayed Columns

Then, the `columns` option needs to be used to configure how which data will be displayed:

```yaml
datagrids:
    acme-demo-question-grid-base:
        columns:
            subject:
                label: acme.demo.question.subject.label
            description:
                label: acme.demo.question.description.label
            dueDate:
                label: acme.demo.question.due_date.label
                frontend_type: datetime
            priorityTitle:
                label: acme.demo.question.priority.label
```

Keep in mind that the frontend datagrid is configured in the `Resources/views/layouts/<theme>/config/datagrids.yml` file within the configuration directory of your bundle.

### Column Sorters

Use the `sorters` option to define on which columns’ header the user can click to order by the data:

```yaml
datagrids:
    acme-demo-question-grid-base:
        sorters:
            columns:
                subject:
                    data_name: e.subject
                description:
                    data_name: e.description
                dueDate:
                    data_name: e.dueDate
                priorityTitle:
                    data_name: priorityTitle
            default:
                dueDate: DESC
```

Each key under `sorters.columns` refers to one displayed column. The `data_name` option is the term that will be used as the `order by` term in the Doctrine query.

### Data Filters

Data filters are UI elements that allow the user to filter the data being displayed in the data grid. List all the attributes for which a filter should be shown under the `filters.columns` key. To configure the filter for a certain property, two options are needed:

* The `type` configures the UI type of the filter. The type of filter should be chosen based on the data type of the underlying attribute.
* The `data_name` denotes the name of the property to filter and will be used as is to modify the datagrid’s query builder.

```yaml
datagrids:
    acme-demo-question-grid-base:
        filters:
            columns:
                subject:
                    data_name: e.subject
                    type: string
                description:
                    data_name: e.description
                    type: string
                dueDate:
                    data_name: e.dueDate
                    type: datetime
                priorityTitle:
                    type: entity
                    data_name: priority.id
                    options:
                        field_type: Symfony\Bridge\Doctrine\Form\Type\EntityType
                        field_options: { class: Acme\Bundle\DemoBundle\Entity\Priority, choice_label: label }
```

### Complete Datagrid Configuration

The final datagrid configuration now looks like this:

```yaml
datagrids:
    acme-demo-question-grid-base:
        extended_entity_name: Acme\Bundle\DemoBundle\Entity\Question
        source:
            type: orm
            query:
                select:
                    - e.id
                    - e.subject
                    - e.description
                    - e.dueDate
                    - 'priority.label as priorityTitle'
                from:
                    -
                        table: Acme\Bundle\DemoBundle\Entity\Question
                        alias: e
                join:
                    left:
                        - { join: e.priority, alias: priority }
        columns:
            subject:
                label: acme.demo.question.subject.label
            description:
                label: acme.demo.question.description.label
            dueDate:
                label: acme.demo.question.due_date.label
                frontend_type: datetime
            priorityTitle:
                label: acme.demo.question.priority.label
        sorters:
            columns:
                subject:
                    data_name: e.subject
                description:
                    data_name: e.description
                dueDate:
                    data_name: e.dueDate
                priorityTitle:
                    data_name: priorityTitle
            default:
                dueDate: DESC
        filters:
            columns:
                subject:
                    data_name: e.subject
                    type: string
                description:
                    data_name: e.description
                    type: string
                dueDate:
                    data_name: e.dueDate
                    type: datetime
                priorityTitle:
                    type: entity
                    data_name: priority.id
                    options:
                        field_type: Symfony\Bridge\Doctrine\Form\Type\EntityType
                        field_options: { class: Acme\Bundle\DemoBundle\Entity\Priority, choice_label: label }
```

<a id="cookbook-entities-grid-controller"></a>

## Create the Controller and View

To make your datagrid accessible, create a controller that the user can visit, which will serve as a view that renders the configured datagrid:

```php
namespace Acme\Bundle\DemoBundle\Controller;

use Acme\Bundle\DemoBundle\Entity\Question;
use Symfony\Bridge\Twig\Attribute\Template;
use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\Routing\Attribute\Route;

#[Route(path: '/question', name: 'acme_demo_question_')]
class QuestionController extends AbstractController
{
    #[Route(path: '/', name: 'index')]
    #[Template('@AcmeDemo/Question/index.html.twig')]
    public function indexAction(): array
    {
        return [
            'entity_class' => Question::class
        ];
    }
}
```

The view can be straightforward if you extend the `@OroUI/actions/index.html.twig` template:

```html
{% extends '@OroUI/actions/index.html.twig' %}
{% import '@OroUI/macros.html.twig' as UI %}
{% set gridName = 'acme-demo-question-grid' %}
{% set pageTitle = 'acme.demo.question.entity_plural_label'|trans %}
```

Configure the name of your datagrid and the title you wish to be displayed. The base template from the OroUIBundle handles everything else.

<a id="cookbook-entities-grid-navigation"></a>

## Link to the Action

At last, you need to make the action accessible by creating a menu item:

```yaml
navigation:
    menu_config:
        items:
            acme_demo_question_list:
                label: acme.demo.question.entity_plural_label
                route: acme_demo_question_index
        tree:
            application_menu:
                children:
                    acme_tab:
                        children:
                            acme_demo_tab:
                                children:
                                    acme_demo_question_list: ~
```

#### NOTE
`application_menu` is the name of the menu you want to hook your item into. In this case, `application_menu` is an existing menu that is part of OroPlatform.

## Key Classes

- `Datagrid\Manager` - responsible for preparing the grid and its configuration.
- `Datagrid\Builder` - responsible for creating and configuring the datagrid object and its datasource. It contains registered datasource types and extensions, and it also performs a check for datasource availability according to ACL.
- `Datagrid\Datagrid` - the main grid object, has only knowledge about the datasource object and its interaction; all further modifications of the results and metadata come from the extensions. Extension\\Acceptor - is a visitable mediator, contains all applied extensions, and provokes visits at different points of the interactions.
- `Extension\ExtensionVisitorInterface` - visitor interface.
- `Extension\AbstractExtension` - basic empty implementation.
- `Datasource\DatasourceInterface` - link object between data and grid. Should provide results as an array of ResultRecordInterface compatible objects.
- `Provider\SystemAwareResolver` - resolves specific grid YAML syntax expressions. For more information, see the [references in configuration](../customize-datagrids/backend/references-in-configuration.md#datagrid-references-configuration) topic.

<a id="datagrids-customize-mixin"></a>

## Mixin

Mixin is a datagrid that contains additional (common) information for use by other datagrids.

**Configuration Syntax**

```yaml
datagrids:
    # configuration mixin with column, sorter and filter for an entity identifier
    acme-demo-common-user-ownership-datagrid-mixin:
        source:
            query:
                select:
                    - CONCAT(ownerUser.firstName, ' ', ownerUser.lastName) as uOwnerName
                join:
                    left:
                        # _root_entity__ alias that will be replaced by an alias of the root entity
                        - { join: __root_entity__.owner, alias: ownerUser }
        columns:
            uOwnerName:
                label: oro.user.entity_label
        sorters:
            columns:
                uOwnerName:
                    data_name: uOwnerName
        filters:
            columns:
                uOwnerName:
                    data_name: uOwnerName
                    type: string
    acme-demo-question-grid-base:
        # one or several mixins
        mixins:
            - acme-demo-common-user-ownership-datagrid-mixin
        source:
            type: orm
            query:
                from:
                    -
                        table: Acme\Bundle\DemoBundle\Entity\Question
                        alias: e
```

**Related Articles**

* [Customizing Datagrid](../customize-datagrids/index.md#customizing-data-grid-in-orocommerce)
* [Datagrid Configuration Reference](../../configuration/yaml/datagrids.md#reference-format-datagrids)
