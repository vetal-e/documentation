<a id="book-entities-extended-entities-custom-form-type-for-fields"></a>

# Define Custom Form Type for Fields

Extended fields are rendered as HTML controls, and control type (text, textarea, number, checkbox, etc.) is defined by
classes that implement <a href="https://github.com/symfony/symfony/blob/6.4/src/Symfony/Component/Form/FormTypeGuesserInterface.php" target="_blank">Symfony\\Component\\Form\\FormTypeGuesserInterface</a>.

In case of extended fields, OroPlatform has three guessers (in decreasing priority): <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/EntityBundle/Form/Guesser/FormConfigGuesser.php" target="_blank">FormConfigGuesser</a>, <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/EntityExtendBundle/Form/Guesser/ExtendFieldTypeGuesser.php" target="_blank">ExtendFieldTypeGuesser</a> and <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/EntityBundle/Form/Guesser/DoctrineTypeGuesser.php" target="_blank">DoctrineTypeGuesser</a>.

Each provides guesses, and the best guess is selected based on the guesser’s confidence (low, medium, high, very high).

There are a few ways to define a custom form type and form options for a particular extended field:

1. Through the compiler pass to add or override the guesser’s mappings:
   > ```php
   > namespace Acme\Bundle\DemoBundle\DependencyInjection\Compiler;

   > use Symfony\Component\DependencyInjection\Compiler\CompilerPassInterface;
   > use Symfony\Component\DependencyInjection\ContainerBuilder;
   > use Symfony\Component\Form\Extension\Core\Type\TimeType;

   > class AcmeExtendGuesserPass implements CompilerPassInterface
   > {
   >     #[\Override]
   >     public function process(ContainerBuilder $container)
   >     {
   >         $guesser = $container->findDefinition('oro_entity_extend.provider.extend_field_form_type');
   >         $guesser->addMethodCall(
   >             'addExtendTypeMapping',
   >             ['time', TimeType::class, ['model_timezone' => "UTC", 'view_timezone' => "UTC"]]
   >         );
   >     }
   > }
   > ```
2. With a custom form extend field options provider that can be used for providing form options that need a complex logic and cannot be declared in compiler pass:
   > ```php
   > namespace Acme\Bundle\DemoBundle\Provider;

   > use Oro\Bundle\EntityExtendBundle\Provider\ExtendFieldFormOptionsProviderInterface;

   > class ExtendFieldCustomFormOptionsProvider implements ExtendFieldFormOptionsProviderInterface
   > {
   >     #[\Override]
   >     public function getOptions(string $className, string $fieldName): array
   >     {
   >         $options = [];
   >         if ($className === '...' && $fieldName === '...') {
   >             $options['custom_option'] = 'custom_value';
   >         }

   >         return $options;
   >     }
   > }
   > ```

   > Register it in the dependency injection container:
   > ```yaml
   > services:
   >     acme.provider.extend_field_form_options:
   >       class: Acme\Bundle\DemoBundle\Provider\ExtendFieldCustomFormOptionsProvider
   >       tags:
   >         - { name: acme_entity_extend.form_options_provider }
   > ```
3. With a custom guesser that will have higher priority or will provide a guess with the highest confidence value:
   > ```php
   > namespace Acme\Bundle\DemoBundle\Form\Guesser;

   > use Symfony\Component\Form\FormTypeGuesserInterface;
   > use Symfony\Component\Form\Guess\TypeGuess;
   > use Symfony\Component\Form\Guess\ValueGuess;

   > class CustomTypeGuesser implements FormTypeGuesserInterface
   > {
   >     #[\Override]
   >     public function guessType(string $class, string $property)
   >     {
   >         // some conditions here
   >         if ($class === '...' && $property === '') {
   >             $guessedType = '';
   >             $options     = [];
   >             return new TypeGuess($guessedType, $options, TypeGuess::HIGH_CONFIDENCE);
   >         }

   >         // not guessed
   >         return new ValueGuess(false, ValueGuess::LOW_CONFIDENCE);
   >     }

   >     #[\Override]
   >     public function guessRequired(string $class, string $property)
   >     {
   >         return new ValueGuess(false, ValueGuess::LOW_CONFIDENCE);
   >     }

   >     #[\Override]
   >     public function guessMaxLength(string $class, string $property)
   >     {
   >         return new ValueGuess(null, ValueGuess::LOW_CONFIDENCE);
   >     }

   >     #[\Override]
   >     public function guessPattern(string $class, string $property)
   >     {
   >         return new ValueGuess(null, ValueGuess::LOW_CONFIDENCE);
   >     }
   > }
   > ```

   > Register it in the dependency injection container:
   > ```yaml
   > acme.form.guesser.extend_field:
   >     class: Acme\Bundle\DemoBundle\Form\Guesser\CustomTypeGuesser
   >     tags:
   >         - { name: form.type_guesser, priority: N }
   > ```

   > Here is an idea of what N should be, the existing guessers have the following priorities:

   > | Guesser                                                                                                                                                                       |   Priority |
   > |-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|------------|
   > | <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/EntityBundle/Form/Guesser/FormConfigGuesser.php" target="_blank">FormConfigGuesser</a>                 |         20 |
   > | <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/EntityExtendBundle/Form/Guesser/ExtendFieldTypeGuesser.php" target="_blank">ExtendFieldTypeGuesser</a> |         15 |
   > | <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/EntityBundle/Form/Guesser/DoctrineTypeGuesser.php" target="_blank">DoctrineTypeGuesser</a>             |         10 |

   > Select it according to what you need to achieve.
4. Using attribute to a field or a related entity (if an extended field is an association)
   > ```php
   > #[Config(
   >     defaultValues: [
   >         ...
   >         'form' => [
   >             'form_type' => 'Oro\Bundle\UserBundle\Form\Type\UserSelectType',
   >             'form_option' => ['option1' => ..., ...]
   >         ]
   >     ]
   > )]
   > ```

<!-- Frontend -->
