<a id="book-entities-entity-configuration"></a>

# Configure Entities

So far, Doctrine offers a wide range of functionality to map your entities to the database, save your data, and retrieve them from the database. However, in an application based on the OroPlatform, you usually want to control how entities are presented to the user. OroPlatform includes the <a href="https://github.com/oroinc/platform/tree/master/src/Oro/Bundle/EntityConfigBundle" target="_blank">EntityConfigBundle</a> that makes it easy to configure additional metadata of your entities, as well as the fields of your entities. For example, you can now configure icons and labels used when showing an entity in the UI, or you can set up access levels to control how entities can be viewed and modified.

## Configure Entities and Their Fields

Entities will not be configurable by default. They must be tagged as configurable entities to let the system apply entity config options to them:

* The #[Config] attribute is used to enable entity-level configuration for an entity.
* Use the #[ConfigField] attribute to enable config options for selected fields.

#### NOTE
The bundles from OroPlatform offer a large set of predefined options that you can use in your entities to configure them and control their behavior. Take a look at the entity_config.yml files that can be found in many bundles and read their dedicated documentation.

### The `#[Config]` Attribute

To make the `Document` entity from the first part of the chapter configurable, import the `Oro\Bundle\EntityConfigBundle\Metadata\Attribute\Config` attribute and use it:

#### NOTE
src/Acme/Bundle/DemoBundle/Entity/Document.php
```php
 namespace Acme\Bundle\DemoBundle\Entity;

 use Doctrine\ORM\Mapping as ORM;
 use Oro\Bundle\EntityConfigBundle\Metadata\Attribute\Config;

 #[ORM\Entity]
 #[ORM\Table(name: 'acme_demo_document')]
 #[Config]
 class Document
 {
     // ...
 }
```

You can also change the default value of each configurable option using the `defaultValues` argument:

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
        'dataaudit' => ['auditable' => true],
    ]
)]
```

### The `#[ConfigField]` Attribute

Similar to the `#[Config]` attribute for entities, you can use the `Oro\Bundle\EntityConfigBundle\Metadata\Attribute\ConfigField` attribute to make properties of an entity configurable. Default values can be changed the same way as for the entity level:

```php
    // ...
    #[ORM\Column(name: 'subject', type: 'string', length: 255, nullable: false)]
    #[ConfigField(defaultValues: ['dataaudit' => ['auditable' => true], 'importexport' => ['identity' => true]])]
    private $subject;
    // ...
```

## Console Commands

### Update Configuration Data

To update configurable entities, use the following:

```bash
php bin/console oro:entity-config:update
```

Execute this command only in the ‘dev’ mode when a new configuration attribute or the whole configuration scope is added.

### Clearing Up Cache

To remove all data related to configurable entities from the application cache, use:

```none
php bin/console oro:entity-config:cache:clear
```

To skip warming up the cache after cleaning, use the `--no-warmup` command:

```none
php bin/console oro:entity-config:cache:clear --no-warmup
```

### Warming Up the Cache

To warm up the entity config cache, use the `oro:entity-config:cache:warmup` command:

```none
php bin/console oro:entity-config:cache:warmup
```

### Debugging Configuration Data

To get a different type of configuration data, add/remove/update configuration of entities, use the `oro:entity-config:debug` command. To see all available options, run this command with the `--help` option.

The example shows all configuration data for the User entity:

```none
php bin/console oro:entity-config:debug "Acme\Bundle\DemoBundle\Entity\Document"
```

#### NOTE
Check out the Attributes topic to learn how to assign functionality to an entity to [create and manipulate attributes](../attributes.md#dev-entities-attributes).

* [Define a New Object Configuration Attribute](configure-entity-config-attribute.md)
* [Implementation](implementation.md)
* [Add Configuration Options](add-configuration-options.md)
* [Access Entities Configuration](access-entities-configuration.md)

<!-- Frontend -->
