<a id="index-0"></a>

<a id="how-to-pass-request-parameter-to-the-grid"></a>

# Pass Request Parameters to the Grid

Sometimes, you must pass parameters from the request to the grid.
For this, use grid event listeners or an existing listener implementation.

## Grid Configuration

Suppose you have a grid configuration and a named parameter inside the where clause of its source query:

#### NOTE
src/Acme/Bundle/DemoBundle/Resources/config/oro/datagrids.yml
```yaml
datagrids:
    acme-demo-question-grid-by-priority:
        source:
            query:
                where:
                    and:
                        - 'IDENTITY(e.priority) = :holder_entity_id'
```

The goal is to set the :holder_entity_id parameter with the value from the request.

## Solution 1: Grid Parameter Binding

The easiest way (sufficient for most situations) is to use the parameter binding option of the datasource to configure the mapping between the datagrid and query parameters.

You can do this by adding the `bind_parameters` option to your `datagrids.yml` using the following syntax:

```yaml
datagrids:
    acme-demo-question-grid-by-priority:
        source:
            query:
                where:
                    and:
                        - 'IDENTITY(e.priority) = :holder_entity_id'
            bind_parameters:
                # Get the "holder_entity_id" parameter from the datagrid
                # and set its value to the "holder_entity_id" parameter in the datasource query
                - holder_entity_id
```

The controller receives an entity “priority” and passes it to the view:

```php
#[Route(path: '/priority', name: 'acme_demo_priority_')]
class PriorityController extends AbstractController
{
    #[Route(path: '/view/{id}', name: 'view', requirements: ['id' => '\d+'])]
    #[Template('@AcmeDemo/Priority/view.html.twig')]
    public function viewAction(Priority $entity): array
    {
        return [
            'entity' => $entity,
        ];
    }
}
```

In this example, the entity mapping page adds a grid with the data of the linked entity.
The view passes the “holder_entity_id” parameter with the value “entity.id” to the grid:

```html
{% import '@OroDataGrid/macros.html.twig' as dataGrid %}
{% block content_data %}

    {% set dataBlocks = [
        {
            'title': 'acme.demo.priority.priority_questions.label'|trans,
            'subblocks': [
                {
                    'data' : [
                        dataGrid.renderGrid('acme-demo-question-grid-by-priority', { holder_entity_id: entity.id }, { cssClass: 'inner-grid' })
                    ]
                }
            ]
        },
    ] %}
```

In case if names of the parameter in the grid and the query do not match, you can pass an associative array of parameters, where the key will be the name of the query parameter, and the value will match the name of the parameter in the grid:

```yaml
 datagrids:
     acme-demo-question-grid-by-priority:
         source:
             query:
                 where:
                     and:
                     - 'IDENTITY(e.priority) = :holder_entity_id'
             bind_parameters:
                 # Get the "priority" parameter from the datagrid
                 # and set its value to the "holder_entity_id" parameter in the datasource query
                 holder_entity_id: priority
         # ...
     # ...
```

#### NOTE
A datasource must implement the <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/DataGridBundle/Datasource/BindParametersInterface.php" target="_blank">BindParametersInterface</a> to support the bind_parameters option.

## Solution 2. Create Custom Event Listener

If the first example does not work for you ( for example, datasource does not support parameters binding, or  you need to implement additional logic before binding parameters), you can create a listener for the
`oro_datagrid.datagrid.build.after` event and set the parameter for the source query in this listener:

```php
namespace Acme\Bundle\DemoBundle\EventListener;

use Doctrine\ORM\QueryBuilder;

use Oro\Bundle\DataGridBundle\Datasource\Orm\OrmDatasource;
use Oro\Bundle\DataGridBundle\Event\BuildAfter;

class ParameterListener
{
    protected $parameterName;
    protected $requestParameterName;

    public function __construct($parameterName, $requestParameterName = null)
    {
        $this->parameterName = $parameterName;
        $this->requestParameterName = $requestParameterName ? $requestParameterName : $this->parameterName;
    }

    public function onBuildAfter(BuildAfter $event)
    {
        $datagrid   = $event->getDatagrid();
        $datasource = $datagrid->getDatasource();
        $parameters = $datagrid->getParameters();

        if ($datasource instanceof OrmDatasource) {
            /** @var QueryBuilder $queryBuilder */
            $queryBuilder = $datasource->getQueryBuilder();

            $queryBuilder->setParameter($this->parameterName, $parameters->get($this->requestParameterName));
        }
    }
}
```

Register this listener in the container:

```yaml
services:
    acme_demo.event_listener.acme_priorities_grid_parameter_listener:
        class: Acme\Bundle\DemoBundle\EventListener\ParameterListener
        arguments:
            - holder_entity_id
        tags:
            - { name: kernel.event_listener, event: oro_datagrid.datagrid.build.after.acme-demo-document-grid-by-priority, method: onBuildAfter }
```

```yaml
datagrids:
    acme-demo-document-grid-by-priority:
        source:
            query:
                where:
                    and:
                        - 'IDENTITY(e.priority) = :holder_entity_id'
```

The view passes the “holder_entity_id” parameter with the value “entity.id” to the grid:

```html
{% import '@OroDataGrid/macros.html.twig' as dataGrid %}
{% block content_data %}

    {% set dataBlocks = [
        {
            'title': 'acme.demo.priority.priority_documents.label'|trans,
            'subblocks': [
                {
                    'data' : [
                        dataGrid.renderGrid('acme-demo-document-grid-by-priority', { holder_entity_id: entity.id }, { cssClass: 'inner-grid' })
                    ]
                }
            ]
        },
```

## References

* <a href="https://symfony.com/doc/6.4/cookbook/doctrine/event_listeners_subscribers.html" target="_blank">Symfony Cookbook How to Register Event Listeners and Subscribers</a>

<!-- Frontend -->
