<a id="dev-guide-application-web-framework"></a>

# Application Framework

**Application Framework** functionality is part of **OroPlatform** that determines the structure of the Oro application (code
organization, file structure, application flow routine) and the way of interaction between independent components in the application.

In this section, you’ll find a description of the main principles for organizing your adjustments to Oro applications.

## How It Works

Application Framework functionality in OroPlatform is based on the <a href="https://symfony.com/" target="_blank">Symfony Framework</a> and extends it in the
<a href="https://github.com/oroinc/platform/tree/master/src/Oro/Bundle/PlatformBundle" target="_blank">OroPlatformBundle</a> and <a href="https://github.com/oroinc/platform/tree/master/src/Oro/Bundle/DistributionBundle" target="_blank">OroDistributionBundle</a> to improve several aspects of development experience.

Please refer to the [Architecture Principles of Oro Applications](architecture-principles.md#dev-guide-application-web-framework-symfony) article
for the synopsis of structural aspects of the Oro applications:

* [Symfony Role in OroPlatform](architecture-principles.md#dev-guide-application-web-framework-symfony-symfony-role-in-oroplatform)
* [HTTP Request Application Flow](architecture-principles.md#dev-guide-application-web-framework-symfony-http-request-application-flow)
* [Event System](architecture-principles.md#dev-guide-application-web-framework-symfony-event-system)
* [Inversion of Control Principle](architecture-principles.md#dev-guide-application-web-framework-symfony-inversion-of-control-principle)
* [Bundle System](architecture-principles.md#dev-guide-application-web-framework-symfony-bundle-system)
* [Application Directory Structure](architecture-principles.md#dev-guide-application-web-framework-symfony-application-directory-structure)
* [Application Configuration](architecture-principles.md#dev-guide-application-web-framework-symfony-application-configuration)
* [Templating System](architecture-principles.md#dev-guide-application-web-framework-symfony-templating-system)
* [Security System (Data Access Management)](architecture-principles.md#dev-guide-application-web-framework-symfony-security-system-data-access-management)
* [Databases Management (Doctrine ORM)](architecture-principles.md#dev-guide-application-web-framework-symfony-databases-management-doctrine-orm)
* [CLI Application](architecture-principles.md#dev-guide-application-web-framework-symfony-cli-application)
* [List of Symfony Components Used in Oro Applications](architecture-principles.md#dev-guide-application-web-framework-symfony-list-of-symfony-components-used-in-oro-applications)

## Getting Started

If you want to adjust Oro application functionality, the **first step** to organizing your changes should always be to
[Prepare Your Custom Application]():

* [Create a Custom Application]() and
* [Create a New Bundle]().

**Next steps** depend on whether you’re going to [Create a New Functionality]() or [Change an Existing Functionality]().

**Finally**, if you’re ready to pack and share your adjustments to the Oro application, you can
[Create and Publish an Extension]() to the <a href="https://extensions.oroinc.com/oroplatform/" target="_blank">Oro Extensions Store</a>.

### Prepare Your Custom Application

#### Create a Custom Application

Please see the [Create Custom Oro Application](../custom-application.md#dev-cookbook-create-custom-oro-application) cookbook article.

#### Create a New Bundle

Please see the [How To Create New Bundle](../../extension/create-bundle.md#dev-cookbook-framework-how-to-create-new-bundle) article.

### Create a New Functionality

Please see the basic example on how to create a new functionality in the
[Create a Simple CRUD](../../entities/crud.md#dev-cookbook-framework-create-simple-crud) article.

### Change an Existing Functionality

If you want to change the existing behavior of the Oro application, please refer to the
[Oro Application Customization](../customization/index.md#architecture-customization-customize) section of the Architecture Guide
for the guidance.

### Create and Publish an Extension

If you are ready to publish your adjustment in the Oro application for free or paid usage of community members, see the
[How to Add an Extension to the OroPlatform Extensions Store](../../extension/add-extension.md#dev-cookbook-framework-how-to-add-extension-to-marketplace)
article for the details on how to do this.

## Related Cookbook Articles

* [Create Custom Oro Application](../custom-application.md#dev-cookbook-create-custom-oro-application)
* [Create a Simple CRUD](../../entities/crud.md#dev-cookbook-framework-create-simple-crud)
* [How to Create a New Bundle](../../extension/create-bundle.md#dev-cookbook-framework-how-to-create-new-bundle)
* [How to Add an Extension to the Oro Extensions Store](../../extension/add-extension.md#dev-cookbook-framework-how-to-add-extension-to-marketplace)
* [How to Manage OroPlatform Extensions](../../extension/install-extension.md#dev-cookbook-framework-how-to-manage-extensions)

<!-- Frontend -->
