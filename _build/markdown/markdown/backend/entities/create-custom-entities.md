<a id="backend-entities-create-custom-entities"></a>

# Create Custom Entities

A custom entity is an entity that has no PHP class in any bundle. The definition of such an entity is created automatically in the Symfony cache. To create a custom entity, you can use <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/EntityExtendBundle/Migration/Extension/ExtendExtension.php" target="_blank">ExtendExtension</a>, as illustrated below:

#### NOTE
src/Acme/Bundle/DemoBundle/Migrations/Schema/v1_6/AddCustomEntity.php
```php
<?php

namespace Acme\Bundle\DemoBundle\Migrations\Schema\v1_6;

use Doctrine\DBAL\Schema\Schema;
use Oro\Bundle\EntityExtendBundle\EntityConfig\ExtendScope;
use Oro\Bundle\EntityExtendBundle\Migration\Extension\ExtendExtension;
use Oro\Bundle\EntityExtendBundle\Migration\Extension\ExtendExtensionAwareInterface;
use Oro\Bundle\MigrationBundle\Migration\Migration;
use Oro\Bundle\MigrationBundle\Migration\QueryBag;

class AddCustomEntity implements Migration, ExtendExtensionAwareInterface
{
    protected ExtendExtension $extendExtension;

    #[\Override]
    public function setExtendExtension(ExtendExtension $extendExtension)
    {
        $this->extendExtension = $extendExtension;
    }

    #[\Override]
    public function up(Schema $schema, QueryBag $queries)
    {
        $table = $this->extendExtension->createCustomEntityTable(
            $schema,
            'CustomEntity'
        );
        $table->addColumn(
            'name',
            'string',
            [
                'length' => 100,
                'oro_options' => [
                    'extend'  => ['owner' => ExtendScope::OWNER_CUSTOM],
                    'entity' => ['label' => 'name']
                ]
            ]
        );
        $this->extendExtension->addManyToOneRelation(
            $schema,
            $table,
            'users',
            'oro_user',
            'username',
            [
                'extend' => [
                    'owner' => ExtendScope::OWNER_CUSTOM
                ]
            ]
        );
    }
}
```

Adding fields and relationships to custom entities is the same as for extended entities.
For details, see [Add Entity Fields](extend-entities/index.md#book-entities-extended-entities-add-fields)
and [Add Entity Relationships](extend-entities/index.md#book-entities-extended-entities-add-relationships).

<!-- Frontend -->
