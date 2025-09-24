<a id="dev-cookbook-framework-create-simple-crud"></a>

# CRUD Operations

To let the users create new questions and edit existing ones, follow these steps:

1. [Create a form type for the Question entity](#cookbook-entity-form-type).
2. [Create a controller](#cookbook-entity-controller).
3. [Render the form in a template](#cookbook-entity-template).
4. [Link data grid entries to the controller](#controller-entity-grid-create-edit-link).

<a id="cookbook-entity-form-type"></a>

## The Form Type

First, you need to create a form type that makes it possible to let the user enter all the data
needed to describe a question:

#### NOTE
src/Acme/Bundle/DemoBundle/Form/Type/QuestionType.php
```php
namespace Acme\Bundle\DemoBundle\Form\Type;

use Acme\Bundle\DemoBundle\Entity\Question;
use Oro\Bundle\FormBundle\Form\Type\OroDateTimeType;
use Symfony\Component\Form\AbstractType;
use Symfony\Component\Form\Extension\Core\Type\TextType;
use Symfony\Component\Form\FormBuilderInterface;
use Symfony\Component\OptionsResolver\OptionsResolver;

/**
 * Form type for Question entity.
 */
class QuestionType extends AbstractType
{
    #[\Override]
    public function buildForm(FormBuilderInterface $builder, array $options): void
    {
        $builder
            ->add(
                'subject',
                TextType::class,
                ['label' => 'acme.demo.question.subject.label', 'required' => true]
            )
            ->add(
                'description',
                TextType::class,
                ['label' => 'acme.demo.question.description.label', 'required' => true]
            )
            ->add(
                'dueDate',
                OroDateTimeType::class,
                ['label' => 'acme.demo.question.due_date.label', 'required' => false]
            )
            ->add(
                'priority',
                PriorityCreateOrSelectType::class,
                ['label' => 'acme.demo.question.priority.label', 'required' => false]
            );
    }

    #[\Override]
    public function configureOptions(OptionsResolver $resolver): void
    {
        $resolver->setDefaults([
            'data_class' => Question::class
        ]);
    }
}
```

#### SEE ALSO
Learn more about <a href="https://symfony.com/doc/6.4/book/forms.html" target="_blank">form types in the Symfony documentation</a>.

<a id="cookbook-entity-controller"></a>

## The Controllers

You then need to create a controller class that comes with two actions: one that is called when a new question should be created and one that can fetch an existing question to let the user modify its data:

```php
namespace Acme\Bundle\DemoBundle\Controller;

use Acme\Bundle\DemoBundle\Entity\Question;
use Acme\Bundle\DemoBundle\Form\Type\QuestionType;
use Oro\Bundle\FormBundle\Model\UpdateHandlerFacade;
use Oro\Bundle\SecurityBundle\Attribute\Acl;
use Oro\Bundle\SecurityBundle\Attribute\AclAncestor;
use Symfony\Bridge\Twig\Attribute\Template;
use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\HttpFoundation\RedirectResponse;
use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\Routing\Attribute\Route;
use Symfony\Contracts\Translation\TranslatorInterface;

/**
 * Contains CRUD actions for Question
 */
#[Route(path: '/question', name: 'acme_demo_question_')]
class QuestionController extends AbstractController
{
    #[Route(path: '/', name: 'index')]
    #[Template('@AcmeDemo/Question/index.html.twig')]
    #[AclAncestor('acme_demo_question_view')]
    public function indexAction(): array
    {
        return [
            'entity_class' => Question::class
        ];
    }

    #[Route(path: '/view/{id}', name: 'view', requirements: ['id' => '\d+'])]
    #[Template('@AcmeDemo/Question/view.html.twig')]
    #[Acl(
        id: 'acme_demo_question_view',
        type: 'entity',
        class: 'Acme\Bundle\DemoBundle\Entity\Question',
        permission: 'VIEW'
    )]
    public function viewAction(Question $entity): array
    {
        return [
            'entity' => $entity,
        ];
    }

    /**
     * Create Question
     */
    #[Route(path: '/create', name: 'create', options: ['expose' => true])]
    #[Template('@AcmeDemo/Question/update.html.twig')]
    #[Acl(
        id: 'acme_demo_question_create',
        type: 'entity',
        class: 'Acme\Bundle\DemoBundle\Entity\Question',
        permission: 'CREATE'
    )]
    public function createAction(Request $request): array|RedirectResponse
    {
        $createMessage = $this->container->get(TranslatorInterface::class)->trans(
            'acme.demo.controller.question.saved.message'
        );

        return $this->update(new Question(), $request, $createMessage);
    }

    /**
     * Edit Question form
     */
    #[Route(path: '/update/{id}', name: 'update', requirements: ['id' => '\d+'])]
    #[Template('@AcmeDemo/Question/update.html.twig')]
    #[Acl(
        id: 'acme_demo_question_update',
        type: 'entity',
        class: 'Acme\Bundle\DemoBundle\Entity\Question',
        permission: 'EDIT'
    )]
    public function updateAction(Question $entity, Request $request): array|RedirectResponse
    {
        $updateMessage = $this->container->get(TranslatorInterface::class)->trans(
            'acme.demo.controller.question.saved.message'
        );

        return $this->update($entity, $request, $updateMessage);
    }

    protected function update(
        Question $entity,
        Request $request,
        string $message = ''
    ): array|RedirectResponse {
        return $this->container->get(UpdateHandlerFacade::class)->update(
            $entity,
            $this->createForm(QuestionType::class, $entity),
            $message,
            $request,
            null
        );
    }

    #[\Override]
    public static function getSubscribedServices(): array
    {
        return array_merge(
            parent::getSubscribedServices(),
            [
                TranslatorInterface::class,
                UpdateHandlerFacade::class,
            ]
        );
    }
}
```

Then, make sure that the controller is loaded in your routing configuration so that Symfony knows
which controller needs to be called for particular routes:

```yaml
acme_demo:
    resource: "@AcmeDemoBundle/Controller"
    type: attribute
    prefix: /demo
```

<a id="cookbook-entity-template"></a>

## The Template

The template that is responsible for displaying the form fields should extend the base template `@OroUI/actions/update.html.twig` from the OroUIBundle. This template defines some basic blocks that you can use. This way, your own forms will provide the same look and feel as the ones coming with OroPlatform:

```html
{% extends '@OroUI/actions/update.html.twig' %}

{% oro_title_set({params : {'%title%': entity|oro_format_name|default('N/A'|trans), '%entityName%': 'acme.demo.question.entity_label'|trans} }) %}

{# choose the appropriate action depending on whether a task is created or modified #}
{# this variable needs to be named formAction as this is what the base template expects #}
{% set formAction = form.vars.value.id ? path('acme_demo_question_update', { 'id': form.vars.value.id }) : path('acme_demo_question_create')  %}

{% block navButtons %}
    {% import '@OroUI/macros.html.twig' as UI %}

    {{ parent() }}

    {# the cancelButton() macro creates a button that discards the entered data and leads
           the user to the linked controller #}
    {{ UI.cancelButton(path('acme_demo_question_index')) }}
    {% set html = UI.saveAndCloseButton({
        'route': 'acme_demo_question_view',
        'params': {'id': '$id'}
    }) %}
    {% if is_granted('acme_demo_question_create') %}
        {% set html = html ~ UI.saveAndNewButton({
            'route': 'acme_demo_question_create'
        }) %}
    {% endif %}
    {% if entity.id or is_granted('acme_demo_question_update') %}
        {% set html = html ~ UI.saveAndStayButton({
            'route': 'acme_demo_question_update',
            'params': {'id': '$id'}
        }) %}
    {% endif %}
    {{ UI.dropdownSaveButton({'html': html}) }}
{% endblock %}

{% block pageHeader %}
    {% if entity.id %}
        {% set breadcrumbs = {
            'entity':      entity,
            'indexPath':   path('acme_demo_question_index'),
            'indexLabel': 'acme.demo.question.entity_plural_label'|trans,
            'entityTitle': entity|oro_format_name|default('N/A'|trans)
        } %}
        {{ parent() }}
    {% else %}
        {% set title = 'oro.ui.create_entity'|trans({'%entityName%': 'acme.demo.question.entity_label'|trans}) %}
        {% include '@OroUI/page_title_block.html.twig' with { title: title } %}
    {% endif %}
{% endblock pageHeader %}

{% block stats %}
    <li>{{ 'oro.ui.created_at'|trans }}: {{ entity.createdAt ? entity.createdAt|oro_format_datetime : 'N/A' }}</li>
    <li>{{ 'oro.ui.updated_at'|trans }}: {{ entity.updatedAt ? entity.updatedAt|oro_format_datetime : 'N/A' }}</li>
{% endblock stats %}

{% block content_data %}
    {% set dataBlocks = [
        {
            'title': 'acme.demo.section.question.general.label'|trans,
            'subblocks': [
                {
                    'data' : [
                        form_row(form.subject),
                        form_row(form.description),
                        form_row(form.dueDate),
                        form_row(form.priority),
                    ]
                }
            ]
        },
    ] %}

    {% set dataBlocks = dataBlocks|merge(oro_form_additional_data(form, 'Additional'|trans)) %}

    {# the data variable is a special variable that is used in the parent content_data block
           to render the visual content "blocks" of a page #}
    {% set data = {
        'formErrors': form_errors(form),
        'dataBlocks': dataBlocks
    }%}

    <div class="responsive-form-inner">
        {% set id = 'acme-demo-question-edit' %}
        {{ parent() }}
    </div>
{% endblock content_data %}
```

<a id="controller-entity-grid-create-edit-link"></a>

## Linking the Data Grid

Finally, link both actions on the page that displays the list of questions:

**1. Add a link to create new questions**

The base `@OroUI/actions/index.html.twig` template from the OroUIBundle that you [already used](data-grids/index.md#cookbook-entities-grid-controller) to embed the data grid comes with a pre-defined `navButtons` block, which you can use to add a button that links to the *create a question action*:

```html
{% extends '@OroUI/actions/index.html.twig' %}
{% import '@OroUI/macros.html.twig' as UI %}
{% set gridName = 'acme-demo-question-grid' %}
{% set pageTitle = 'acme.demo.question.entity_plural_label'|trans %}

{% block navButtons %}
    {% import '@OroUI/macros.html.twig' as UI %}

    {% include '@OroImportExport/ImportExport/buttons_from_configuration.html.twig' with {
        'alias': 'acme_demo_question'
    } %}

    {% if is_granted('acme_demo_question_create') %}
        <div class="btn-group">
        {{ UI.addButton({
            'path': path('acme_demo_question_create'),
            'entity_label': 'acme.demo.question.entity_label'|trans
        }) }}
        </div>
    {% endif %}
{% endblock %}
```

**2. Link question rows to the related update action**

To make it possible to modify each question, you need to define a property that describes how the URL of the update action is built, and then add this URL to the list of available actions in your data grid configuration:

```yaml
 datagrids:
     acme-demo-question:
         # ...
         properties:
             id: ~
             update_link:
                 type: url
                 route: acme_demo_question_update
                 params:
                     - id
             # ...
         actions:
             # ...
             edit:
                 type: navigate
                 label: Edit
                 link: update_link
                 icon: edit
```

<a id="book-crud-delete"></a>

## Deleting Entities

You can delete a question through the `DELETE` operation available for all entities by default or through the customized one. When running `DELETE`, ensure that your entity has a route from the `routeName` option of the entity configuration.

You can delete an entity through the [DELETE operation](../entities-data-management/actions/index.md#bundle-docs-platform-action-bundle-default-operations) which is enabled by default for all entities. To run the operation, you need to ensure that your entity has the `routeName` option of the entity configuration, which will be used as a route name to redirect a user after the `DELETE` operation (as in the example below).

```php
#[Config(
    routeName: 'acme_demo_question_index',
    routeView: 'acme_demo_question_view',
    defaultValues: [
        'entity' => ['icon' => 'fa-question'],
    ]
)]
```

See the sample configuration of the default `DELETE` operation in the <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ActionBundle/Resources/config/oro/actions.yml" target="_blank">Actions</a> topic.

If the default configuration is not valid for your particular case, create your own operation that would inherit from the default one, following the example:

```yaml
DELETE:
    exclude_entities:
        - Oro\Bundle\CatalogBundle\Entity\Category

oro_catalog_category_delete:
    extends: DELETE
    replace:
        - exclude_entities
        - entities
        - for_all_datagrids
        - for_all_entities
    for_all_datagrids: false
    for_all_entities: false
    entities:
        - Oro\Bundle\CatalogBundle\Entity\Category
    preconditions:
        '@and':
            - '@not_equal': [$.data.parentCategory, null]
```

#### NOTE
When creating your own operation, make sure to exclude the entity from the default operation. See more details on [available operations and their configuration](../entities-data-management/actions/index.md#bundle-docs-platform-action-bundle-operations) in the related article.

<!-- Frontend -->
