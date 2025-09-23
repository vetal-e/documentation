<a id="dev-guide-field-sales-app-setup"></a>

# OroCommerce Field Sales Application Setup and Configuration

#### NOTE
Please <a href="https://oroinc.com/contact-us/" target="_blank">contact our support team</a> to learn more about the Field Sales Application and its implementation.

This guide outlines steps required to set up the OroCommerce Field Sales application in a development environment. The Field Sales application is a headless Progressive Web Application (PWA) built with React and powered by OroCommerce backend APIs. It is designed to help sales representatives operate in offline or low-connectivity environments by offering capabilities such as order entry, visit tracking, and digital catalog browsing.

## System Requirements

Two components are required to run the Field Sales application: an instance of the OroCommerce Enterprise application and the Field Sales application itself. You must ensure that your environment meets the system requirements for both components.

#### HINT
If you are not installing the <a href="https://github.com/oroinc/sales-frontend" target="_blank">SalesFrontendBundle</a> on your OroCommerce instance and prefer to use an already prepared OroCommerce instance (e.g., hosted on Oro Cloud), you can skip the section about the OroCommerce Enterprise system requirements.

### Field Sales Application System Requirements

* Node.js is version 22 or higher
* PNPM is version 9 or higher
* You have received access and cloned the <a href="https://github.com/oroinc/field-sales-frontend" target="_blank">Field Sales application</a> source code from GitHub. Once you have the application in place, proceed with one of the installation options described below.

  #### HINT
  This repository contains the React-based frontend application that connects to your OroCommerce backend API either locally or remotely, depending on your setup. Please contact our support team to request access.

### OroCommerce Enterprise System Requirements

The list of the system requirements for OroCommerce Enterprise application can be found [on the system requirements page](../setup/system-requirements/index.md#system-requirements).

## Installation

The process includes preparing the OroCommerce Enterprise instance and installing the Field Sales application. If you already have a running instance (such as one hosted in Oro Cloud), you can skip the preparation step for OroCommerce Enterprise.

### Prepare OroCommerce Enterprise Instance

#### HINT
If you have not yet set up a local OroCommerce Enterprise instance, please follow the steps outlined in the [Installation Guide](../setup/installation.md#install-for-dev).

Install <a href="https://github.com/oroinc/sales-frontend" target="_blank">SalesFrontendBundle</a> to your OroCommerce Enterprise instance. Use this approach if you have permission to install additional backend packages to the OroCommerce instance:

1. While being in the root directory of your OroCommerce Enterprise instance, use composer to add the package code:

```php
composer require oro/sales-frontend --prefer-dist --update-no-dev
```

1. Next, remove old cache:

```shell
rm -rf var/cache/prod
```

1. Perform the application upgrade to make the OroCommerce Enterprise application aware of the newly installed extension:

```php
php bin/console oro:platform:update  --env=prod --force
```

1. Configure the OroSalesFrontendBundle by adding the following to your config/config.yml file:

```yaml
# config/config.yml

oro_sales_frontend:
    app_base_urls:
        - '/sales-frontend' # Default URL for the Field Sales application in case it is installed under a subpath of your OroCommerce instance
        - 'https://field-sales.dev.loc' # Example URL for the Field Sales application
        - 'http://localhost:5173' # Default Vite development server URL
```

This configuration allows the OroSalesFrontendBundle to automatically set up the <a href="https://www.w3.org/TR/cors/" target="_blank">CORS</a> and <a href="https://www.w3.org/TR/CSP/" target="_blank">CSP</a> headers, enabling the Field Sales application to communicate with the OroCommerce backend API.

#### HINT
You can find more information on the possible configuration options in the OroSalesFrontendBundle configuration.

1. Clear the cache:

```php
php bin/console cache:clear --env=prod
```

#### HINT
For more information on installing extensions from the Oro Extensions Store, see [Installing an Extension](../extension/install-extension.md#cookbook-extensions-composer).

### Install the Field Sales Application

#### Configuration

Create a file named .env.development.local in the root directory of the Field Sales application worktree.

#### NOTE
The .env.development.local file is used for local development. For production builds, create .env.production.local file with equivalent values for your instance.

```shell
cp .env .env.development.local
```

If you run you OroCommerce Enterprise instance locally and the Field Sales application is installed under a subpath of the OroCommerce Enterprise public directory (e.g., /sales-frontend), it should include the following content:

```none
BUILD_DIR="dist"
VITE_BASE_URL=""
VITE_BASE_API_URL="/admin/api/"
VITE_LOGIN_URL="/admin/sales-frontend/user/login"
VITE_LOGOUT_URL="/admin/sales-frontend/user/logout"
VITE_CHECK_TOKEN_URL="/admin/sales-frontend/oauth2/check-token"
VITE_REFRESH_TOKEN_URL="/admin/sales-frontend/oauth2/refresh-token"
```

If you OroCommerce instance runs on a separate domain or hosted in the Oro Cloud, specify the absolute URLs for the OroCommerce Enterprise instance:

```none
BUILD_DIR="dist"
VITE_BASE_URL=""
VITE_BASE_API_URL="https://orocommerce.dev.loc/admin/api/"
VITE_LOGIN_URL="https://orocommerce.dev.loc/admin/sales-frontend/user/login"
VITE_LOGOUT_URL="https://orocommerce.dev.loc/admin/sales-frontend/user/logout"
VITE_CHECK_TOKEN_URL="https://orocommerce.dev.loc/admin/sales-frontend/oauth2/check-token"
VITE_REFRESH_TOKEN_URL="https://orocommerce.dev.loc/admin/sales-frontend/oauth2/refresh-token"
```

Here, https://orocommerce.dev.loc is the URL of the OroCommerce Enterprise instance you want the Field Sales application to connect to.

You can find more information on the available environment variables in the [Field Sales application configuration reference](configuration.md#dev-guide-field-sales-app-configuration-reference).

#### Installation and Operation

1. Install the dependencies using pnpm while in the root directory of the Field Sales application:

```shell
pnpm install
```

1. Build the application using:

```shell
pnpm run build --mode=development
```

This will build the application source code in the specified BUILD_DIR (default is dist). Make sure you have the web server configured to serve the files from this directory.

If you want to run the application for development or debugging purposes, or if you do not have the appropriate web server set up, you can use the Vite development server:

```shell
pnpm run dev
```

This command will start the Vite development server, which offers features like hot module replacement (HMR) and other development tools. You do not need to configure a web server to serve the application files, as Vite will take care of that during development.

<!-- Frontend -->

* [Configuration Reference](configuration.md)
