<a id="book-entities-extended-entities-serialized-fields"></a>

# Serialized Fields

OroPlatform provides the ability to create custom entities or custom fields for extended entities.
This package provides a possibility to avoid schema update when you create custom fields.

However, these fields have some restrictions. Their data is stored in the serialized_data column as a serialized array but the serialized_data field is hidden from the UI on entity config page.

#### NOTE
Serialized Enum Fields

Serialized fields have different restrictions than enum fields (select, multiselect), which are also stored in the serialized_data column. The Enum fields functionality is described in Option Set Fields.

Not supported features:

- grid filtering and sorting
- segments and reports
- charts
- search
- relations, and option set field types
- data audit
- usage of such fields in Doctrine query builder

The Serialized Fields bundle adds a new field called Storage Type within New field creation page where you need to choose one of the two storage types:

- The Table Column option enables to create custom field as usual;
- The Serialized field option means that you can avoid schema update and start to use this field immediately. Keep in mind that in this case field types are limited to the following:
  > - BigInt
  > - Boolean
  > - Date
  > - DateTime
  > - Decimal
  > - Float
  > - Integer
  > - Select
  > - Multi-select
  > - Money
  > - Percent
  > - SmallInt
  > - String
  > - Text
  > - WYSIWYG

![Basic properties available when creating a new field for an entity](user/img/system/entity_management/new_entity_field.png)

To create a serialized field via migration, use <a href="https://github.com/oroinc/OroEntitySerializedFieldsBundle/blob/master/Migration/Extension/SerializedFieldsExtension.php" target="_blank">SerializedFieldsExtension</a>. For example:

#### NOTE
src/Acme/Bundle/DemoBundle/Migrations/Schema/v1_4/AddSerializedFieldMigration.php
```php
<?php

namespace Acme\Bundle\DemoBundle\Migrations\Schema\v1_4;

use Doctrine\DBAL\Schema\Schema;
use Oro\Bundle\EntityExtendBundle\EntityConfig\ExtendScope;
use Oro\Bundle\EntitySerializedFieldsBundle\Migration\Extension\SerializedFieldsExtension;
use Oro\Bundle\EntitySerializedFieldsBundle\Migration\Extension\SerializedFieldsExtensionAwareInterface;
use Oro\Bundle\MigrationBundle\Migration\Migration;
use Oro\Bundle\MigrationBundle\Migration\QueryBag;

class AddSerializedFieldMigration implements Migration, SerializedFieldsExtensionAwareInterface
{
    protected SerializedFieldsExtension $serializedFieldsExtension;

    #[\Override]
    public function setSerializedFieldsExtension(SerializedFieldsExtension $serializedFieldsExtension)
    {
        $this->serializedFieldsExtension = $serializedFieldsExtension;
    }

    #[\Override]
    public function up(Schema $schema, QueryBag $queries)
    {
        $this->serializedFieldsExtension->addSerializedField(
            $schema->getTable('acme_demo_document'),
            'my_serialized_field',
            'string',
            [
                'extend'    => [
                    'owner' => ExtendScope::OWNER_CUSTOM,
                ],
                'entity' => ['label' => 'My serialized field'],
            ]
        );
    }
}
```

Serialized files support the same set of config options as other [configurable fields](../../configuration/annotation/config-field.md#backend-configuration-annotation-config-field).

<!-- Frontend -->
