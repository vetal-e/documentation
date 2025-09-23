<a id="customize-datagrids-datasource-orm"></a>

# ORM Datasource

This datasource provides an adapter to access data from the doctrine ORM using the doctrine query builder. You can configure a query using the `query` param under the source tree. This query will be converted via <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/DataGridBundle/Datasource/Orm/QueryConverter/YamlConverter.php" target="_blank">YamlConverter</a> to the doctrine `QueryBuilder` object.

**Example**

```yaml
datagrids:
    DATAGRID_NAME_HERE:
        source:
            type: orm
            query:
                select:
                    - email.id
                    - email.subject
                from:
                    - { table: Oro\Bundle\EmailBundle\Entity\Email, alias: email }
```

## Important Notes

By default, all datagrids that use ORM datasource are marked by the <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Component/DoctrineUtils/README.md#preciseorderbywalker-class" target="_blank">HINT_PRECISE_ORDER_BY</a> query hint. This guarantees that rows are sorted the same way independently of the state of the SQL server and the values of OFFSET and LIMIT clauses. More details are available in the <a href="https://www.postgresql.org/docs/8.1/static/queries-limit.html" target="_blank">Queries Limits</a> section of PostgreSQL documentation.

If you need to disable this behavior for your datagrid, use the following configuration:

```yaml
datagrids:
    DATAGRID_NAME_HERE:
        source:
            type: orm
            query:
                ...
            hints:
                - { name: HINT_PRECISE_ORDER_BY, value: false }
```

## How to

### Modify Query Configuration from PHP Code

You can modify query configuration from PHP code, for example from the datagrid [extensions](../extensions/index.md#customize-datagrid-extensions) or [listeners](../index.md#customizing-data-grid-in-orocommerce-backend-extendability). This can be done using <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/DataGridBundle/Datasource/Orm/OrmQueryConfiguration.php" target="_blank">OrmQueryConfiguration</a> class. To get an instance of this class, use the getOrmQuery method of <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/DataGridBundle/Datagrid/Common/DatagridConfiguration.php" target="_blank">DatagridConfiguration</a>. For example:

```php
$query = $config->getOrmQuery();
$rootAlias = $query->getRootAlias();
$query->addSelect($rootAlias . '.myField');
```

In addition to query modification methods, the <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/DataGridBundle/Datasource/Orm/OrmQueryConfiguration.php" target="_blank">OrmQueryConfiguration</a> contains several valuable methods:

- `getRootAlias()` - Returns the FIRST root alias of the query.
- `getRootEntity($entityClassResolver = null, $lookAtExtendedEntityClassName = false)` - Returns the FIRST root entity of the query.
- `findRootAlias($entityClass, $entityClassResolver = null)` - Tries to find the root alias for the given entity.
- `getJoinAlias($join, $conditionType = null, $condition = null)` - Returns an alias for the given join. If the query does not contain the specified join, its alias will be generated automatically. This might be helpful if you need to get an alias to extend the association that will be joined later.
- `convertAssociationJoinToSubquery($joinAlias, $columnAlias, $joinEntityClass)` - Converts an association based join to a subquery. This can be helpful in case of performance issues with a datagrid.
- `convertEntityJoinToSubquery($joinAlias, $columnAlias)` - Converts an entity based join to a subquery. This can be helpful in case of performance issues with a datagrid.

Example of `convertAssociationJoinToSubquery` usage in a datagrid listener:

```none
public function onPreBuild(PreBuild $event)
{
    $config = $event->getConfig();
    $parameters = $event->getParameters();

    $filters = $parameters->get(OrmFilterExtension::FILTER_ROOT_PARAM, []);
    $sorters = $parameters->get(OrmSorterExtension::SORTERS_ROOT_PARAM, []);
    if (empty($filters['channelName']) && empty($sorters['channelName'])) {
        $config->getOrmQuery()->convertAssociationJoinToSubquery(
            'g',
            'groupName',
            'Acme\Bundle\DemoBundle\Entity\UserGroup'
        );
    }
}
```

The original query:

```yaml
query:
    select:
        - g.name as groupName
    from:
        - { table: Acme\Bundle\DemoBundle\Entity\User, alias: u }
    join:
        left:
            - { join: u.group, alias: g }
```

The converted query:

```yaml
query:
    select:
        - (SELECT g.name FROM Acme\Bundle\DemoBundle\Entity\UserGroup AS g WHERE g = u.group) as groupName
    from:
        - { table: Acme\Bundle\DemoBundle\Entity\User, alias: u }
```

Make sure you investigate this class to find out all the other features.

### Add Query Hints

The following example shows how <a href="https://www.doctrine-project.org/projects/doctrine-orm/en/latest/reference/dql-doctrine-query-language.html#query-hints" target="_blank">Doctrine query hints</a> can be set:

```yaml
datagrids:
    DATAGRID_NAME_HERE:
        source:
            type: orm
            query:
                select:
                    - partial g.{id, label}
                from:
                    - { table: Oro\Bundle\ContactBundle\Entity\Group, alias: g }
            hints:
                - HINT_FORCE_PARTIAL_LOAD
```

If you need to set the hint’s value, use the following syntax:

```yaml
datagrids:
    DATAGRID_NAME_HERE:
        source:
            type: orm
            query:
                select:
                    - c
                from:
                    - { table: Oro\Bundle\ContactBundle\Entity\Contact, alias: c }
                join:
                    left:
                        - { join: c.addresses, alias: address, conditionType: WITH, condition: 'address.primary = true' }
                        - { join: address.country, alias: country }
                        - { join: address.region, alias: region }
            hints:
                - { name: HINT_CUSTOM_OUTPUT_WALKER, value: Gedmo\Translatable\Query\TreeWalker\TranslationWalker }
```

Please keep in mind that ORM datasource uses the Query Hint Resolver service to handle hints. If you create your own query walker and wish to use it in a grid, register it in the Query Hint Resolver. For example, hint `HINT_TRANSLATABLE` is registered as an alias for the translation walker, and as a result, the following configurations are equal:

```yaml
hints:
    - { name: HINT_CUSTOM_OUTPUT_WALKER, value: Gedmo\Translatable\Query\TreeWalker\TranslationWalker }

hints:
    - HINT_TRANSLATABLE
```

#### HINT
See [Resolve ORM Query Hints](../../../query-hint-resolver.md#dev-entities-resolving-orm-query-hints) for more information.

**Related Articles**

* [Datagrids](../../../data-grids/index.md#data-grids)
* [Datagrid Configuration Reference](../../../../configuration/yaml/datagrids.md#reference-format-datagrids)

<!-- Frontend -->
