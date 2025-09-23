<a id="dev-doc-workflows"></a>

# Workflows

In a business application, a `workflow` is a sequence of steps or rules applied to a process from its initiation to completion.
In Oro applications, workflows organize and direct users’ work, making them follow particular steps in a pre-defined order, or preventing them from performing actions that either contradict or conflict with the logical steps of a process.

A workflow `step` is a state of an entity record. It is represented by an instance of the `Oro\Bundle\WorkflowBundle\Entity\WorkflowStep` class.
The process of moving an entity from one step to another is called a `transition`.

This guide explains and illustrates how to create workflows through configuration files, and provides details on workflow components and their translation.

**Table of Contents**

* [Introduction](intro.md#backend-workflows-intro)
* [Configuration Reference](configuration-reference.md#backend-workflows-config-reference)
* [Basic Workflow Configuration](create.md#backend-workflows-create)
* [Transition Services](transition-services.md#backend-workflows-transition-services)
* [Transition Forms](transition-forms.md#backend-workflows-transition-forms)
* [Managing Elements](elements.md#backend-workflows-managing-elements) ([Actions](elements.md#backend-workflows-transition-actions) and [Conditions](elements.md#backend-workflows-transition-conditions))
* [Workflow Translations](translations-wizard.md#backend-workflows-translation-wizard)
* [Workflow Events](workflow-events.md#backend-workflows-workflow-events)
* [Example of Workflow Configuration](config-example.md#backend-workflows-example)

#### HINT
For information on how to create simple workflows via the user interface (application back-office), see the Workflow Management user guide.
