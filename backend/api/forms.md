<a id="web-api-forms"></a>

# Forms and Validators Configuration

The Symfony <a href="https://symfony.com/doc/6.4/validation.html" target="_blank">Validation Component</a> and <a href="https://symfony.com/doc/6.4/forms.html" target="_blank">Forms Component</a> are used to validate and transform input data to an entity in [create](actions.md#create-action), [update](actions.md#update-action), [update_relationship](actions.md#update-relationship-action), [add_relationship](actions.md#add-relationship-action) and [delete_relationship](actions.md#delete-relationship-action) actions.

## Validation

The validation rules are loaded from Resources/config/validation.yml and annotations, as is commonly done in Symfony applications. So, all validation rules defined for an entity also apply to the API. API uses two validation groups: **Default** and **api**. If you need to add validation constraints that should be applied in the API only, add them to the **api** validation group.

Suppose a validation rule cannot be implemented as a regular validation constraint due to its complexity. In that case, you can implement it as a processor for the `post_validate` event of  the [customize_form_data](actions.md#customize-form-data-action) action. Keep in mind that <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Form/FormUtil.php" target="_blank">FormUtil class</a> contains methods that may be useful in such a processor.

If the input data violate validation constraints, they will be automatically converted into the [validation errors](processors.md#web-api-processors) that help build the correct response of the API. The conversion is performed by the <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Processor/Shared/CollectFormErrors.php" target="_blank">CollectFormErrors</a> processor. By default, the HTTP status code for validation errors is `400 Bad Request`. To change it, you can:

- Implement <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Validator/Constraints/ConstraintWithStatusCodeInterface.php" target="_blank">ConstraintWithStatusCodeInterface</a> in you constraint class.
- Implement your own constraint text extractor. The API bundle has the <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Request/ConstraintTextExtractor.php" target="_blank">default implementation of constraint text extractor</a>. To add a new extractor, create a class implements <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Request/ConstraintTextExtractorInterface.php" target="_blank">ConstraintTextExtractorInterface</a> and tag it with the `oro.api.constraint_text_extractor` in the dependency injection container. This service can also be used to change an error code and type for a validation constraint.

The following example shows how to add validation constraints to the API resources using the Resources/config/oro/api.yml configuration file:

```yaml
api:
    entities:
        Acme\Bundle\DemoBundle\Entity\SomeEntity:
            fields:
                primaryEmail:
                    form_options:
                        constraints:
                            # add Symfony\Component\Validator\Constraints\Email validation constraint
                            - Email: ~
                userName:
                    form_options:
                        constraints:
                            # add Symfony\Component\Validator\Constraints\Length validation constraint
                            - Length:
                                max: 50
                            # add Acme\Bundle\DemoBundle\Validator\Constraints\Alphanumeric validation constraint
                            - Acme\Bundle\DemoBundle\Validator\Constraints\Alphanumeric: ~
```

Also, see how to [validate virtual fields](how-to.md#validate-virtual-fields).

In case you need to replace an error title returned by the API with another error title,
use the `error_title_overrides` configuration section in Resources/config/oro/app.yml in any bundle
or config/config.yml of your application, e.g.:

```yaml
oro_api:
    error_title_overrides:
        'percent range constraint': 'range constraint'
```

## Forms

The API forms are isolated from the UI forms. This helps avoid collisions and prevent unnecessary performance overhead in the API. Consequently, all the API form types, extensions, and guessers should be registered separately. There are two ways to complete this:

- Use the application configuration file.
- Tag the form elements with appropriate tags in the dependency injection container.

To register new form elements using the application configuration file, add Resources/config/oro/app.yml in any bundle or use config/config.yml of your application.

```yaml
oro_api:
    form_types:
        - Symfony\Component\Form\Extension\Core\Type\DateType # the class name of a form type
        - form.type.date # the service id of a form type
    form_type_extensions:
        - form.type_extension.form.http_foundation # service id of a form type extension
    form_type_guessers:
        - acme.form.type_guesser # service id of a form type guesser
    form_type_guesses:
        datetime: # data type
            form_type: Symfony\Component\Form\Extension\Core\Type\DateTimeType # the guessed form type
            options: # guessed form type options
                model_timezone: UTC
                view_timezone: UTC
                with_seconds: true
                widget: single_text
                format: "yyyy-MM-dd'T'HH:mm:ssZZZZZ" # HTML5
```

#### NOTE
The form_types section can contain either the class name or the service id of a form type. Usually, the service id is used if a form type depends on other services in the dependency injection container.

You can find the already registered API form elements in <a href="https://github.com/oroinc/platform/tree/master/src/Oro/Bundle/ApiBundle/Resources/config/oro/app.yml" target="_blank">Resources/config/oro/app.yml</a>.

If you need to add new form elements can by tagging them in the dependency injection container, use tags from the following table:

| Tag                         | Description                                 |
|-----------------------------|---------------------------------------------|
| oro.api.form.type           | Create a new form type                      |
| oro.api.form.type_extension | Create a new form extension                 |
| oro.api.form.type_guesser   | Add your own logic for “form type guessing” |

Example:

```yaml
acme.form.type.datetime:
    class: Acme\Bundle\DemoBundle\Form\Type\DateTimeType
    tags:
        - { name: form.type, alias: acme_datetime } # allow to use the form type on UI
        - { name: oro.api.form.type, alias: acme_datetime } # allow to use the form type in API

acme.form.extension.datetime:
    class: Acme\Bundle\DemoBundle\Form\Extension\DateTimeExtension
    tags:
        - { name: form.type_extension, alias: acme_datetime } # add the form extension to UI forms
        - { name: oro.api.form.type_extension, alias: acme_datetime } # add the form extension to API forms

acme.form.guesser.test:
    class: Acme\Bundle\DemoBundle\Form\Guesser\TestGuesser
    tags:
        - { name: form.type_guesser } # add the form type guesser to UI forms
        - { name: oro.api.form.type_guesser } # add the form type guesser to API forms
```

To switch between the general and API forms, use the <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Processor/Shared/InitializeApiFormExtension.php" target="_blank">Processor\\Shared\\InitializeApiFormExtension</a> and <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Processor/Shared/RestoreDefaultFormExtension.php" target="_blank">Processor\\Shared\\RestoreDefaultFormExtension</a> processors.

The <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Processor/Shared/BuildFormBuilder.php" target="_blank">Processor\\Shared\\BuildFormBuilder</a> processor builds the form for a particular entity on the fly based on the [API configuration](configuration.md#web-api-configuration) and the entity metadata.

<!-- Frontend -->
