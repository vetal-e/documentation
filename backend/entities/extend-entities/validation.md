<a id="book-entities-extended-entities-validation-for-fields"></a>

# Validation for Extended Fields

By default, all extended fields are not validated. In general, extended fields are rendered as usual forms, the same way as not extended, but there is a way to define validation constraints for all extended fields by their type.

This is done through the configuration of `oro_entity_extend.validation_loader`:

```yaml
oro_entity_extend.validation_loader:
    class: Oro\Bundle\EntityExtendBundle\Validator\ExtendFieldValidationLoader
    arguments:
        - '@oro_entity_config.provider.extend'
        - '@oro_entity_config.provider.form'
    calls:
        -
            - addConstraints
            -
                - integer
                -
                    - Regex:
                        pattern: '^(-?[1-9]\d*|0)$'
                        message: 'This value should contain only numbers.'

        - [addConstraints, ['percent', [{ Type: {type: 'numeric'} }]]]
```

There are two ways to pass the constraints:

* use a compiler pass to add the ‘addConstraints’ call with the necessary constraint configuration
* directly call the service

For example:

```php
namespace Acme\Bundle\DemoBundle\DependencyInjection\Compiler;

use Symfony\Component\DependencyInjection\Compiler\CompilerPassInterface;
use Symfony\Component\DependencyInjection\ContainerBuilder;

class AcmeExtendValidationPass implements CompilerPassInterface
{
    private const INTEGER_CONSTRAINT = [
        'Regex' => [
            'pattern' => '/^(-?[1-9]\d*|0)$/',
            'message' => 'This value should contain only numbers!'
        ]
    ];

    #[\Override]
    public function process(ContainerBuilder $container): void
    {
        if (!$container->has('oro_entity_extend.validation_loader')) {
            return;
        }

        $definition = $container->findDefinition('oro_entity_extend.validation_loader');
        $definition->addMethodCall(
            'addConstraints',
            ['integer', [self::INTEGER_CONSTRAINT]]
        );
    }
```

Make sure to insert CompilerPass to the bundle root file.

```php

namespace Acme\Bundle\DemoBundle;
use Acme\Bundle\DemoBundle\DependencyInjection\Compiler\AcmeExtendValidationPass;
use Oro\Bundle\DataAuditBundle\Model\AuditFieldTypeRegistry;
use Symfony\Component\DependencyInjection\ContainerBuilder;
use Symfony\Component\HttpKernel\Bundle\Bundle;

class AcmeDemoBundle extends Bundle
{
    #[\Override]
    public function build(ContainerBuilder $container): void
    {
        parent::build($container);

        $container->addCompilerPass(new AcmeExtendValidationPass());
    }
}
```

Keep in mind that all constraints defined here are applied to all extended fields with a corresponding type.

<!-- Frontend -->
