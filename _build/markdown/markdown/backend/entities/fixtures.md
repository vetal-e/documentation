<a id="backend-entities-fixtures"></a>

# Fixtures

## Data Fixtures

Symfony allows loading data using data fixtures, and these fixtures run every time the doctrine:fixtures:load command is executed.

To avoid loading the same fixture several times, use **oro:migration:data:load**. This command guarantees that each data fixture is loaded only once.

This command supports two types of migration files: main data fixtures and demo data fixtures. During an installation, you can choose whether you want to load demo data or not.

Data fixtures for this command should be placed either into the Migrations/Data/ORM or Migrations/Data/Demo/ORM directory and must implement `Doctrine\Common\DataFixtures\FixtureInterface` interface.

The order of fixtures can be changed using the standard Doctrine ordering or dependency functionality. More information about fixture ordering s available in the <a href="https://github.com/doctrine/data-fixtures#fixture-ordering" target="_blank">doctrine data fixtures manual</a>.

## Versioned Fixtures

Some fixtures require execution time after time. An example is a fixture that uploads data on countries. Usually, if you add a new list with countries, you need to create a new data fixture that will upload this data. To avoid this, you can use versioned data fixtures.

To make a fixture versioned, it must implement <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/MigrationBundle/Fixture/VersionedFixtureInterface.php" target="_blank">VersionedFixtureInterface</a> and the getVersion method that returns a version of the fixture data.

Example:

#### NOTE
src/Acme/Bundle/DemoBundle/Migrations/Data/ORM/LoadFavoritesData.php
```php
<?php

namespace Acme\Bundle\DemoBundle\Migrations\Data\ORM;

use Acme\Bundle\DemoBundle\Entity\Favorite;
use Doctrine\Common\DataFixtures\AbstractFixture;
use Doctrine\Persistence\ObjectManager;
use Oro\Bundle\MigrationBundle\Fixture\VersionedFixtureInterface;
use Oro\Bundle\OrganizationBundle\Entity\Organization;

/**
 * Load favorites data 1.0.
 */
class LoadFavoritesData extends AbstractFixture implements VersionedFixtureInterface
{
    #[\Override]
    public function getVersion(): string
    {
        return '1.0';
    }

    #[\Override]
    public function load(ObjectManager $manager)
    {
        $organization = $manager->getRepository(Organization::class)->getFirst();

        $newFavorite = new Favorite();
        $newFavorite->setName('First favorite');
        $newFavorite->setValue('First favorite value');
        $newFavorite->setViewCount(14);
        $newFavorite->setOrganization($organization);
        $manager->persist($newFavorite);

        $manager->flush();
    }
}
```

In this example, the fixture will be loaded, and version 1.0 will be saved as its current loaded version.

To have the possibility to load this fixture again, it must return a version greater than 1.0, for example, 1.0.1 or 1.1. The version number must be a PHP-standardized version number string. You can find more information about PHP-standardized version number string in the <a href="http://php.net/manual/en/function.version-compare.php" target="_blank">PHP manual</a>.

If the fixture needs to know the last loaded version, it must implement <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/MigrationBundle/Fixture/LoadedFixtureVersionAwareInterface.php" target="_blank">LoadedFixtureVersionAwareInterface</a> and the setLoadedVersion method:

```php
<?php

namespace Acme\Bundle\DemoBundle\Migrations\Data\ORM;

use Acme\Bundle\DemoBundle\Entity\Favorite;
use Doctrine\Common\DataFixtures\AbstractFixture;
use Doctrine\Persistence\ObjectManager;
use Oro\Bundle\MigrationBundle\Fixture\LoadedFixtureVersionAwareInterface;
use Oro\Bundle\MigrationBundle\Fixture\VersionedFixtureInterface;
use Oro\Bundle\OrganizationBundle\Entity\Organization;

/**
 * Load versioned favorites data.
 */
class LoadVersionedFavoriteData extends AbstractFixture implements
    VersionedFixtureInterface,
    LoadedFixtureVersionAwareInterface
{
    #[\Override]
    public function getVersion(): string
    {
        return '2.0';
    }

    #[\Override]
    public function setLoadedVersion($version = null): void
    {
    }

    #[\Override]
    public function load(ObjectManager $manager): void
    {
        $newFavorite = new Favorite();
        $newFavorite->setName('Last favorite');
        $newFavorite->setValue('Last favorite value');
        $newFavorite->setViewCount(18);
        $newFavorite->setOrganization($manager->getRepository(Organization::class)->getFirst());
        $manager->persist($newFavorite);
        $manager->flush();
    }
}
```

## Rename Fixtures

When refactoring, you may need to change the fixture namespace or class name.

To prevent the fixture from loading again, this fixture must implement <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/MigrationBundle/Fixture/RenamedFixtureInterface.php" target="_blank">RenamedFixtureInterface</a> and the getPreviousClassNames method, which returns a list of all previous fully specified class names.

Example:

```php
<?php

namespace Acme\Bundle\DemoBundle\Migrations\Data\ORM;

use Acme\Bundle\DemoBundle\Entity\Favorite;
use Doctrine\Common\DataFixtures\AbstractFixture;
use Doctrine\Persistence\ObjectManager;
use Oro\Bundle\MigrationBundle\Fixture\RenamedFixtureInterface;
use Oro\Bundle\OrganizationBundle\Entity\Organization;

/**
 * Load renamed favorites data.
 */
class LoadRenamedFavoritesData extends AbstractFixture implements RenamedFixtureInterface
{
    #[\Override]
    public function load(ObjectManager $manager)
    {
        $organization = $manager->getRepository(Organization::class)->getFirst();

        $newFavorite = new Favorite();
        $newFavorite->setName('Second favorite');
        $newFavorite->setValue('Second favorite value');
        $newFavorite->setViewCount(5);
        $newFavorite->setOrganization($organization);
        $manager->persist($newFavorite);

        $manager->flush();
    }

    #[\Override]
    public function getPreviousClassNames(): array
    {
        return [
            'Acme\Bundle\DemoBundle\Migrations\Data\ORM\LoadFavoritesData'
        ];
    }
}
```

<!-- Frontend -->
