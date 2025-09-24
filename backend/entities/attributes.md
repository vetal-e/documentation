<a id="dev-entities-attributes"></a>

# Attributes Configuration

Attributes allow you to create additional entity fields dynamically. An attribute is a configuration field with an assigned value. Every attribute has a dedicated CRUD and field types, similar to the extend fields. For easier management, attributes can be grouped and nested into attribute families.

## Enabling Attributes for an Entity

You can enable attributes for any extendable and configurable entity by doing the following:

1. Add #[Config] attribute to the class with the ‘attribute’ scope and add key ‘has_attributes’ set to true.
2. Add the **attributeFamily** field with many-to-one relation to `Oro\Bundle\EntityConfigBundle\Attribute\Entity\AttributeFamily`. Make the field configurable, activate import if necessary, and add migration.
3. Implement **AttributeFamilyAwareInterface** and accessors for the **attributeFamily** field.

The following example illustrates enabling attributes for the *Document* entity:

```php
/**
 * ORM Entity Document.
 */
#[Config(
    defaultValues: [
        'attribute' => ['has_attributes' => true]
    ]
)]
class Document implements
    AttributeFamilyAwareInterface,
    // ...
{
    // ...
    /**
     * @var AttributeFamily
     */
    #[ORM\ManyToOne(targetEntity: 'Oro\Bundle\EntityConfigBundle\Attribute\Entity\AttributeFamily')]
    #[ORM\JoinColumn(name: 'attribute_family_id', referencedColumnName: 'id', onDelete: 'RESTRICT')]
    #[ConfigField(defaultValues: ['dataaudit' => ['auditable' => false], 'importexport' => ['order' => 10]])]
    protected $attributeFamily;

    /**
     * @param AttributeFamily $attributeFamily
     */
    #[\Override]
    public function setAttributeFamily(AttributeFamily $attributeFamily): self
    {
        $this->attributeFamily = $attributeFamily;

        return $this;
    }

    #[\Override]
    public function getAttributeFamily(): ?AttributeFamily
    {
        return $this->attributeFamily;
    }
}
```

```php
namespace Acme\Bundle\DemoBundle\Migrations\Schema\v1_9;

use Doctrine\DBAL\Schema\Schema;
use Oro\Bundle\MigrationBundle\Migration\Migration;
use Oro\Bundle\MigrationBundle\Migration\QueryBag;

class AddAttributeFamilyField implements Migration
{
    #[\Override]
    public function up(Schema $schema, QueryBag $queries)
    {
        $this->addAttributeFamilyField($schema);
    }

    public function addAttributeFamilyField(Schema $schema)
    {
        $table = $schema->getTable('acme_demo_document');
        $table->addColumn('attribute_family_id', 'integer', ['notnull' => false]);
        $table->addIndex(['attribute_family_id']);

        $table->addForeignKeyConstraint(
            $schema->getTable('oro_attribute_family'),
            ['attribute_family_id'],
            ['id'],
            ['onUpdate' => null, 'onDelete' => 'RESTRICT']
        );
    }
}
```

#### NOTE
Remember to clear cache and update configuration after these changes.

## Creating an Attribute

After enabling attributes for an entity, you can use routes - *oro_attribute_index*, *oro_attribute_family_index*, etc., to create and manipulate the attributes. Alias of your entity class should be passed in the route parameters to help the controller’s action identify the necessary entity. The action is not accessible when the alias is missing or invalid and when no ‘attributes’ are configured for the provided entity.

You can add routes to the navigation tree to simplify access, like in the following example:

```yaml
            document_attributes_index:
                label: acme.demo.menu.document_attributes
                route: 'oro_attribute_index'
                route_parameters:
                    alias: 'document'
                extras:
                    routes: ['oro_attribute_*']
            document_attribute_families:
                label: acme.demo.menu.document_attribute_families
                route: 'oro_attribute_family_index'
                route_parameters:
                    alias: 'document'
                extras:
                    routes: ['oro_attribute_family_*']
```

The ‘oro_attribute_create’ route is responsible for creating a new attribute. Attribute creation is split into two steps. In step 1, a user provides the attribute code used as a unique slug representation and attribute type (string, bigint, select, etc.) that defines the data that should be captured in the following step. In step 2, a user provides a label to display an attribute on the website (e.g., OroCommerce Web Store) and any other information that should be captured about the attribute. Oro application can store the attribute as a *serialized field* or as a *table column*. The type of storage is selected based on the attribute type (simple types vs. Select and Multi-Select), as well as the setting of the *Filterable* and *Sortable* options. The product attribute storage type is set to *table column* for the attribute with Select of Multi-Select data type and for an attribute of any type with Filterable or Sortable option enabled. This data type requires a reindex launched by the user when they click **Update schema** on the *All Product Attributes* page. This triggers the field to be physically created in the table.

#### NOTE
Attributes created by the user are labeled as custom, while attributes created during migrations are labeled as a system. For system attributes, deleting is disabled.

#### WARNING
Schema changes are permanent and cannot be easily rolled back. We recommend that developers back up data before any database schema change if changes have to be rolled back.

## Attribute Families and Groups

An entity has no direct relation to the attribute. Attributes are bound to the entity using the *AttributeFamily*. You can create a new attribute family for the entity using the *oro_attribute_family_create* route with the corresponding alias. The *AttributeFamily* contains a collection of *AttributeGroups*. *AttributeFamily* requires *Code* and *Labels* values to be provided and must contain at least one attribute group. Attribute groups can be created directly on the family create/edit page by adding a new group to the collection. Each group (a collection element) has a required field, ‘Label’, and a select control that allows picking one or many attributes that were previously created for the entity (in a specific class). Attributes can be added to the group, moved from one group to another, and deleted from the group (except for the system attributes that are moved to the default group on deletion).

## Attribute ACL

Attributes provide supplementary logic that helps extend entity fields marked as attributes despite limited access to the entity management.

#### NOTE
Next, you can modify the shape of the Document so that there are several steps when creating the entity. For example, you can use OroProductBundle.
