#### NOTE
<a id="dev-feature-toggle"></a>

# Feature Toggle

## Define a New Feature

Features are defined in configuration files placed into Resources/config/oro/features.yml.

Each feature consists of one required option, the label. You can configure the following sections, out-of-the-box:

> - `label` - A feature title.
> - `description` - A feature description.
> - `toggle` - A [system configuration](../system-configuration/index.md#backend-system-configuration) option key that is used as a feature toggle.
> - `dependencies` - A list of feature names that the feature depends on. The feature is enabled when all the features from this list are also enabled.
> - `routes` - A list of route names.
> - `configuration` - A list of [system configuration](../system-configuration/index.md#backend-system-configuration) group and field names.
> - `workflows` - A list of [workflow](../entities-data-management/workflows/intro.md#backend-workflows-intro) names.
> - `processes` - A list of [process](../entities-data-management/processes.md#backend-entities-data-management-processes) names.
> - `operations` - A list of [operation](../entities-data-management/actions/index.md#bundle-docs-platform-action-bundle-operations) names.
> - `api_resources` - A list of entity FQCNs that are available as API resources.
> - `frontend_api_resources` - A list of entity FQCNs that are available as the storefront API resources.
> - `commands` - A list of commands that depend on the feature. Running these commands is impossible or is not reasonable when the feature is disabled.
> - `entities` - A list of entity FQCNs.
> - `dashboard_widgets` - A list of [dashboard widget](../dashboards/index.md#dev-dashboards) names.
> - `sidebar_widgets` - A list of sidebar widget names.
> - `cron_jobs` - A list of CRON commands that depend on the feature. These commands are not executed by the [cron](../cron.md#dev-guide-system-cron-jobs) when the feature is disabled.
> - `navigation_items` - A list of [navigation item](../navigation/index.md#doc-managing-app-menu) names.
> - `placeholder_items` - A list of placeholder item names.
> - `mq_topics` - A list of [message queue topic](../mq/message-queue-topics.md#dev-guide-mq-topics) names.

An example of the features.yml configuration:

```yaml
features:
    acme:
        label: acme.feature.label
        description: acme.feature.description
        toggle: acme.feature_enabled
        dependencies:
            - foo
            - bar
        routes:
            - acme_entity_view
            - acme_entity_create
        configuration:
            - acme_general_section
            - acme.some_option
        workflows:
            - acme_sales_flow
        processes:
            - acme_some_process
        operations:
            - acme_some_operation
        api_resources:
            # bind whole API resource / bind all API actions for API resource
            - Acme\Bundle\DemoBundle\Entity\Page
            # bind only specific API actions for API resource
            - [Acme\Bundle\DemoBundle\Entity\Page, [create, update, delete, delete_list]]
        frontend_api_resources:
            # bind whole API resource / bind all API actions for API resource
            - Acme\Bundle\DemoBundle\Entity\Page
            # bind only specific API actions for API resource
            - [Acme\Bundle\DemoBundle\Entity\Page, [create, update, delete, delete_list]]
        commands:
            - oro:search:index
        entities:
            - Acme\Bundle\DemoBundle\Entity\Page
        dashboard_widgets:
            - page_dashboard_widget
        sidebar_widgets:
            - acme_sidebar_widget
        cron_jobs:
            - acme:cron:sync-job
        navigation_items:
            - application_menu.sales_tab.acme_order_list
        placeholder_items:
            - acme_create_page_button
        mq_topics:
            - acme.mq_topics.calculate
```

#### NOTE
The `oro:feature-toggle:config:dump-reference` command can be used to dump the reference structure for Resources/config/oro/features.yml:

```none
php bin/console oro:feature-toggle:config:dump-reference
```

<a id="feature-toggle-new-options"></a>

## Add New Options to Feature Configuration

Feature configuration may be extended with new configuration options. To add a new configuration option, you need to add a feature configuration that implements ConfigurationExtensionInterface and register it with the oro_feature.config_extension tag.
For example, there are some Acme processors which should be configured with the acme_processor option.

Configuration extension:

```php
namespace Acme\Bundle\DemoBundle\Config;

use Oro\Bundle\FeatureToggleBundle\Configuration\ConfigurationExtensionInterface;
use Symfony\Component\Config\Definition\Builder\NodeBuilder;

class FeatureConfigurationExtension implements ConfigurationExtensionInterface
{
    #[\Override]
    public function extendConfigurationTree(NodeBuilder $node)
    {
        $node
            ->arrayNode('acme_processor')
                ->prototype('variable')
                ->end()
            ->end();
    }
}
```

Extension registration:

```yaml
services:
    acme.configuration.feature_configuration_extension:
        class: Acme\Bundle\DemoBundle\Config\FeatureConfigurationExtension
        tags:
            - { name: oro_feature.config_extension }
```

<a id="feature-toggle-check-feature-state"></a>

## Check Feature State

Feature state is determined by FeatureChecker. There are proxy classes that expose a feature check functionality to layout updates, operations, workflows, processes, and twig.

Feature state is resolved by isFeatureEnabled($featureName, $scopeIdentifier = null)

Feature resource types are nodes of feature configuration (routes, workflows, configuration, processes, operations, API resources, etc.), resources are their values. Resource is disabled if it is included into at least one disabled feature.
Resource state is resolved by public function isResourceEnabled($resource, $resourceType, $scopeIdentifier = null)

### Layout Updates

* Check the feature state =data[‘feature’].isFeatureEnabled(‘feature_name’)
* Check the resource state =data[‘feature’].isResourceEnabled(‘acme_product_view’, ‘routes’)

> Set the block visibility based on the feature state:
```yaml
layout:
    actions:
        - '@add':
            id: products
            parentId: page_content
            blockType: datagrid
            options:
                grid_name: products-grid
                visible: '=data["feature"].isFeatureEnabled("product_feature")'
```

### Processes, Workflows, Operations

In processes, workflows and operations, config expressions may be used to check the feature state

* Check the feature state
  > ```yaml
  > '@feature_enabled':
  >     feature: 'feature_name'
  >     scope_identifier: $.scopeIdentifier
  > ```
* Check the resource state
  > ```yaml
  > '@feature_resource_enabled':
  >     resource: 'some_route'
  >     resource_type: 'routes'
  >     scope_identifier: $.scopeId
  > ```

### Twig

* Check the feature state feature_enabled($featureName, $scopeIdentifier = null)
* Check the resource state feature_resource_enabled($resource, $resourceType, $scopeIdentifier = null)

<a id="feature-toggle-include-services"></a>

## Include a Service Into a Feature

Any service that requires a feature functionality, needs to implement the FeatureToggleableInterface interface.
All checks are done by developer.

OroFeatureToggleBundle provides helper functionality to inject a feature checker and a feature name into services marked with the oro_featuretogle.feature tag.
FeatureCheckerHolderTrait contains implementation of methods from FeatureToggleableInterface.

Some extensions can extend the form, and we need to include this extension functionality into a feature. In this case, FeatureChecker should be injected into service, and feature availability should be checked where needed.

Extension:

```php
namespace Acme\Bundle\DemoBundle\Form\Extension;

use Symfony\Component\Form\AbstractTypeExtension;
use Symfony\Component\Form\FormBuilderInterface;

use Oro\Bundle\FeatureToggleBundle\Checker\FeatureToggleableInterface;
use Oro\Bundle\FeatureToggleBundle\Checker\FeatureCheckerHolderTrait;

class ProductFormExtension extends AbstractTypeExtension implements FeatureToggleableInterface
{
    use FeatureCheckerHolderTrait;

    #[\Override]
    public static function getExtendedTypes(): iterable
    {
        return ['acme_product'];
    }

    #[\Override]
    public function buildForm(FormBuilderInterface $builder, array $options)
    {
        if (!$this->isFeaturesEnabled()) {
            return;
        }

        $builder->add(
            'category',
            AcmeCategoryTreeType::class,
            [
                'required' => false,
                'mapped' => false,
                'label' => 'Category'
            ]
        );
    }
}
```

Extension registration:

```yaml
services:
    acme_demo.form.extension.product_form:
        class: Acme\Bundle\DemoBundle\Form\Extension\ProductFormExtension
    tags:
        - { name: oro_featuretogle.feature, feature: acme_feature }
```

<a id="feature-toggle-feature-voter"></a>

## Check Feature State with a Feature Voter

Feature state is checked by feature voters. All voters are called each time you use the isFeatureEnabled() or isResourceEnabled() method on the feature checker.
The feature checker makes the decision based on the configured strategy defined in the system configuration or per feature, which can be: affirmative, consensus, or unanimous.

By default, ConfigVoter is registered to check features availability.
It checks the feature state based on the value of a toggle option defined in the features.yml configuration.

A custom voter needs to implement `Oro\Bundle\FeatureToggleBundle\Checker\Voter\VoterInterface`.
Imagine that we have the state checker that returns decision based on a feature name and a scope identifier.
The feature is enabled for the valid state and disabled for the invalid state. In other cases, do not vote.

Such voter looks as follows:

```php
namespace Acme\Bundle\DemoBundle\Voter;

use Oro\Bundle\FeatureToggleBundle\Checker\Voter\VoterInterface;

class FeatureVoter implements VoterInterface
{
    private StateChecker $stateChecker;

    public function __construct(StateChecker $stateChecker) {
        $this->stateChecker = $stateChecker;
    }

    /**
     * @param string $feature
     * @param object|int|null $scopeIdentifier
     * return int either FEATURE_ENABLED, FEATURE_ABSTAIN, or FEATURE_DISABLED
     */
    public function vote($feature, $scopeIdentifier = null)
    {
        if ($this->stateChecker($feature, $scopeIdentifier) === StateChecker::VALID_STATE) {
            return self::FEATURE_ENABLED;
        }
        if ($this->stateChecker($feature, $scopeIdentifier) === StateChecker::INVALID_STATE) {
            return self::FEATURE_DISABLED;
        }

        return self::FEATURE_ABSTAIN;
    }
}
```

Now, configure a voter:

```yaml
services:
    acme_demo.voter.feature_voter:
        class: Acme\Bundle\DemoBundle\Voter\FeatureVoter
        arguments: [ '@acme_demo.voter.state_checker' ]
        tags:
            - { name: oro_featuretogle.voter }
```

<a id="feature-toggle-change-decision-strategy"></a>

## Change Decision Strategy

There are three strategies available:

* *affirmative* – The strategy grants access if one voter grants access;
* *consensus* – The strategy grants access if there are more voters that grant access than those that deny;
* *unanimous* (default) – The strategy grants access only if all voters grant access.

Strategy configuration (may be defined in Resources/config/oro/app.yml)

```yaml
oro_featuretoggle:
    strategy: affirmative
    allow_if_all_abstain: true
    allow_if_equal_granted_denied: false
```

or in feature definition

```yaml
features:
    acme:
        label: acme.feature.label
        strategy: affirmative
        allow_if_all_abstain: true
        allow_if_equal_granted_denied: false
```

<a id="feature-toggle-checker-for-commands"></a>

## Use Checker for Commands

Commands launched as subcommands cannot be skipped globally. To avoid running such commands, add an implementation of FeatureCheckerAwareInterface to your parent command, import FeatureCheckerHolderTrait (via use FeatureCheckerHolderTrait;), and check the feature status via featureChecker that is automatically injected into your command.

```php
namespace Acme\Bundle\DemoBundle\Command;

use Oro\Bundle\FeatureToggleBundle\Checker\FeatureCheckerHolderTrait;
use Oro\Bundle\FeatureToggleBundle\Checker\FeatureCheckerAwareInterface;

class LoadDataFixturesCommand implements FeatureCheckerAwareInterface
{

    use FeatureCheckerHolderTrait;

    protected function execute(InputInterface $input, OutputInterface $output)
    {
        $commands = [
            'oro:cron:analytic:calculate' => [],
            'oro:b2b:lifetime:recalculate'          => ['--force' => true]
        ];

        foreach ($commands as $commandName => $options) {
            if ($this->featureChecker->isResourceEnabled($commandName, 'commands')) {
                $command = $this->getApplication()->find($commandName);
                $input = new ArrayInput(array_merge(['command' => $commandName], $options));
                $command->run($input, $output);
            }
        }
    }
}
```
