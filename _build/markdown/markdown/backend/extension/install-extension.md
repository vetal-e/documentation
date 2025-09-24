<a id="admin-package-manager"></a>

<a id="dev-cookbook-framework-how-to-manage-extensions"></a>

<a id="cookbook-extensions-composer"></a>

<a id="index-0"></a>

# Install Extension from the Oro Extensions Store

#### NOTE
Before installing an extension it is recommended to back up the database and the application
source code. There is no simple way to uninstall an extension.

Installing an extension from the Oro Extensions Store is the least resource-consuming way to expand the existing functionality of the Oro application.

Oro application’s extensions store is a catalog service for sharing packages that extend a particular Oro application. On the Extensions Store, Oro and third-party vendors may publish free or chargeable custom packages to distribute commonly-used extension solutions to the Oro community.

#### NOTE
See the [Oro PHP application structure](../architecture/structure/index.md#architecture-oro-php-application-structure) topic for more information on the definition of a package and levels of extension and customization.

Browse published extensions for Oro applications on the following Extensions Stores:

* OroPlatform — <a href="https://extensions.oroinc.com/oroplatform/" target="_blank">https://extensions.oroinc.com/oroplatform/</a>
* OroCRM — <a href="https://extensions.oroinc.com/orocrm/" target="_blank">https://extensions.oroinc.com/orocrm/</a>
* OroCommerce — <a href="https://extensions.oroinc.com/orocommerce/" target="_blank">https://extensions.oroinc.com/orocommerce/</a>

#### NOTE
Once the Oro application extension package is [published on the Oro Extensions Store](add-extension.md#dev-extend-how-to-publish-extension-on-the-marketplace), it is automatically registered in the <a href="https://packagist.oroinc.com/" target="_blank">Oro Packagist repository</a>. See a topic on a [Distribution Model](../architecture/structure/index.md#architecture-oro-php-application-structure) for more information on using composer service with Packagist and OroPackagist repositories.

You can install extensions from the command-line.

Start with upgrading Composer to the latest version. This may be needed in case the extension to be
installed uses some bleeding edge feature in its `composer.json` file:

```none
$ composer self-update
# add and download extension
```

Then, install the extension’s Composer package using the Composer `require` command:

```none
$ composer require <extension name> --prefer-dist --update-no-dev
```

#### NOTE
Find the required extension name on the extension view page on the Oro Extensions Store website.

![Classes of OroImportExportBundle](img/backend/extension/extension_name.png)

Next, remove old cache:

```none
sudo rm -rf var/cache/prod
```

Repeat this for any other extension you want to install. When you are finished with adding new
packages, use the `oro:platform:update` command to make the application aware of the newly
installed extensions:

```bash
php bin/console oro:platform:update --env=prod --force
```

Finally, make sure to properly clean the cache:

```bash
php bin/console cache:clear --env=prod
```

<!-- Frontend -->
