<a id="customize-install"></a>

# Customizing the Installation Process

To customize the installation process and modify the database structure and/or data that are loaded in the OroCommerce after installation, you can:

> * [Execute Custom Migrations](#execute-custom-migrations)
> * [Load Custom Data Fixtures](#load-custom-data-fixtures)

<a id="customize-install-execute-custom-migrations"></a>

## Execute Custom Migrations

You can create your own [migrations](../../entities/migration.md#backend-entities-migrations) that can be executed during the installation.
A migration is a class which implements the `Oro\Bundle\MigrationBundle\Migration\Migration` interface:

```php
namespace Acme\Bundle\DemoBundle\Migrations\Schema\v1_0;

use Doctrine\DBAL\Schema\Schema;
use Oro\Bundle\MigrationBundle\Migration\Migration;
use Oro\Bundle\MigrationBundle\Migration\QueryBag;

class CustomMigration implements Migration
{
    #[\Override]
    public function up(Schema $schema, QueryBag $queries)
    {
        // ...
    }
}
```

#### NOTE
Entity metadata in the PHP entity classes (attributes) should exactly match what the schema migration is doing. If you create a migration that modifies the type, length, or another property of an existing entity field, please remember to make the same change in the PHP entity class attributes.

In the `Oro\Bundle\MigrationBundle\Migration\Migration::up`, you can modify the database schema and/or add additional SQL queries executed before and after the schema changes.

<a id="load-custom-data-fixtures"></a>

## Load Custom Data Fixtures

To load your own data [fixtures](../../entities/fixtures.md#backend-entities-fixtures), you will need to implement Doctrine’s  *“FixtureInterface”*:

```php
namespace Acme\Bundle\DemoBundle\Migrations\Data\ORM;

use Doctrine\Common\DataFixtures\FixtureInterface;
use Doctrine\Persistence\ObjectManager;

class CustomFixture implements FixtureInterface
{
    #[\Override]
    public function load(ObjectManager $manager)
    {
        // ...
    }
}
```

<!-- Frontend -->
