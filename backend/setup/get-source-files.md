<a id="platform-installation-source-files"></a>

<a id="installation-get-files"></a>

# Get the Oro Application Source Code

You can obtain the application source code and the required dependencies in one of the following ways:

* [Create a project with composer](#platform-installation-composer-create-project).
* [Clone the GitHub repository and install dependencies](#platform-installation-github-clone).
* [Download the source code archive](#platform-installation-download-archive).

These methods are detailed below.

<a id="platform-installation-composer-create-project"></a>

## Method 1: Create a Project with Composer

1. Make sure you use PHP >=8.4 and have Composer installed. If you do not, use the Composer
   installation process described in the <a href="https://getcomposer.org/doc/00-intro.md#installation-linux-unix-osx" target="_blank">Composer installation documentation</a>.
2. Create your new Oro application project with Composer by running one of the commands below, depending on the base application you want to install:
   ```none
   # OroCommerce Community Edition
   composer create-project oro/commerce-crm-application my_project_name 6.1.0
   # OroCommerce Enterprise Edition
   composer create-project oro/commerce-crm-enterprise-application my_project_name 6.1.0 --repository=https://packagist.oroinc.com
   # OroCRM Community Edition
   composer create-project oro/crm-application my_project_name 6.1.0
   # OroCRM Enterprise Edition
   composer create-project oro/crm-enterprise-application my_project_name 6.1.0 --repository=https://packagist.oroinc.com
   # OroPlatform Community Edition
   composer create-project oro/platform-application my_project_name 6.1.0
   # OroCommerce Community Edition for Germany
   composer create-project oro/commerce-crm-application-de oroapp my_project_name 6.1.0
   # OroCommerce Enterprise Edition for Germany
   composer create-project oro/commerce-crm-enterprise-application-de my_project_name 6.1.0 --repository=https://packagist.oroinc.com
   # OroCommerce Enterprise Edition (without CRM)
   composer create-project oro/commerce-enterprise-application my_project_name 6.1.0 --repository=https://packagist.oroinc.com
   ```

   * Replace the `6.1.0` with the version to download.
   * This command creates a new directory called my_project_name/ that contains an empty project.

<a id="platform-installation-github-clone"></a>

<a id="clone-the-github-repository"></a>

## Method 2: Use the GitHub Repository

1. Clone the Oro application GitHub repository by running one of the commands below:
   ```none
   # OroCommerce Community Edition
   git clone -b 6.1.0 https://github.com/oroinc/orocommerce-application my_project_name
   # OroCommerce Enterprise Edition
   git clone -b 6.1.0 https://github.com/oroinc/orocommerce-enterprise-application my_project_name
   # OroCommerce Platform Application
   git clone -b 5.0.0 https://github.com/oroinc/orocommerce-platform-application my_project_name
   # OroCRM Community Edition
   git clone -b 6.1.0 https://github.com/oroinc/crm-application my_project_name
   # OroCRM Enterprise Edition
   git clone -b 6.1.0 https://github.com/oroinc/crm-enterprise-application my_project_name
   # OroPlatform Community Edition
   git clone -b 6.1.0 https://github.com/oroinc/platform-application my_project_name
   # OroCommerce Community Edition for Germany
   git clone -b 6.1.0 https://github.com/oroinc/orocommerce-application-de my_project_name
   # OroCommerce Enterprise Edition for Germany
   git clone -b 6.1.0 https://github.com/oroinc/orocommerce-enterprise-application-de my_project_name
   # OroCommerce Enterprise Edition (without CRM)
   git clone -b 6.1.0 https://github.com/oroinc/orocommerce-enterprise-nocrm-application my_project_name
   ```

   * Replace the `6.1.0` with the version to download.
   * `my_project_name` is the directory into which you need to clone the application source files.
2. Run the `composer install` command with `--prefer-dist --no-dev` parameter to install all Oro application
   dependencies:
   ```none
   $ cd <application-root-folder>
   $ composer install --prefer-dist --no-dev
   ```

   Note that you can find the description for every environment variable in the [Infrastructure-related Oro Application Configuration](dev-environment/parameters-yml.md#installation-parameters-yml-description) article.

<a id="platform-installation-download-archive"></a>

## Method 3: Download the Source Code Archive

1. Download the latest version of the application source code from the download section on <a href="http://www.oroinc.com/" target="_blank">the website</a> (you may be required to fill in a form to request access):
   * <a href="https://oroinc.com/b2b-ecommerce/download/#source" target="_blank">Download OroCommerce</a>
   * <a href="https://oroinc.com/orocrm/download/#source" target="_blank">Download OroCRM</a>
   * <a href="https://oroinc.com/oroplatform/download/#source" target="_blank">Download OroPlatform</a>

   Click the **download zip**, **download tar.gz**, or **download tar.bz2** link to download the archive.

   #### NOTE
   You can also download the **virtual machine** to quickly [deploy the application in the virtual sandbox environment](demo-environment/vm.md#virtual-machine-deployment).

   Then extract the source files. For example, on a Linux-based OS run:
   ```none
   $ cd <application-root-folder>
   $ tar -xzvf crm-application.tar.gz
   ```
2. All required dependencies are already installed in the vendor folder in the extracted archive.

   #### WARNING
   If necessary, update the configuration parameters in the `.env-app.local` file once the command execution is complete.

<!-- Frontend -->
