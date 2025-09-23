<a id="web-api-configuration-extra"></a>

# Configuration Extras

The configuration extras are the way to get varying configuration information.

There are two types of configuration extras:

- A configuration extra used to request additional configuration options for existing configuration sections. This extra is represented by  `Oro\Bundle\ApiBundle\Config\Extra\ConfigExtraInterface`.
- A configuration extra used to request additional configuration sections. This extra is represented by `Oro\Bundle\ApiBundle\Config\Extra\ConfigExtraSectionInterface`.

Both types of configuration extras work in the following way:

- The actions like [get](actions.md#get-action), [get_list](actions.md#get-list-action) or [delete](actions.md#delete-action) register the configuration extras in the [context](actions.md#web-api-context-class) using the `addConfigExtra` method. All required extras must be registered before any of the `getConfig`, `getConfigOf`, `getConfigOfFilters` or `getConfigOfSorters` methods of the Context is called. Typically the registration happens in processors from the `initialize` group. For example, <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Processor/Get/InitializeConfigExtras.php" target="_blank">InitializeConfigExtras</a>.
- When a processor needs a configuration, it calls the appropriate method of the [context](actions.md#web-api-context-class). For example `getConfig`, `getConfigOf`, `getConfigOfFilters` or `getConfigOfSorters`. The first call of any of these methods causes the loading of the configuration.
- The loading of the configuration is performed by the [get_config](actions.md#get-config-action) action. Any processors registered for this action can check which configuration data is requested. There are two ways a processor can find out which configuration data is requested. The first one is to use the [processor conditions](processors.md#web-api-processors). The second one is to use the `hasExtra` method of the  <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Processor/GetConfig/ConfigContext.php" target="_blank">ConfigContext</a>.

For more details on the config structure, sections, properties, etc., see the [Configuration Reference](configuration.md#web-api-configuration).

<a id="web-api-configuration-extra-configextrainterface"></a>

## ConfigExtraInterface

The <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Config/Extra/ConfigExtraInterface.php" target="_blank">ConfigExtraInterface</a> has the following methods:

- **getName** - Returns a string which is used as unique identifier of configuration data.
- **getCacheKeyPart** - Returns a string to add to a cache key used by the <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Provider/ConfigProvider.php" target="_blank">configuration providers</a>. In most cases, this method returns the same value as the `getName` method. However, more complicated extras can build the cache key part based on other properties, e.g., <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Config/Extra/MaxRelatedEntitiesConfigExtra.php" target="_blank">MaxRelatedEntitiesConfigExtra</a>.
- **configureContext** - Adds additional values into the <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Processor/GetConfig/ConfigContext.php" target="_blank">ConfigContext</a>. For example, the mentioned above <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Config/Extra/MaxRelatedEntitiesConfigExtra.php" target="_blank">MaxRelatedEntitiesConfigExtra</a> adds the maximum number of related entities into the context of the [get_config](actions.md#get-config-action) action, and this value is used by the <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Processor/GetConfig/SetMaxRelatedEntities.php" target="_blank">SetMaxRelatedEntities</a> processor to make necessary modifications to the configuration.
- **isPropagable** - Indicates whether this config extra should be used when a configuration of related entities is built. For example,  <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Config/Extra/DescriptionsConfigExtra.php" target="_blank">DescriptionsConfigExtra</a> is propagable; as a result, field value data transformers will be returned for the main entity and all related entities.

<a id="web-api-configuration-extra-configextrasectioninterface"></a>

## ConfigExtraSectionInterface

The <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Config/Extra/ConfigExtraSectionInterface.php" target="_blank">ConfigExtraSectionInterface</a> extends <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Config/Extra/ConfigExtraInterface.php" target="_blank">ConfigExtraInterface</a> and has one additional method:

- **getConfigType** - Returns the configuration type that should be loaded into the corresponding section. The <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Config/Loader/ConfigLoaderFactory.php" target="_blank">ConfigLoaderFactory</a> uses the return value of this method to find the appropriate loader.

There is a list of existing configuration extras that implement this interface:

- <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Config/Extra/FiltersConfigExtra.php" target="_blank">FiltersConfigExtra</a>
- <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Config/Extra/SortersConfigExtra.php" target="_blank">SortersConfigExtra</a>

<a id="web-api-configuration-extra-example"></a>

## Example of configuration extra

The <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Config/Extra/DescriptionsConfigExtra.php" target="_blank">DescriptionsConfigExtra</a> is used to request human-readable descriptions of entities and their fields:

```php
namespace Oro\Bundle\ApiBundle\Config;

use Oro\Bundle\ApiBundle\Processor\GetConfig\ConfigContext;

class DescriptionsConfigExtra implements ConfigExtraInterface
{
    public const NAME = 'descriptions';

    public function getName(): string
    {
        return self::NAME;
    }

    public function configureContext(ConfigContext $context): void
    {
        // no modifications of the ConfigContext are required
    }

    public function isPropagable(): bool
    {
        return false;
    }

    public function getCacheKeyPart(): ?string
    {
        return self::NAME;
    }
}
```

Usually, configuration extras are added to the context by the `InitializeConfigExtras` processors, which belong to the `initialize group`, e.g., the <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Processor/Get/InitializeConfigExtras.php" target="_blank">InitializeConfigExtras</a> processor for the `get` action. However, the API documentation requires human-readable descriptions. Therefore, <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Config/Extra/DescriptionsConfigExtra.php" target="_blank">DescriptionsConfigExtra</a> is added by <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/ApiDoc/AnnotationHandler/RestDocHandler.php" target="_blank">RestDocHandler</a>.

The <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Processor/GetConfig/CompleteDescriptions.php" target="_blank">CompleteDescriptions</a> processor adds entities, fields, and filter descriptions. This processor is registered as a service in <a href="https://github.com/oroinc/platform/tree/master/src/Oro/Bundle/ApiBundle/Resources/config/processors.get_config.yml" target="_blank">processors.get_config.yml</a>. Note that the processor tag contains the `extra` attribute with the `descriptions` value. This means that the processor will be executed only if the extra configuration (in this case `descriptions`) were requested. For more details, see [processor conditions](processors.md#web-api-processors).

<!-- Frontend -->
