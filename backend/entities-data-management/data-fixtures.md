<a id="entities-data-management-fixtures"></a>

# Fixtures and Demo Data

Before your application contains an interface to create new tasks, you need to load them
programmatically. In OroPlatform, this can be done by creating [fixture classes](../entities/fixtures.md#backend-entities-fixtures) that are placed in the
`Migrations/Data/ORM` subdirectory of your bundle and that implement the `FixtureInterface`:

```php
namespace Acme\Bundle\DemoBundle\Migrations\Data\ORM;

use Acme\Bundle\DemoBundle\Entity\Priority;
use Acme\Bundle\DemoBundle\Entity\Question;
use Doctrine\Common\DataFixtures\FixtureInterface;
use Doctrine\Persistence\ObjectManager;
use Oro\Bundle\OrganizationBundle\Entity\Organization;

/**
 * Load Question data fixture.
 */
class LoadQuestions implements FixtureInterface
{
    #[\Override]
    public function load(ObjectManager $manager)
    {
        $organization = $manager->getRepository(Organization::class)->getFirst();

        $majorPriority = new Priority();
        $majorPriority->setLabel('major');
        $majorPriority->setOrganization($organization);
        $manager->persist($majorPriority);

        $importantTask = new Question();
        $importantTask->setSubject('Important task');
        $importantTask->setDescription('This is an important task');
        $importantTask->setDueDate(new \DateTime('+1 week'));
        $importantTask->setPriority($majorPriority);
        $importantTask->setOrganization($organization);
        $manager->persist($importantTask);

        $minorPriority = new Priority();
        $minorPriority->setLabel('minor');
        $minorPriority->setOrganization($organization);
        $manager->persist($minorPriority);

        $unimportantTask = new Question();
        $unimportantTask->setSubject('Unimportant task');
        $unimportantTask->setDescription('This is a not so important task');
        $unimportantTask->setDueDate(new \DateTime('+2 weeks'));
        $unimportantTask->setPriority($minorPriority);
        $unimportantTask->setOrganization($organization);
        $manager->persist($unimportantTask);

        $manager->flush();
    }
}
```

## Console Commands

Use the `oro:migration:data:load` command to load all fixtures that have not been loaded yet:

```none
php bin/console oro:migration:data:load
```

The fixtures type (“main”, or “demo”) can be specified with the `--fixtures-type=<type>` option. For example, you can create data fixtures that should only be loaded when you want to present your application with some demo data. To do so, place your data fixture classes in the `Migrations/Data/Demo/ORM` subdirectory of your bundle and use the `--fixtures-type` option of the `oro:migration:data:load` command to indicate that the demo data should be loaded:

```none
php bin/console oro:migration:data:load --fixtures-type=demo
```

The `--dry-run` option can be used to print the list of fixtures without applying them:

```none
php bin/console oro:migration:data:load --dry-run
```

The `--bundles` option can be used to load the fixtures only from the specified bundles:

```none
php bin/console oro:migration:data:load --bundles=<BundleOne> --bundles=<BundleTwo> --bundles=<BundleThree>
```

The `--exclude` option will skip loading fixtures from the specified bundles:

```none
php bin/console oro:migration:data:load --exclude=<BundleOne> --exclude=<BundleTwo> --exclude=<BundleThree>
```

## Non-Default Object Managers and Reference Repositories in Fixtures

To create and reference entities managed by non-default object manager, extend your fixture class from  `Oro\Bundle\TestFrameworkBundle\Test\DataFixtures\AbstractFixture`.

#### NOTE
The default object manager is available as the load method argument.

```php
namespace Oro\Bundle\ConfigBundle\Tests\Functional\DataFixtures;

use Doctrine\Persistence\ObjectManager;
use Oro\Bundle\ConfigBundle\Entity\Config;
use Oro\Bundle\ConfigBundle\Entity\ConfigValue;
use Oro\Bundle\TestFrameworkBundle\Test\DataFixtures\AbstractFixture;
use Symfony\Component\Yaml\Yaml;

class LoadConfigValue extends AbstractFixture
{
    const FILENAME = 'config_value.yml';

    public function load(ObjectManager $manager)
    {
        $config = $this->getConfig(
            $this->getObjectManagerForClass(Config::class)
        );

        $configValueObjectManager = $this->getObjectManagerForClass(ConfigValue::class);

        foreach ($this->getConfigValuesData() as $name => $data) {
            $configValue = new ConfigValue();
            $configValue->setConfig($config)
                ->setName($name)
                ->setSection($data['section'])
                ->setValue($data['value']);

            $configValueObjectManager->persist($configValue);
            $this->setReference($name, $configValue);
        }

        $configValueObjectManager->flush();
    }

    /**
     * @return array
     */
    protected function getConfigValuesData()
    {
        return Yaml::parse(file_get_contents(__DIR__.'/data/'.static::FILENAME));
    }

    /**
     * @param ObjectManager $manager
     * @return null|Config
     */
    protected function getConfig(ObjectManager $manager)
    {
        return $manager->getRepository(Config::class)->findOneBy([]);
    }
}
```

<!-- Frontend -->
