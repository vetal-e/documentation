<a id="dev-guide-application-web-framework-symfony"></a>

# Architecture Principles of Oro Applications

<a id="dev-guide-application-web-framework-symfony-symfony-role-in-oroplatform"></a>

## Symfony Role in OroPlatform

#### NOTE
We use <a href="https://symfony.com/" target="_blank">Symfony Framework</a> v. 6.4 LTS in Oro Applications v.5.x.

Symfony is the most mature PHP framework that provides a wide range of functions typical for any web application. Symfony takes care of numerous tasks, for instance:

- Receiving a user request and transforming it into a response for the user’s browser
- Organizing the source code in a conventional structure
- Providing tools for continuous storage of application data, validation of income data against some set of rules
- etc.

For more information on Symfony, its purpose and benefits, see the following articles on the Symfony website:

* <a href="https://symfony.com/doc/6.4/introduction/from_flat_php_to_symfony.html" target="_blank">Symfony versus Flat PHP: Why is Symfony better than just opening up a file and writing flat PHP?</a>.
* <a href="https://hackernoon.com/7-good-reasons-to-use-symfony-framework-for-your-project-265f96dcf759" target="_blank">7 Good Reasons to Use Symfony Framework for Your Project</a>
* <a href="https://matthiasnoback.nl/2013/08/why-symfony-seven-facts/" target="_blank">Why Symfony? Seven Facts</a>

Using Symfony allows web applications to avoid the development of low-level components responsible for the application’s organization and focus on developing a functionality specific to a particular web application.

Oro applications have a lot in common with regular Symfony applications based on the 3rd version of the framework. So, if you are not familiar with Symfony yet - start your acquaintance with the Oro application from the *Getting Started* and *Guides* sections of the official <a href="https://symfony.com/doc/6.4/index.html" target="_blank">Symfony documentation</a>.

Many Symfony features substantially impacted the architecture of all Oro applications. Some are listed below with the description of how they were adjusted for Oro applications compared to other Symfony-based applications.

<a id="dev-guide-application-web-framework-symfony-http-request-application-flow"></a>

## HTTP Request Application Flow

Symfony framework (and its <a href="https://symfony.com/doc/6.4/components/http_kernel.html" target="_blank">HttpKernel Component</a>) determines
the main flow of the Oro web application, i.e., the transformation of user requests into responses.

This flow in Oro applications is similar to the flows of other <a href="https://symfony.com/doc/6.4/introduction/http_fundamentals.html#the-symfony-application-flow" target="_blank">Symfony applications </a>, in particular:

1. A client sends an HTTP request to the application.
2. The request is executed by the application’s front controller (index.php) file, which takes a request, boots the application, and passes the request information.
3. Internally, the application uses registered routes, controllers, event dispatchers, and event subscribers to process the request and create a response object.
4. The application turns the response object into the text headers and content, which are sent back to the client.

![Oro HTTP Request Application Flow](img/backend/architecture/request-flow.png)

This routine defines several important architectural features of Oro applications, such as using the **Front Controller** and **MVC** structural patterns and an **Event system** to organize the application functionality and manage the application flow.

<a id="dev-guide-application-web-framework-symfony-event-system"></a>

## Event System

Oro applications actively use <a href="https://symfony.com/doc/6.4/event_dispatcher.html" target="_blank">Symfony Event Dispatcher</a> to create
and use the points of functional extension and control of the application flow.

As illustrated throughout the Developer Guide, a significant part of the Oro application’s functionality allows you to adjust the application behavior by creating listeners or subscribers for system events.

<a id="dev-guide-application-web-framework-symfony-inversion-of-control-principle"></a>

## Inversion of Control Principle

The <a href="https://en.wikipedia.org/wiki/Inversion_of_control" target="_blank">Inversion of Control principle</a> is widely used in the architecture of Oro applications to loosen the coupling between classes and objects, facilitate extensibility and eliminate code duplication.

The principle is embodied in the | Symfony’s Service container| and the <a href="https://symfony.com/doc/6.4/components/dependency_injection.html" target="_blank">Dependency Injection Component</a> used to create and manage all objects and organize their interaction in Oro applications.

<a id="dev-guide-application-web-framework-symfony-bundle-system"></a>

## Bundle System

The <a href="https://symfony.com/doc/6.4/bundles.html" target="_blank">bundle system</a> is one of the main principles of feature and code organization in Oro applications. However, there are some differences in it between the Symfony framework and Oro applications.

At the beginning of Oro applications’ development, there were certain constraints with the Symfony bundle management system that we decided to address, namely:

- Installing a bundle was too cumbersome
- Removing a bundle was even more cumbersome

As a workaround, we changed the bundle registration system in Oro applications so that the bundles acquired the option of auto-registration in the application without needing to modify any of the application files.

For the bundle to be registered and enabled in Oro applications, it is sufficient to mention the bundle in its  *Resources/config/oro/bundles.yml* file. You can activate any bundle in the application by putting its primary class name in the *Resources/config/oro/bundles.yml* of your bundle (keep in mind, though, that you must physically install the bundle with the help of Composer).

#### NOTE
src/Acme/Bundle/DemoBundle/Resources/config/oro/bundles.yml
```yaml
bundles:
    - { name: Acme\Bundle\DemoBundle\AcmeDemoBundle, priority: 70 }
```

#### NOTE
For more details on creating a bundle, please refer to the [How to Create a New Bundle](../../extension/create-bundle.md#dev-cookbook-framework-how-to-create-new-bundle) topic.

<a id="dev-guide-application-web-framework-symfony-application-directory-structure"></a>

## Application Directory Structure

Oro applications follow Symfony’s recommendations to organize the structure of the application files and source code.

Please refer to the [Architecture Guide](../structure/index.md#architecture-oro-php-application-structure) for the detailed description of Oro application directory structure.

<a id="dev-guide-application-web-framework-symfony-application-configuration"></a>

## Application Configuration

Oro widely uses Symfony conventions to configure the application and specific features in YAML configuration files.

On the application level (<a href="https://symfony.com/doc/6.4/best_practices/configuration.html" target="_blank">according to Symfony conventions</a>), the configuration is divided into infrastructure-related ( *.env-app*) and application-related (*config/config.yml* file).

On the bundle level, Oro applications have slight changes in the configuration technologies but a considerable shift in the role and purpose of the configuration files.

Simultaneously with [bundle auto-registration](#dev-guide-application-web-framework-symfony-bundle-system) in the *Resources/config/oro/bundles.yml* file, the policy of auto-registration of the feature’s configuration files that follow special naming conventions was enabled.

For example, the files called

- *Resource/config/oro/api.yml*
- *Resource/config/oro/navigation.yml*
- *Resource/config/oro/datagrids.yml*

are auto-discovered and applied to the configuration of corresponding features.

The role of the configuration files underwent the most changes. In the Oro application, Yaml configuration files of many features are used not only to configure a feature but also to create parts of the application functionality.

For example, there are three ways in Oro applications to create navigation items in the UI:

1. Use admin UI to manage the navigation items.
2. Declare new navigation items in the DI service with a special dependency injection tag “oro_menu.builder”
3. Add navigation item information to the *Resources/config/oro/navigation.yml* file of your bundle:

```yaml
navigation:
    menu_config:
        items:
            new_menu_item:
                label: New Menu Item
                route: acme_demo_new

        tree:
            application_menu:
                children:
                    new_menu_item: ~
```

<a id="dev-guide-application-web-framework-symfony-templating-system"></a>

## Templating System

<a href="https://symfony.com/doc/6.4/templates.html" target="_blank">Symfony Templating</a> is widely extended in Oro Applications by the <a href="https://github.com/oroinc/platform/tree/master/src/Oro/Bundle/LayoutBundle" target="_blank">Layouts</a> concept, which allows addressing Symfony’s limitations in UI extension and composition.

However, all features of the <a href="https://twig.symfony.com/" target="_blank">TWIG templating engine</a> and <a href="https://symfony.com/doc/6.4/components/templating.html" target="_blank">Symfony Templating Component</a> are widely used in Oro
applications in UI building.

We recommend you get acquainted with the <a href="https://symfony.com/doc/6.4/templates.html" target="_blank">Symfony Templating</a> documentation to work with Oro applications.

<a id="dev-guide-application-web-framework-symfony-security-system-data-access-management"></a>

## Security System (Data Access Management)

Data Access Management in OroPlatform is based on the proprietary <a href="https://en.wikipedia.org/wiki/Role-based_access_control" target="_blank">Role Based Access Control</a> system as it is necessary for
business applications.

#### NOTE
More information on this is available in the next section of this guide which is dedicated to OroPlatform and its Security System.

However, this RBAC system of Oro applications widely uses <a href="https://symfony.com/doc/6.4/components/security.html" target="_blank">Symfony Security Components</a>. It is, therefore, vital that you familiarize yourself with them to be able to work with Oro applications.

<a id="dev-guide-application-web-framework-symfony-databases-management-doctrine-orm"></a>

## Databases Management (Doctrine ORM)

Oro applications support the storage of application data in relational databases, such as PostgreSQL,
EnterpriseDB. Support for these databases is provided by the <a href="https://www.doctrine-project.org/" target="_blank">Doctrine</a> layer.

Oro applications widely use all Doctrine features to manage the persistent data: Database Abstraction Layer,
Object Relation Mapping, Event Manager, etc.

However, Doctrine Migrations, one of the most popular Doctrine projects, was entirely replaced by proprietary. It allows for versioning management of the database schema using specific migration classes on the bundle level, not the application level.

Because the Oro application functionality is constantly evolving, it was essential to be able to extend the Doctrine data types and DQL functions. As a result, the <a href="https://github.com/oroinc/doctrine-extensions" target="_blank">Oro Doctrine Extensions</a> component was created.

<a id="dev-guide-application-web-framework-symfony-cli-application"></a>

## CLI Application

All Symfony-based applications <a href="https://symfony.com/doc/6.4/console.html" target="_blank">come with a command line interface tool</a> (bin/console) that helps you
maintain your application. It provides commands that boost your productivity by automating
tedious and repetitive tasks, such as clearing the application cache, debugging routing, viewing the list of registered commands, etc.

Oro application follows this convention and extends the set of the CLI tools by commands for:

- Database migrations management (see [Databases Management (Doctrine ORM)]() above)
- Translations management
- Application upgrading
- Periodical task management
- Unit tests generation
- etc.

To see all Oro applications console commands, run the following command in the console:

```bash
php bin/console list oro
```

<a id="dev-guide-application-web-framework-symfony-list-of-symfony-components-used-in-oro-applications"></a>

## List of Symfony Components Used in Oro Applications

Symfony is a *set of program components* and a *web framework*. Components represent separate independent parts
of the functionality used to perform typical web programming tasks, while the framework is a functionality responsible for organizing the interaction of these independent components in a web application.

Description of Symfony’s contribution to the Oro applications architecture would not be complete without
mentioning Symfony components that are actively used in the development of Oro applications:

- <a href="https://symfony.com/doc/6.4/components/asset.html" target="_blank">Asset Component</a>
- <a href="https://symfony.com/doc/6.4/components/console.html" target="_blank">Console Component</a>
- <a href="https://symfony.com/doc/6.4/components/dependency_injection.html" target="_blank">DependencyInjection Component</a>
- <a href="https://symfony.com/doc/6.4/components/event_dispatcher.html" target="_blank">EventDispatcher Component</a>
- <a href="https://symfony.com/doc/6.4/components/expression_language.html" target="_blank">ExpressionLanguage Component</a>
- <a href="https://symfony.com/doc/6.4/components/form.html" target="_blank">Form Component</a>
- <a href="https://symfony.com/doc/6.4/components/http_foundation.html" target="_blank">The HttpFoundation Component</a>
- <a href="https://symfony.com/doc/6.4/components/http_kernel.html" target="_blank">HttpKernel Component</a>
- <a href="https://symfony.com/doc/6.4/components/options_resolver.html">OptionsResolver Component</a>
- <a href="https://symfony.com/doc/6.4/components/property_access.html" target="_blank">PropertyAccess Component</a>
- <a href="https://symfony.com/doc/6.4/components/routing.html" target="_blank">Routing Component</a>
- <a href="https://symfony.com/doc/6.4/components/security.html" target="_blank">Security Component</a>
- <a href="https://symfony.com/doc/6.4/components/serializer.html" target="_blank">Serializer Component</a>
- <a href="https://symfony.com/doc/6.4/components/templating.html" target="_blank">Templating Component</a>
- <a href="https://symfony.com/doc/6.4/components/translation.html" target="_blank">Translation Component</a>
- <a href="https://symfony.com/doc/6.4/components/validator.html" target="_blank">Validator Component</a>
- <a href="https://symfony.com/doc/6.4/components/yaml.html" target="_blank">Yaml Component</a>

You must be familiar with these components to work comfortably with Oro applications.

<!-- Frontend -->
