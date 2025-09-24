<a id="customizing-data-grid-in-orocommerce"></a>

# Customize Datagrids

Most business application users have to deal with significant amounts of data on a daily basis. Thus, efficiently navigating through large data sets becomes a must-have requirement, and OroCommerce is not an exception. The application users must be able to filter, sort, and search easily through thousands (or millions) of records, usually represented in the form of a datagrid on a page.

This topic uses existing OroCommerce datagrids for illustration. If you are not familiar with OroPlatform datagrids, check the [articles on how to create a simple datagrid](../data-grids/index.md#data-grids), and [how to pass request parameters to your datagrid](../data-grids/how-to-pass-request-parameter-to-grid.md#how-to-pass-request-parameter-to-the-grid). The [datagrid.yml configuration reference](../../configuration/yaml/datagrids.md#reference-format-datagrids) and the OroDataGridBundle documentation contain additional useful information.

![image](img/backend/entities/grid1.png)

## Data Sources

A datagrid is usually used to visualize some data coming from a data source. OroDataGridBundle allows for the use of various data sources for datagrids and includes the ORM data source adapter out of the box. It is possible to [implement your own data source adapters](backend/datasources/index.md#customize-datagrids-datasource) as well.

The ORM data source types allow for database query specification, sorters, and filter definitions to be specified in the datagrid configuration. Datagrid configuration can be supplied by a developer in YAML format. By convention, the datagrid.yml files placed in the Resources/config folders of any application bundle are processed automatically. All supported data source configuration options that can be used in the data source configuration section are described in the [datasources section of the DataGridBundle documentation](backend/datasources/index.md#customize-datagrids-datasource).

## Inner Workings of Datagrids

Datagrids in Oro applications are highly customizable. It is possible to modify an existing grid in order to fetch more data than was originally defined in the grid configuration. In this article, we will retrieve and present to a user some additional data in the existing <a href="https://github.com/oroinc/orocommerce/blob/master/src/Oro/Bundle/ProductBundle/Resources/config/oro/datagrids.yml#L305" target="_blank">products-grid</a>.

And before we start customizing it, let’s take a deeper look into two aspects of how the datagrids actually work:

* Building and configuring a new DataGrid instance
* Fetching data

### Building Grids

![OroDataGridBundle base class diagram](img/backend/entities/datagrid_base_uml-800x487.jpg)

Datagrid\\Builder class is responsible for creating and configuring an object and its data source. This is how the build method is processing the grid configuration internally:

![Datagrid build process](img/backend/entities/build-flow-551x600.png)

Imagine that you need to show a list of related price lists for every product record in the product grid. You want it to be displayed in a separate column with a multi-select filter. You also want to add one more column to display the owner of each of the product records.

One of the possible ways to customize this grid would be through events triggered by the system during the grid build process and when the data is fetched from the data source.

There are several events triggered before processing the datagrid configuration files. In this case, a good choice is the onBuildBefore event. By listening to this event, you can add new elements to the grid configuration or modify the already existing configuration in your event listener.

#### NOTE
You can find more information about grid column definition configuration options in the [columns and properties section of the DataGrid documentation](backend/index.md#customizing-data-grid-columns-properties).

The Product entity has a many-to-one relation with the Business Unit entity, so to add the owner column to the grid and load the owner data from the data source, you should modify its query configuration by adding additional join and select parts.

### Fetching Data

However, retrieving data for the price lists column is a little bit more complicated because the Product entity has a many-to-many relation with price lists, and the join result will contain duplicate rows. In such situations or when some other dynamic data should be included in the query results, a possible solution would be data modification after the rows were fetched from the data source.

This is how data retrieval works in general:

![Datagrid records retrieval](img/backend/entities/orm-result.png)

So in our customization, we will fetch the price list data in a separate query and then attach this data to each of the product records in the grid.

## Product Grid Customization

The resulting implementation of the ProductsGridListener may look similar to this example:

```php
class ProductsGridListener
{
   ...

   /**
    * @param BuildBefore $event
    */
   public function onBuildBefore(BuildBefore $event)
   {
       $datagridConfiguration = $event->getConfig();
       $this->addBusinessUnitColumn($datagridConfiguration);
       $this->addPriceListsColumn($datagridConfiguration);
   }

   /**
    * @param OrmResultAfter $event
    */
   public function onResultAfter(OrmResultAfter $event)
   {
       $records = $event->getRecords();
       $this->addPriceListsToRecords($records);
   }

   /**
    * @param DatagridConfiguration $datagridConfiguration
    */
   protected function addPriceListsColumn(DatagridConfiguration $datagridConfiguration)
   {
       $column = [
           'label' => 'Price Lists',
           'type' => 'twig',
           'template' => '@OroPricing/Datagrid/Column/priceLists.html.twig',
           'frontend_type' => 'html',
           'renderable' => true,
       ];
       $datagridConfiguration->addColumn('price_lists', $column);
   }

   /**
    * @param DatagridConfiguration $datagridConfiguration
    */
   protected function addBusinessUnitColumn(DatagridConfiguration $datagridConfiguration)
   {
       $datagridConfiguration->joinTable(
           'left',
           [
               'join' => BusinessUnit::class,
               'alias' => 'business_unit',
               'conditionType' => 'WITH',
               'condition' => 'product.owner = business_unit',
           ]
       );

       $column = [
           'label' => 'Owner'
       ];

       // column name should be the same as the field alias in the select query
       $datagridConfiguration->addColumn('owner', $column, 'business_unit.name as owner');
   }

   /**
    * @param ResultRecord[] $records
    * @throws \Doctrine\ORM\ORMException
    */
   protected function addPriceListsToRecords(array $records)
   {
       $repository = $this->doctrine->getRepository(PriceListToProduct::class);
       /** @var EntityManager $objectManager */
       $objectManager = $this->doctrine->getManager();

       $products = [];
       foreach ($records as $record) {
           $products[] = $objectManager->getReference(Product::class, $record->getValue('id'));
       }

       $priceLists = [];
       foreach ($repository->findBy(['product' => $products]) as $item) {
           $priceLists[$item->getProduct()->getId()][] = $item->getPriceList();
       }

       /** @var ResultRecord $record */
       foreach ($records as $record) {
           $id = $record->getValue('id');
           $products[] = $objectManager->getReference(Product::class, $id);

           $record->addData(['price_lists' => $priceLists[$id]]);
       }
   }
}
```

We will need to register this event listener in the service container:

```none
grid_event_listener.product:
    class: Oro\Bundle\CustomGridBundle\Datagrid\ProductsGridListener
    arguments:
        - '@doctrine'
    tags:
        - { name: kernel.event_listener, event: oro_datagrid.datagrid.build.before.products-grid, method: onBuildBefore }
        - { name: kernel.event_listener, event: oro_datagrid.orm_datasource.result.after.products-grid, method: onResultAfter }
```

After the application cache is refreshed (or immediately in the dev mode), two new columns will appear in the product grid.

## Custom Filters

Our second customization task will be to add filters for the newly introduced column.

In most cases, the built-in filters would work well. But in the case of the price lists column, a custom filter is required. The purpose of this filter will be to modify the data retrieval query depending on the filter values entered by a user.

```php
class ProductPriceListsFilter extends EntityFilter
{
    #[\Override]
    public function apply(FilterDatasourceAdapterInterface $ds, $data)
    {
        /** @var array $data */
        $data = $this->parseData($data);
        if (!$data) {
            return false;
        }

        $this->restrictByPriceList($ds, $data['value']);

        return true;
    }

    /**
     * @param OrmFilterDatasourceAdapter|FilterDatasourceAdapterInterface $ds
     * @param array $priceLists
     */
    public function restrictByPriceList($ds, array $priceLists)
    {
        $queryBuilder = $ds->getQueryBuilder();
        $parentAlias = $queryBuilder->getRootAliases()[0];
        $parameterName = $ds->generateParameterName('price_lists');

        $repository = $this->doctrine->getRepository(PriceListToProduct::class);
        $subQueryBuilder = $repository->createQueryBuilder('relation');
        $subQueryBuilder->where(
            $subQueryBuilder->expr()->andX(
                $subQueryBuilder->expr()->eq('relation.product', $parentAlias),
                $subQueryBuilder->expr()->in('relation.priceList', ":$parameterName")
            )
        );

        $queryBuilder->andWhere($subQueryBuilder->expr()->exists($subQueryBuilder->getQuery()->getDQL()));
        $queryBuilder->setParameter($parameterName, $priceLists);
    }
}
```

Our new filter should be registered in the service container with the oro_filter.extension.orm_filter.filter tag:

```none
grid_filter.price_lists:
    class: Oro\Bundle\CustomGridBundle\Filter\ProductPriceListsFilter
    public: false
    arguments:
        - '@form.factory'
        - '@oro_filter.filter_utility'
        - '@doctrine'
    tags:
        - { name: oro_filter.extension.orm_filter.filter, type: product-price-lists }
```

This filter can be added to the grid configuration similarly to how we added new columns – in an event listener. Thus the final implementation of the ProductsGridListener would look like this:

```php
class ProductsGridListener
{
    /**
     * @var ManagerRegistry
     */
    protected $doctrine;

    /**
     * @param ManagerRegistry $doctrine
     */
    public function __construct(ManagerRegistry $doctrine)
    {
        $this->doctrine = $doctrine;
    }

    /**
     * @param BuildBefore $event
     */
    public function onBuildBefore(BuildBefore $event)
    {
        $datagridConfiguration = $event->getConfig();
        $this->addBusinessUnitColumn($datagridConfiguration);
        $this->addPriceListsColumn($datagridConfiguration);
        $this->addPriceListsFilter($datagridConfiguration);
    }

    /**
     * @param OrmResultAfter $event
     */
    public function onResultAfter(OrmResultAfter $event)
    {
        $records = $event->getRecords();
        $this->addPriceListsToRecords($records);
    }

    /**
     * @param DatagridConfiguration $datagridConfiguration
     */
    protected function addPriceListsColumn(DatagridConfiguration $datagridConfiguration)
    {
        $column = [
            'label' => 'Price Lists',
            'type' => 'twig',
            'template' => '@OroCustomGrid/Datagrid/Column/price_lists.html.twig',
            'frontend_type' => 'html',
            'renderable' => true,
        ];
        $datagridConfiguration->addColumn('price_lists', $column);
    }

    /**
     * @param DatagridConfiguration $datagridConfiguration
     */
    protected function addBusinessUnitColumn(DatagridConfiguration $datagridConfiguration)
    {
        $datagridConfiguration->joinTable(
            'left',
            [
                'join' => BusinessUnit::class,
                'alias' => 'business_unit',
                'conditionType' => 'WITH',
                'condition' => 'product.owner = business_unit',
            ]
        );

        $column = [
            'label' => 'Owner'
        ];

        // column name should be the same as the field alias in the select query
        $datagridConfiguration->addColumn('owner', $column, 'business_unit.name as owner');
    }

    /**
     * @param DatagridConfiguration $datagridConfiguration
     */
    protected function addPriceListsFilter(DatagridConfiguration $datagridConfiguration)
    {
        $filter = [
            'type' => 'product-price-lists',
            'data_name' => 'price_lists',
            'options' => [
                'field_type' => 'entity',
                'field_options' => [
                    'class' => PriceList::class,
                    'property' => 'name',
                    'multiple' => true
                ]
            ]
        ];

        $datagridConfiguration->addFilter('price_lists', $filter);
    }

    /**
     * @param ResultRecord[] $records
     * @throws \Doctrine\ORM\ORMException
     */
    protected function addPriceListsToRecords(array $records)
    {
        $repository = $this->doctrine->getRepository(PriceListToProduct::class);
        /** @var EntityManager $objectManager */
        $objectManager = $this->doctrine->getManager();

        $products = [];
        foreach ($records as $record) {
            $products[] = $objectManager->getReference(Product::class, $record->getValue('id'));
        }

        $priceLists = [];
        foreach ($repository->findBy(['product' => $products]) as $item) {
            $priceLists[$item->getProduct()->getId()][] = $item->getPriceList();
        }

        /** @var ResultRecord $record */
        foreach ($records as $record) {
            $id = $record->getValue('id');
            $products[] = $objectManager->getReference(Product::class, $id);

            $record->addData(['price_lists' => $priceLists[$id]]);
        }
    }
}
```

A fully working example, organized into a custom bundle, is available in <a href="https://github.com/oroinc/orocommerce-sample-extensions/releases/download/0.1/CustomGridBundle.zip" target="_blank">the CustomGridBundle.zip file</a> (Download 13.47 KB).

To add this bundle to your application, please extract the content of the zip archive into a source code directory that is recognized by your composer autoload settings.

**Related Articles**

* [Datagrids](../data-grids/index.md#data-grids)
* [Datagrid Configuration Reference](../../configuration/yaml/datagrids.md#reference-format-datagrids)

<!-- Frontend -->
