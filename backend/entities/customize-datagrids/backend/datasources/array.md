<a id="customize-datagrids-datasource-array"></a>

# Array Datasource

This datasource provides the ability to set data for the datagrid from the array.

**Example**

```yaml
datagrids:
    DATAGRID_NAME_HERE:
        source:
            type: array
```

## Configuration

To configure datasource, create a datagrid event listener and subscribe to the `oro_datagrid.datagrid.build.after.DATAGRID_NAME_HERE` event.

```yaml
acme_demo.event_listener.datagrid.my_custom_listener:
    class: Acme\Bundle\DemoBundle\EventListener\Datagrid\MyCustomListener
    tags:
        - { name: kernel.event_listener, event: oro_datagrid.datagrid.build.after.DATAGRID_NAME_HERE, method: onBuildAfter }
```

```php
namespace Acme\Bundle\DemoBundle\EventListener\Datagrid;

use Oro\Bundle\DataGridBundle\Datasource\ArrayDatasource\ArrayDatasource;
use Oro\Bundle\DataGridBundle\Event\BuildAfter;
use Oro\Bundle\DataGridBundle\Exception\UnexpectedTypeException;

class MyCustomListener
{
    /**
     * @param BuildAfter $event
     * @return void
     */
    public function onBuildAfter(BuildAfter $event): void
    {
        $datagrid   = $event->getDatagrid();
        $datasource = $datagrid->getDatasource();

        if (!$datasource instanceof ArrayDatasource) {
            throw new UnexpectedTypeException($datasource, ArrayDatasource::class);
        }

        // Create datagrid source array
        $source = [
            // row 1
            [
                'first_column'  => 'Value in first row and first column',
                'second_column' => 'Value in first row and second column',
            ],
            // row 2
            [
                'first_column'  => 'Value in second row and first column',
                'second_column' => 'Value in second row and second column',
            ],
            // ...
        ];

        $datasource->setArraySource($source);
    }
}
```

Predefined columns can be defined using the following configuration:

```yaml
datagrids:
    DATAGRID_NAME_HERE:
        source:
            type: array
        columns:
            first_column:
                label: Column 1 Label
        sorters:
            columns:
                first_column:
                    data_name: first_column
```

**Related Articles**

* [Datagrids](../../../data-grids/index.md#data-grids)
* [Datagrid Configuration Reference](../../../../configuration/yaml/datagrids.md#reference-format-datagrids)
