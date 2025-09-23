<a id="index-0"></a>

# Add OroCommerce Capabilities to an OroCRM Application

#### NOTE
Before installing OroCommerce over OroCRM, you should change default parameter `web_backend_prefix` to some non-empty prefix that should start with “/” but do not finish with “/”, e.g., ‘/admin’.

#### WARNING
To avoid access permissions issues, please review the Symfony <a href="https://symfony.com/doc/6.4/setup/file_permissions.html" target="_blank">Setting up or Fixing File Permissions</a>  guide before running any commands. On top of that, consider running the command(s) below with sudo -u [web server user name] prefix.

To install OroCommerce and OroCRM from scratch, please [install OroCommerce application](../setup/installation.md#installation) that has OroCRM capabilities embedded out-of the-box.

To add OroCommerce to an existing instance of OroCRM, please follow the ordinary [OroCRM upgrade process](../setup/upgrade-to-new-version.md#upgrade-application) and ensure you add the OroCommerce package as a dependency during the step 5. Once the upgrade process is complete, please run following commands to add necessary initial OroCommerce configuration:

```bash
php bin/console oro:config:update oro_website.url https://unsecure.url
php bin/console oro:config:update oro_website.secure_url http://secure.url
```

where http://unsecure.url and https://secure.url are urls for the OroCommerce storefront. The `oro:config:update` command updates a configuration value in the global scope.

<!-- Frontend -->
