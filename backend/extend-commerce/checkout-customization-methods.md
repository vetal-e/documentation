<a id="dev-extend-commerce-mastering-checkouts"></a>

# Common Checkout Customization Methods

This document provides details on the most common checkout workflow customizations. To customize the checkout process, you can follow two methods. The first method requires modifying the configuration and logic of an existing checkout. The second method involves creating a new checkout workflow with a custom name and then making all the necessary customizations based on that custom name. For the examples provided, let us assume that we have extended a multistep checkout workflow and named the new workflow acme_demo_checkout.

```yaml
imports:
    -
        resource: '@OroCheckoutBundle/Resources/config/oro/workflows/b2b_flow_checkout.yml'
        workflow: b2b_flow_checkout
        as: acme_demo_checkout
        replace: []
```

## Transfer Custom Data from Shopping List to Order with Checkout

The most common checkout change is the addition of new attributes and their display on the checkout form.
As an illustration, let us add the `external_po_number` field to the Shopping List and transfer it to the Order:

1. Add an extendable field `external_po_number` to the Shopping List, Checkout and Order entities with migration.
   ```php
   <?php

   namespace Acme\Bundle\DemoBundle\Migrations\Schema\v1_13;

   use Doctrine\DBAL\Schema\Schema;
   use Oro\Bundle\EntityBundle\EntityConfig\DatagridScope;
   use Oro\Bundle\EntityExtendBundle\EntityConfig\ExtendScope;
   use Oro\Bundle\MigrationBundle\Migration\Migration;
   use Oro\Bundle\MigrationBundle\Migration\QueryBag;

   class AddExternalPoNumberColumn implements Migration
   {
       public function up(Schema $schema, QueryBag $queries)
       {
           $this->addExternalPoNumber($schema, 'oro_shopping_list');
           $this->addExternalPoNumber($schema, 'oro_checkout');
           $this->addExternalPoNumber($schema, 'oro_order');
       }

       private function addExternalPoNumber(Schema $schema, string $tableName): void
       {
           $table = $schema->getTable($tableName);
           $table->addColumn(
               'external_po_number',
               'string',
               [
                   'notnull' => false,
                   'length' => 255,
                   'oro_options' => [
                       'extend' => [
                           'is_extend' => true,
                           'owner' => ExtendScope::OWNER_CUSTOM
                       ],
                       'entity' => ['label' => 'External Po Number'],
                       'datagrid' => ['is_visible' => DatagridScope::IS_VISIBLE_TRUE]
                   ]
               ]
           );
       }
   }
   ```
2. Define the storage for `external_po_number` during the Checkout. Add an extendable field `external_po_number` to the Checkout Entity and expose it as an attribute in the Checkout workflow:
   > ```yaml
   > workflows:
   >     acme_demo_checkout:
   >         attributes:
   >             # Extends the list of attributes of the b2b_flow_checkout.attributes
   >             external_po_number:
   >                 property_path: checkout.external_po_number
   > ```

1. Transfer the `external_po_number` value from the Shopping List to the Checkout.
   > To transfer the data from the source object to the checkout during the checkout start decorate an action group `Oro\Bundle\CheckoutBundle\Workflow\ActionGroup\StartShoppingListCheckout` that is responsible for the start logic:
   ```php
   <?php

   namespace Acme\Bundle\DemoBundle\Workflow\ActionGroup;

   use Oro\Bundle\CheckoutBundle\Workflow\ActionGroup\StartShoppingListCheckoutInterface;
   use Oro\Bundle\ShoppingListBundle\Entity\ShoppingList;

   /**
    * Transfer external_po_number from shopping list to the checkout on checkout creation.
    */
   class StartShoppingListCheckout implements StartShoppingListCheckoutInterface
   {
       public function __construct(
           private StartShoppingListCheckoutInterface $innerAction
       ) {
       }

       public function execute(
           ShoppingList $shoppingList,
           bool $forceStartCheckout = false,
           bool $showErrors = false,
           bool $validateOnStartCheckout = true,
           bool $allowManualSourceRemove = true,
           bool $removeSource = true,
           bool $clearSource = false
       ): array {
           $result = $this->innerAction->execute(
               $shoppingList,
               $forceStartCheckout,
               $showErrors,
               $validateOnStartCheckout,
               $allowManualSourceRemove,
               $removeSource,
               $clearSource
           );

           $result['checkout']->setExternalPoNumber($shoppingList->getExternalPoNumber());

           return $result;
       }
   }
   ```

   Make sure you register the service.
   ```yaml
   services:
       acme_demo.action_group.start_from_shopping_list:
           class: Acme\Bundle\DemoBundle\Workflow\ActionGroup\StartShoppingListCheckout
           decorates: oro_checkout.action_group.start_from_shopping_list
           arguments:
               - '@.inner'
           tags:
               - { name: 'oro_action_group_service' }
   ```

1. Modify the `place_order` transition form to include the new attribute.
   1. Add the attribute to the transition form fields

   ```yaml
   workflows:
       acme_demo_checkout:
           transitions:
               place_order:
                   form_options:
                       # Extends the list of attribute_fields
                       attribute_fields:
                           external_po_number: ~
   ```

   1. Render the new attribute. For more information, see documentation on Layouts.
2. Transfer `external_po_number` from the Checkout to the Order during Order placement by adding an event listener to the `extendable_action.finish_checkout` event.
   ```php
   <?php

   namespace Acme\Bundle\DemoBundle\Workflow\EventListener;

   use Oro\Component\Action\Event\ExtendableActionEvent;

   /**
    * Transfer external_po_number from checkout to order on checkout finish.
    */
   class FinishCheckoutEventListener
   {
       public function onFinishCheckout(ExtendableActionEvent $event): void
       {
           $data = $event->getData();
           if (!$data) {
               return;
           }

           $checkout = $data->offsetGet('checkout');
           $order = $data->offsetGet('order');

           $order->setExternalPoNumber($checkout->getExternalPoNumber());
       }
   }
   ```

   ```yaml
   services:
       Acme\Bundle\DemoBundle\Workflow\EventListener\FinishCheckoutEventListener:
           tags:
               - { name: kernel.event_listener, event: extendable_action.finish_checkout, method: onFinishCheckout }
   ```

## Add Intermediate Step to Existing Multistep Checkout

Another frequently implemented Checkout workflow customization is adding a new step to gather additional information.

#### NOTE
For simplicity, less important allowed transitions (such as back_to_\*) are not listed. Be sure to include them in your customization.

To illustrate such customization, consider a case where only a customer user with manager permissions can place an Order if `external_po_number` starts with the **EXT-** prefix.

This scenario covers the following aspects:

- Adding an intermediate step to the existing multistep checkout
- Modifying/extending the transition logic with service-based transitions
- Implementing the ability to direct users to different destinations based on a specific condition
- Adding and checking a new ACL permission

1. Define a new workflow with additional step `manager_approval`. To reach this step, modify the configuration of the `place_order` transition by adding the `conditional_steps_to` option and rewriting the `transition_service`.
2. After this change, if the *external_po_number* field starts with the *EXT-* prefix, buyers without the *acme_demo_checkout_approve* ACL permission cannot proceed with the checkout and are redirected to the *manager_approval* step. Only users with manager permissions will be able to complete orders in this workflow. Managers will also have the ability to place such orders directly from the Order Review step without restrictions.
   ```yaml
   imports:
       -
           resource: '@OroCheckoutBundle/Resources/config/oro/workflows/b2b_flow_checkout.yml'
           workflow: b2b_flow_checkout
           as: acme_demo_checkout
           replace: []

   workflows:
       acme_demo_checkout:
           defaults:
               active: false

           attributes:
               # Extends the list of attributes of the b2b_flow_checkout.attributes
               external_po_number:
                   property_path: checkout.external_po_number

           steps:
               manager_approval:
                   order: 80
                   allowed_transitions:
                       - place_order
                       - finish_checkout
                       # A set of additional transitions are here

               order_created:
                   # Overrides b2b_flow_checkout.steps.order_created.order
                   order: 90

           transitions:
               place_order:
                   # Overrides b2b_flow_checkout.transitions.place_order.transition_service
                   transition_service: 'acme_demo.workflow.transition.place_order'
                   # Adds conditional_step_to
                   conditional_steps_to:
                       manager_approval:
                           conditions:
                               '@and':
                                   - '@start_with': [$external_po_number, 'EXT-']
                                   - '@not':
                                         - '@acl_granted': 'acme_demo_checkout_approve'
   ```
3. Define the ACL permission.
   ```yaml
   acls:
       acme_demo_checkout_approve:
           label: acme.demo.security.permission.checkout_approve
           type: action
           group_name: "commerce"
           category: "checkout"
   ```
4. Change the implementation of the Place Order transition to avoid creating an order when it is now allowed.
   ```php
   <?php

   namespace Acme\Bundle\DemoBundle\Workflow\Transition;

   use Doctrine\Common\Collections\Collection;
   use Oro\Bundle\ActionBundle\Model\ActionExecutor;
   use Oro\Bundle\CheckoutBundle\Entity\Checkout;
   use Oro\Bundle\WorkflowBundle\Entity\WorkflowItem;
   use Oro\Bundle\WorkflowBundle\Model\TransitionServiceInterface;
   use Symfony\Component\Security\Core\Authorization\AuthorizationCheckerInterface;

   /**
    * PlaceOrder transition customization for Demo Checkout.
    */
   class PlaceOrder implements TransitionServiceInterface
   {
       public function __construct(
           private TransitionServiceInterface $baseTransition,
           private AuthorizationCheckerInterface $authorizationChecker,
           private ActionExecutor $actionExecutor
       ) {
       }

       public function isPreConditionAllowed(WorkflowItem $workflowItem, Collection $errors = null): bool
       {
           if ($workflowItem->getCurrentStep()?->getName() === 'manager_approval'
               && !$this->authorizationChecker->isGranted('acme_demo_checkout_approve')
           ) {
               $errors?->add(['message' => 'Pending approval']);

               return false;
           }

           return $this->baseTransition->isPreConditionAllowed($workflowItem, $errors);
       }

       public function isConditionAllowed(WorkflowItem $workflowItem, Collection $errors = null): bool
       {
           return $this->baseTransition->isConditionAllowed($workflowItem, $errors);
       }

       public function execute(WorkflowItem $workflowItem): void
       {
           /** @var Checkout $checkout */
           $checkout = $workflowItem->getEntity();
           // Do not execute place order transition logic if conditions are not met.
           if (str_starts_with($checkout->getExternalPoNumber(), 'EXT-')
               && !$this->authorizationChecker->isGranted('acme_demo_checkout_approve')
           ) {
               $this->actionExecutor->executeAction(
                   'flash_message',
                   [
                       'message' => 'This checkout requires manager approval.',
                       'type' => 'warning'
                   ]
               );

               return;
           }

           $this->baseTransition->execute($workflowItem);
       }
   }
   ```

   ```yaml
   services:
       acme_demo.workflow.transition.place_order:
           class: Acme\Bundle\DemoBundle\Workflow\Transition\PlaceOrder
           arguments:
               - '@oro_checkout.workflow.b2b_flow_checkout.transition.place_order'
               - '@security.authorization_checker'
               - '@oro_action.action_executor'
           tags:
               - { name: 'oro_workflow.transition_service' }
   ```

## Block Checkout Transition Availability or Execution

To limit the availability and execution of the transition, use workflow guard events, such as `oro_workflow.pre_announce`, `oro_workflow.announce`, `oro_workflow.pre_guard` and `oro_workflow.guard`. Thr `pre_announce` and `pre_guard` events are executed before any transition logic, while  the `announce` and `guard` are executed immediately after. The `*announce` events serve to limit transition availability, whereas the `*guard` events are used to limit execution.

The example below illustrates a scenario where customer users belonging to the Guest customer group are not allowed to place orders if the total amount is less than 100 USD. Here, the limit should apply only to `acme_demo_checkout`.

```php
<?php

namespace Acme\Bundle\DemoBundle\Workflow\EventListener;

use Oro\Bundle\CheckoutBundle\DataProvider\Manager\CheckoutLineItemsManager;
use Oro\Bundle\CheckoutBundle\Entity\Checkout;
use Oro\Bundle\ConfigBundle\Config\ConfigManager;
use Oro\Bundle\CustomerBundle\Provider\CustomerUserRelationsProvider;
use Oro\Bundle\OrderBundle\Entity\Order;
use Oro\Bundle\PricingBundle\SubtotalProcessor\TotalProcessorProvider;
use Oro\Bundle\WorkflowBundle\Event\Transition\PreAnnounceEvent;

/**
 * Disallow order placement for guests if total is less than 100 USD
 */
class DisallowCheapOrdersForGuestsEventListener
{
    private const TOTAL_LIMIT = 100.0;

    public function __construct(
        private CustomerUserRelationsProvider $customerUserRelationsProvider,
        private ConfigManager $configManager,
        private TotalProcessorProvider $totalsProvider,
        private CheckoutLineItemsManager $checkoutLineItemsManager
    ) {
    }

    public function onPreAnnounce(PreAnnounceEvent $event): void
    {
        // Nothing to do, already denied
        if (!$event->isAllowed()) {
            return;
        }

        $workflowItem = $event->getWorkflowItem();
        /** @var Checkout $checkout */
        $checkout = $workflowItem->getEntity();
        if (!$checkout instanceof Checkout) {
            return;
        }

        // Do not apply the limit for non-anonymous customer group
        $customerUserGroup = $this->customerUserRelationsProvider->getCustomerGroup($checkout->getCustomerUser());
        if ($customerUserGroup?->getId() != $this->configManager->get('oro_customer.anonymous_customer_group')) {
            return;
        }

        if ($checkout->getCurrency() !== 'USD') {
            return;
        }

        if ($this->getOrderTotalForCheckout($checkout) < self::TOTAL_LIMIT) {
            $event->getErrors()?->add([
                'message' => 'Cannot proceed to checkout because total amount is less than 100 USD'
            ]);

            $event->setAllowed(false);
        }
    }

    private function getOrderTotalForCheckout(Checkout $checkout): float
    {
        $orderLineItems = $this->checkoutLineItemsManager->getData($checkout);
        $order = new Order();
        $order->setLineItems($orderLineItems);

        return $this->totalsProvider->enableRecalculation()->getTotal($order)->getAmount();
    }
}
```

```yaml
services:
    Acme\Bundle\DemoBundle\Workflow\EventListener\DisallowCheapOrdersForGuestsEventListener:
        arguments:
            - '@oro_customer.provider.customer_user_relations_provider'
            - '@oro_config.manager'
            - '@oro_pricing.subtotal_processor.total_processor_provider'
            - '@oro_checkout.data_provider.manager.checkout_line_items'
        tags:
            - { name: kernel.event_listener, event: oro_workflow.acme_demo_checkout.pre_announce, method: onPreAnnounce }
```

## Import Workflow Configuration Conditionally

Workflow bundle provides different ways to organize workflow configuration. Workflow configuration can be split into separate parts and added to the workflow configuration using the `imports` directive.

#### NOTE
Consider following the advice below when organizing the checkout workflow configuration:

* For complex workflows, use imports and to separate different parts of the configuration, such as steps and transitions.
* For simple workflows with a limited number of changes, keep all configurations in one place.

While developing a workflow, you may find it necessary to switch to a new implementation of transition logic, such as when migrating to service-based transitions. To solve this and retain the option to easily revert to the old implementation, you can import different versions of the transition configuration by including an `import_condition` expression. Another potential use for this feature is to load workflow configuration only when a specific 3rd party package is available.

```yaml
 imports:
     # BC. Load workflows with definition-based transitions
     # when acme_demo.use_transition_services DI parameter is not present or set to false.
     -
         resource: 'workflows/checkout.yml'
         import_condition: "parameter_or_null('acme_demo.use_transition_services') !== true"

     # NEW. Load workflows with transition as a service implementation
     # when acme_demo.use_transition_services DI parameter is present and set to true
     -
         resource: 'workflows/checkout_with_services.yml'
         import_condition: "parameter_or_null('acme_demo.use_transition_services') === true"
```

## Choose Storage for Additional Checkout Data

When working with checkouts, you have three storage options for additional data: **Checkout Entity**, **Workflow Data**, and **Workflow Result**.

The **Checkout Entity** is a suitable storage option for any data useful for the entire checkout workflow or any logic that may use the Checkout entity outside the workflow. Opting for this method means you must add entity migration and execute the update process. This operation requires a DB schema update for non-extend fields and may require downtime.

On the other hand, data can be stored in the WorkflowData when the workflow attribute is configured. This storage is easier to set up and only requires reloading the workflow definition. It is a good option when data is needed in the checkout workflow itself or is specific to that workflow. For instance, if an additional checkout workflow is initiated for a customer group that requires approval, the approval information is specific to that particular checkout with approval workflow and should be stored in the WorkflowData.

There is a third possible place to store workflow data at runtime, the Workflow Result. In YAML-based checkouts, it is used to store variable values for a transition. It can be used to transfer non-persistent data in the WorkflowItem across various logic parts that have access to the WorkflowItem.

#### WARNING
The data stored in the Workflow Result is not persisted and is only available during the execution of the workflow.

## Access the WorkflowItem by the Given Workflow Entity

As illustrated in the examples above, sometimes only the workflow entity is available. In cases when the data is stored in the WorkflowItem, retrieve it from the available workflow entity first. For this, use the `oro_workflow.manager` service. For example, to work with data stored in the WorkflowItem, you can modify the `FinishCheckoutEventListener` as follows:

```php
<?php

namespace Acme\Bundle\DemoBundle\Workflow\EventListener\Alternatives;

use Oro\Bundle\WorkflowBundle\Model\WorkflowManager;
use Oro\Component\Action\Event\ExtendableActionEvent;

/**
 * Transfer external_po_number from WorkflowItem to order on checkout finish.
 */
class FinishCheckoutEventListener
{
    public function __construct(
        private WorkflowManager $workflowManager
    ) {
    }

    public function onFinishCheckout(ExtendableActionEvent $event): void
    {
        $data = $event->getData();
        if (!$data) {
            return;
        }

        $checkout = $data->offsetGet('checkout');
        $order = $data->offsetGet('order');

        $workflowItem = $this->workflowManager->getWorkflowItem($checkout, 'acme_demo_checkout');
        $externalPoNumber = $workflowItem?->offsetGet('external_po_number');

        $order->setExternalPoNumber($externalPoNumber);
    }
}
```

**Related Articles**

* Checkout Customization
* Checkout Finish
* [Workflow Configuration Reference](../entities-data-management/workflows/configuration-reference.md#backend-workflows-config-reference)
* [Workflow Transition Forms](../entities-data-management/workflows/transition-forms.md#backend-workflows-transition-forms)
* [Workflow Transition Services](../entities-data-management/workflows/transition-services.md#backend-workflows-transition-services)
* [Workflow Events](../entities-data-management/workflows/workflow-events.md#backend-workflows-workflow-events)
* [Action Groups](../entities-data-management/actions/action-groups.md#bundle-docs-platform-action-bundle-action-groups)
* Layouts.
