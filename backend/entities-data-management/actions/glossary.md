<a id="bundle-docs-platform-action-bundle-glossary"></a>

# Operations (Actions) Glossary

* [Buttons](buttons.md#bundle-docs-platform-action-bundle-buttons) are a user interface component that helps deliver custom actions for user interaction. They provide a way to demonstrate any actions (operations, for example) to UI for a proper context through specific <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ActionBundle/Extension/ButtonProviderExtensionInterface.php" target="_blank">ButtonsProviderExtension</a> together with <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ActionBundle/Button/ButtonInterface.php" target="_blank">Buttons</a> matched by a context.
* [Operation](index.md#bundle-docs-platform-action-bundle-operations) are configured user interaction elements (buttons, links or even further: forms, pages) with customized execution logic. One of the main components is ActionBundle which handles information about a specific operation logic, how and when a UI element is displayed, the reaction it provides, and how to aggregate the data retrieved from a user (usually through a form) into execution unit values and launch configured *Actions* afterward.

The operation definition contains the most important information, such as operation related entity classes (‘AcmeBundleDemoBundleEntityMyEntity’), or routes (‘acme_demo_myentity_view’), or datagrids (‘acme-demo-grid’).

The operation can be enabled or disabled. Other fields of the operation contain information about its name, extended options, and order of displayed buttons. For more options please refer to [Operation Configuration](index.md#bundle-docs-platform-action-bundle-operations).

* [Action Group](action-groups.md#bundle-docs-platform-action-bundle-action-groups) is complex business logic sets of backend actions grouped together under the named configuration nodes. It is another key component in ActionBundle. A named group of actions with entry parameters (required or optional, typed or not) and conditions.

  *Action groups* can be used not only from an operation but within the workflow processes and in any part of the OroPlatform configuration nodes that understand [Actions](actions-conditions.md#bundle-docs-platform-action-bundle-action-component).

A special @run_action_group action is designed to run a group of actions as a single one. (For more information please refer to [\*ActionGroup\* configuration](action-groups.md#bundle-docs-platform-action-bundle-action-groups) and @run_action_group action.

* [Condition](actions-conditions.md#bundle-docs-platform-action-bundle-conditions) - defines whether *Operation* or *ActionGroup* is allowed. Conditions use <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Component/ConfigExpression/README.md" target="_blank">ConfigExpression</a> syntax and can be nested within each other.
* [Actions](actions-conditions.md#bundle-docs-platform-action-bundle-action-component)- simple functional blocks (that are described in Action Component). They can be used in *ActionGroups* or *Operations* to implement the preparation logic before *conditions*, to retrieve rendering data, to initialize and execute the logic afterward.
  * *Operations* contain the following *actions*: **Preactions** (preactions), the **Form Init** actions (form_init), and **Actions** themselves with the functions of Action Component. The difference is that preactions are executed before the operation button rendering, though the form_init actions are executed before the form display. Actions can be used to perform any operations with data in their context (called Action Data) or other entities.
  * **Definition** - part of *Operation* or *ActionGroup* that contains the configuration of the component itself and describes its behavior.
* **Attribute** - an entity that represents a value (mostly in *Operation*) and is used to render a field value in a step of a form. The attribute knows about its type (string, object, entity, etc.) and additional options. The attribute contains a name and label as additional parameters.

<!-- Frontend -->
