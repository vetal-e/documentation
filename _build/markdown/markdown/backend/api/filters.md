<a id="api-filters"></a>

# Filters

This chapter provides information on the existing filters and illustrates how to create new filters.

Filters are used to limit a set of data or request additional information returned by the API.

Filters for fields that have a database index are enabled automatically. Filters by all other fields should be
[enabled explicitly](configuration.md#filters-config).

<a id="comparisonfilter-filter"></a>

## ComparisonFilter Filter

The <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Filter/ComparisonFilter.php" target="_blank">ComparisonFilter</a> is the default filter used to filter data by a field value
using various comparison types.

All supported comparison types are listed in the following table:

| Comparison Type   | Operator   | Description                                                                                                                                                                                                                                                                                                                                               |
|-------------------|------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| eq                | =          | For fields and not collection valued associations, it checks whether a field value is equal to a filter value. For collection valued associations, it checks whether a collection contains any of the filter values.                                                                                                                                      |
| neq               | !=         | For fields and not collection valued associations, it checks whether a field value is not equal to a filter value. For collection valued associations, it checks that a collection does not contain any filter values. Records with null as the field value or an empty collection are not returned. To return them, use the neq_or_null comparison type. |
| lt                | <          | Checks whether a field value is less than a filter value. Supports numeric, date, and time fields.                                                                                                                                                                                                                                                        |
| lte               | <=         | Checks whether a field value is less than or equal to a filter value. Supports numeric, date, and time fields.                                                                                                                                                                                                                                            |
| gt                | >          | Checks whether a field value is greater than a filter value. Supports numeric, date, and time fields.                                                                                                                                                                                                                                                     |
| gte               | >=         | Checks whether a field value is greater than or equal to a filter value. Supports numeric, date, and time fields.                                                                                                                                                                                                                                         |
| exists            | \*         | For fields and not collection valued associations, it checks whether a field value is not null (if a filter value is true) or a field value is null (if a filter value is false). For collection valued associations, it checks whether a collection is not empty (if a filter value is true) or empty (if a filter value is false).                      |
| neq_or_null       | !\*        | For fields and not collection valued associations checks whether a field value is not equal to a filter value, or is null. For collection valued associations, it  checks whether a collection does not contain any filter values or is empty.                                                                                                            |
| contains          | ~          | For string fields, it checks whether a field value contains a filter value. The LIKE ‘%value%’ comparison is used. For collection valued associations, it checks whether a collection contains all of the filter values.                                                                                                                                  |
| not_contains      | !~         | For string fields, it checks that a field value does not contain a filter value. The NOT LIKE ‘%value%’ comparison is used. For collection valued associations, it checks that a collection does not contain all of the filter values.                                                                                                                    |
| starts_with       | ^          | Checks whether a field value starts with a filter value. The LIKE ‘value%’ comparison is used. Supports only string fields.                                                                                                                                                                                                                               |
| not_starts_with   | !^         | Checks that a field value does not start with a filter value. The NOT LIKE ‘value%’ comparison is used. Supports only string fields.                                                                                                                                                                                                                      |
| ends_with         | $          | Checks whether a field value ends with a filter value. The LIKE ‘%value’ comparison is used. Supports only string fields.                                                                                                                                                                                                                                 |
| not_ends_with     | !$         | Checks that a field value does not end with a filter value. The NOT LIKE ‘%value’ comparison is used. Supports only string fields.                                                                                                                                                                                                                        |
| empty             |            | Checks whether a field value is empty or null (if a filter value is true) or a field value is not empty and not null (if a filter value is false). Supports string and array fields.                                                                                                                                                                      |

<a id="web-api-existing-filters"></a>

## Existing Filters

A list of filters that are configured automatically according to the data type of a field:

| Data Type / Filter Type   | Operators enabled by default   |
|---------------------------|--------------------------------|
| string                    | =, !=, \*, !\*                 |
| boolean                   | =, !=, \*, !\*                 |
| integer                   | =, !=, <, <=, >, >=, \*, !\*   |
| smallint                  | =, !=, <, <=, >, >=, \*, !\*   |
| bigint                    | =, !=, <, <=, >, >=, \*, !\*   |
| unsignedInteger           | =, !=, <, <=, >, >=, \*, !\*   |
| decimal                   | =, !=, <, <=, >, >=, \*, !\*   |
| float                     | =, !=, <, <=, >, >=, \*, !\*   |
| date                      | =, !=, <, <=, >, >=, \*, !\*   |
| time                      | =, !=, <, <=, >, >=, \*, !\*   |
| datetime                  | =, !=, <, <=, >, >=, \*, !\*   |
| guid                      | =, !=, \*, !\*                 |
| text                      | \*                             |
| percent                   | =, !=, <, <=, >, >=, \*, !\*   |
| money                     | =, !=, <, <=, >, >=, \*, !\*   |
| money_value               | =, !=, <, <=, >, >=, \*, !\*   |
| currency                  | =, !=, \*, !\*                 |
| duration                  | =, !=, <, <=, >, >=, \*, !\*   |

All these filters are implemented by [ComparisonFilter](#comparisonfilter-filter).

See [Enable Advanced Operators for String Filter](how-to.md#advanced-operators-for-string-filter) and [Enable Case-insensitive String Filter](how-to.md#case-insensitive-string-filter) for examples of advanced filter configuration.

The following filters are also configured automatically:

- The composite_identifier filter for the ID field if an entity has a composite identifier.
  The operators enabled for this filter are =, !=.
  It is implemented by <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Filter/CompositeIdentifierFilter.php" target="_blank">CompositeIdentifierFilter</a>.
- The association_composite_identifier filter for the ID field if an association has a composite identifier.
  The operators enabled for this filter are =, !=.
  It is implemented by <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Filter/AssociationCompositeIdentifierFilter.php" target="_blank">AssociationCompositeIdentifierFilter</a>.
- The association filter for [multi-target associations](../entities/extend-entities/multi-target-associations.md#book-entities-extended-entities-multi-target-associations).
  The operators enabled for this filter are =, !=, \*, !\*.
  It is implemented by <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Filter/ExtendedAssociationFilter.php" target="_blank">ExtendedAssociationFilter</a>.
  More details on how to configure multi-target associations are available in the following topics:
  [Configure an Extended Many-To-One Association](how-to.md#extended-many-to-one-association),
  [Configure an Extended Many-To-Many Association](how-to.md#extended-many-to-many-association) and
  [Configure an Extended Multiple Many-To-One Association](how-to.md#extended-multiple-many-to-one-association).

A list of filters that should be configured explicitly using the [type](configuration.md#filters-config) option:

| Filter Type       | Enabled Operators   | Implemented by                                                                                                                                                          |
|-------------------|---------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| primaryField      | =, !=, \*, !\*      | <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Filter/PrimaryFieldFilter.php" target="_blank">PrimaryFieldFilter</a>                  |
| nestedAssociation | =, !=, \*, !\*      | <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Filter/NestedAssociationFilter.php" target="_blank">NestedAssociationFilter</a>        |
| nestedTree        | >, >=               | <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Filter/NestedTreeFilter.php" target="_blank">NestedTreeFilter</a>                      |
| searchQuery       | =                   | <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/SearchBundle/Api/Filter/SearchQueryFilter.php" target="_blank">SearchQueryFilter</a>             |
| searchAggregation | =                   | <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/SearchBundle/Api/Filter/SearchAggregationFilter.php" target="_blank">SearchAggregationFilter</a> |

You can also run the php var/console debug:config oro_api command to view all the existing filters
in the  filters section and all the existing operators for filters in the filter_operators section.

<a id="web-api-filterinterface"></a>

## FilterInterface Interface

All filters must implement the <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Filter/FilterInterface.php" target="_blank">FilterInterface</a> interface.

Consider checking out the following classes before implementing your own filters, as each of them may serve as a good base class for your own filters:

* [StandaloneFilter](#standalonefilter-base-class)
* [StandaloneFilterWithDefaultValue](#standalonefilterwithdefaultvalue-base-class),
* [ComparisonFilter](#comparisonfilter-filter) and
* <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Filter/AssociationFilter.php" target="_blank">AssociationFilter</a>

<a id="fieldfilterinterface"></a>

## FieldFilterInterface Interface

The <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Filter/FieldFilterInterface.php" target="_blank">FieldFilterInterface</a> is a marker interface that filters applied to a field must implement.

Examples of such filters are [ComparisonFilter](#comparisonfilter-filter), <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Filter/StringComparisonFilter.php" target="_blank">StringComparisonFilter</a>, <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Filter/CompositeIdentifierFilter.php" target="_blank">CompositeIdentifierFilter</a>, <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Filter/AssociationCompositeIdentifierFilter.php" target="_blank">AssociationCompositeIdentifierFilter</a>, <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Filter/NestedTreeFilter.php" target="_blank">NestedTreeFilter</a>, <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Filter/ExtendedAssociationFilter.php" target="_blank">ExtendedAssociationFilter</a>, <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Filter/NestedAssociationFilter.php" target="_blank">NestedAssociationFilter</a> and <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Filter/PrimaryFieldFilter.php" target="_blank">PrimaryFieldFilter</a>.

<a id="fieldawarefilterinterface"></a>

## FieldAwareFilterInterface Interface

Filters that are applied to a field and need to know the field name. must implement the <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Filter/FieldAwareFilterInterface.php" target="_blank">FieldAwareFilterInterface</a> interface.

Examples of such filters are [ComparisonFilter](#comparisonfilter-filter), <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Filter/StringComparisonFilter.php" target="_blank">StringComparisonFilter</a>, <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Filter/ExtendedAssociationFilter.php" target="_blank">ExtendedAssociationFilter</a>, <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Filter/NestedAssociationFilter.php" target="_blank">NestedAssociationFilter</a>, <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Filter/PrimaryFieldFilter.php" target="_blank">PrimaryFieldFilter</a> and <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Filter/AssociationCompositeIdentifierFilter.php" target="_blank">AssociationCompositeIdentifierFilter</a>.

<a id="collectionawarefilterinterface"></a>

## CollectionAwareFilterInterface Interface

Filters that can handle a collection valued association must implement the <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Filter/CollectionAwareFilterInterface.php" target="_blank">CollectionAwareFilterInterface</a> interface.

Examples of such filters are [ComparisonFilter](#comparisonfilter-filter), <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Filter/StringComparisonFilter.php" target="_blank">StringComparisonFilter</a>, <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Filter/ExtendedAssociationFilter.php" target="_blank">ExtendedAssociationFilter</a> and <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Filter/PrimaryFieldFilter.php" target="_blank">PrimaryFieldFilter</a>.

<a id="configawarefilterinterface"></a>

## ConfigAwareFilterInterface Interface

Filters that depend on the <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Config/EntityDefinitionConfig.php" target="_blank">entity configuration</a> must implement the <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Filter/ConfigAwareFilterInterface.php" target="_blank">ConfigAwareFilterInterface</a> interface.

Examples of such filters are <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Filter/ExtendedAssociationFilter.php" target="_blank">ExtendedAssociationFilter</a> and <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Filter/NestedAssociationFilter.php" target="_blank">NestedAssociationFilter</a>.

<a id="metaawarefilterinterface"></a>

## MetadataAwareFilterInterface Interface

Filters that depend on the <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Metadata/EntityMetadata.php" target="_blank">entity metadata</a> must implement the <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Filter/MetadataAwareFilterInterface.php" target="_blank">MetadataAwareFilterInterface</a> interface.

Examples of such filters are <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Filter/CompositeIdentifierFilter.php" target="_blank">CompositeIdentifierFilter</a> and <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Filter/AssociationCompositeIdentifierFilter.php" target="_blank">AssociationCompositeIdentifierFilter</a>.

<a id="requestawarefilterinterface"></a>

## RequestAwareFilterInterface Interface

Filters that depend on a [request type](request-type.md#api-request-type) must implement the <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Filter/RequestAwareFilterInterface.php" target="_blank">RequestAwareFilterInterface</a> interface.

Examples of such filters are <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Filter/ExtendedAssociationFilter.php" target="_blank">ExtendedAssociationFilter</a>, <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Filter/NestedAssociationFilter.php" target="_blank">NestedAssociationFilter</a>, <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Filter/CompositeIdentifierFilter.php" target="_blank">CompositeIdentifierFilter</a> and <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Filter/AssociationCompositeIdentifierFilter.php" target="_blank">AssociationCompositeIdentifierFilter</a>.

<a id="selfidentifiablefilterinterface"></a>

## SelfIdentifiableFilterInterface Interface

Filters that should search for their value themselves must implement the <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Filter/SelfIdentifiableFilterInterface.php" target="_blank">SelfIdentifiableFilterInterface</a> interface.

Examples of such filters are <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Filter/ExtendedAssociationFilter.php" target="_blank">ExtendedAssociationFilter</a> and <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Filter/NestedAssociationFilter.php" target="_blank">NestedAssociationFilter</a>.

<a id="namedvaluefilterinterface"></a>

## NamedValueFilterInterface Interface

Filters with a named value should implement the <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Filter/NamedValueFilterInterface.php" target="_blank">NamedValueFilterInterface</a> interface.

Examples of such filters are <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Filter/ExtendedAssociationFilter.php" target="_blank">ExtendedAssociationFilter</a> and <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Filter/NestedAssociationFilter.php" target="_blank">NestedAssociationFilter</a>.

<a id="specialhandlingfilterinterface"></a>

## SpecialHandlingFilterInterface Interface

Filters with special handling must implement the <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Filter/SpecialHandlingFilterInterface.php" target="_blank">SpecialHandlingFilterInterface</a> interface. As a result, common normalization should not be applied to their values.

Examples of such filters are <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Filter/MetaPropertyFilter.php" target="_blank">MetaPropertyFilter</a>, <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Filter/FieldsFilter.php" target="_blank">FieldsFilter</a>, and <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Filter/IncludeFilter.php" target="_blank">IncludeFilter</a>.

<a id="standalonefilter-base-class"></a>

## StandaloneFilter Base Class

The <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Filter/StandaloneFilter.php" target="_blank">StandaloneFilter</a> is the base class for filters you can use independently of other filters.

Examples of such filters are:

* [ComparisonFilter](#comparisonfilter-filter)
* <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Filter/StringComparisonFilter.php" target="_blank">StringComparisonFilter</a>
* <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Filter/ExtendedAssociationFilter.php" target="_blank">ExtendedAssociationFilter</a>
* <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Filter/NestedAssociationFilter.php" target="_blank">NestedAssociationFilter</a>
* <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Filter/CompositeIdentifierFilter.php" target="_blank">CompositeIdentifierFilter</a>
* <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Filter/AssociationCompositeIdentifierFilter.php" target="_blank">AssociationCompositeIdentifierFilter</a>
* <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Filter/NestedTreeFilter.php" target="_blank">NestedTreeFilter</a>
* <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/SearchBundle/Api/Filter/SearchQueryFilter.php" target="_blank">SearchQueryFilter</a>
* <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/SearchBundle/Api/Filter/SearchAggregationFilter.php" target="_blank">SearchAggregationFilter</a>
* <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/SearchBundle/Api/Filter/SimpleSearchFilter.php" target="_blank">SimpleSearchFilter</a>
* <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Filter/PrimaryFieldFilter.php" target="_blank">PrimaryFieldFilter</a>

<a id="standalonefilterwithdefaultvalue-base-class"></a>

## StandaloneFilterWithDefaultValue Base Class

The <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Filter/StandaloneFilter.php" target="_blank">StandaloneFilterWithDefaultValue</a> is the base class for filters
that you can use independently of other filters and have a predefined default value.

Examples of such filters are <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Filter/PageNumberFilter.php" target="_blank">PageNumberFilter</a>, <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Filter/PageSizeFilter.php" target="_blank">PageSizeFilter</a>  and <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Filter/SortFilter.php" target="_blank">SortFilter</a>.

## Criteria Class

The <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Collection/Criteria.php" target="_blank">Criteria</a> class represents criteria for filtering data returned by ORM queries.
This class extends the <a href="https://github.com/doctrine/collections/blob/2.0.x/src/Criteria.php" target="_blank">Doctrine Criteria</a> class and adds methods to work with joins.
It is required as API filters can be applied to associations at any nesting level.

## CriteriaConnector Class

The <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Util/CriteriaConnector.php" target="_blank">CriteriaConnector</a> class is used to apply criteria stored in the Criteria object to the QueryBuilder object.

This class uses <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Util/CriteriaNormalizer.php" target="_blank">CriteriaNormalizer</a> to prepare the Criteria object before criteria are applied to the QueryBuilder object.

Keep in mind that you can decorate <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Util/RequireJoinsDecisionMakerInterface.php" target="_blank">RequireJoinsDecisionMakerInterface</a> and <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Util/OptimizeJoinsDecisionMakerInterface.php" target="_blank">OptimizeJoinsDecisionMakerInterface</a> interfaces
and oro_api.query.require_joins_decision_maker and oro_api.query.optimize_joins_decision_maker services if your expressions require this.

## QueryExpressionVisitor Class

The <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Collection/QueryExpressionVisitor.php" target="_blank">QueryExpressionVisitor</a> is used to walk a graph of DQL expressions from the Criteria object and turns them into a query. This class is similar to
<a href="https://github.com/doctrine/doctrine2/blob/master/lib/Doctrine/ORM/Query/QueryExpressionVisitor.php" target="_blank">Doctrine QueryExpressionVisitor</a>, but allows adding new types of expressions easily and helps to build subquery-based expressions.

<a id="web-api-query-expressions"></a>

## Query Expressions

The following query expressions are implemented out-of-the-box:

| Operator              | Class                                                                                                                                                                                                           | Description                                                                              |
|-----------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|------------------------------------------------------------------------------------------|
| AND                   | <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Collection/QueryVisitorExpression/AndCompositeExpression.php" target="_blank">AndCompositeExpression</a>                       | Logical AND                                                                              |
| OR                    | <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Collection/QueryVisitorExpression/OrCompositeExpression.php" target="_blank">OrCompositeExpression</a>                         | Logical OR                                                                               |
| NOT                   | <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Collection/QueryVisitorExpression/NotCompositeExpression.php" target="_blank">NotCompositeExpression</a>                       | Logical NOT                                                                              |
| =                     | <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Collection/QueryVisitorExpression/EqComparisonExpression.php" target="_blank">EqComparisonExpression</a>                       | EQUAL TO comparison                                                                      |
| <>                    | <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Collection/QueryVisitorExpression/NeqComparisonExpression.php" target="_blank">NeqComparisonExpression</a>                     | NOT EQUAL TO comparison                                                                  |
| <                     | <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Collection/QueryVisitorExpression/LtComparisonExpression.php" target="_blank">LtComparisonExpression</a>                       | LESS THAN comparison                                                                     |
| <=                    | <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Collection/QueryVisitorExpression/LteComparisonExpression.php" target="_blank">LteComparisonExpression</a>                     | LESS THAN OR EQUAL TO comparison                                                         |
| >                     | <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Collection/QueryVisitorExpression/GtComparisonExpression.php" target="_blank">GtComparisonExpression</a>                       | GREATER THAN comparison                                                                  |
| >=                    | <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Collection/QueryVisitorExpression/GteComparisonExpression.php" target="_blank">GteComparisonExpression</a>                     | GREATER THAN OR EQUAL TO comparison                                                      |
| IN                    | <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Collection/QueryVisitorExpression/InComparisonExpression.php" target="_blank">InComparisonExpression</a>                       | IN comparison                                                                            |
| NIN                   | <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Collection/QueryVisitorExpression/NinComparisonExpression.php" target="_blank">NinComparisonExpression</a>                     | NOT IN comparison                                                                        |
| EXISTS                | <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Collection/QueryVisitorExpression/ExistsComparisonExpression.php" target="_blank">ExistsComparisonExpression</a>               | EXISTS (IS NOT NULL) and DOES NOT EXIST (IS NULL) comparisons                            |
| EMPTY                 | <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Collection/QueryVisitorExpression/EmptyComparisonExpression.php" target="_blank">EmptyComparisonExpression</a>                 | EMPTY and NOT EMPTY comparisons for collections                                          |
| NEQ_OR_NULL           | <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Collection/QueryVisitorExpression/NeqOrNullComparisonExpression.php" target="_blank">NeqOrNullComparisonExpression</a>         | NOT EQUAL TO OR IS NULL comparison                                                       |
| NEQ_OR_EMPTY          | <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Collection/QueryVisitorExpression/NeqOrEmptyComparisonExpression.php" target="_blank">NeqOrEmptyComparisonExpression</a>       | NOT EQUAL TO OR EMPTY comparison for collections                                         |
| MEMBER_OF             | <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Collection/QueryVisitorExpression/MemberOfComparisonExpression.php" target="_blank">MemberOfComparisonExpression</a>           | checks whether a collection contains any of specific values                              |
| ALL_MEMBER_OF         | <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Collection/QueryVisitorExpression/AllMemberOfComparisonExpression.php" target="_blank">AllMemberOfComparisonExpression</a>     | checks whether a collection contains all of specific values                              |
| ALL_NOT_MEMBER_OF     | <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Collection/QueryVisitorExpression/AllMemberOfComparisonExpression.php" target="_blank">AllMemberOfComparisonExpression</a>     | checks whether a collection does not contain all of specific values                      |
| CONTAINS              | <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Collection/QueryVisitorExpression/ContainsComparisonExpression.php" target="_blank">ContainsComparisonExpression</a>           | LIKE `%value%` comparison                                                                |
| NOT_CONTAINS          | <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Collection/QueryVisitorExpression/NotContainsComparisonExpression.php" target="_blank">NotContainsComparisonExpression</a>     | NOT LIKE `%value%` comparison                                                            |
| STARTS_WITH           | <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Collection/QueryVisitorExpression/StartsWithComparisonExpression.php" target="_blank">StartsWithComparisonExpression</a>       | LIKE `value%` comparison                                                                 |
| NOT_STARTS_WITH       | <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Collection/QueryVisitorExpression/NotStartsWithComparisonExpression.php" target="_blank">NotStartsWithComparisonExpression</a> | NOT LIKE `value%` comparison                                                             |
| ENDS_WITH             | <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Collection/QueryVisitorExpression/EndsWithComparisonExpression.php" target="_blank">EndsWithComparisonExpression</a>           | LIKE `%value` comparison                                                                 |
| NOT_ENDS_WITH         | <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Collection/QueryVisitorExpression/NotEndsWithComparisonExpression.php" target="_blank">NotEndsWithComparisonExpression</a>     | NOT LIKE `%value` comparison                                                             |
| EMPTY_VALUE           | <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Collection/QueryVisitorExpression/EmptyValueComparisonExpression.php" target="_blank">EmptyValueComparisonExpression</a>       | EQUAL TO empty value OR IS NULL and NOT EQUAL TO empty value AND NOT IS NULL comparisons |
| ENTITY                | <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Collection/QueryVisitorExpression/EntityComparisonExpression.php" target="_blank">EntityComparisonExpression</a>               | checks whether a field contains an identifier of an entity matched specified criteria    |
| NESTED_TREE           | <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Collection/QueryVisitorExpression/NestedTreeComparisonExpression.php" target="_blank">NestedTreeComparisonExpression</a>       | returns all child nodes for a given node depending on the nesting level                  |
| NESTED_TREE_WITH_ROOT | <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Collection/QueryVisitorExpression/NestedTreeComparisonExpression.php" target="_blank">NestedTreeComparisonExpression</a>       | returns a given node and all child nodes for this node depending on the nesting level    |

You can add new comparison expressions and use them in your filters if necessary. For this, create a class that implements the expression logic, register it as a service tagged with the oro.api.query.comparison_expression in the dependency injection container, and decorate the <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Util/RequireJoinsDecisionMaker.php" target="_blank">oro_api.query.require_joins_decision_maker</a>
and <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Util/OptimizeJoinsDecisionMaker.php" target="_blank">oro_api.query.optimize_joins_decision_maker</a> services if required.

<a id="web-api-creating-filter"></a>

## Creating a New Filter

To create a new filter:

* Create a class that implements the filtering logic. This class must implement [FilterInterface Interface]() or extend one of the classes implementing this interface.
* If your filter is complex and depends on other services, create a factory to create the filter.
  Register the factory as a service in the dependency injection container.
* Register this class in the oro_api / filters section using Resources/config/oro/app.yml.
  You can find examples of filters registration in
  <a href="https://github.com/oroinc/platform/tree/master/src/Oro/Bundle/ApiBundle/Resources/config/oro/app.yml" target="_blank">ApiBundle/Resources/config/oro/app.yml</a>.
  > #### NOTE
  > The default value for class attribute in the oro_api / filters section is OroBundleApiBundleFilterComparisonFilter. The default value for supported_operators attribute there is [‘=’, ‘!=’, ‘\*’, ‘!\*’].

To configure your filter for an API resource, use the [type](configuration.md#filters-config) option of the filter.

<a id="web-api-other-classes"></a>

## Other Classes

Consider checking out the list of other classes below, as they can provide insight on how data filtering works:

* <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Filter/FilterNames.php" target="_blank">FilterNames</a> - contains names of predefined filters for a specific request type.
* <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Filter/FilterNamesRegistry.php" target="_blank">FilterNamesRegistry</a> - a container for names of predefined filters for all registered request types.
* <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Filter/FilterValue.php" target="_blank">FilterValue</a> - represents a filter value.
* <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Filter/FilterValueAccessorInterface.php" target="_blank">FilterValueAccessorInterface</a> - represents a collection of the FilterValue objects.
* <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Request/RestFilterValueAccessor.php" target="_blank">RestFilterValueAccessor</a> - extracts values of filters from REST API HTTP request.
* <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Filter/FilterHelper.php" target="_blank">FilterHelper</a> - reusable utility methods that can be used to get filter values.
* <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Filter/FilterCollection.php" target="_blank">FilterCollection</a> - a collection of filters.
* <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Filter/SimpleFilterFactory.php" target="_blank">SimpleFilterFactory</a> - the default implementation of a factory to create filters.
* <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Filter/FilterOperatorRegistry.php" target="_blank">FilterOperatorRegistry</a> - the container for all registered operators for filters.
* <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Filter/MetaPropertyFilter.php" target="_blank">MetaPropertyFilter</a> - a filter used to request to add entity meta properties to the result or to perform some additional operations.
* <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Processor/Shared/AddMetaPropertyFilter.php" target="_blank">AddMetaPropertyFilter</a> - a processor that adds the “meta” filter that is used to specify which entity meta properties should be returned or which additional operations should be performed.
* <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Processor/Shared/HandleMetaPropertyFilter.php" target="_blank">HandleMetaPropertyFilter</a> - a processor that handles the “meta” filter.
* <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Processor/Shared/ValidateMetaPropertyFilterSupported.php" target="_blank">ValidateMetaPropertyFilterSupported</a> - a processor that validates that the “meta” filter is supported and all requested meta properties are allowed.
* <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Processor/GetConfig/AddMetaProperties.php" target="_blank">AddMetaProperties</a> - a processor that adds the configuration of meta properties requested via the “meta” filter.
* <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Filter/FieldsFilter.php" target="_blank">FieldsFilter</a> - a filter that is used to filter entity fields.
* <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Processor/Shared/AddFieldsFilter.php" target="_blank">AddFieldsFilter</a> - a processor that adds “fields” filters that are used to filter entity fields.
* <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Processor/Shared/HandleFieldsFilter.php" target="_blank">HandleFieldsFilter</a> - a processor that handles “fields” filters.
* <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Processor/GetConfig/FilterFieldsByExtra.php" target="_blank">FilterFieldsByExtra</a> - a processor that modifies configuration of entities according to “fields” filters.
* <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Filter/IncludeFilter.php" target="_blank">IncludeFilter</a> - a filter that is used to request information about related entities.
* <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Processor/Shared/AddIncludeFilter.php" target="_blank">AddIncludeFilter</a> - a processor that adds “include” filters that are used to request information about related entities.
* <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Processor/Shared/HandleIncludeFilter.php" target="_blank">HandleIncludeFilter</a> - a processor that handles “include” filters.
* <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Processor/GetConfig/ExpandRelatedEntities.php" target="_blank">ExpandRelatedEntities</a> - a processor that adds configuration of related entities requested via “include” filters.
* <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Processor/Shared/BuildCriteria.php" target="_blank">BuildCriteria</a> - a processor that applies all requested filters to the Criteria object.
* <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Processor/Shared/NormalizeFilterValues.php" target="_blank">NormalizeFilterValues</a> - a processor that converts values of all requested filters according to the type of the filters and validates that all requested filters are supported.
* <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Processor/Shared/RegisterConfiguredFilters.php" target="_blank">RegisterConfiguredFilters</a> - a processor that registers filters according to the [filters](configuration.md#filters-config) configuration section.
* <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Processor/Shared/RegisterDynamicFilters.php" target="_blank">RegisterDynamicFilters</a> - a processor that registers nested filters.

<!-- Frontend -->
