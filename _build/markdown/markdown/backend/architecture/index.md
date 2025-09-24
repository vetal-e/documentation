#### NOTE
<a id="architecture-guide"></a>

# Application Architecture

## What is Oro Application

Oro application is a PHP web application that uses the Symfony framework. It provides the following benefits:

* **Runs on any OS**, although Linux is recommended. See [system requirements](../setup/system-requirements/index.md#system-requirements) for more information.
* **Scalable** - Oro application can be easily scaled up and down to meet your company needs (message queue, indexes, search).
* **Extendable** - Oro application can be extended via the packages from the <a href="https://extensions.oroinc.com/oroplatform/" target="_blank">Oro Extensions Store</a> designed by Oro, Oro partners, or the Oro community. Also, you can design your own packages to implement additional functionality.
* **Customizable** - The Oro application inherits most of the development techniques enabled by the Symfony framework and extends them. It helps quickly customize the Oro application for any business needs.

With these out-of-the-box benefits, developers can focus on implementing their unique business logic and build Oro-based applications in less time.

### Oro Licensing

Community versions of Oro applications are distributed under the <a href="http://opensource.org/licenses/OSL-3.0" target="_blank">OSL-3.0</a> license. The community edition of OroPlatform is distributed under the <a href="https://opensource.org/licenses/MIT" target="_blank">MIT</a> license. Enterprise editions of Oro applications are distributed under a custom End User License Agreement.

<a id="architecture-overview"></a>

## Architecture Overview

* [Technology Stack](tech-stack/index.md)
  * [Client Side](tech-stack/index.md#client-side)
    * [Web Browser](tech-stack/index.md#web-browser)
    * [API Client](tech-stack/index.md#api-client)
  * [Server Side](tech-stack/index.md#server-side)
    * [Oro PHP Application](tech-stack/index.md#oro-php-application)
    * [Web Server and PHP](tech-stack/index.md#web-server-and-php)
    * [Database and RDBMS](tech-stack/index.md#database-and-rdbms)
    * [File Storage](tech-stack/index.md#file-storage)
    * [Session Storage](tech-stack/index.md#session-storage)
    * [Message Queue](tech-stack/index.md#message-queue)
    * [Search Engine](tech-stack/index.md#search-engine)
    * [Cache Storage](tech-stack/index.md#cache-storage)
    * [Notes on Deployment Options](tech-stack/index.md#notes-on-deployment-options)
* [Application Structure](structure/index.md)
  * [Source Code Repositories](structure/index.md#source-code-repositories)
  * [Distribution Model](structure/index.md#distribution-model)
  * [File System Structure](structure/index.md#file-system-structure)
    * [Oro PHP Application File System Structure](structure/index.md#oro-php-application-file-system-structure)
    * [Oro Package File System Structure](structure/index.md#oro-package-file-system-structure)
  * [File System Permissions](structure/index.md#file-system-permissions)
* [Application Framework](framework/index.md)
  * [How It Works](framework/index.md#how-it-works)
  * [Getting Started](framework/index.md#getting-started)
    * [Prepare Your Custom Application](framework/index.md#prepare-your-custom-application)
    * [Create a New Functionality](framework/index.md#create-a-new-functionality)
    * [Change an Existing Functionality](framework/index.md#change-an-existing-functionality)
    * [Create and Publish an Extension](framework/index.md#create-and-publish-an-extension)
  * [Related Cookbook Articles](framework/index.md#related-cookbook-articles)
    * [Architecture Principles of Oro Applications](framework/architecture-principles.md)
* [Application Customization](customization/index.md)
  * [Customize the Source Code](customization/index.md#customize-the-source-code)
    * [Prepare for Source Code Customization](customization/index.md#prepare-for-source-code-customization)
    * [Implement the Customization](customization/index.md#implement-the-customization)
    * [Publish Your Complete Customization as a Package on the Oro Extensions Store](customization/index.md#publish-your-complete-customization-as-a-package-on-the-oro-extensions-store)
  * [Customize via UI](customization/index.md#customize-via-ui)
  * [Related Articles](customization/index.md#related-articles)
* [Differences to Common Symfony Applications](differences.md)
  * [The Application Kernel](differences.md#the-application-kernel)
  * [Routing Configuration](differences.md#routing-configuration)
  * [Access Control Lists](differences.md#access-control-lists)
  * [Extension Management](differences.md#extension-management)
  * [Related Articles](differences.md#related-articles)
* [Custom Oro Application](custom-application.md)
  * [Application Repository and Installation](custom-application.md#application-repository-and-installation)
  * [Application Custom Code](custom-application.md#application-custom-code)
  * [Application Deployment](custom-application.md#application-deployment)
  * [Related Articles](custom-application.md#related-articles)
* [Bundle-less Structure](bundle-less-structure.md)
  * [Application-level Structure Changes](bundle-less-structure.md#application-level-structure-changes)
  * [Moving Existing Bundle to Bundle-less Structure](bundle-less-structure.md#moving-existing-bundle-to-bundle-less-structure)
    * [Migrations](bundle-less-structure.md#migrations)
    * [Fixtures](bundle-less-structure.md#fixtures)
    * [Entity](bundle-less-structure.md#entity)
    * [Controllers](bundle-less-structure.md#controllers)
    * [Views](bundle-less-structure.md#views)
    * [Translations](bundle-less-structure.md#translations)
    * [Gridview](bundle-less-structure.md#gridview)
    * [Search](bundle-less-structure.md#search)
    * [Navigation](bundle-less-structure.md#navigation)
    * [Entity](bundle-less-structure.md#id1)
    * [ACLS](bundle-less-structure.md#acls)
    * [API](bundle-less-structure.md#api)
    * [Channels](bundle-less-structure.md#channels)
    * [Charts](bundle-less-structure.md#charts)
    * [Workflows](bundle-less-structure.md#workflows)
    * [Processes](bundle-less-structure.md#processes)
    * [ACL Categories](bundle-less-structure.md#acl-categories)
    * [Address Format](bundle-less-structure.md#address-format)
    * [API Frontend](bundle-less-structure.md#api-frontend)
    * [Application Base Configuration](bundle-less-structure.md#application-base-configuration)
    * [Configurable Permissions](bundle-less-structure.md#configurable-permissions)
    * [Dashboards](bundle-less-structure.md#dashboards)
    * [Entity Extend](bundle-less-structure.md#entity-extend)
    * [Entity Hidden Fields](bundle-less-structure.md#entity-hidden-fields)
    * [Locale Data](bundle-less-structure.md#locale-data)
    * [Naming Format](bundle-less-structure.md#naming-format)
    * [Permissions](bundle-less-structure.md#permissions)
    * [Placeholders](bundle-less-structure.md#placeholders)
    * [System Configurations](bundle-less-structure.md#system-configurations)
    * [Cache Metadata](bundle-less-structure.md#cache-metadata)
  * [Moving Extension and Configuration](bundle-less-structure.md#moving-extension-and-configuration)
  * [Themes & Layouts](bundle-less-structure.md#themes-layouts)
    * [Asset Handling in Application Development](bundle-less-structure.md#asset-handling-in-application-development)
    * [Overriding SASS Variables](bundle-less-structure.md#overriding-sass-variables)
    * [Referencing Assets Using the asset() Function in Twig](bundle-less-structure.md#referencing-assets-using-the-asset-function-in-twig)
    * [Placement of SVG Icons (Storefront)](bundle-less-structure.md#placement-of-svg-icons-storefront)
    * [JS Modules](bundle-less-structure.md#js-modules)
    * [Admin Theme Configuration](bundle-less-structure.md#admin-theme-configuration)
  * [Tests](bundle-less-structure.md#tests)

#### NOTE
Business Tip

Are you in the process of selecting the <a href="https://oroinc.com/b2b-ecommerce/b2b-ecommerce-comparison" target="_blank">best eCommerce platform for B2B</a>? Evaluate nine alternatives on our B2B comparison page.
<!-- Frontend -->
