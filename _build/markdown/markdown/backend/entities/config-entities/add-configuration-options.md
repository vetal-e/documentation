<a id="book-entities-entity-configuration-add-configuration-options"></a>

# Add Configuration Options

To add new configuration options:

1. Define the options that you want to be configurable. You can create new options per bundle, which means a bundle can extend the available options. To add new options, create the `entity_config.yml` file in your bundle as follows:

#### NOTE
src/Acme/Bundle/DemoBundle/Resources/config/oro/entity_config.yml
```yaml
entity_config:
  acme_demo:
    entity:
      items:
        comment:
          options:
            default_value: ""
            translatable: true
            indexed: true
          grid:
            type: string
            label: 'Comment'
            show_filter: true
            filterable: true
            filter_type: string
            sortable: true
          form:
            type: Symfony\Component\Form\Extension\Core\Type\TextType
            options:
              block: entity
              label: 'Comment'
    field:
      items:
        auditable:
          options:
            indexed: true
            priority: 60
          grid:
            type: boolean
            label: 'Auditable'
            show_filter: false
            filterable: true
            filter_type: boolean
            sortable: true
            required: true
          form:
            type: Oro\Bundle\EntityConfigBundle\Form\Type\ChoiceType
            options:
              block: entity
              label: 'Auditable'
              choices:
                No: false
                Yes: true
```

The key used in the first level of the entity configuration is a custom identifier used to create a kind of namespace for the additional options. For each scope, a different service is created (its name follows the schema `oro_entity_config.provider.<scope>`). For example, the service name for
the options configured in the example above is `oro_entity_config.provider.acme_demo`. It is an instance of the `Oro\Bundle\EntityConfigBundle\Provider\ConfigProvider` class.

Options can be configured on two levels: on the entity level or per field. The example above adds a new `comment` property that allows the users to add custom comments per configurable entity. It also adds the `auditable` option on the field level. This means that the user can decide for every field on an entity whether or not it should be audited.

The configured values are stored in different tables:

* Values for options on the entity level are stored in the `oro_entity_config` table.
* The `oro_entity_config_field` table stores configured values for the field level.

Below the configuration level, each option’s configuration is divided into three sections:

<a id="book-entities-configuration-options"></a>
* `options` - These values are used to configure additional behavior for the config field:

  | Option          | Description                                                                                                                                                               |
  |-----------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
  | `default_value` | The value that is used by default when no custom value was added.                                                                                                         |
  | `translatable`  | If `true`, the value entered by the user is treated as a key which is<br/>then used to look up the actual value using the Symfony translation<br/>procedure.              |
  | `indexed`       | Set this to `true` when the attribute needs to be accessed in SQL<br/>queries (see [Indexed Attributes](#book-entities-indexed-attributes)).                              |
  | `priority`      | Defines the order in which options will be shown in grid views and<br/>forms (options with a higher priority will be displayed before options<br/>with a lower priority). |
* `grid` - Configures the way the field is presented in a datagrid:

  | Option                                                 | Description                                                                                                             |
  |--------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------|
  | `type`                                                 | The attribute type                                                                                                      |
  | `label`                                                | The grid column headline                                                                                                |
  | * `show_filter`<br/>* `filterable`<br/>* `filter_type` | These options control whether the view can be filtered by the attribute<br/>value and how the filter options look like. |
  | `sortable`                                             | When enabled, the user can sort the table by clicking on the attribute<br/>column’s title.                              |

  #### NOTE
  To use the attribute in a grid view, it
  [needs to be indexed](#book-entities-indexed-attributes).
* `form` - You use these options to control how the user can configure the actual value:

  | Option          | Description                                                                                                              |
  |-----------------|--------------------------------------------------------------------------------------------------------------------------|
  | `type`          | The form type                                                                                                            |
  | `options`       | Additional options controlling the form layout:                                                                          |
  | * `block`       | The block of the form in which the attribute will be displayed                                                           |
  | * `label`       | The field label                                                                                                          |
  | * `choices`     | Possible values from which the user can choose one (this option is only<br/>available when the form type is `choice`)    |
  | * `empty_value` | The value that is taken when the user makes no choice (this option is<br/>only available when the form type is `choice`) |

2. Update all configurable entities after configuration parameters have been
modified or added using the `oro:entity-config:update` command:

```none
php bin/console oro:entity-config:update --force
```

When the `oro:entity-config:update` command is executed without using the `--force` option,
only new values will be added, but no existing parameters will be updated.

```none
php bin/console oro:entity-config:update
```

The `--dry-run` option outputs modifications without actually applying them.

```none
php bin/console oro:entity-config:update --dry-run
```

A regular expression provided with the `--filter` option is used to filter entities by their class names:

```none
php bin/console oro:entity-config:update --filter=<regexp>
```

```none
php bin/console oro:entity-config:update --filter='Oro\\Bundle\\User*'
```

```none
php bin/console oro:entity-config:update --filter='^Oro\\(.*)\\Region$'
```

<a id="book-entities-indexed-attributes"></a>

## Indexed Attributes

<a id="book-entities-entity-extension"></a>

The values the user enters when editing additional entity attributes are stored as serialized arrays in the database by default. However, when the application uses attributes in an SQL query, it needs to get the *raw* data. To achieve this, you must enable the index using the [indexed key](#book-entities-configuration-options) in the `options` section. When this option is enabled, the system will store a copy of the attribute’s value and keep it in sync when it gets updated (the indexed value is stored in the `oro_entity_config_index_value` table).

For example, it is required for fields to be visible in grids in the System > Entities section and have a filter or allow sorting in this grid. In this case, you can mark a field as indexed. For example:

```yaml
entity_config:
  acme:                                 # a configuration scope name
    entity:                             # a section describes an entity
      items:                            # starts a description of entity attributes
        demo_attr:                      # adds an attribute named 'demo_attr'
          options:
            indexed:  true              # TRUE if an attribute should be filterable or sortable in a data grid
```

When you do this, a copy of this attribute will be stored in the oro_entity_config_index_value table (and kept synchronized if a value is changed). As a result, you can write SQL query like this:

```sql
select *
from oro_entity_config c
         inner join oro_entity_config_index_value v on v.entity_id = c.id
where v.scope = 'acme_demo' and v.code = 'comment' and v.value like '%comment%';
```

<!-- Frontend -->
