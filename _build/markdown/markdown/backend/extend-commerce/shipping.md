# Create Shipping Method Integrations

This topic describes how to add a custom shipping method to your OroCommerce-based store.

It is recommended to manage shipping methods through integrations. Therefore, to create a new shipping method:

- Implement an integration for the shipping method
- Implement the shipping method itself

Usually, a shipping method has several services to provide a flexible choice of price and delivery time. As an example, we will implement the “Fast Shipping” method — a simple method that requires just the minimum set of options to operate. It will have two services (types): “With present” and “Without present”. Thus, at the end of the topic, you will have the understanding of what steps are necessary to add a workable shipment method and the basic template that you can further extend when the need arises.

## Create a Bundle

First, create and enable the FastShippingBundle bundle for your shipping method as described in the [How to create a new bundle](../extension/create-bundle.md#how-to-create-new-bundle) topic:

1. In the /src/Acme/Bundle/FastShippingBundle/ directory of your application, create class AcmeFastShippingBundle.php:

```php
<?php

namespace Acme\Bundle\FastShippingBundle;

use Symfony\Component\HttpKernel\Bundle\Bundle;

class AcmeFastShippingBundle extends Bundle
{
}
```

#### NOTE
The body of your class can be empty if you use regular case in the name of your organization (i.e. Acme or ACME in our example). getExtension() is necessary when you use uppercase, as Symfony treats uppercase letters in the organization prefix as separate words when creating aliases.
1. To enable the bundle, create Resources/config/oro/bundles.yml in the same directory, with the following content:

```yaml
bundles:
    # Set a priority higher than the priority of the ShippingBundle (i.e. 200) to ensure that all dependencies
    # from the ShippingBundle are loaded before the dependent methods of your bundle.
    - { name: Acme\Bundle\FastShippingBundle\AcmeFastShippingBundle, priority: 200 }
```

#### HINT
To fully enable a bundle, you need to regenerate the application cache. However, to save time, you can do it after creation of the shipping integration.

## Create a Shipping Integration

### Create an Entity to Store the Shipping Method Settings

Define an entity that to store the configuration settings of the shipping method in the database. To do this, create <bundle_root>/Entity/FastShippingSettings.php:

```php
<?php

namespace Acme\Bundle\FastShippingBundle\Entity;

use Doctrine\Common\Collections\ArrayCollection;
use Doctrine\Common\Collections\Collection;
use Doctrine\ORM\Mapping as ORM;
use Oro\Bundle\IntegrationBundle\Entity\Transport;
use Oro\Bundle\LocaleBundle\Entity\LocalizedFallbackValue;
use Symfony\Component\HttpFoundation\ParameterBag;

/**
 * Entity with settings for Fast Shipping integration
 */
#[ORM\Entity]
class FastShippingSettings extends Transport
{
    /**
     * @var Collection|LocalizedFallbackValue[]
     */
    #[ORM\ManyToMany(targetEntity: LocalizedFallbackValue::class, cascade: ['ALL'], orphanRemoval: true)]
    #[ORM\JoinTable(name: 'acme_fast_ship_transport_label')]
    #[ORM\JoinColumn(name: 'transport_id', referencedColumnName: 'id', onDelete: 'CASCADE')]
    #[ORM\InverseJoinColumn(name: 'localized_value_id', referencedColumnName: 'id', onDelete: 'CASCADE', unique: true)]
    private $labels;

    /**
     * @var ParameterBag
     */
    private $settings;

    public function __construct()
    {
        $this->labels = new ArrayCollection();
    }

    /**
     * @return Collection|LocalizedFallbackValue[]
     */
    public function getLabels(): Collection
    {
        return $this->labels;
    }

    public function addLabel(LocalizedFallbackValue $label): FastShippingSettings
    {
        if (!$this->labels->contains($label)) {
            $this->labels->add($label);
        }

        return $this;
    }

    public function removeLabel(LocalizedFallbackValue $label): FastShippingSettings
    {
        if ($this->labels->contains($label)) {
            $this->labels->removeElement($label);
        }

        return $this;
    }

    #[\Override]
    public function getSettingsBag(): ParameterBag
    {
        if (null === $this->settings) {
            $this->settings = new ParameterBag([]);
        }

        return $this->settings;
    }
}
```

As you can see from the code above, the only necessary parameter defined for the FastShipping shipping method is the `label` parameter.

#### IMPORTANT
When naming DB columns, make sure that the name does not exceed 31 symbols. Pay attention to the `acme_fast_ship_transport_label` name in the following extract:

```php
    #[ORM\ManyToMany(targetEntity: LocalizedFallbackValue::class, cascade: ['ALL'], orphanRemoval: true)]
    #[ORM\JoinTable(name: 'acme_fast_ship_transport_label')]
    #[ORM\JoinColumn(name: 'transport_id', referencedColumnName: 'id', onDelete: 'CASCADE')]
    #[ORM\InverseJoinColumn(name: 'localized_value_id', referencedColumnName: 'id', onDelete: 'CASCADE', unique: true)]
```

### Create a User Interface Form for the Shipping Method Integration

When you add an integration via the user interface of the back-office, a form that contains the integration settings appears. In this step, implement the form. To do this, create <bundle_root>/Form/Type/FastShippingTransportSettingsType.php:

```php
<?php

namespace Acme\Bundle\FastShippingBundle\Form\Type;

use Acme\Bundle\FastShippingBundle\Entity\FastShippingSettings;
use Oro\Bundle\LocaleBundle\Form\Type\LocalizedFallbackValueCollectionType;
use Symfony\Component\Form\AbstractType;
use Symfony\Component\Form\FormBuilderInterface;
use Symfony\Component\OptionsResolver\OptionsResolver;
use Symfony\Component\Validator\Constraints\NotBlank;

/**
 * Form type for Fast Shipping integration settings
 */
class FastShippingTransportSettingsType extends AbstractType
{
    private const BLOCK_PREFIX = 'acme_fast_shipping_settings';

    #[\Override]
    public function buildForm(FormBuilderInterface $builder, array $options)
    {
        $builder
            ->add(
                'labels',
                LocalizedFallbackValueCollectionType::class,
                [
                    'label'    => 'acme.fast_shipping.settings.labels.label',
                    'required' => true,
                    'entry_options'  => ['constraints' => [new NotBlank()]],
                ]
            );
    }

    #[\Override]
    public function configureOptions(OptionsResolver $resolver)
    {
        $resolver->setDefaults([
            'data_class' => FastShippingSettings::class
        ]);
    }

    #[\Override]
    public function getBlockPrefix()
    {
        return self::BLOCK_PREFIX;
    }
}
```

### Add Translations for the Form Texts

To present the information on the user interface in a user-friendly way, add translations for the shipping method settings’ names. To do this, create <bundle_root>/Resources/translations/messages.en.yml:

```yaml
acme:
    fast_shipping:
        settings:
            labels:
                label: 'Label'
```

This defines the name of the field that contains the label.

### Create the Integration Channel Type

When you select the type of the integration on the user interface, you will see the integration name and the icon that you define in this step.

To implement a channel type, create <bundle_root>/Integration/FastShippingChannelType.php:

```php
<?php

namespace Acme\Bundle\FastShippingBundle\Integration;

use Oro\Bundle\IntegrationBundle\Provider\ChannelInterface;
use Oro\Bundle\IntegrationBundle\Provider\IconAwareIntegrationInterface;

/**
 * Integration channel type for Fast Shipping integration
 */
class FastShippingChannelType implements ChannelInterface, IconAwareIntegrationInterface
{
    #[\Override]
    public function getLabel(): string
    {
        return 'acme.fast_shipping.channel_type.label';
    }

    #[\Override]
    public function getIcon(): string
    {
        return 'bundles/acmefastshipping/img/fast-shipping-logo.png';
    }
}
```

### Add an Icon for the Integration

To add an icon:

1. Save the file to the <bundle_root>/Resources/public/img directory.
2. Install assets:
   ```none
   bin/console assets:install --symlink
   ```

To make sure that the icon is accessible for the web interface, check if it appears (as a copy or a symlink depending on the settings selected during the application installation) in the /public/bundles/acmefastshipping/img directory of your application.

### Create the Integration Transport

Transport is generally responsible for how the data is obtained from the external system. While the Fast Shipping method does not interact with external systems, you still need to define transport and implement all methods of the TransportInterface for the integration to work properly. To add transport, create <bundle_root>/Integration/FastShippingTransport.php:

```php
<?php

namespace Acme\Bundle\FastShippingBundle\Integration;

use Acme\Bundle\FastShippingBundle\Entity\FastShippingSettings;
use Acme\Bundle\FastShippingBundle\Form\Type\FastShippingTransportSettingsType;
use Oro\Bundle\IntegrationBundle\Entity\Transport;
use Oro\Bundle\IntegrationBundle\Provider\TransportInterface;
use Symfony\Component\HttpFoundation\ParameterBag;

/**
 * Transport for Fast Shipping integration
 */
class FastShippingTransport implements TransportInterface
{
    /** @var ParameterBag */
    protected $settings;

    #[\Override]
    public function init(Transport $transportEntity)
    {
        $this->settings = $transportEntity->getSettingsBag();
    }

    #[\Override]
    public function getSettingsFormType(): string
    {
        return FastShippingTransportSettingsType::class;
    }

    #[\Override]
    public function getSettingsEntityFQCN(): string
    {
        return FastShippingSettings::class;
    }

    #[\Override]
    public function getLabel(): string
    {
        return 'acme.fast_shipping.transport.label';
    }
}
```

### Create a Configuration File for the Service Container

To start using a service container for your bundle, first create the bundle configuration file <bundle_root>/Resources/config/services.yml.

### Add the Channel Type and Transport to the Services Container

To register the channel type and transport, append the following key-values to <bundle_root>/Resources/config/services.yml:

```yaml
parameters:
    acme_fast_shipping.integration.type: 'fast_shipping'

services:
    acme_fast_shipping.integration.channel:
        class: 'Acme\Bundle\FastShippingBundle\Integration\FastShippingChannelType'
        tags:
            - { name: oro_integration.channel, type: '%acme_fast_shipping.integration.type%' }

    acme_fast_shipping.integration.transport:
        class: 'Acme\Bundle\FastShippingBundle\Integration\FastShippingTransport'
        tags:
            - { name: oro_integration.transport, type: '%acme_fast_shipping.integration.type%', channel_type: '%acme_fast_shipping.integration.type%' }
```

### Set up Services with DependencyInjection

To set up services, load your configuration file (services.yml) using the DependencyInjection component. For this, create <bundle_root>/DependencyInjection/AcmeFastShippingExtension.php with the following content:

```php
<?php

namespace Acme\Bundle\FastShippingBundle\DependencyInjection;

use Symfony\Component\Config\FileLocator;
use Symfony\Component\DependencyInjection\ContainerBuilder;
use Symfony\Component\DependencyInjection\Loader;
use Symfony\Component\HttpKernel\DependencyInjection\Extension;

class AcmeFastShippingExtension extends Extension
{
    #[\Override]
    public function load(array $configs, ContainerBuilder $container): void
    {
        $loader = new Loader\YamlFileLoader($container, new FileLocator(__DIR__ . '/../Resources/config'));
        $loader->load('services.yml');
    }
}
```

### Add Translations for the Channel Type and Transport

The channel type and, in general, transport labels also appear on the user interface (you will not see the transport label for Fast Shipping). Provide translations for them by appending the <bundle_root>/Resources/translations/messages.en.yml. Now, the messages.en.yml content must look as follows:

```yaml
acme:
    fast_shipping:
        settings:
            labels:
                label: 'Label'
        transport:
            label: 'Fast Shipping'
        channel_type:
            label: 'Fast Shipping'
```

### Add an Installer

An installer ensures that upon the application installation, the database will contain the entity that you defined within your bundle.

Follow the instructions provided in the [How to generate an installer](../entities/migration.md#installer-generate) topic to apply the changes without migration and generate an installer file based on the current schema of the DB.

#### NOTE
If you have not performed the steps mentioned in [How to generate an installer](../entities/migration.md#installer-generate), because you already have the installer file, then make sure to run the `php bin/console oro:migration:load --force` command to apply the changes from the file.

After you complete the process, you will have the <bundle_root>/Migrations/Schema/FastShippingBundleInstaller.php class with the following content:

```php
<?php

namespace Acme\Bundle\FastShippingBundle\Migrations\Schema;

use Doctrine\DBAL\Schema\Schema;
use Oro\Bundle\MigrationBundle\Migration\Installation;
use Oro\Bundle\MigrationBundle\Migration\QueryBag;

/**
 * @SuppressWarnings(PHPMD.TooManyMethods)
 * @SuppressWarnings(PHPMD.ExcessiveClassLength)
 */
class FastShippingBundleInstaller implements Installation
{
    #[\Override]
    public function getMigrationVersion()
    {
        return 'v1_0';
    }

    #[\Override]
    public function up(Schema $schema, QueryBag $queries)
    {
        /** Tables generation **/
        $this->createAcmeFastShipTransportLabelTable($schema);

        /** Foreign keys generation **/
        $this->addAcmeFastShipTransportLabelForeignKeys($schema);
    }

    /**
     * Create acme_fast_ship_transport_label table
     */
    protected function createAcmeFastShipTransportLabelTable(Schema $schema)
    {
        $table = $schema->createTable('acme_fast_ship_transport_label');
        $table->addColumn('transport_id', 'integer', []);
        $table->addColumn('localized_value_id', 'integer', []);
        $table->setPrimaryKey(['transport_id', 'localized_value_id']);
        $table->addUniqueIndex(['localized_value_id'], 'UNIQ_15E6E6F3EB576E89');
        $table->addIndex(['transport_id'], 'IDX_15E6E6F39909C13F', []);
    }

    /**
     * Add acme_fast_ship_transport_label foreign keys.
     */
    protected function addAcmeFastShipTransportLabelForeignKeys(Schema $schema)
    {
        $table = $schema->getTable('acme_fast_ship_transport_label');
        $table->addForeignKeyConstraint(
            $schema->getTable('oro_integration_transport'),
            ['transport_id'],
            ['id'],
            ['onDelete' => 'CASCADE', 'onUpdate' => null]
        );
        $table->addForeignKeyConstraint(
            $schema->getTable('oro_fallback_localization_val'),
            ['localized_value_id'],
            ['id'],
            ['onDelete' => 'CASCADE', 'onUpdate' => null]
        );
    }
}
```

### Check That the Integration is Created Successfully

1. Clear the application cache:
   ```none
   bin/console cache:clear
   ```

   #### NOTE
   If you are working in a production environment, you have to use the `--env=prod` parameter with the command.
2. Open the user interface and check that the changes have applied and you can add an integration of the Fast Shipping type. Note that at this point you are not yet able to add this shipping method to a shipping rule.
   ![View the Fast Shipping integration details.](img/backend/extend_commerce/shipping_method_create2.png)

## Implement a Shipping Method

Now implement the shipping method itself using the following steps:

> * [Implement the Main Method](#implement-the-main-method)
> * [Add the Shipping Method Identifier Generator to the Services Container](#add-the-shipping-method-identifier-generator-to-the-services-container)
> * [Create a Factory for the Shipping Method](#create-a-factory-for-the-shipping-method)
> * [Add the Shipping Method Factory to the Services Container](#add-the-shipping-method-factory-to-the-services-container)
> * [Register a Shipping Method Provider](#register-a-shipping-method-provider)
> * [Create a Shipping Method Type](#create-a-shipping-method-type)
> * [Define Translation for the Shipping Method Type](#define-translation-for-the-shipping-method-type)
> * [Create a Shipping Method Options Form](#create-a-shipping-method-options-form)
> * [Add the Shipping Method Options Form to the Services Container](#add-the-shipping-method-options-form-to-the-services-container)
> * [Define Translation for the Shipping Method Form Options](#define-translation-for-the-shipping-method-form-options)
> * [Add a Template](#add-a-template)

### Implement the Main Method

To implement the main method, create the <bundle_root>/Method/FastShippingMethod.php class that implements two standard interfaces `\Oro\Bundle\ShippingBundle\Method\ShippingMethodInterface` and `\Oro\Bundle\ShippingBundle\Method\ShippingMethodIconAwareInterface`:

```php
<?php

namespace Acme\Bundle\FastShippingBundle\Method;

use Oro\Bundle\ShippingBundle\Method\ShippingMethodIconAwareInterface;
use Oro\Bundle\ShippingBundle\Method\ShippingMethodInterface;
use Oro\Bundle\ShippingBundle\Method\ShippingMethodTypeInterface;
use Symfony\Component\Form\Extension\Core\Type\HiddenType;

/**
 * Represents Fast Shipping method.
 */
class FastShippingMethod implements ShippingMethodInterface, ShippingMethodIconAwareInterface
{
    private string $identifier;
    private string $name;
    private string $label;
    private ?string $icon;
    private bool $enabled;
    private array $types;

    public function __construct(
        string $identifier,
        string $name,
        string $label,
        ?string $icon,
        bool $enabled,
        array $types
    ) {
        $this->identifier = $identifier;
        $this->name = $name;
        $this->label = $label;
        $this->icon = $icon;
        $this->enabled = $enabled;
        $this->types = $types;
    }

    #[\Override]
    public function getIdentifier(): string
    {
        return $this->identifier;
    }

    #[\Override]
    public function isGrouped(): bool
    {
        return true;
    }

    #[\Override]
    public function isEnabled(): bool
    {
        return $this->enabled;
    }

    #[\Override]
    public function getName(): string
    {
        return $this->name;
    }

    #[\Override]
    public function getLabel(): string
    {
        return $this->label;
    }

    #[\Override]
    public function getIcon(): ?string
    {
        return $this->icon;
    }

    #[\Override]
    public function getTypes(): array
    {
        return $this->types;
    }

    #[\Override]
    public function getType(string $identifier): ?ShippingMethodTypeInterface
    {
        return $this->types[$identifier] ?? null;
    }

    #[\Override]
    public function getOptionsConfigurationFormType(): ?string
    {
        return HiddenType::class;
    }

    #[\Override]
    public function getSortOrder(): int
    {
        return 150;
    }
}
```

The methods are the following:

* `getIdentifier` — Provides a unique identifier of the shipping method in the scope of the Oro application.
* `getName` — Returns the shipping method’s name that appears on the shipping rule edit page.
* `getLabel` — Returns the shipping method’s label that appears on the shipping method step on checkout. It can also be a Symfony translated message.
* `getIcon` — Returns the icon that appears on the shipping rule edit page.
* `isEnabled` — Defines, whether the integration of the shipping method is enabled by default.
* `isGrouped` — Defines how shipping method’s types appear in the shipping method configuration on the user interface. If set to `true`, the types appear in the table where each line contains the **Active** checkbox that enables users to enable individual shipping method types for a particular shipping method configuration.
* `getSortOrder` —  Defines the order in which shipping methods appear on the user interface. For example, in the following screenshot, the Flat rate sort order is lower than the UPS sort order:
  ![image](img/backend/extend_commerce/shipping_methods_frontend.png)
* `getType` — Returns the selected shipping method type based on the type identifier.
* `getTypes` — Returns a set of the shipping method types.
* `getOptionsConfigurationFormType` — Returns the user interface form with the configuration options. The form appears on the shipping rule edit page. If the method returns `HiddenType::class`, the form does not appear.

### Add the Shipping Method Identifier Generator to the Services Container

Append the following lines to <bundle_root>/Resources/config/services.yml:

```yaml
    acme_fast_shipping.method.identifier_generator.method:
        parent: oro_integration.generator.prefixed_identifier_generator
        public: true
        arguments:
            - '%acme_fast_shipping.integration.type%'
```

### Create a Factory for the Shipping Method

This factory generates an individual configuration set for each instance of the integration of the Fast Shipping type. In our case, it also contains the method createTypes() that generates the services (types) of the fast shipping type and assigns them labels.

Create the <bundle_root>/Factory/FastShippingMethodFromChannelFactory.php class with the following content:

```php
<?php

namespace Acme\Bundle\FastShippingBundle\Factory;

use Acme\Bundle\FastShippingBundle\Entity\FastShippingSettings;
use Acme\Bundle\FastShippingBundle\Method\FastShippingMethod;
use Acme\Bundle\FastShippingBundle\Method\FastShippingMethodType;
use Oro\Bundle\IntegrationBundle\Entity\Channel;
use Oro\Bundle\IntegrationBundle\Generator\IntegrationIdentifierGeneratorInterface;
use Oro\Bundle\IntegrationBundle\Provider\IntegrationIconProviderInterface;
use Oro\Bundle\LocaleBundle\Helper\LocalizationHelper;
use Oro\Bundle\ShippingBundle\Method\Factory\IntegrationShippingMethodFactoryInterface;
use Oro\Bundle\ShippingBundle\Method\ShippingMethodInterface;
use Symfony\Contracts\Translation\TranslatorInterface;

/**
 * The factory to create Fast Shipping method.
 */
class FastShippingMethodFromChannelFactory implements IntegrationShippingMethodFactoryInterface
{
    private IntegrationIdentifierGeneratorInterface $identifierGenerator;
    private LocalizationHelper $localizationHelper;
    private TranslatorInterface $translator;
    private IntegrationIconProviderInterface $integrationIconProvider;

    public function __construct(
        IntegrationIdentifierGeneratorInterface $identifierGenerator,
        LocalizationHelper $localizationHelper,
        TranslatorInterface $translator,
        IntegrationIconProviderInterface $integrationIconProvider
    ) {
        $this->identifierGenerator = $identifierGenerator;
        $this->localizationHelper = $localizationHelper;
        $this->translator = $translator;
        $this->integrationIconProvider = $integrationIconProvider;
    }

    #[\Override]
    public function create(Channel $channel): ShippingMethodInterface
    {
        /** @var FastShippingSettings $transport */
        $transport = $channel->getTransport();

        return new FastShippingMethod(
            $this->identifierGenerator->generateIdentifier($channel),
            $channel->getName(),
            (string)$this->localizationHelper->getLocalizedValue($transport->getLabels()),
            $this->integrationIconProvider->getIcon($channel),
            $channel->isEnabled(),
            $this->createTypes()
        );
    }

    private function createTypes(): array
    {
        $withoutPresent = new FastShippingMethodType(
            $this->translator->trans('acme.fast_shipping.method.processing_type.without_present.label'),
            false
        );
        $withPresent = new FastShippingMethodType(
            $this->translator->trans('acme.fast_shipping.method.processing_type.with_present.label'),
            true
        );

        return [
            $withoutPresent->getIdentifier() => $withoutPresent,
            $withPresent->getIdentifier() => $withPresent,
        ];
    }
}
```

### Add the Shipping Method Factory to the Services Container

To register the shipping method factory, append the following key-values to <bundle_root>/Resources/config/services.yml under the services section:

```yaml
    acme_fast_shipping.factory.method:
        class: 'Acme\Bundle\FastShippingBundle\Factory\FastShippingMethodFromChannelFactory'
        arguments:
            - '@acme_fast_shipping.method.identifier_generator.method'
            - '@oro_locale.helper.localization'
            - '@translator'
            - '@oro_integration.provider.integration_icon'

```

### Register a Shipping Method Provider

Append the following lines to <bundle_root>/Resources/config/services.yml under the services section:

```yaml
    acme_fast_shipping.method.provider:
        class: 'Oro\Bundle\ShippingBundle\Method\Provider\Integration\ChannelShippingMethodProvider'
        arguments:
            - '%acme_fast_shipping.integration.type%'
            - '@acme_fast_shipping.factory.method'
            - '@oro_shipping.method.loader'
        tags:
            - { name: oro_shipping_method_provider }
```

### Create a Shipping Method Type

Shipping method types define different specifics of the same shipping services. For example, for Flat Rate, the type defines whether to calculate shipping price per order or per item. The Fast Shipping will have two types: “With Present” and “Without Present”.

To create a shipping method type, add the <bundle_root>/Method/FastShippingMethodType.php class with the following content:

```php
<?php

namespace Acme\Bundle\FastShippingBundle\Method;

use Acme\Bundle\FastShippingBundle\Form\Type\FastShippingMethodOptionsType;
use Oro\Bundle\CurrencyBundle\Entity\Price;
use Oro\Bundle\ShippingBundle\Context\ShippingContextInterface;
use Oro\Bundle\ShippingBundle\Method\ShippingMethodTypeInterface;

/**
 * Represents Fast Shipping method type.
 */
class FastShippingMethodType implements ShippingMethodTypeInterface
{
    public const PRICE_OPTION = 'price';

    private const WITHOUT_PRESENT_TYPE = 'without_present';
    private const WITH_PRESENT_TYPE = 'with_present';

    private string $label;
    private bool $isWithPresent;

    public function __construct(string $label, bool $isWithPresent)
    {
        $this->label = $label;
        $this->isWithPresent = $isWithPresent;
    }

    #[\Override]
    public function getIdentifier(): string
    {
        return $this->isWithPresent ? self::WITH_PRESENT_TYPE : self::WITHOUT_PRESENT_TYPE;
    }

    #[\Override]
    public function getLabel(): string
    {
        return $this->label;
    }

    #[\Override]
    public function getSortOrder(): int
    {
        return 0;
    }

    #[\Override]
    public function getOptionsConfigurationFormType(): ?string
    {
        return FastShippingMethodOptionsType::class;
    }

    #[\Override]
    public function calculatePrice(
        ShippingContextInterface $context,
        array $methodOptions,
        array $typeOptions
    ): ?Price {
        $price = $typeOptions[self::PRICE_OPTION];

        // Provide additional price calculation logic here if required.

        return Price::create((float)$price, $context->getCurrency());
    }
}
```

* `getIdentifier` — Returns a unique identifier of a shipping method type in the scope of the shipping method.
* `getLabel` — Returns the label of the shipping method type. The label appears on the shipping rule edit page in the back-office and in the storefront.
* `getSortOrder` —  Defines the order in which shipping method types appear on the user interface. For example, see the UPS shipping types below. The number that defines the sort order of the UPS Ground is lower than that of the UPS 2nd Day Air (i.e. the lower the number, the higher up the list the method type appears):
  ![image](img/backend/extend_commerce/shipping_methods_frontend.png)
* `getOptionsConfigurationFormType` — Returns the user interface form with the configuration options. The form appears on the shipping rule edit page. If the method returns `HiddenType::class`, the form does not appear.
* `calculatePrice`– Contains the main logic and returns the shipping price for the given `$context`.

#### NOTE
If you implement a more complicated shipping method, see Oro\\Bundle\\ShippingBundle\\Context\\ShippingContextInterface for attributes that can affect a shipping price (e.g., shipping address information or line items).

### Define Translation for the Shipping Method Type

Provide translations by appending the <bundle_root>/Resources/translations/messages.en.yml. Now, the messages.en.yml content must look as follows:

```yaml
acme:
    fast_shipping:
        settings:
            labels:
                label: 'Label'
        transport:
            label: 'Fast Shipping'
        channel_type:
            label: 'Fast Shipping'
        method:
            price.label: 'Price'
            processing_type:
                without_present.label: 'Fast Shipping Rate Without Present'
                with_present.label: 'Fast Shipping Rate With Present'
```

### Create a Shipping Method Options Form

This form with options for a shipping method appears on the user interface of the back-office when you add the shipping method to a shipping rule. Add FastShippingMethodOptionsType.php to the <bundle_root>/Form/Type/ directory:

```php
<?php

namespace Acme\Bundle\FastShippingBundle\Form\Type;

use Acme\Bundle\FastShippingBundle\Method\FastShippingMethodType;
use Oro\Bundle\CurrencyBundle\Rounding\RoundingServiceInterface;
use Symfony\Component\Form\AbstractType;
use Symfony\Component\Form\Extension\Core\Type\NumberType;
use Symfony\Component\Form\FormBuilderInterface;
use Symfony\Component\OptionsResolver\Exception\AccessException;
use Symfony\Component\OptionsResolver\OptionsResolver;
use Symfony\Component\Validator\Constraints\NotBlank;
use Symfony\Component\Validator\Constraints\Type;

/**
 * Form type for fast shipping method options which are displayed on shipping rules page
 */
class FastShippingMethodOptionsType extends AbstractType
{
    private const BLOCK_PREFIX = 'acme_fast_shipping_options_type';

    /**
     * @var RoundingServiceInterface
     */
    protected $roundingService;

    public function __construct(RoundingServiceInterface $roundingService)
    {
        $this->roundingService = $roundingService;
    }

    #[\Override]
    public function buildForm(FormBuilderInterface $builder, array $options)
    {
        $priceOptions = [
            'scale' => $this->roundingService->getPrecision(),
            'rounding_mode' => $this->roundingService->getRoundType(),
            'attr' => ['data-scale' => $this->roundingService->getPrecision()],
        ];

        $builder
            ->add(FastShippingMethodType::PRICE_OPTION, NumberType::class, array_merge([
                'label' => 'acme.fast_shipping.method.price.label',
                'constraints' => [new NotBlank(), new Type(['type' => 'numeric'])],
            ], $priceOptions));
    }

    /**
     * @throws AccessException
     */
    #[\Override]
    public function configureOptions(OptionsResolver $resolver)
    {
        $resolver->setDefaults([
            'label' => 'acme.fast_shipping.form.acme_fast_shipping_options_type.label',
        ]);
    }

    #[\Override]
    public function getBlockPrefix()
    {
        return self::BLOCK_PREFIX;
    }
}
```

### Add the Shipping Method Options Form to the Services Container

Append the following lines to <bundle_root>/Resources/config/services.yml under the services section:

```yaml
    acme_fast_shipping.form.type.fast_shipping_options:
        class: 'Acme\Bundle\FastShippingBundle\Form\Type\FastShippingMethodOptionsType'
        arguments:
            - '@oro_currency.rounding.price_rounding_service'
        tags:
            - { name: form.type }
```

### Define Translation for the Shipping Method Form Options

Provide translations by appending the <bundle_root>/Resources/translations/messages.en.yml. Now, the messages.en.yml content must look as follows:

```yaml
acme:
    fast_shipping:
        settings:
            labels:
                label: 'Label'
        transport:
            label: 'Fast Shipping'
        channel_type:
            label: 'Fast Shipping'
        method:
            price.label: 'Price'
            processing_type:
                without_present.label: 'Fast Shipping Rate Without Present'
                with_present.label: 'Fast Shipping Rate With Present'
        form:
            acme_fast_shipping_options_type.label: 'Fast Shipping Rate'
```

### Add a Template

In the shipping rules, this template is used to display the configured settings of the Fast Shipping integration.

Create the /Resources/views/method/fastShippingMethodWithOptions.html.twig file with the following content:

```html
{% import '@OroShipping/ShippingMethodsConfigsRule/macros.html.twig' as ShipRuleMacro %}

   {% apply spaceless %}
       {% set methodLabel = get_shipping_method_label(methodData.identifier)|trans %}
       {% if methodLabel|length > 0 %}
           <li>{{ methodLabel }}
           <ul>
       {% endif %}
       {% for type in methodData.types %}
           {%- if type.enabled -%}
               <li>{{ get_shipping_method_type_label(methodData.identifier, type.identifier)|trans }} ({{ 'acme.fast_shipping.method.price.label'|trans }}: {{ type.options['price']|oro_format_currency({'currency': currency}) }}
                   ) {{ ShipRuleMacro.renderShippingMethodDisabledFlag(methodData.identifier) }}</li>
           {%- endif -%}
       {% endfor %}
       {% if methodLabel|length > 0 %}
           </ul>
           </li>
       {% endif %}
   {% endapply %}
```

## Add a Check for When Users Disable Used Shipping Method Types

To show a notification when a user disables or removes the integration currently used in shipping rules, use the event listeners to catch the corresponding event and the event handlers.

### Add Event Listeners to the System Container

Append the following lines to <bundle_root>/Resources/config/services.yml under the parameters and services sections:

```yaml
parameters:
    acme_fast_shipping.admin_view.method_template: '@@AcmeFastShipping/method/fastShippingMethodWithOptions.html.twig'

services:
    acme_fast_shipping.event_listener.shipping_method_config_data:
        parent: oro_shipping.admin_view.method_template.listener
        arguments:
            - '%acme_fast_shipping.admin_view.method_template%'
            - '@acme_fast_shipping.method.provider'
        tags:
            - { name: kernel.event_listener, event: oro_shipping_method.config_data, method: onGetConfigData }

    acme_fast_shipping.remove_integration_listener:
        parent: oro_shipping.remove_integration_listener
        arguments:
            - '%acme_fast_shipping.integration.type%'
            - '@acme_fast_shipping.method.identifier_generator.method'
            - '@oro_shipping.method.event.dispatcher.method_removal'
        tags:
            - { name: kernel.event_listener, event: oro_integration.channel_delete, method: onRemove }

    acme_fast_shipping.disable_integration_listener:
        parent: oro_shipping.disable_integration_listener
        arguments:
            - '%acme_fast_shipping.integration.type%'
            - '@acme_fast_shipping.method.identifier_generator.method'
            - '@oro_shipping.method_disable_handler.decorator'
        tags:
            - { name: kernel.event_listener, event: oro_integration.channel_disable, method: onIntegrationDisable }
```

### Add Actions

Create actions.yml in the <bundle_root>/Resources/config/oro/ directory:

```yaml
operations:
    # Disable the default deactivate method for the Fast Shipping integration.
    oro_integration_deactivate:
        preconditions:
            '@and':
                - '@not_equal': [$type, '%acme_fast_shipping.integration.type%']

    # Disable the default delete method for the Fast Shipping integration.
    oro_integration_delete:
        preconditions:
            '@and':
                - '@not_equal': [$type, '%acme_fast_shipping.integration.type%']

    # Use the deactivate method that:
    #    a. first checks whether there are shipping rules that use the Fast Shipping method, and
    #    b. if yes, displays to a user the confirmation dialog with the notification message and the link to the list of the corresponding rules.
    acme_fast_shipping_integration_deactivate:
        extends: oro_integration_deactivate
        for_all_entities: false
        for_all_datagrids: false
        replace:
            - preactions
            - preconditions
            - frontend_options

        # Filter the grid with the active shipping rules that use the Fast Shipping method and generate the link to it.
        preactions:
            - '@call_service_method':
                  attribute: $.actionAllowed
                  service: oro_integration.utils.edit_mode
                  method: isSwitchEnableAllowed
                  method_parameters: [$.data.editMode]
            - '@call_service_method':
                  attribute: $.methodIdentifier
                  service: acme_fast_shipping.method.identifier_generator.method
                  method: generateIdentifier
                  method_parameters: [$.data]
            - '@call_service_method':
                  attribute: $.linkGrid
                  service: oro_shipping.helper.filtered_datagrid_route
                  method: generate
                  method_parameters:  [{'methodConfigs': $.methodIdentifier}]

        # Check that the method is used in the shipping rules.
        preconditions:
            '@and':
                - '@shipping_method_has_enabled_shipping_rules':
                      parameters:
                          method_identifier: $.methodIdentifier
                - '@equal': [$type, '%acme_fast_shipping.integration.type%']
                - '@equal': [$.actionAllowed, true]
                - '@equal': [$.data.enabled, true]

        # Show the confirmation dialog with the notification message.
        frontend_options:
            confirmation:
                title: oro.shipping.integration.deactivate.title
                okText: oro.shipping.integration.deactivate.button.okText
                message: oro.shipping.integration.deactivate.message
                message_parameters:
                    linkGrid: $.linkGrid
                component: oroui/js/standart-confirmation


    # If there are no shipping rules that use this method, deactivate without displaying to a user the confirmation dialog.
    acme_fast_shipping_integration_deactivate_without_rules:
        extends: acme_fast_shipping_integration_deactivate
        for_all_entities: false
        for_all_datagrids: false
        replace:
            - preconditions
            - frontend_options
        preconditions:
            '@and':
                - '@not':
                      - '@shipping_method_has_enabled_shipping_rules':
                            parameters:
                                method_identifier: $.methodIdentifier
                - '@equal': [$type, '%acme_fast_shipping.integration.type%']
                - '@equal': [$.actionAllowed, true]
                - '@equal': [$.data.enabled, true]
        frontend_options: ~

    # Use the delete method that:
    #    a. first checks whether there are shipping rules that use the Fast Shipping method, and
    #    b. if yes, displays to a user the confirmation dialog with the notification message and the link to the list of the corresponding rules.
    acme_fast_shipping_integration_delete:
        extends: oro_integration_delete
        for_all_entities: false
        for_all_datagrids: false
        replace:
            - preactions
            - preconditions
            - frontend_options
        preactions:
            - '@call_service_method':
                  service: oro_integration.utils.edit_mode
                  method: isEditAllowed
                  method_parameters: [$.data.editMode]
                  attribute: $.actionAllowed
            - '@call_service_method':
                  attribute: $.methodIdentifier
                  service: acme_fast_shipping.method.identifier_generator.method
                  method: generateIdentifier
                  method_parameters: [$.data]
            - '@call_service_method':
                  attribute: $.linkGrid
                  service: oro_shipping.helper.filtered_datagrid_route
                  method: generate
                  method_parameters:  [{'methodConfigs': $.methodIdentifier}]
        preconditions:
            '@and':
                - '@shipping_method_has_shipping_rules':
                      parameters:
                          method_identifier: $.methodIdentifier
                - '@equal': [$type, '%acme_fast_shipping.integration.type%']
                - '@equal': [$.actionAllowed, true]
        frontend_options:
            confirmation:
                title: oro.shipping.integration.delete.title
                okText: oro.shipping.integration.delete.button.okText
                message: oro.shipping.integration.delete.message
                message_parameters:
                    linkGrid: $.linkGrid
                component: oroui/js/standart-confirmation

    acme_fast_shipping_integration_delete_without_rules:
        extends: acme_fast_shipping_integration_delete
        for_all_entities: false
        for_all_datagrids: false
        replace:
            - preconditions
            - frontend_options
        preconditions:
            '@and':
                - '@not':
                      - '@shipping_method_has_shipping_rules':
                            parameters:
                                method_identifier: $.methodIdentifier
                - '@equal': [$type, '%acme_fast_shipping.integration.type%']
                - '@equal': [$.actionAllowed, true]
        frontend_options:
            title: oro.action.delete_entity
            confirmation:
                title: oro.action.delete_entity
                message: oro.action.delete_confirm
                message_parameters:
                    entityLabel: $name
                component: oroui/js/delete-confirmation
```

To enable this shipping method, you need to set up a corresponding shipping rule. Follow the Shipping Rules Configuration topic for more details.
