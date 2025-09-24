<a id="dev-guide-development-practice-setup-dev-env"></a>

<a id="doc-dev-env-best-practices"></a>

# Set Up Development Environment for OroPlatform Based Application

Please follow the steps outlined in the sections below to set up the development environment for Oro application customization tasks.

#### NOTE
Business Tip

Are you searching for the <a href="https://oroinc.com/b2b-ecommerce/b2b-ecommerce-comparison" target="_blank">best B2B eCommerce solution</a>? Utilize our platform comparison chart to weigh your alternatives.

<a id="dev-guide-development-practice-setup-dev-env-requirements"></a>

## Meet the Hardware and OS Requirements

* **Operating System**

  The recommended OS for Oro applications is Oracle Linux. However, it is possible to set up the development environment on any Linux, Mac, or Windows with WSL2.
* **Disk Drive**

  A solid-state drive (SSD) is recommended. As the Oro application uses many files (vendors, cache), an SSD speeds up everyday development operations much faster than an HDD.
* **Available RAM**

  The recommended amount of available RAM is 8Gb for most development operations (e.g., upgrading the application or updating Composer dependencies). However, 2Gb of available RAM is usually enough to run the application.

<a id="dev-guide-development-practice-setup-dev-env-prepare-tools"></a>

## Prepare Development Tools

1. <a href="https://git-scm.com/book/en/v2/Getting-Started-Installing-Git" target="_blank">Install Git</a>.
2. <a href="https://www.jetbrains.com/help/phpstorm/install-and-set-up-product.html" target="_blank">Install PHPStorm</a> as the recommended IDE.

   #### NOTE
   Although PHPStorm is recommended, it is not the required IDE for Oro application development. If you use a different IDE, skip the PHPStorm configuration steps below.
3. Configure PHPStorm:
   * Install and configure <a href="https://plugins.jetbrains.com/plugin/7219-symfony-plugin" target="_blank">Symfony plugin</a> and <a href="https://plugins.jetbrains.com/plugin/8449-oro-phpstorm-plugin" target="_blank">Oro plugin</a> by following the <a href="https://www.jetbrains.com/help/phpstorm/managing-plugins.html" target="_blank">official PHPStorm plugin management instructions</a>.
   * Exclude the following directories in PhpStorm (to avoid class duplication and indexation overhead) by right-clicking on the directory and selecting **Mark Directory As > Excluded**:
     * var/cache
     * public/bundles
   * Enable code quality checks in PHPStorm:
     * <a href="https://www.jetbrains.com/help/phpstorm/using-php-code-sniffer.html" target="_blank">Enable PHP Code Sniffer</a> (use PSR2 or Symfony2 code standards)
     * <a href="https://www.jetbrains.com/help/phpstorm/using-php-mess-detector.html" target="_blank">Enable PHP Mess Detector</a>, making sure that:
       * **Cyclomatic complexity** DOES NOT exceed the limit of **15**.
       * The limit of the **NPath complexity** is set to **200** (the default PHPMD limit).
     * <a href="https://www.jetbrains.com/help/phpstorm/using-php-cs-fixer.html" target="_blank">Enable PHP CS Fixer</a> (use <a href="https://github.com/oroinc/platform/blob/master/build/.php-cs-fixer.php" target="_blank">PHP CS Fixer settings</a> )
   * Configure xDebug
     * Integrate a debugging tool into your IDE that provides a range of features to improve the PHP development experience. See <a href="https://www.jetbrains.com/help/phpstorm/configuring-xdebug.html" target="_blank">how to set up integration between PhpStorm and xDebug</a> for more information.
   * Configure PhpUnit
     * If you write tests for your code, integrate <a href="https://confluence.jetbrains.com/display/PhpStorm/PHPUnit+support+in+PhpStorm" target="_blank">PhpUnit with PhpStorm</a> and use it <a href="https://www.jetbrains.com/help/phpstorm/testing-with-phpunit.html" target="_blank">for actual testing</a>.

     #### NOTE
     You can also set the default configuration for the PhpUnit test runner (path to phpunit.xml, the working directory, etc.). Then you can right-click a test file and select **Run <file>** to run all tests from the file.

<a id="dev-guide-development-practice-setup-dev-env-create-app"></a>

## Create a Custom Application

1. Fork Oro application repository.

   Use the <a href="https://docs.github.com/en/get-started/quickstart/fork-a-repo" target="_blank">Github guide on forking a repo</a> as an illustration of how to fork the Oro application repository.

   #### NOTE
   Pay attention to the <a href="https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/working-with-forks/syncing-a-fork" target="_blank">Keep your fork synced</a> section of this Github guide. You have to set the original Oro application repository as the remote upstream in order to be able to pull improvements and fixes from the original Oro application.
2. (optional) Change the **README.md** file in your repo to describe your application.
3. (optional) Change the package name of your application in the **composer.json** file.

#### IMPORTANT
Please be aware that in accordance with the [Oro PHP Application structure](../../architecture/structure/index.md#architecture-oro-php-application-structure), you have to use only the following folders and files to place your code in your custom application:

* **src**: the main folder for your customization code
* **templates**: the folder for template files
* **config**: folder the folder for config files
* **translations**: the folder for translation files
* **README.MD**: the file for the description of your custom application
* **composer.json**: the file which you can change if you want to <a href="https://symfonycasts.com/screencast/question-answer-day/create-composer-package" target="_blank">make a package</a> from your custom application

<a id="dev-guide-development-practice-setup-dev-env-setup-env"></a>

## Set Up Application Environment

We recommend using [Docker and Symfony Server](docker-and-symfony/index.md#setup-dev-env-docker-symfony) to set up the environment for your custom Oro application.

#### HINT
There are quick guides to set up the Docker and Symfony Server development stack:

- [Setup on Mac OS X](mac.md#setup-dev-env-docker-symfony-mac)
- [Setup on Ubuntu 24.04 LTS](ubuntu.md#setup-dev-env-docker-symfony-ubuntu)
- [Setup on Windows Subsystem for Linux (WSL) 2](windows.md#setup-dev-env-docker-symfony-windows)

Alternatively, to have a fully dockerized environment, you can use <a href="https://github.com/kiboko-labs/kloud" target="_blank">Docker images and stacks for OroPlatform based applications by the Kiboko team</a>.

To check that the environment meets the application requirements, use the `oro:check-requirements` command:

```none
php bin/console oro:check-requirements
```

By default, this command shows only errors, but you can increase the verbosity to see warnings and informational messages too:

```none
php bin/console oro:check-requirements -v
```

```none
php bin/console oro:check-requirements -vv
```

The command will return 0 on exit if all application requirements are met and 1 if some of the requirements are not fulfilled.

<a id="dev-guide-development-practice-setup-dev-env-install-app"></a>

## Install Your Application

When the environment is set up, follow the instructions in the [Installation Guide](../installation.md#install-for-dev) to install your application.

#### NOTE
If you use [Docker and Symfony Server](docker-and-symfony/index.md#setup-dev-env-docker-symfony), follow [this guide](docker-and-symfony/index.md#setup-dev-env-docker-symfony-install-application).

<a id="dev-guide-development-practice-setup-dev-env-create-bundle"></a>

## Create a Custom Bundle

All OroPlatform-based applications have unique features that facilitate smooth development routines, like autoregistration of bundles and configuration files, for example.

However, these features assume that all application code is **organized in bundles**. For this reason, you have to create your own bundle for your custom code to perform customization tasks.

Please, follow the [How to Create a New Bundle](../../extension/create-bundle.md#how-to-create-new-bundle) cookbook article to create a bundle in your custom application.

#### NOTE
A **priority** parameter of your bundle should be greater than **210** to make the bundle loaded after all Oro application bundles and to allow override of configuration files from Oro bundles.

<!-- Frontend -->
