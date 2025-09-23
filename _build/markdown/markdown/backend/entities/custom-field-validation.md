<a id="dev-entities-custom-field-validaton"></a>

# Custom Field Validation

Using the oro_entity.manager.entity_field_validator service, you can add custom field validation that you can place in your bundle.

Example:

```yaml
    # Validator
    acme_demo.validator.acme_custom_grid_field_validator:
        class: Acme\Bundle\DemoBundle\Validator\CustomGridFieldValidator
        arguments:
            - '@property_accessor'
        tags:
            - { name: oro_entity.custom_grid_field_validator, entity_name: Acme_Bundle_DemoBundle_Entity_Priority }
```

Each validator should implement `Oro\Bundle\EntityBundle\Entity\Manager\Field\CustomGridFieldValidatorInterface` and add a tag description.

The tag should contain name and entity_name:

* entity_name - should contain the entity name which will be performed. Use `str_replace('\\', '_', ClassUtils::getClass($entity))` transformation of the object to get the entity_name which could be written into the service tag block.

Example:

```php
namespace Acme\Bundle\DemoBundle\Validator;

use Acme\Bundle\DemoBundle\Entity\Priority;
use Doctrine\Common\Util\ClassUtils;
use Oro\Bundle\EntityBundle\Entity\Manager\Field\CustomGridFieldValidatorInterface;
use Oro\Bundle\EntityBundle\Exception\IncorrectEntityException;
use Symfony\Component\PropertyAccess\Exception\InvalidArgumentException;
use Symfony\Component\PropertyAccess\PropertyAccessorInterface;

/**
 * Validates priority entity fields to be editable inline in grid.
 */
class CustomGridFieldValidator implements CustomGridFieldValidatorInterface
{
    protected $accessor;

    public function __construct(
        PropertyAccessorInterface $accessor
    ) {
        $this->accessor = $accessor;
    }

    #[\Override]
    public function hasAccessEditField($entity, $fieldName): bool
    {
        if (!$entity instanceof Priority) {
            $className = ClassUtils::getClass($entity);
            throw new IncorrectEntityException(
                sprintf('Entity %s, is not instance of Priority class', $className)
            );
        }

        return $this->hasField($entity, $fieldName)
            && !in_array($fieldName, $this->getPriorityFieldBlockList(), true);
    }

    #[\Override]
    public function hasField($entity, $fieldName): bool
    {
        try {
            $this->accessor->isWritable($entity, $fieldName);
        } catch (InvalidArgumentException $e) {
            return false;
        }

        return true;
    }

    protected function getPriorityFieldBlockList(): array
    {
        return [
            'updated_at',
        ];
    }
}
```
