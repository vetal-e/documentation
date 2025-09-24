<a id="dev-entities-entity-class-name-provider"></a>

# Entity Class Name Provider

This service aims to provide human-readable representation in **English** of an entity class name. OroPlatform uses this provider to generate a description of REST API resources that are generated on the fly. See <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/EntityBundle/Routing/DictionaryEntityApiDocHandler.php" target="_blank">DictionaryEntityApiDocHandler</a> for details.

**Interface of an entity class name provider**

The <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/EntityBundle/Provider/ChainEntityClassNameProvider.php" target="_blank">entity class name provider</a> service is a “chain” service. It works by asking a set of prioritized providers to get a human-readable representation of an entity class name. Each child service must implement <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/EntityBundle/Provider/EntityClassNameProviderInterface.php" target="_blank">EntityClassNameProviderInterface</a>. This interface declares the following methods:

- *getEntityClassName* - returns a human-readable representation for an entity class.
- *getEntityClassPluralName* - returns a human-readable representation in plural for an entity class.

**Create custom entity class name provider**

To create own provider just create a class implementing <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/EntityBundle/Provider/EntityClassNameProviderInterface.php" target="_blank">EntityClassNameProviderInterface</a> and register it in DI container with the tag **oro_entity.class_name_provider**. You can also use the existing <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/EntityBundle/Provider/AbstractEntityClassNameProvider.php" target="_blank">abstract provider</a> as a superclass for your provider.

#### NOTE
src/Acme/Bundle/DemoBundle/Provider/AcmeClassNameProvider.php
```php
namespace Acme\Bundle\DemoBundle\Provider;

use Acme\Bundle\DemoBundle\Entity\Priority;
use Oro\Bundle\EntityBundle\Provider\AbstractEntityClassNameProvider;
use Oro\Bundle\EntityBundle\Provider\EntityClassNameProviderInterface;

/**
 * The default implementation of a service to get human-readable names in English of entity classes.
 */
class AcmeClassNameProvider extends AbstractEntityClassNameProvider implements EntityClassNameProviderInterface
{
    #[\Override]
    public function getEntityClassName(string $entityClass): ?string
    {
        // add your implementation here
        if ($entityClass !== Priority::class) {
            return null;
        }

        return "PRIORITY";
    }

    #[\Override]
    public function getEntityClassPluralName(string $entityClass): ?string
    {
        // add your implementation here
        return $this->getName($entityClass, true);
    }
}
```

```yaml
services:
    acme_demo.priority_entity_name_provider:
        class: Acme\Bundle\DemoBundle\Provider\AcmeClassNameProvider
        arguments:
            - "@oro_entity_config.config_manager"
            - "@translator"
            - '@Doctrine\Inflector\Inflector'
        tags:
            - { name: oro_entity.class_name_provider, priority: 100 }
```

You can specify the priority to move the provider up or down the provider’s chain. The bigger the priority number is, the earlier the provider will be executed. The priority value is optional and defaults to 0.

<!-- Frontend -->
