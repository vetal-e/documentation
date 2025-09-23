<a id="book-entities-extended-entities-associations"></a>

# Extended Associations

This topic shows how to create different types of relationships between entities when you know the entity type for
the target side of the relationship. For more complex cases, see
[Multi-Target Extended Associations](multi-target-associations.md#book-entities-extended-entities-multi-target-associations).

## Limitations

A new relationship can be created between two entities when at least one entity on the *owning* side of the relationship
(the one that owns the foreign key in the database) is extendable. This rule enables you to create a relationship for
the following combinations of entities:

|                       | Extendable entity                                                                                                      | Non-extendable entity                                           |
|-----------------------|------------------------------------------------------------------------------------------------------------------------|-----------------------------------------------------------------|
| Extendable entity     | bidirectional and unidirectional many-to-many, bidirectional and unidirectional many-to-one, bidirectional one-to-many | many-to-many and many-to-one relationships, unidirectional only |
| Non-extendable entity | None                                                                                                                   | None                                                            |

## Many-To-One, Unidirectional

```php
namespace Acme\Bundle\DemoBundle\Migrations\Schema\v1_5;

use Doctrine\DBAL\Schema\Schema;
use Oro\Bundle\EntityExtendBundle\EntityConfig\ExtendScope;
use Oro\Bundle\EntityExtendBundle\Migration\Extension\ExtendExtension;
use Oro\Bundle\EntityExtendBundle\Migration\Extension\ExtendExtensionAwareInterface;
use Oro\Bundle\MigrationBundle\Migration\Migration;
use Oro\Bundle\MigrationBundle\Migration\QueryBag;

//Extended Associations. Many-To-One, Unidirectional

class AddDocument1RelationToUser implements Migration, ExtendExtensionAwareInterface
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
        $this->addRelationsToUser($schema);
    }

    private function addRelationsToUser(Schema $schema): void
    {
        $this->extendExtension->addManyToOneRelation(
            $schema,
            'oro_user', // owning side table
            'doc1_many_to_one_unidirect_rel', // owning side field name
            'acme_demo_document', // inverse side table
            'id', // column name is used to show related entity
            [
                'extend' => [
                    'owner' => ExtendScope::OWNER_CUSTOM
                ],
                'entity' => ['label' => 'Doc1 ManyToOneUnidirectRel'],
            ]
        );
    }
}
```

## Many-To-One, Bidirectional

```php
namespace Acme\Bundle\DemoBundle\Migrations\Schema\v1_5;

use Doctrine\DBAL\Schema\Schema;
use Oro\Bundle\EntityExtendBundle\EntityConfig\ExtendScope;
use Oro\Bundle\EntityExtendBundle\Migration\Extension\ExtendExtension;
use Oro\Bundle\EntityExtendBundle\Migration\Extension\ExtendExtensionAwareInterface;
use Oro\Bundle\MigrationBundle\Migration\Migration;
use Oro\Bundle\MigrationBundle\Migration\QueryBag;

//Extended Associations. Many-To-One, Bidirectional

class AddDocument2RelationToUser implements Migration, ExtendExtensionAwareInterface
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
        $this->addRelationsToUser($schema);
    }

    private function addRelationsToUser(Schema $schema): void
    {
        $this->extendExtension->addManyToOneRelation(
            $schema,
            'oro_user', // owning side table
            'doc2_many_to_one_bidirect_rel', // owning side field name
            'acme_demo_document', // inverse side table
            'id', // column name is used to show related entity
            [
                'extend' => [
                    'owner' => ExtendScope::OWNER_CUSTOM
                ],
                'entity' => ['label' => 'Doc2 ManyToOneBidirectRel'],
            ]
        );
        $this->extendExtension->addManyToOneInverseRelation(
            $schema,
            'oro_user', // owning side table
            'doc2_many_to_one_bidirect_rel', // owning side field name
            'acme_demo_document', // inverse side table
            'users2_many_to_one_bidirect_rel', // inverse side field name
            ['username'], // column names are used to show a title of owning side entity
            ['username'], // column names are used to show detailed info about owning side entity
            ['username'], // Column names are used to show owning side entity in a grid
            [
                'extend' => [
                    'owner' => ExtendScope::OWNER_CUSTOM
                ],
                'entity' => ['label' => 'Users2 ManyToOneBidirectRel'],
            ]
        );
    }
}
```

## Many-To-Many, Unidirectional

```php
namespace Acme\Bundle\DemoBundle\Migrations\Schema\v1_5;

use Doctrine\DBAL\Schema\Schema;
use Oro\Bundle\EntityExtendBundle\EntityConfig\ExtendScope;
use Oro\Bundle\EntityExtendBundle\Migration\Extension\ExtendExtension;
use Oro\Bundle\EntityExtendBundle\Migration\Extension\ExtendExtensionAwareInterface;
use Oro\Bundle\MigrationBundle\Migration\Migration;
use Oro\Bundle\MigrationBundle\Migration\QueryBag;

//Extended Associations. Many-To-Many, Unidirectional

class AddDocument3RelationToUser implements Migration, ExtendExtensionAwareInterface
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
        $this->addRelationsToUser($schema);
    }

    private function addRelationsToUser(Schema $schema): void
    {
        $this->extendExtension->addManyToManyRelation(
            $schema,
            'oro_user', // owning side table
            'doc3_many_to_many_unidirect_rel', // owning side field name
            'acme_demo_document', // inverse side table
            ['subject'], // column names are used to show a title of related entity
            ['description'], // column names are used to show detailed info about related entity
            ['subject'], // Column names are used to show related entity in a grid
            [
                'extend' => [
                    'owner' => ExtendScope::OWNER_CUSTOM
                ],
                'entity' => ['label' => 'Doc3 ManyToManyUnidirectRel'],
            ]
        );
    }
}
```

## Many-To-Many, Unidirectional, Without Default Entity

```php
namespace Acme\Bundle\DemoBundle\Migrations\Schema\v1_5;

use Doctrine\DBAL\Schema\Schema;
use Oro\Bundle\EntityExtendBundle\EntityConfig\ExtendScope;
use Oro\Bundle\EntityExtendBundle\Migration\Extension\ExtendExtension;
use Oro\Bundle\EntityExtendBundle\Migration\Extension\ExtendExtensionAwareInterface;
use Oro\Bundle\MigrationBundle\Migration\Migration;
use Oro\Bundle\MigrationBundle\Migration\QueryBag;

//Extended Associations. Many-To-Many, Unidirectional, Without Default Entity

class AddDocument4RelationToUser implements Migration, ExtendExtensionAwareInterface
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
        $this->addRelationsToUser($schema);
    }

    private function addRelationsToUser(Schema $schema): void
    {
        $this->extendExtension->addManyToManyRelation(
            $schema,
            'oro_user', // owning side table
            'doc4_many_to_many_unidirect_w_rel', // owning side field name
            'acme_demo_document', // inverse side table
            ['subject'], // column names are used to show a title of related entity
            ['description'], // column names are used to show detailed info about related entity
            ['subject'], // Column names are used to show related entity in a grid
            [
                'extend' => [
                    'owner' => ExtendScope::OWNER_CUSTOM,
                    'without_default' => true
                ],
                'entity' => ['label' => 'Doc4 ManyToManyUnidirectWRel'],
            ]
        );
    }
}
```

<!-- Many-To-Many, Bidirectional
--------------------------- -->
<!-- .. code-block:: php
:caption: src/Acme/Bundle/DemoBundle/Migrations/Schema/v1_7/AddDocumentRelationToUser.php -->
<!-- namespace Acme\Bundle\DemoBundle\Migrations\Schema\v1_7; -->
<!-- use Doctrine\DBAL\Schema\Schema;
use Oro\Bundle\EntityExtendBundle\EntityConfig\ExtendScope;
use Oro\Bundle\EntityExtendBundle\Migration\Extension\ExtendExtension;
use Oro\Bundle\EntityExtendBundle\Migration\Extension\ExtendExtensionAwareInterface;
use Oro\Bundle\MigrationBundle\Migration\Migration;
use Oro\Bundle\MigrationBundle\Migration\QueryBag; -->
<!-- class AddDocumentRelationToUser implements Migration, ExtendExtensionAwareInterface
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
        $this->addRelationsToUser($schema);
    }
    /**
     * @param Schema $schema
     * @return void
     */
    private function addRelationsToUser(Schema $schema): void
    {
        $this->extendExtension->addManyToManyRelation(
            $schema,
            'oro_user', // owning side table
            'document', // owning side field name
            'acme_document', // inverse side table
            ['subject'], // column names are used to show a title of related entity
            ['description'], // column names are used to show detailed info about related entity
            ['subject'], // Column names are used to show related entity in a grid
            [
                'extend' => [
                    'owner' => ExtendScope::OWNER_CUSTOM,
                    'without_default' => true
                ]
            ]
        );
        $this->extendExtension->addManyToManyInverseRelation(
            $schema,
            'oro_user', // owning side table
            'document', // owning side field name
            'acme_document', // inverse side table
            'users', // inverse side field name
            ['username'], // column names are used to show a title of owning side entity
            ['username'], // column names are used to show detailed info about owning side entity
            ['username'], // Column names are used to show owning side entity in a grid
            [
                'extend' => [
                    'owner' => ExtendScope::OWNER_CUSTOM
                ]
            ]
        );
    }
} -->
<!-- Many-To-Many, Bidirectional, Without Default Entity
--------------------------------------------------- -->
<!-- .. code-block:: php
:caption: src/Acme/Bundle/DemoBundle/Migrations/Schema/v1_7/AddDocumentRelationToUser.php -->
<!-- namespace Acme\Bundle\DemoBundle\Migrations\Schema\v1_7; -->
<!-- use Doctrine\DBAL\Schema\Schema;
use Oro\Bundle\EntityExtendBundle\EntityConfig\ExtendScope;
use Oro\Bundle\EntityExtendBundle\Migration\Extension\ExtendExtension;
use Oro\Bundle\EntityExtendBundle\Migration\Extension\ExtendExtensionAwareInterface;
use Oro\Bundle\MigrationBundle\Migration\Migration;
use Oro\Bundle\MigrationBundle\Migration\QueryBag; -->
<!-- class AddDocumentRelationToUser implements Migration, ExtendExtensionAwareInterface
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
        $this->addRelationsToUser($schema);
    }
    /**
     * @param Schema $schema
     * @return void
     */
    private function addRelationsToUser(Schema $schema): void
    {
        $this->extendExtension->addManyToManyRelation(
            $schema,
            'oro_user', // owning side table
            'document', // owning side field name
            'acme_document', // inverse side table
            ['subject'], // column names are used to show a title of related entity
            ['description'], // column names are used to show detailed info about related entity
            ['subject'], // Column names are used to show related entity in a grid
            [
                'extend' => [
                    'owner' => ExtendScope::OWNER_CUSTOM,
                    'without_default' => true
                ]
            ]
        );
        $this->extendExtension->addManyToManyInverseRelation(
            $schema,
            'oro_user', // owning side table
            'document', // owning side field name
            'acme_document', // inverse side table
            'users', // inverse side field name
            ['username'], // column names are used to show a title of owning side entity
            ['username'], // column names are used to show detailed info about owning side entity
            ['username'], // Column names are used to show owning side entity in a grid
            [
                'extend' => [
                    'owner' => ExtendScope::OWNER_CUSTOM
                ]
            ]
        );
    }
} -->

## One-To-Many, Unidirectional

```php
namespace Acme\Bundle\DemoBundle\Migrations\Schema\v1_5;

use Doctrine\DBAL\Schema\Schema;
use Oro\Bundle\EntityExtendBundle\EntityConfig\ExtendScope;
use Oro\Bundle\EntityExtendBundle\Migration\Extension\ExtendExtension;
use Oro\Bundle\EntityExtendBundle\Migration\Extension\ExtendExtensionAwareInterface;
use Oro\Bundle\MigrationBundle\Migration\Migration;
use Oro\Bundle\MigrationBundle\Migration\QueryBag;

//Extended Associations. One-To-Many, Unidirectional

class AddDocument5RelationToUser implements Migration, ExtendExtensionAwareInterface
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
        $this->addRelationsToUser($schema);
    }

    private function addRelationsToUser(Schema $schema): void
    {
        $this->extendExtension->addOneToManyRelation(
            $schema,
            'oro_user', // owning side table
            'doc5_one_to_many_unidirect_rel', // owning side field name
            'acme_demo_document', // inverse side table
            ['subject'], // column names are used to show a title of related entity
            ['description'], // column names are used to show detailed info about related entity
            ['subject'], // Column names are used to show related entity in a grid
            [
                'extend' => [
                    'owner' => ExtendScope::OWNER_CUSTOM
                ],
                'entity' => ['label' => 'Doc5 OneToManyUnidirectRel'],
            ]
        );
    }
}
```

## One-To-Many, Unidirectional, Without Default Entity

```php
namespace Acme\Bundle\DemoBundle\Migrations\Schema\v1_5;

use Doctrine\DBAL\Schema\Schema;
use Oro\Bundle\EntityExtendBundle\EntityConfig\ExtendScope;
use Oro\Bundle\EntityExtendBundle\Migration\Extension\ExtendExtension;
use Oro\Bundle\EntityExtendBundle\Migration\Extension\ExtendExtensionAwareInterface;
use Oro\Bundle\MigrationBundle\Migration\Migration;
use Oro\Bundle\MigrationBundle\Migration\QueryBag;

//Extended Associations. One-To-Many, Unidirectional, Without Default Entity

class AddDocument6RelationToUser implements Migration, ExtendExtensionAwareInterface
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
        $this->addRelationsToUser($schema);
    }

    private function addRelationsToUser(Schema $schema): void
    {
        $this->extendExtension->addOneToManyRelation(
            $schema,
            'oro_user', // owning side table
            'doc6_one_to_many_unidirect_w_rel', // owning side field name
            'acme_demo_document', // inverse side table
            ['subject'], // column names are used to show a title of related entity
            ['description'], // column names are used to show detailed info about related entity
            ['subject'], // Column names are used to show related entity in a grid
            [
                'extend' => [
                    'owner' => ExtendScope::OWNER_CUSTOM,
                    'without_default' => true
                ],
                'entity' => ['label' => 'Doc6 OneToManyUnidirectRel'],
            ]
        );
    }
}
```

## One-To-Many, Bidirectional

```php
namespace Acme\Bundle\DemoBundle\Migrations\Schema\v1_5;

use Doctrine\DBAL\Schema\Schema;
use Oro\Bundle\EntityExtendBundle\EntityConfig\ExtendScope;
use Oro\Bundle\EntityExtendBundle\Migration\Extension\ExtendExtension;
use Oro\Bundle\EntityExtendBundle\Migration\Extension\ExtendExtensionAwareInterface;
use Oro\Bundle\MigrationBundle\Migration\Migration;
use Oro\Bundle\MigrationBundle\Migration\QueryBag;

//Extended Associations. One-To-Many, Bidirectional

class AddDocument7RelationToUser implements Migration, ExtendExtensionAwareInterface
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
        $this->addRelationsToUser($schema);
    }

    private function addRelationsToUser(Schema $schema): void
    {
        $this->extendExtension->addOneToManyRelation(
            $schema,
            'oro_user', // owning side table
            'doc7_one_to_many_bidirect_rel', // owning side field name
            'acme_demo_document', // inverse side table
            ['subject'], // column names are used to show a title of related entity
            ['description'], // column names are used to show detailed info about related entity
            ['subject'], // Column names are used to show related entity in a grid
            [
                'extend' => [
                    'owner' => ExtendScope::OWNER_CUSTOM
                ],
                'entity' => ['label' => 'Doc7 OneToManyBidirectRel'],
            ]
        );
        $this->extendExtension->addOneToManyInverseRelation(
            $schema,
            'oro_user', // owning side table
            'doc7_one_to_many_bidirect_rel', // owning side field name
            'acme_demo_document', // inverse side table
            'users7_one_to_many_bidirect_rel', // inverse side field name
            'username', // column name are used to show a title of owning side entity
            [
                'extend' => [
                    'owner' => ExtendScope::OWNER_CUSTOM
                ],
                'entity' => ['label' => 'Users7 OneToManyBidirectRel'],
            ]
        );
    }
}
```

## One-To-Many, Bidirectional, Without Default Entity

```php
namespace Acme\Bundle\DemoBundle\Migrations\Schema\v1_5;

use Doctrine\DBAL\Schema\Schema;
use Oro\Bundle\EntityExtendBundle\EntityConfig\ExtendScope;
use Oro\Bundle\EntityExtendBundle\Migration\Extension\ExtendExtension;
use Oro\Bundle\EntityExtendBundle\Migration\Extension\ExtendExtensionAwareInterface;
use Oro\Bundle\MigrationBundle\Migration\Migration;
use Oro\Bundle\MigrationBundle\Migration\QueryBag;

//Extended Associations. One-To-Many, Bidirectional, Without Default Entity

class AddDocument8RelationToUser implements Migration, ExtendExtensionAwareInterface
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
        $this->addRelationsToUser($schema);
    }

    private function addRelationsToUser(Schema $schema): void
    {
        $this->extendExtension->addOneToManyRelation(
            $schema,
            'oro_user', // owning side table
            'doc8_one_to_many_bidirect_w_rel', // owning side field name
            'acme_demo_document', // inverse side table
            ['subject'], // column names are used to show a title of related entity
            ['description'], // column names are used to show detailed info about related entity
            ['subject'], // Column names are used to show related entity in a grid
            [
                'extend' => [
                    'owner' => ExtendScope::OWNER_CUSTOM,
                    'without_default' => true
                ],
                'entity' => ['label' => 'Doc8 OneToManyBidirectWRel'],
            ]
        );
        $this->extendExtension->addOneToManyInverseRelation(
            $schema,
            'oro_user', // owning side table
            'doc8_one_to_many_bidirect_w_rel', // owning side field name
            'acme_demo_document', // inverse side table
            'user8_one_to_many_bidirect_rel_w', // inverse side field name
            'username', // column name are used to show a title of owning side entity
            [
                'extend' => [
                    'owner' => ExtendScope::OWNER_CUSTOM
                ],
                'entity' => ['label' => 'Users8 OneToManyBidirectWRel'],
            ]
        );
    }
}
```

<!-- Frontend -->
