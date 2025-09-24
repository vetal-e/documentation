<a id="web-api-configuration-extensions"></a>

# Configuration Extensions

Configuration extensions help add new options to existing configuration sections and new configuration sections.

<a id="web-api-configuration-extensions-create"></a>

## Creating a Configuration Extension

Each configuration extension must implement <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Config/Extension/ConfigExtensionInterface.php" target="_blank">ConfigExtensionInterface</a> (you can also use <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Config/Extension/AbstractConfigExtension.php" target="_blank">AbstractConfigExtension</a> as a superclass). To register a new configuration extension, add it to Resources/config/oro/app.yml in your bundle or use config/config.yml of your application. Here is an example:

```php
namespace Acme\Bundle\DemoBundle\Api;

use Oro\Bundle\ApiBundle\Config\Extension\AbstractConfigExtension;

class MyConfigExtension extends AbstractConfigExtension
{
}
```

#### NOTE
config/config.yml
```yaml
services:
    acme.api.my_config_extension:
        class: Acme\Bundle\DemoBundle\Api\MyConfigExtension
        public: false

oro_api:
    config_extensions:
        - acme.api.my_config_extension
```

<a id="web-api-configuration-extensions-add-options"></a>

## Add Options to an Existing Configuration Section

To add options to an existing configuration section, implement the `getConfigureCallbacks` method of <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Config/Extension/ConfigExtensionInterface.php" target="_blank">ConfigExtensionInterface</a>. If you need to add logic before the normalization during the configuration validation, implement the `getPreProcessCallbacks` and `getPostProcessCallbacks` methods.

The following table describes existing sections to which you can add new options.

| Section Name                          | When to use                                                                                                                             |
|---------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------|
| entities.entity                       | Add entity options                                                                                                                      |
| entities.entity.field                 | Add field options                                                                                                                       |
| filters                               | Add options to `filters` section                                                                                                        |
| filters.field                         | Add filter options                                                                                                                      |
| sorters                               | Add options to `sorters` section                                                                                                        |
| sorters.field                         | Add sorter options                                                                                                                      |
| actions.action                        | Add action options                                                                                                                      |
| actions.action.status_code            | Add response status code options                                                                                                        |
| actions.action.field                  | Add field options specific for a particular action. These options override options defined in `entities.entity.field`                   |
| subresources.subresource              | Add sub-resource options                                                                                                                |
| subresources.subresource.action       | Add sub-resource action options                                                                                                         |
| subresources.subresource.action.field | Add field options specific for a particular action of a sub-resource. These options override options defined in `entities.entity.field` |

Example:

```php
namespace Acme\Bundle\DemoBundle\Api;

use Symfony\Component\Config\Definition\Builder\NodeBuilder;
use Oro\Bundle\ApiBundle\Config\Extension\AbstractConfigExtension;

class MyConfigExtension extends AbstractConfigExtension
{
    #[\Override]
    public function getConfigureCallbacks(): array
    {
        return [
            'entities.entity' => function (NodeBuilder $node) {
                $node->scalarNode('some_option');
            }
        ];
    }

    #[\Override]
    public function getPreProcessCallbacks(): array
    {
        return [
            'entities.entity' => function (array $config) {
                // do something
                return $config;
            }
        ];
    }

    #[\Override]
    public function getPostProcessCallbacks(): array
    {
        return [
            'entities.entity' => function (array $config) {
                // do something
                return $config;
            }
        ];
    }
}
```

<a id="web-api-configuration-extensions-add-section"></a>

## Add New Configuration Section

To add new configuration section,  create a class implements <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Config/Definition/ConfigurationSectionInterface.php" target="_blank">ConfigurationSectionInterface</a> and return instance of it in the `getEntityConfigurationSections` method of your configuration extension.

By default, the configuration is returned as an array, but if you want to provide a class that represents the configuration of your section, you can implement a configuration loader. The loader is a class that implements <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Config/Loader/ConfigLoaderInterface.php" target="_blank">ConfigLoaderInterface</a>. An instance of the loader should be returned by the `getEntityConfigurationLoaders` method of your configuration extension.

An example of a simple configuration section:

```php
namespace Acme\Bundle\DemoBundle\Api\Config\Definition;

use Oro\Bundle\ApiBundle\Config\Definition\AbstractConfigurationSection;
use Symfony\Component\Config\Definition\Builder\NodeBuilder;

class MyConfiguration extends AbstractConfigurationSection
{
    #[\Override]
    public function configure(NodeBuilder $node): void
    {
        $node->scalarNode('some_option');
    }
}
```

An example of a configuration section that other bundles can extend:

```php
namespace Acme\Bundle\DemoBundle\Api\Config\Definition;

use Oro\Bundle\ApiBundle\Config\Definition\AbstractConfigurationSection;
use Symfony\Component\Config\Definition\Builder\ArrayNodeDefinition;
use Symfony\Component\Config\Definition\Builder\NodeBuilder;

class MyConfiguration extends AbstractConfigurationSection
{
    #[\Override]
    public function configure(NodeBuilder $node): void
    {
        $sectionName = 'my_section';

        /** @var ArrayNodeDefinition $parentNode */
        $parentNode = $node->end();
        $this->callConfigureCallbacks($node, $sectionName);
        $this->addPreProcessCallbacks($parentNode, $sectionName);
        $this->addPostProcessCallbacks($parentNode, $sectionName);

        $node->scalarNode('some_option');
    }
}
```

An example of a configuration section loader:

```php
namespace Acme\Bundle\DemoBundle\Api\Config\Loader;

use Acme\Bundle\DemoBundle\Api\Config\MyConfigSection;
use Oro\Bundle\ApiBundle\Config\Loader\AbstractConfigLoader;

class MyConfigLoader extends AbstractConfigLoader
{
    #[\Override]
    public function load(array $config): mixed
    {
        $result = new MyConfigSection();
        foreach ($config as $key => $value) {
            $this->loadConfigValue($result, $key, $value);
        }

        return $result;
    }
}
```

An example of a configuration extension:

```php
namespace Acme\Bundle\DemoBundle\Api\Config\Extension;

use Acme\Bundle\DemoBundle\Api\Config\Definition\MyConfiguration;
use Acme\Bundle\DemoBundle\Api\Config\Loader\MyConfigLoader;
use Oro\Bundle\ApiBundle\Config\Extension\AbstractConfigExtension;

class MyConfigExtension extends AbstractConfigExtension
{
    #[\Override]
    public function getEntityConfigurationSections(): array
    {
        return ['my_section' => new MyConfiguration()];
    }

    #[\Override]
    public function getEntityConfigurationLoaders(): array
    {
        return ['my_section' => new MyConfigLoader()];
    }
}
```

An example of how to use the created configuration section:

```yaml
api:
    entities:
        Acme\Bundle\DemoBundle\Entity\SomeEntity:
            my_section:
                my_option: value
```

To check that your configuration section is added correctly, run `php bin/console oro:api:config:dump-reference`. The output should look similar to the following:

```yaml
api:
    entities:
        name:
            my_section:
                my_option: ~
```

<!-- Frontend -->
