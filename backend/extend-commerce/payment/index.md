<a id="dev-extend-commerce-payment-create-payment-method"></a>

# Create Payment Method Integrations

This topic describes how to add a custom payment method to your OroCommerce-based store.

It is recommended to manage payment methods through integrations. Therefore, to create a new payment method:

- Implement an integration for a payment method
- Implement a payment method itself

As an example, let us implement a collect on delivery (cash on delivery, COD) payment option. This is a simple method that does not utilize external services (like credit card payment interfaces) and requires just the minimum set of options to operate. Thus, at the end of the topic, you will have the understanding of what steps are necessary to add a workable payment method and the basic template that you can further extend when the need arises.

## Create a Bundle

First, create and enable the CollectOnDeliveryBundle bundle for your payment method as described in the [How to create a new bundle](../../extension/create-bundle.md#how-to-create-new-bundle) topic:

1. In the /src/Acme/Bundle/CollectOnDeliveryBundle/ directory of your application, create class AcmeCollectOnDeliveryBundle.php:

```php
<?php

namespace Acme\Bundle\CollectOnDeliveryBundle;

use Symfony\Component\HttpKernel\Bundle\Bundle;

class AcmeCollectOnDeliveryBundle extends Bundle
{
}
```

1. To enable the bundle, create Resources/config/oro/bundles.yml in the same directory, with the following content:

```yaml
bundles:
    - Acme\Bundle\CollectOnDeliveryBundle\AcmeCollectOnDeliveryBundle
```

#### HINT
To fully enable a bundle, you need to regenerate the application cache. However, to save time, you can do it after creation of the payment integration.

## Create a Payment Integration

### Create an Entity to Store the Payment Method Settings

Define an entity to store the configuration settings of the payment method in the database. To do this, create <bundle_root>/Entity/CollectOnDeliverySettings.php:

```php
<?php

namespace Acme\Bundle\CollectOnDeliveryBundle\Entity;

use Acme\Bundle\CollectOnDeliveryBundle\Entity\Repository\CollectOnDeliverySettingsRepository;
use Doctrine\Common\Collections\ArrayCollection;
use Doctrine\Common\Collections\Collection;
use Doctrine\ORM\Mapping as ORM;
use Oro\Bundle\IntegrationBundle\Entity\Transport;
use Oro\Bundle\LocaleBundle\Entity\LocalizedFallbackValue;
use Symfony\Component\HttpFoundation\ParameterBag;

/**
 * Entity with settings for Collect on delivery integration
 */
#[ORM\Entity(repositoryClass: CollectOnDeliverySettingsRepository::class)]
class CollectOnDeliverySettings extends Transport
{
    /**
     * @var Collection|LocalizedFallbackValue[]
     */
    #[ORM\ManyToMany(targetEntity: 'Oro\Bundle\LocaleBundle\Entity\LocalizedFallbackValue', cascade: ['ALL'], orphanRemoval: true)]
    #[ORM\JoinTable(name: 'acme_coll_on_deliv_trans_label')]
    #[ORM\JoinColumn(name: 'transport_id', referencedColumnName: 'id', onDelete: 'CASCADE')]
    #[ORM\InverseJoinColumn(name: 'localized_value_id', referencedColumnName: 'id', onDelete: 'CASCADE', unique: true)]
    private $labels;

    /**
     * @var Collection|LocalizedFallbackValue[]
     */
    #[ORM\ManyToMany(targetEntity: 'Oro\Bundle\LocaleBundle\Entity\LocalizedFallbackValue', cascade: ['ALL'], orphanRemoval: true)]
    #[ORM\JoinTable(name: 'acme_coll_on_deliv_short_label')]
    #[ORM\JoinColumn(name: 'transport_id', referencedColumnName: 'id', onDelete: 'CASCADE')]
    #[ORM\InverseJoinColumn(name: 'localized_value_id', referencedColumnName: 'id', onDelete: 'CASCADE', unique: true)]
    private $shortLabels;

    /**
     * @var ParameterBag
     */
    private $settings;

    public function __construct()
    {
        $this->labels = new ArrayCollection();
        $this->shortLabels = new ArrayCollection();
    }

    /**
     * @return Collection|LocalizedFallbackValue[]
     */
    public function getLabels()
    {
        return $this->labels;
    }

    /**
     * @param LocalizedFallbackValue $label
     *
     * @return $this
     */
    public function addLabel(LocalizedFallbackValue $label)
    {
        if (!$this->labels->contains($label)) {
            $this->labels->add($label);
        }

        return $this;
    }

    /**
     * @param LocalizedFallbackValue $label
     *
     * @return $this
     */
    public function removeLabel(LocalizedFallbackValue $label)
    {
        if ($this->labels->contains($label)) {
            $this->labels->removeElement($label);
        }

        return $this;
    }

    /**
     * @return Collection|LocalizedFallbackValue[]
     */
    public function getShortLabels()
    {
        return $this->shortLabels;
    }

    /**
     * @param LocalizedFallbackValue $label
     *
     * @return $this
     */
    public function addShortLabel(LocalizedFallbackValue $label)
    {
        if (!$this->shortLabels->contains($label)) {
            $this->shortLabels->add($label);
        }

        return $this;
    }

    /**
     * @param LocalizedFallbackValue $label
     *
     * @return $this
     */
    public function removeShortLabel(LocalizedFallbackValue $label)
    {
        if ($this->shortLabels->contains($label)) {
            $this->shortLabels->removeElement($label);
        }

        return $this;
    }

    /**
     * @return ParameterBag
     */
    #[\Override]
    public function getSettingsBag()
    {
        if (null === $this->settings) {
            $this->settings = new ParameterBag(
                [
                    'labels' => $this->getLabels(),
                    'short_labels' => $this->getShortLabels(),
                ]
            );
        }

        return $this->settings;
    }
}
```

As you can see from the code above, the only two necessary parameters are defined for our collect on delivery payment method: `labels` and `shortLabels`.

#### IMPORTANT
When naming DB columns, make sure that the name does not exceed 31 symbols. Pay attention to the acme_coll_on_deliv_short_label name in the following extract:

```php
    #[ORM\JoinTable(name: 'acme_coll_on_deliv_trans_label')]
    #[ORM\JoinColumn(name: 'transport_id', referencedColumnName: 'id', onDelete: 'CASCADE')]
    #[ORM\InverseJoinColumn(name: 'localized_value_id', referencedColumnName: 'id', onDelete: 'CASCADE', unique: true)]
```

### Create a Repository That Returns the Payment Method Settings

The repository returns on request the configuration settings stored by the entity that you created in the previous step. To add the repository, create <bundle_root>/Entity/Repository/CollectOnDeliverySettingsRepository.php:

```php
<?php

namespace Acme\Bundle\CollectOnDeliveryBundle\Entity\Repository;

use Acme\Bundle\CollectOnDeliveryBundle\Entity\CollectOnDeliverySettings;
use Doctrine\ORM\EntityRepository;

/**
 * Repository for CollectOnDeliverySettings entity
 */
class CollectOnDeliverySettingsRepository extends EntityRepository
{
    /**
     * @return CollectOnDeliverySettings[]
     */
    public function getEnabledSettings()
    {
        return $this->createQueryBuilder('settings')
            ->innerJoin('settings.channel', 'channel')
            ->andWhere('channel.enabled = true')
            ->getQuery()
            ->getResult();
    }
}
```

### Create a User Interface Form for the Payment Method Integration

When you add an integration via the user interface of the back-office, a form that contains the integration settings appears. In this step, implement the form. To do this, create <bundle_root>/Form/Type/CollectOnDeliverySettingsType.php:

```php
<?php

namespace Acme\Bundle\CollectOnDeliveryBundle\Form\Type;

use Acme\Bundle\CollectOnDeliveryBundle\Entity\CollectOnDeliverySettings;
use Oro\Bundle\LocaleBundle\Form\Type\LocalizedFallbackValueCollectionType;
use Symfony\Component\Form\AbstractType;
use Symfony\Component\Form\FormBuilderInterface;
use Symfony\Component\OptionsResolver\OptionsResolver;
use Symfony\Component\Validator\Constraints\NotBlank;

/**
 * Form type for Collect on delivery integration settings
 */
class CollectOnDeliverySettingsType extends AbstractType
{
    const BLOCK_PREFIX = 'acme_collect_on_delivery_setting_type';

    #[\Override]
    public function buildForm(FormBuilderInterface $builder, array $options)
    {
        $builder
            ->add(
                'labels',
                LocalizedFallbackValueCollectionType::class,
                [
                    'label' => 'acme.collect_on_delivery.settings.labels.label',
                    'required' => true,
                    'entry_options' => ['constraints' => [new NotBlank()]],
                ]
            )
            ->add(
                'shortLabels',
                LocalizedFallbackValueCollectionType::class,
                [
                    'label' => 'acme.collect_on_delivery.settings.short_labels.label',
                    'required' => true,
                    'entry_options' => ['constraints' => [new NotBlank()]],
                ]
            );
    }

    #[\Override]
    public function configureOptions(OptionsResolver $resolver)
    {
        $resolver->setDefaults(
            [
                'data_class' => CollectOnDeliverySettings::class,
            ]
        );
    }

    #[\Override]
    public function getBlockPrefix()
    {
        return self::BLOCK_PREFIX;
    }
}
```

### Create a Configuration File for the Service Container

To start using a service container for your bundle, first create the configuration file <bundle_root>/Resources/config/services.yml.

### Set up Services with DependencyInjection

To set up services, load your configuration file (services.yml) using the DependencyInjection component. For this, create <bundle_root>/DependencyInjection/CollectOnDeliveryExtension.php with the following content:

```php
<?php

namespace Acme\Bundle\CollectOnDeliveryBundle\DependencyInjection;

use Symfony\Component\Config\FileLocator;
use Symfony\Component\DependencyInjection\ContainerBuilder;
use Symfony\Component\DependencyInjection\Loader;
use Symfony\Component\HttpKernel\DependencyInjection\Extension;

class AcmeCollectOnDeliveryExtension extends Extension
{
    #[\Override]
    public function load(array $configs, ContainerBuilder $container): void
    {
        $loader = new Loader\YamlFileLoader($container, new FileLocator(__DIR__ . '/../Resources/config'));
        $loader->load('services.yml');
    }
}
```

### Add Translations for the Form Texts

To present the information on the user interface in the user-friendly way, add translations for the payment method settings’ names. To do this, create <bundle_root>/Resources/translations/messages.en.yml:

```yaml
acme:
    collect_on_delivery:
        settings:
            labels.label: 'Labels'
            short_labels.label: 'Short Labels'
```

### Create the Integration Channel Type

When you select the type of the integration on the user interface, you will see the name and the icon that you define in this step. To implement a channel type, create <bundle_root>/Integration/CollectOnDeliveryChannelType.php:

```php
<?php

namespace Acme\Bundle\CollectOnDeliveryBundle\Integration;

use Oro\Bundle\IntegrationBundle\Provider\ChannelInterface;
use Oro\Bundle\IntegrationBundle\Provider\IconAwareIntegrationInterface;

/**
 * Integration channel type for Collect on delivery payment integration
 */
class CollectOnDeliveryChannelType implements ChannelInterface, IconAwareIntegrationInterface
{
    const TYPE = 'collect_on_delivery';

    #[\Override]
    public function getLabel()
    {
        return 'acme.collect_on_delivery.channel_type.label';
    }

    #[\Override]
    public function getIcon()
    {
        return 'bundles/oromoneyorder/img/money-order-icon.png';
    }
}
```

### Add an Icon for the Integration

To add an icon:

1. Save the file to the <bundle_root>/Resources/public/img directory.
2. Install assets:
   ```bash
   php bin/console assets:install --symlink
   ```

To make sure that the icon is accessible for the web interface, check if it appears (as a copy or a symlink depending on the settings selected during the application installation) in the /public/bundles/collect_on_delivery/img directory of your application.

### Create the Integration Transport

A transport is generally responsible for how the data is obtained from the external system. While the Collect On Delivery method does not interact with external systems, you still need to define a transport and implement all methods of the TransportInterface for the integration to work properly. To add a transport, create <bundle_root>/Integration/CollectOnDeliveryTransport.php:

```php
<?php

namespace Acme\Bundle\CollectOnDeliveryBundle\Integration;

use Acme\Bundle\CollectOnDeliveryBundle\Entity\CollectOnDeliverySettings;
use Acme\Bundle\CollectOnDeliveryBundle\Form\Type\CollectOnDeliverySettingsType;
use Oro\Bundle\IntegrationBundle\Entity\Transport;
use Oro\Bundle\IntegrationBundle\Provider\TransportInterface;

/**
 * Transport for Collect on delivery payment integration
 */
class CollectOnDeliveryTransport implements TransportInterface
{
    #[\Override]
    public function init(Transport $transportEntity)
    {
    }

    #[\Override]
    public function getLabel()
    {
        return 'acme.collect_on_delivery.settings.transport.label';
    }

    #[\Override]
    public function getSettingsFormType()
    {
        return CollectOnDeliverySettingsType::class;
    }

    #[\Override]
    public function getSettingsEntityFQCN()
    {
        return CollectOnDeliverySettings::class;
    }
}
```

### Add the Channel Type and Transport to the Services Container

To register the channel type and transport, append the following key-values to <bundle_root>/Resources/config/services.yml:

```yaml
parameters:
    acme_collect_on_delivery.method.identifier_prefix.collect_on_delivery: 'collect_on_delivery'

services:
    acme_collect_on_delivery.generator.collect_on_delivery_config_identifier:
        parent: oro_integration.generator.prefixed_identifier_generator
        public: true
        arguments:
            - '%acme_collect_on_delivery.method.identifier_prefix.collect_on_delivery%'

    acme_collect_on_delivery.integration.channel:
        class: Acme\Bundle\CollectOnDeliveryBundle\Integration\CollectOnDeliveryChannelType
        public: true
        tags:
            - { name: oro_integration.channel, type: collect_on_delivery }

    acme_collect_on_delivery.integration.transport:
        class: Acme\Bundle\CollectOnDeliveryBundle\Integration\CollectOnDeliveryTransport
        public: false
        tags:
            - { name: oro_integration.transport, type: collect_on_delivery, channel_type: collect_on_delivery }
```

### Add Translations for the Channel Type and Transport

The channel type and, in general, transport labels also appear on the user interface (you will not see the transport label for Collect On Delivery). Provide translations for them by appending the <bundle_root>/Resources/translations/messages.en.yml. Now, the messages.en.yml content must look as follows:

```yaml
acme:
    collect_on_delivery:
        settings:
            labels.label: 'Labels'
            short_labels.label: 'Short Labels'
            transport.label: 'Collect on delivery'

        channel_type.label: 'Collect on delivery'
        payment_method_message: 'Pay on delivery'
```

### Add an Installer

An installer ensures that upon the application installation, the database will contain the entity that you defined within your bundle.

Follow the instructions provided in the [How to generate an installer](../../entities/migration.md#installer-generate) topic to apply the changes without migration and generate an installer file based on the current schema of the DB.

#### NOTE
If you have not performed the steps mentioned in [How to generate an installer](../../entities/migration.md#installer-generate), because you already have the installer file, then make sure to run the `php bin/console oro:migration:load --force` command to apply the changes from the file.

After you complete it, you will have the class <bundle_root>/Migrations/Schema/CollectOnDeliveryBundleInstaller.php with the following content:

```php
<?php

namespace Acme\Bundle\CollectOnDeliveryBundle\Migrations\Schema;

use Doctrine\DBAL\Schema\Schema;
use Oro\Bundle\MigrationBundle\Migration\Installation;
use Oro\Bundle\MigrationBundle\Migration\QueryBag;

/**
 * @SuppressWarnings(PHPMD.TooManyMethods)
 * @SuppressWarnings(PHPMD.ExcessiveClassLength)
 */
class AcmeCollectOnDeliveryBundleInstaller implements Installation
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
        $this->createAcmeCollOnDelivTransLabelTable($schema);
        $this->createAcmeCollOnDelivShortLabelTable($schema);

        /** Foreign keys generation **/
        $this->addAcmeCollOnDelivTransLabelForeignKeys($schema);
        $this->addAcmeCollOnDelivShortLabelForeignKeys($schema);
    }

    /**
     * Create acme_coll_on_deliv_trans_label table
     */
    protected function createAcmeCollOnDelivTransLabelTable(Schema $schema)
    {
        $table = $schema->createTable('acme_coll_on_deliv_trans_label');
        $table->addColumn('transport_id', 'integer', []);
        $table->addColumn('localized_value_id', 'integer', []);
        $table->setPrimaryKey(['transport_id', 'localized_value_id']);
        $table->addIndex(['transport_id'], 'idx_13476d069909c13f', []);
        $table->addUniqueIndex(['localized_value_id'], 'uniq_13476d06eb576e89');
    }

    /**
     * Create acme_coll_on_deliv_short_label table
     */
    protected function createAcmeCollOnDelivShortLabelTable(Schema $schema)
    {
        $table = $schema->createTable('acme_coll_on_deliv_short_label');
        $table->addColumn('transport_id', 'integer', []);
        $table->addColumn('localized_value_id', 'integer', []);
        $table->addUniqueIndex(['localized_value_id'], 'uniq_2c81a8dceb576e89');
        $table->addIndex(['transport_id'], 'idx_2c81a8dc9909c13f', []);
        $table->setPrimaryKey(['transport_id', 'localized_value_id']);
    }

    /**
     * Add acme_coll_on_deliv_trans_label foreign keys.
     */
    protected function addAcmeCollOnDelivTransLabelForeignKeys(Schema $schema)
    {
        $table = $schema->getTable('acme_coll_on_deliv_trans_label');
        $table->addForeignKeyConstraint(
            $schema->getTable('oro_fallback_localization_val'),
            ['localized_value_id'],
            ['id'],
            ['onUpdate' => null, 'onDelete' => 'CASCADE']
        );
        $table->addForeignKeyConstraint(
            $schema->getTable('oro_integration_transport'),
            ['transport_id'],
            ['id'],
            ['onUpdate' => null, 'onDelete' => 'CASCADE']
        );
    }

    /**
     * Add acme_coll_on_deliv_short_label foreign keys.
     */
    protected function addAcmeCollOnDelivShortLabelForeignKeys(Schema $schema)
    {
        $table = $schema->getTable('acme_coll_on_deliv_short_label');
        $table->addForeignKeyConstraint(
            $schema->getTable('oro_fallback_localization_val'),
            ['localized_value_id'],
            ['id'],
            ['onUpdate' => null, 'onDelete' => 'CASCADE']
        );
        $table->addForeignKeyConstraint(
            $schema->getTable('oro_integration_transport'),
            ['transport_id'],
            ['id'],
            ['onUpdate' => null, 'onDelete' => 'CASCADE']
        );
    }
}
```

### Check That the Integration is Created Successfully

1. Clear the application cache:
   ```bash
   php bin/console cache:clear
   ```

   #### NOTE
   If you are working in production environment, you have to use the `--env=prod` parameter  with the command.
2. Open the user interface and check that the changes have applied and you can add an integration of the Collect On Delivery type.

## Implement a Payment Method

Now implement the payment method itself.

### Create a Factory for the Payment Method Configuration

A configuration factory generates an individual configuration set for each instance of the integration of the Collect On Delivery type.

To add a payment method configuration factory, in the directory <bundle_root>/PaymentMethod/Config/Factory/ create interface CollectOnDeliveryConfigFactoryInterface.php and the class CollectOnDeliveryConfigFactory.php that implements this interface:

#### Configuration Factory Interface

```php
<?php

namespace Acme\Bundle\CollectOnDeliveryBundle\PaymentMethod\Config\Factory;

use Acme\Bundle\CollectOnDeliveryBundle\Entity\CollectOnDeliverySettings;
use Acme\Bundle\CollectOnDeliveryBundle\PaymentMethod\Config\CollectOnDeliveryConfigInterface;

/**
 * Interface for Collect on delivery payment method config factory
 * Creates instances of CollectOnDeliverySettings with configuration for payment method
 */
interface CollectOnDeliveryConfigFactoryInterface
{
    /**
     * @param CollectOnDeliverySettings $settings
     * @return CollectOnDeliveryConfigInterface
     */
    public function create(CollectOnDeliverySettings $settings);
}
```

#### Configuration Factory Class

```php
<?php

namespace Acme\Bundle\CollectOnDeliveryBundle\PaymentMethod\Config\Factory;

use Acme\Bundle\CollectOnDeliveryBundle\Entity\CollectOnDeliverySettings;
use Acme\Bundle\CollectOnDeliveryBundle\PaymentMethod\Config\CollectOnDeliveryConfig;
use Doctrine\Common\Collections\Collection;
use Oro\Bundle\IntegrationBundle\Generator\IntegrationIdentifierGeneratorInterface;
use Oro\Bundle\LocaleBundle\Helper\LocalizationHelper;

/**
 * Creates instances of configurations for Collect on delivery payment method
 */
class CollectOnDeliveryConfigFactory implements CollectOnDeliveryConfigFactoryInterface
{
    /**
     * @var LocalizationHelper
     */
    private $localizationHelper;

    /**
     * @var IntegrationIdentifierGeneratorInterface
     */
    private $identifierGenerator;

    public function __construct(
        LocalizationHelper $localizationHelper,
        IntegrationIdentifierGeneratorInterface $identifierGenerator
    ) {
        $this->localizationHelper = $localizationHelper;
        $this->identifierGenerator = $identifierGenerator;
    }

    #[\Override]
    public function create(CollectOnDeliverySettings $settings)
    {
        $params = [];
        $channel = $settings->getChannel();

        $params[CollectOnDeliveryConfig::FIELD_LABEL] = $this->getLocalizedValue($settings->getLabels());
        $params[CollectOnDeliveryConfig::FIELD_SHORT_LABEL] = $this->getLocalizedValue($settings->getShortLabels());
        $params[CollectOnDeliveryConfig::FIELD_ADMIN_LABEL] = $channel->getName();
        $params[CollectOnDeliveryConfig::FIELD_PAYMENT_METHOD_IDENTIFIER] =
            $this->identifierGenerator->generateIdentifier($channel);

        return new CollectOnDeliveryConfig($params);
    }

    /**
     * @param Collection $values
     *
     * @return string
     */
    private function getLocalizedValue(Collection $values)
    {
        return (string)$this->localizationHelper->getLocalizedValue($values);
    }
}
```

### Create a Provider for the Payment Method Configuration

A configuration provider accepts and integration id and returns settings based on it.

To add a payment method configuration provider, in the directory <bundle_root>/PaymentMethod/Config/Provider/ create interface CollectOnDeliveryConfigProviderInterface.php and the class CollectOnDeliveryConfigProvider.php that implements this interface:

#### Configuration Provider Interface

```php
<?php

namespace Acme\Bundle\CollectOnDeliveryBundle\PaymentMethod\Config\Provider;

use Acme\Bundle\CollectOnDeliveryBundle\PaymentMethod\Config\CollectOnDeliveryConfigInterface;

/**
 * Interface for config provider which allows to get configs based on payment method identifier
 */
interface CollectOnDeliveryConfigProviderInterface
{
    /**
     * @return CollectOnDeliveryConfigInterface[]
     */
    public function getPaymentConfigs();

    /**
     * @param string $identifier
     * @return CollectOnDeliveryConfigInterface|null
     */
    public function getPaymentConfig($identifier);

    /**
     * @param string $identifier
     * @return bool
     */
    public function hasPaymentConfig($identifier);
}
```

#### Configuration Provider Class

```php
<?php

namespace Acme\Bundle\CollectOnDeliveryBundle\PaymentMethod\Config\Provider;

use Acme\Bundle\CollectOnDeliveryBundle\Entity\CollectOnDeliverySettings;
use Acme\Bundle\CollectOnDeliveryBundle\Entity\Repository\CollectOnDeliverySettingsRepository;
use Acme\Bundle\CollectOnDeliveryBundle\PaymentMethod\Config\CollectOnDeliveryConfigInterface;
use Acme\Bundle\CollectOnDeliveryBundle\PaymentMethod\Config\Factory\CollectOnDeliveryConfigFactoryInterface;
use Doctrine\Persistence\ManagerRegistry;
use Psr\Log\LoggerInterface;

/**
 * Allows to get configs of Collect on delivery payment method
 */
class CollectOnDeliveryConfigProvider implements CollectOnDeliveryConfigProviderInterface
{
    /**
     * @var ManagerRegistry
     */
    protected $doctrine;

    /**
     * @var CollectOnDeliveryConfigFactoryInterface
     */
    protected $configFactory;

    /**
     * @var CollectOnDeliveryConfigInterface[]
     */
    protected $configs;

    /**
     * @var LoggerInterface
     */
    protected $logger;

    public function __construct(
        ManagerRegistry $doctrine,
        LoggerInterface $logger,
        CollectOnDeliveryConfigFactoryInterface $configFactory
    ) {
        $this->doctrine = $doctrine;
        $this->logger = $logger;
        $this->configFactory = $configFactory;
    }

    #[\Override]
    public function getPaymentConfigs()
    {
        $configs = [];

        $settings = $this->getEnabledIntegrationSettings();

        foreach ($settings as $setting) {
            $config = $this->configFactory->create($setting);

            $configs[$config->getPaymentMethodIdentifier()] = $config;
        }

        return $configs;
    }

    #[\Override]
    public function getPaymentConfig($identifier)
    {
        $paymentConfigs = $this->getPaymentConfigs();

        if ([] === $paymentConfigs || false === array_key_exists($identifier, $paymentConfigs)) {
            return null;
        }

        return $paymentConfigs[$identifier];
    }

    #[\Override]
    public function hasPaymentConfig($identifier)
    {
        return null !== $this->getPaymentConfig($identifier);
    }

    /**
     * @return CollectOnDeliverySettings[]
     */
    protected function getEnabledIntegrationSettings()
    {
        try {
            /** @var CollectOnDeliverySettingsRepository $repository */
            $repository = $this->doctrine
                ->getManagerForClass(CollectOnDeliverySettings::class)
                ->getRepository(CollectOnDeliverySettings::class);

            return $repository->getEnabledSettings();
        } catch (\UnexpectedValueException $e) {
            $this->logger->critical($e->getMessage());

            return [];
        }
    }
}
```

### Implement Payment Method Configuration

In the <bundle_root>/PaymentMethod/Config directory, create the CollectOnDeliveryConfigInterface.php interface and the CollectOnDeliveryConfig.php class that implements this interface:

#### Configuration Interface

```php
<?php

namespace Acme\Bundle\CollectOnDeliveryBundle\PaymentMethod\Config;

use Oro\Bundle\PaymentBundle\Method\Config\PaymentConfigInterface;

/**
 * Interface that describes specific configuration for Collect on delivery payment method
 */
interface CollectOnDeliveryConfigInterface extends PaymentConfigInterface
{
}
```

#### Configuration Class

```php
<?php

namespace Acme\Bundle\CollectOnDeliveryBundle\PaymentMethod\Config;

use Oro\Bundle\PaymentBundle\Method\Config\ParameterBag\AbstractParameterBagPaymentConfig;

/**
 * Configuration class which is used to get specific configuration for Collect on delivery payment method
 * Usually it has additional get methods for payment type specific configurations
 */
class CollectOnDeliveryConfig extends AbstractParameterBagPaymentConfig implements CollectOnDeliveryConfigInterface
{
}
```

### Add the Payment Method Configuration Factory and Provider to the Services Container

To register the payment method configuration factory and provider, append the following key-values to <bundle_root>/Resources/config/services.yml:

```yaml
    acme_collect_on_delivery.factory.collect_on_delivery_config:
        class: Acme\Bundle\CollectOnDeliveryBundle\PaymentMethod\Config\Factory\CollectOnDeliveryConfigFactory
        public: false
        arguments:
            - '@oro_locale.helper.localization'
            - '@acme_collect_on_delivery.generator.collect_on_delivery_config_identifier'

    acme_collect_on_delivery.payment_method.config.provider:
        class: Acme\Bundle\CollectOnDeliveryBundle\PaymentMethod\Config\Provider\CollectOnDeliveryConfigProvider
        arguments:
            - '@doctrine'
            - '@logger'
            - '@acme_collect_on_delivery.factory.collect_on_delivery_config'
```

### Create a Factory for the Payment Method View

Views provide the set of options for the payment method blocks that users see when they select the Collect on Delivery payment method and review the orders during the checkout.

To add a payment method view factory, in the directory <bundle_root>/PaymentMethod/View/Factory/ create interface CollectOnDeliveryViewFactoryInterface.php and the class CollectOnDeliveryViewFactory.php that implements this interface:

#### Payment Method View Factory Interface

```php
<?php

namespace Acme\Bundle\CollectOnDeliveryBundle\PaymentMethod\View\Factory;

use Acme\Bundle\CollectOnDeliveryBundle\PaymentMethod\Config\CollectOnDeliveryConfigInterface;
use Oro\Bundle\PaymentBundle\Method\View\PaymentMethodViewInterface;

/**
 * Factory for creating views of Collect on delivery payment method
 */
interface CollectOnDeliveryViewFactoryInterface
{
    /**
     * @param CollectOnDeliveryConfigInterface $config
     * @return PaymentMethodViewInterface
     */
    public function create(CollectOnDeliveryConfigInterface $config);
}
```

#### Payment Method View Factory Class

```php
<?php

namespace Acme\Bundle\CollectOnDeliveryBundle\PaymentMethod\View\Factory;

use Acme\Bundle\CollectOnDeliveryBundle\PaymentMethod\Config\CollectOnDeliveryConfigInterface;
use Acme\Bundle\CollectOnDeliveryBundle\PaymentMethod\View\CollectOnDeliveryView;

/**
 * Factory for creating views of Collect on delivery payment method
 */
class CollectOnDeliveryViewFactory implements CollectOnDeliveryViewFactoryInterface
{
    #[\Override]
    public function create(CollectOnDeliveryConfigInterface $config)
    {
        return new CollectOnDeliveryView($config);
    }
}
```

### Create Provider for the Payment Method View

To add a payment method view provider, create <bundle_root>/PaymentMethod/View/Provider/CollectOnDeliveryViewProvider.php:

#### Payment Method View Provider Class

```php
<?php

namespace Acme\Bundle\CollectOnDeliveryBundle\PaymentMethod\View\Provider;

use Acme\Bundle\CollectOnDeliveryBundle\PaymentMethod\Config\CollectOnDeliveryConfigInterface;
use Acme\Bundle\CollectOnDeliveryBundle\PaymentMethod\Config\Provider\CollectOnDeliveryConfigProviderInterface;
use Acme\Bundle\CollectOnDeliveryBundle\PaymentMethod\View\Factory\CollectOnDeliveryViewFactoryInterface;
use Oro\Bundle\PaymentBundle\Method\View\AbstractPaymentMethodViewProvider;

/**
 * Provider for retrieving payment method view instances
 */
class CollectOnDeliveryViewProvider extends AbstractPaymentMethodViewProvider
{
    /** @var CollectOnDeliveryViewFactoryInterface */
    private $factory;

    /** @var CollectOnDeliveryConfigProviderInterface */
    private $configProvider;

    public function __construct(
        CollectOnDeliveryConfigProviderInterface $configProvider,
        CollectOnDeliveryViewFactoryInterface $factory
    ) {
        $this->factory = $factory;
        $this->configProvider = $configProvider;

        parent::__construct();
    }

    #[\Override]
    protected function buildViews()
    {
        $configs = $this->configProvider->getPaymentConfigs();
        foreach ($configs as $config) {
            $this->addCollectOnDeliveryView($config);
        }
    }

    protected function addCollectOnDeliveryView(CollectOnDeliveryConfigInterface $config)
    {
        $this->addView(
            $config->getPaymentMethodIdentifier(),
            $this->factory->create($config)
        );
    }
}
```

### Implement the Payment Method View

Finally, to implement the payment method view, create <bundle_root>/PaymentMethod/ViewCollectOnDeliveryView.php:

#### Payment Method View Class

```php
<?php

namespace Acme\Bundle\CollectOnDeliveryBundle\PaymentMethod\View;

use Acme\Bundle\CollectOnDeliveryBundle\PaymentMethod\Config\CollectOnDeliveryConfigInterface;
use Oro\Bundle\PaymentBundle\Context\PaymentContextInterface;
use Oro\Bundle\PaymentBundle\Method\View\PaymentMethodViewInterface;

/**
 * View for Collect on delivery payment method
 */
class CollectOnDeliveryView implements PaymentMethodViewInterface
{
    /**
     * @var CollectOnDeliveryConfigInterface
     */
    protected $config;

    public function __construct(CollectOnDeliveryConfigInterface $config)
    {
        $this->config = $config;
    }

    #[\Override]
    public function getOptions(PaymentContextInterface $context)
    {
        return [];
    }

    #[\Override]
    public function getBlock()
    {
        return '_payment_methods_collect_on_delivery_widget';
    }

    #[\Override]
    public function getLabel()
    {
        return $this->config->getLabel();
    }

    #[\Override]
    public function getShortLabel()
    {
        return $this->config->getShortLabel();
    }

    #[\Override]
    public function getAdminLabel()
    {
        return $this->config->getAdminLabel();
    }


    #[\Override]
    public function getPaymentMethodIdentifier()
    {
        return $this->config->getPaymentMethodIdentifier();
    }
}
```

### Add the Payment Method View Factory and Provider to the Services Container

To register the payment method view factory and provider, append the following key-values to <bundle_root>/Resources/config/services.yml:

```yaml
    acme_collect_on_delivery.payment_method_view_provider.collect_on_delivery:
        class: Acme\Bundle\CollectOnDeliveryBundle\PaymentMethod\View\Provider\CollectOnDeliveryViewProvider
        public: false
        arguments:
            - '@acme_collect_on_delivery.payment_method.config.provider'
            - '@acme_collect_on_delivery.factory.method_view.collect_on_delivery'
        tags:
            - { name: oro_payment.payment_method_view_provider }

    acme_collect_on_delivery.factory.method.collect_on_delivery:
        class: Acme\Bundle\CollectOnDeliveryBundle\PaymentMethod\Factory\CollectOnDeliveryPaymentMethodFactory
        public: false

```

### Create a Factory for the Main Method

To add a payment method factory, in the directory <bundle_root>/PaymentMethod/Factory/ create interface CollectOnDeliveryPaymentMethodFactoryInterface.php and the class CollectOnDeliveryPaymentMethodFactory.php that implements this interface:

#### Factory Interface

```php
<?php

namespace Acme\Bundle\CollectOnDeliveryBundle\PaymentMethod\Factory;

use Acme\Bundle\CollectOnDeliveryBundle\PaymentMethod\Config\CollectOnDeliveryConfigInterface;
use Oro\Bundle\PaymentBundle\Method\PaymentMethodInterface;

/**
 * Interface of factories which create payment method instances based on configuration
 */
interface CollectOnDeliveryPaymentMethodFactoryInterface
{
    /**
     * @param CollectOnDeliveryConfigInterface $config
     * @return PaymentMethodInterface
     */
    public function create(CollectOnDeliveryConfigInterface $config);
}
```

#### Factory Class

```php
<?php

namespace Acme\Bundle\CollectOnDeliveryBundle\PaymentMethod\Factory;

use Acme\Bundle\CollectOnDeliveryBundle\PaymentMethod\CollectOnDelivery;
use Acme\Bundle\CollectOnDeliveryBundle\PaymentMethod\Config\CollectOnDeliveryConfigInterface;

/**
 * Factory creates payment method instances based on configuration
 */
class CollectOnDeliveryPaymentMethodFactory implements CollectOnDeliveryPaymentMethodFactoryInterface
{
    #[\Override]
    public function create(CollectOnDeliveryConfigInterface $config)
    {
        return new CollectOnDelivery($config);
    }
}
```

### Create Provider for the Main Method

To add a payment method provider, create <bundle_root>/PaymentMethod/Provider/CollectOnDeliveryProvider.php:

#### Provider Class

```php
<?php

namespace Acme\Bundle\CollectOnDeliveryBundle\PaymentMethod\Provider;

use Acme\Bundle\CollectOnDeliveryBundle\PaymentMethod\Config\CollectOnDeliveryConfigInterface;
use Acme\Bundle\CollectOnDeliveryBundle\PaymentMethod\Config\Provider\CollectOnDeliveryConfigProviderInterface;
use Acme\Bundle\CollectOnDeliveryBundle\PaymentMethod\Factory\CollectOnDeliveryPaymentMethodFactoryInterface;
use Oro\Bundle\PaymentBundle\Method\Provider\AbstractPaymentMethodProvider;

/**
 * Provider for retrieving configured payment method instances
 */
class CollectOnDeliveryMethodProvider extends AbstractPaymentMethodProvider
{
    /**
     * @var CollectOnDeliveryPaymentMethodFactoryInterface
     */
    protected $factory;

    /**
     * @var CollectOnDeliveryConfigProviderInterface
     */
    private $configProvider;

    public function __construct(
        CollectOnDeliveryConfigProviderInterface $configProvider,
        CollectOnDeliveryPaymentMethodFactoryInterface $factory
    ) {
        parent::__construct();

        $this->configProvider = $configProvider;
        $this->factory = $factory;
    }

    #[\Override]
    protected function collectMethods()
    {
        $configs = $this->configProvider->getPaymentConfigs();
        foreach ($configs as $config) {
            $this->addCollectOnDeliveryMethod($config);
        }
    }

    protected function addCollectOnDeliveryMethod(CollectOnDeliveryConfigInterface $config)
    {
        $this->addMethod(
            $config->getPaymentMethodIdentifier(),
            $this->factory->create($config)
        );
    }
}
```

### Implement the Main Method

Now, implement the main method. To do this, create the <bundle_root>/PaymentMethod/CollectOnDelivery.php class:

#### Class

```php
<?php

namespace Acme\Bundle\CollectOnDeliveryBundle\PaymentMethod;

use Acme\Bundle\CollectOnDeliveryBundle\PaymentMethod\Config\CollectOnDeliveryConfigInterface;
use Oro\Bundle\PaymentBundle\Context\PaymentContextInterface;
use Oro\Bundle\PaymentBundle\Entity\PaymentTransaction;
use Oro\Bundle\PaymentBundle\Method\PaymentMethodInterface;

/**
 * Payment method class that describes main business logic of Collect on delivery payment method
 * It creates invoice payment transaction
 */
class CollectOnDelivery implements PaymentMethodInterface
{
    /**
     * @var CollectOnDeliveryConfigInterface
     */
    private $config;

    public function __construct(CollectOnDeliveryConfigInterface $config)
    {
        $this->config = $config;
    }

    #[\Override]
    public function execute($action, PaymentTransaction $paymentTransaction)
    {
        $paymentTransaction->setAction(PaymentMethodInterface::INVOICE);
        $paymentTransaction->setActive(true);
        $paymentTransaction->setSuccessful(true);

        return [];
    }

    #[\Override]
    public function getIdentifier()
    {
        return $this->config->getPaymentMethodIdentifier();
    }

    #[\Override]
    public function isApplicable(PaymentContextInterface $context)
    {
        return true;
    }

    #[\Override]
    public function supports($actionName)
    {
        return $actionName === self::PURCHASE;
    }
}
```

#### HINT
Pay attention to the lines:

```php
    public function supports($actionName)
    {
        return $actionName === self::PURCHASE;
    }
```

This is where you define which transaction types are associated with the payment method. To keep it simple, for Collect On Delivery a single transaction is defined. Thus, it will work the following way: when a user submits an order, the “purchase” transaction takes place, and the order status becomes “purchased”.

Check <a href="https://github.com/oroinc/orocommerce/blob/master/src/Oro/Bundle/PaymentBundle/Method/PaymentMethodInterface.php" target="_blank">PaymentMethodInterface</a> for more information on other predefined transactions.

#### NOTE
You can additionally implement the OroBundlePaymentBundleMethodPaymentMethodGroupAwareInterface to restrict the payment method to a specific group of payment methods. You can get the list of available payment methods in a specific group via OroBundlePaymentBundleMethodProviderPaymentMethodGroupAwareProvider.

### Add the Payment Method Factory and Provider to the Services Container

To register the payment method main factory and provider, append the following key-values to <bundle_root>/Resources/config/services.yml:

```yaml
    acme_collect_on_delivery.factory.method.collect_on_delivery:
        class: Acme\Bundle\CollectOnDeliveryBundle\PaymentMethod\Factory\CollectOnDeliveryPaymentMethodFactory
        public: false

    acme_collect_on_delivery.payment_method_provider.collect_on_delivery:
        class: Acme\Bundle\CollectOnDeliveryBundle\PaymentMethod\Provider\CollectOnDeliveryMethodProvider
        public: false
        arguments:
            - '@acme_collect_on_delivery.payment_method.config.provider'
            - '@acme_collect_on_delivery.factory.method.collect_on_delivery'
        tags:
            - { name: oro_payment.payment_method_provider }
```

### Define the Payment Method’s Layouts for the Storefront

Layouts provide the html template for the payment method blocks that users see when doing the checkout in the storefront. There are two different blocks: one that users see during selection of the payment method, and the other that they see when reviewing the order. You need to define templates for each of these blocks.

For this, in the directory <bundle_root>/Resources/views/layouts/default/imports/, create templates for the payment method selection checkout step:

- oro_payment_method_options/layout.html.twig
- oro_payment_method_options/layout.html

> and for the order review:
- oro_payment_method_order_submit/layout.html.twig
- oro_payment_method_order_submit/layout.html

#### layout.html.twig for the Payment Method Selection

```html
{% block _payment_methods_collect_on_delivery_widget %}
    <div class="{{ class_prefix }}-form__payment-methods">
        <table class="grid">
            <tr>
                <td>{{ 'acme.collect_on_delivery.payment_method_message'|trans }}</td>
            </tr>
        </table>
    </div>
{% endblock %}
```

Note that the custom message to appear in the block is defined. Do not forget to add translations in the messages.en.yml for any custom text that you add.

#### layout.html for the Payment Method Selection

```yaml
layout:
    actions:
        - '@setBlockTheme':
            themes:
                - 'layout.html.twig'
```

#### layout.html.twig for the Order Review

```html
{% block _order_review_payment_methods_collect_on_delivery_widget -%}
    {% if options.payment_method is defined %}
        <div class="hidden"
             data-page-component-module="oropayment/js/app/components/payment-method-component"
             data-page-component-options="{{ {paymentMethod: options.payment_method}|json_encode }}">
        </div>
    {% endif %}
{%- endblock %}
```

#### layout.html for the Order Review

```html
layout:
    actions:
        - '@setBlockTheme':
            themes:
                - 'layout.html.twig'
```

### Define a Translation for the Custom Message

In step, you have added a custom message to the payment method block. Define a translation for it in the messages.en.yml which now should look like the following:

```yaml
acme:
    collect_on_delivery:
        settings:
            labels.label: 'Labels'
            short_labels.label: 'Short Labels'
            transport.label: 'Collect on delivery'

        channel_type.label: 'Collect on delivery'
        payment_method_message: 'Pay on delivery'
```

### Check That Payment Method Is Added

Now, the Collect On Delivery payment method is fully implemented.

Clear the application cache, open the user interface and try to submit an order.

<!-- Frontend -->
<!-- .. toctree::
:maxdepth: 1
:hidden: -->
<!-- CyberSource Integration <cybersource> -->
