<a id="book-entities-extended-entities-enums"></a>

# Option Enum Set Fields

The option set, also named as the enum, is a special type of a field which allows to choose one or more options
from a predefined set of options. The OroPlatform provides two different data types for these purposes:

* `enum` (named **Select** on UI) - only one option can be selected
* `multiEnum` (named **Multi-Select** on UI) - several options can be selected

The option sets are quite complex. Both the `enum` and `multiEnum` types are based on [serialized fields](serialized-fields.md#book-entities-extended-entities-serialized-fields). The main difference between them is that the `enum` type is based on a virtual many-to-one association, while the `multiEnum` type is based on a virtual many-to-many association.

To add the option set field to an entity, you can use <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/EntityExtendBundle/Migration/Extension/ExtendExtension.php" target="_blank">ExtendExtension</a>.

The following example illustrates how to do it:

#### NOTE
src/Acme/Bundle/DemoBundle/Migrations/Schema/v1_3/AddEnumFieldOroUser.php
```php
<?php

namespace Acme\Bundle\DemoBundle\Migrations\Schema\v1_3;

use Doctrine\DBAL\Schema\Schema;
use Oro\Bundle\EntityExtendBundle\EntityConfig\ExtendScope;
use Oro\Bundle\EntityExtendBundle\Migration\Extension\ExtendExtension;
use Oro\Bundle\EntityExtendBundle\Migration\Extension\ExtendExtensionAwareInterface;
use Oro\Bundle\MigrationBundle\Migration\Migration;
use Oro\Bundle\MigrationBundle\Migration\QueryBag;

class AddEnumFieldOroUser implements Migration, ExtendExtensionAwareInterface
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
        $table = $schema->getTable('oro_user');
        $this->extendExtension->addEnumField(
            $schema,
            $table,
            'internal_rating', // field name
            'user_internal_rating', // enum code
            false, // only one option can be selected
            false, // an administrator can add new options and remove existing ones
            [
                'extend' => ['owner' => ExtendScope::OWNER_CUSTOM],
                'entity' => ['label' => 'Internal rating']
            ]
        );
    }
}
```

Please mind the enum code parameter. Each option set should have code and be unique system-wide,
and with length of no more than 21 characters (due to dynamic name generation and prefix).
The same principle applies to the field name. In the case above, it should be less than 27 symbols.

To load a list of options, use data fixtures, for example:

```php
<?php

namespace Acme\Bundle\DemoBundle\Migrations\Data\ORM;

use Doctrine\Common\DataFixtures\AbstractFixture;
use Doctrine\Common\DataFixtures\DependentFixtureInterface;
use Doctrine\Persistence\ObjectManager;
use Oro\Bundle\EntityExtendBundle\Entity\EnumOption;
use Oro\Bundle\EntityExtendBundle\Entity\Repository\EnumOptionRepository;
use Oro\Bundle\EntityExtendBundle\Tools\ExtendHelper;
use Oro\Bundle\TranslationBundle\Migrations\Data\ORM\LoadLanguageData;

class LoadUserInternalRatingData extends AbstractFixture implements DependentFixtureInterface
{
    protected array $data = [
        '1' => true,
        '2' => false,
        '3' => false,
        '4' => false,
        '5' => false
    ];

    #[\Override]
    public function load(ObjectManager $manager): void
    {
        /** @var EnumOptionRepository $enumRepo */
        $enumRepo = $manager->getRepository(EnumOption::class);
        $priority = 1;
        foreach ($this->data as $name => $isDefault) {
            $enumOption = $enumRepo->createEnumOption(
                'user_internal_rating',
                ExtendHelper::buildEnumInternalId($name),
                $name,
                $priority++,
                $isDefault,
            );
            $manager->persist($enumOption);
        }

        $manager->flush();
    }

    #[\Override]
    public function getDependencies(): array
    {
        return [LoadLanguageData::class];
    }
}
```

<a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/EntityExtendBundle/Tools/ExtendHelper.php" target="_blank">ExtendHelper</a> class which can be helpful when you work with option sets:

* **buildEnumCode()** - Builds an option set code based on its name.
* **generateEnumCode()** - Generates an option set code based on a field for which this option set is created.
* **isEnumerableType()** - Checks if the passed type is one of the enumerable.
* **isSingleEnumType()** - Checks if the passed type is ‘enum’, (named **Select** on UI)
* **isMultiEnumType()** - Checks if the passed type is ‘multiEnum’, (named **Multi-Select** on UI)
* **buildEnumInternalId()** - Builds an option identifier based on the option name The option internal identifier is a
  32 characters length string.
* **buildEnumOptionId()** - Builds an option identifier based on the option name and enum code. The option identifier is a
  100 characters length string.
* **getEnumTranslationKey()** - Builds label names for option set related translations.
* **buildEnumOptionTranslationKey()** - Builds enum option translation key (symfony translation).
* **extractEnumCode()** - Extracts the enum code from the enum option identifier.

As mentioned above, each option set has its own table to store available options. But translations for all options of all option sets are stored in one table. You can find more details in <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/EntityExtendBundle/Entity/EnumOptionTranslation.php" target="_blank">EnumOptionTranslation</a> and <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/EntityExtendBundle/Entity/EnumOptionInterface.php" target="_blank">EnumOptionInterface</a>.
The EnumOptionTranslation class is used to store translations. The EnumOptionInterface is the base interface for all option set entities.

If for some reason you create system option sets and you have to render them manually, the following components can be helpful:

* <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/EntityExtendBundle/Twig/EnumExtension.php" target="_blank">TWIG extension</a> to sort and translate options. It can be used the following way:
  `optionIds|sort_enum`, `optionId|trans_enum`.
* Symfony form types that can be used to build forms contain option set fields: <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/EntityExtendBundle/Form/Type/EnumChoiceType.php" target="_blank">EnumChoiceType</a> and <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/EntityExtendBundle/Form/Type/EnumSelectType.php" target="_blank">EnumSelectType</a>.
* Grid filters: <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/FilterBundle/Filter/EnumFilter.php" target="_blank">EnumFilter</a> and <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/FilterBundle/Filter/MultiEnumFilter.php" target="_blank">MultiEnumFilter</a>. Check out how to use these filters in datagrids.yml. You can learn
  how to configure datagrid formatters for option sets in <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/EntityExtendBundle/Grid/ExtendColumnOptionsGuesser.php" target="_blank">ExtendColumnOptionsGuesser</a>. Keep in mind that the backend datagrid is configured in the `/config/oro/datagrids.yml` file, while the frontend datagrid is configured in the `/views/layouts/<theme>/config/datagrids.yml` file within the configuration directory of your bundle.

  Please take in account that this class passes the class name as the option set identifier, but you can also use the enum code.

<!-- Frontend -->
