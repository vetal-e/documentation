<a id="web-api-post-processors"></a>

# Post Processors

A post-processor is a data transformer that converts a field value to a format suitable for the API.
Post-processors are used only in the [get](actions.md#get-action), [get_list](actions.md#get-list-action) and
[get_subresource](actions.md#get-subresource-action) actions.

The following table shows all post processors provided out-of-the-box:

| Name   | Description              | Options                                                |
|--------|--------------------------|--------------------------------------------------------|
| twig   | Applies a TWIG template. | **template** *string* - The name of the TWIG template. |

All post processors are registered in <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/PostProcessor/PostProcessorRegistry.php" target="_blank">PostProcessorRegistry</a>. You can use it when you need to get a specific post processor in your code.

<a id="web-api-post-processors-create"></a>

## Creating a New Post Processor

To create a new post processor, you need to do the following:

1. Create a class that implements <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/PostProcessor/PostProcessorInterface.php" target="_blank">PostProcessorInterface</a>.

```php
namespace Acme\Bundle\DemoBundle\Api\PostProcessor;

use Oro\Bundle\ApiBundle\PostProcessor\PostProcessorInterface;

class SomePostProcessor implements PostProcessorInterface
{
    #[\Override]
    public function process(mixed $value, array $options): mixed
    {
    }
}
```

1. Register the post-processor in the dependency injection container using the `oro.api.post_processor` tag
   with the `alias` attribute that contains a unique name of the post-processor:

```yaml
acme.api.post_processor.some:
    class: Acme\Bundle\DemoBundle\Api\PostProcessor\SomePostProcessor
    tags:
        - { name: oro.api.post_processor, alias: some }
```

1. Create a [config extension](configuration-extensions.md#web-api-configuration-extensions-create) if you need to validate
   the post-processor options.

```php
namespace Acme\Bundle\DemoBundle\Api\PostProcessor;

use Oro\Bundle\ApiBundle\Util\ConfigUtil;
use Symfony\Component\Config\Definition\Builder\NodeBuilder;

class SomePostProcessorConfigExtension extends AbstractConfigExtension
{
    #[\Override]
    public function getConfigureCallbacks(): array
    {
        return [
            'entities.entity.field'                 => function (NodeBuilder $node) {
                $this->addValidationOfPostProcessorOptions($node);
            },
            'actions.action.field'                  => function (NodeBuilder $node) {
                $this->addValidationOfPostProcessorOptions($node);
            },
            'subresources.subresource.action.field' => function (NodeBuilder $node) {
                $this->addValidationOfPostProcessorOptions($node);
            }
        ];
    }

    /**
     * @param NodeBuilder $node
     */
    private function addValidationOfPostProcessorOptions(NodeBuilder $node): void
    {
        $node->end()->validate()
            ->ifTrue(function ($v) {})
            ->thenInvalid();
    }
}
```

1. Register the config extension as a service in the dependency injection container.

```yaml
acme.api.config_extension.post_processor.some:
    class: Acme\Bundle\DemoBundle\Api\PostProcessor\SomePostProcessorConfigExtension
```

1. Register the config extension in Resources/config/oro/app.yml in your bundle
   or config/config.yml of your application.

```yaml
oro_api:
    config_extensions:
        - acme.api.config_extension.post_processor.some
```

<!-- Frontend -->
