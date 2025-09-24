<a id="book-entities-entity-configuration-configure-entity-config-attribute"></a>

# Define a New Object Configuration Attribute

You can use configuration to define a new entity config attribute:

1. Create a configuration file that implements `EntityConfigInterface` or `FieldConfigInterface`. For entity config, use `EntityConfigInterface` and the class that ends with EntityConfiguration. For field config, use `FieldConfigInterface` and the class that ends with FieldConfiguration.

Example:

```php
<?php

namespace Acme\Bundle\DemoBundle\EntityConfig;

use Oro\Bundle\EntityConfigBundle\EntityConfig\EntityConfigInterface;
use Symfony\Component\Config\Definition\Builder\NodeBuilder;

/**
 * Provides validations entity config for acme scope.
 */
class AcmeEntityConfiguration implements EntityConfigInterface
{
    #[\Override]
    public function getSectionName(): string
    {
        return 'acme';
    }

    #[\Override]
    public function configure(NodeBuilder $nodeBuilder): void
    {
        $nodeBuilder
            ->scalarNode('demo_attr')
            ->info('`string` demo attribute description.')
            ->defaultFalse()
            ->end()
        ;
    }
}
```

1. Add this class to `services.yml` with tag `oro_entity_config.validation.entity_config`.

Example:

```yaml
services:
    Acme\Bundle\DemoBundle\EntityConfig\AcmeEntityConfiguration:
        tags:
            - oro_entity_config.validation.entity_config
```

## Add Settings to entity_config.yml

To illustrate how you can add metadata to an entity, add the following YAML file (this file must be located in `[BundleName]/Resources/config/oro/entity_config.yml`):

```yaml
entity_config:
  acme:                                 # a configuration scope name
    entity:                             # a section describes an entity
      items:                            # starts a description of entity attributes
        demo_attr:                      # adds an attribute named 'demo_attr'
          options:
            indexed:  true              # TRUE if an attribute should be filterable or sortable in a data grid
            priority: 100
```

This configuration adds the ‘demo_attr’ attribute with the ‘Demo’ value to all configurable entities. The configurable entity is an entity marked with the #[Config] attribute. This code also automatically adds a service named **oro_entity_config.provider.acme** into the DI container. You can use this service to get the value of a particular entity’s ‘demo_attr’ attribute.

To apply this change, execute the **oro:entity-config:update** command that updates configuration data for entities:

```bash
php bin/console oro:entity-config:update
```

An example how to get a value of a configuration attribute:

```php
 /** @var Symfony\Component\DependencyInjection\ContainerInterface $container */
 $container = ...;

/** @var Oro\Bundle\EntityConfigBundle\Provider\ConfigProvider $acmeConfigProvider */
$acmeConfigProvider = $container->get('oro_entity_config.provider.acme');

// retrieve a value of 'demo_attr' attribute for 'Acme\Bundle\DemoBundle\Entity\Document' entity
// the value of $demoAttr variable will be 'Demo'
$demoAttr = $acmeConfigProvider->getConfig('Acme\Bundle\DemoBundle\Entity\Document')->get('demo_attr');
```

If you want to set a value different than the default one for some entity, write it in the #[Config] attribute for this entity. For example:

```php
namespace Acme\Bundle\DemoBundle\Entity;

use Doctrine\ORM\Mapping as ORM;
use Oro\Bundle\EntityConfigBundle\Attribute\Entity\AttributeFamily;

/**
 * ORM Entity Document.
 */
#[ORM\Table(name: 'acme_demo_document')]
#[Config(
    defaultValues: [
        'acme' => ['demo_attr' => 'MyValue'],
    ]
)]
```

The result is demonstrated in the following code:

```php
 /** @var Symfony\Component\DependencyInjection\ContainerInterface $container */
 $container = ...;

/** @var Oro\Bundle\EntityConfigBundle\Provider\ConfigProvider $acmeConfigProvider */
$acmeConfigProvider = $container->get('oro_entity_config.provider.acme');

// retrieve a value of 'demo_attr' attribute for 'Acme\Bundle\DemoBundle\Entity\Document' entity
// the value of $demoAttr1 variable will be 'Demo'
$demoAttr1 = $acmeConfigProvider->getConfig('Acme\Bundle\DemoBundle\Entity\Document')->get('demo_attr');

// retrieve a value of 'demo_attr' attribute for 'Acme\Bundle\DemoBundle\Entity\MyEntity' entity
// the value of $demoAttr2 variable will be 'MyValue'
$demoAttr2 = $acmeConfigProvider->getConfig('Acme\Bundle\DemoBundle\Entity\MyEntity')->get('demo_attr');
```

Essentially, it is all you need to add metadata to any entity. But in most cases, you want to allow an administrator to manage your attribute in UI. To accomplish this, let’s change the YAML file the following way:

```yaml
entity_config:
  acme:                                 # a configuration scope name
    entity:                             # a section describes an entity
      items:                            # starts a description of entity attributes
        demo_attr:                      # adds an attribute named 'demo_attr'
          options:
            default_value: 'Demo'       # sets the default value for 'demo_attr' attribute
            translatable:  true         # means that value of this attribute is translation key
              # and actual value should be taken from translation table
            # or in twig via "|trans" filter
            indexed:  true              # TRUE if an attribute should be filterable or sortable in a data grid
            priority: 100
          grid:                         # configure a data grid to display 'demo_attr' attribute
            type:          string       # sets the attribute type
            label:         'Demo Attr'  # sets the data grid column name
            show_filter:   true         # the next three lines configure a filter for 'Demo Attr' column
            filterable:    true
            filter_type:   string
            sortable:      true         # allows an administrator to sort rows clicks on 'Demo Attr' column
          form:
            type:          Symfony\Component\Form\Extension\Core\Type\TextType         # sets the attribute type
            options:
              block:     entity       # specifies in which block on the form this attribute should be displayed
```

Now you can go to System > Entities in the back-office. The ‘Demo Attr’ column should be displayed in the grid. Click Edit on any entity to open the edit entity form. The ‘Demo Attr’ field should be displayed there.

#### HINT
Check out the [example of YAML config](../../configuration/yaml/entity-config.md#yaml-format-config-entity).

<!-- Frontend -->
