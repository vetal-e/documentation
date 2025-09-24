<a id="deploy-the-update"></a>

# Deploy Application Changes to a New Environment (Testing or Production)

This guide explains how to deploy changes for Oro applications.

#### HINT
The instructions below will help you update a self-hosted application instance. If your application is hosted on OroCloud, follow the OroCloud guide to run the automated <a href="https://doc.oroinc.com/cloud/maintenance/basic-use/#upgrade" target="_blank">upgrade</a>.

#### WARNING
Ensure that you have the recent database and source code dump to roll back changes in case of errors before processing the upgrade

An absolute path to the directory where an application is installed will be used in the guide and will
be referred to as **<application-root-folder>** further in this topic.

#### NOTE
We highly recommend running all the commands in this guide from the same user the web server runs (e.g., **nginx** or **www-data**).

To retrieve a new version and upgrade your Oro application instance:

1. Navigate to the Oro application root folder.
   ```none
   cd <application-root-folder>
   ```
2. Make sure that no changes require the database schema update.
   ```none
   php bin/console oro:entity-extend:update --dry-run --env=prod
   ```

   The platform update is possible only when the database schema is up-to-date.
3. Switch the application to the maintenance mode.
   ```none
   php bin/console lexik:maintenance:lock --env=prod
   ```
4. Stop the cron tasks.
   ```none
   crontab -e
   ```

   Comment the line below:
   ```text
   */1 * * * * /usr/bin/php <application-root-folder>/bin/console --env=prod oro:cron >> /dev/null
   ```
5. Stop all running consumers. They are usually managed by Supervisord.
6. Create backups of your database and source code.
7. Check out the new code version from the release tag of your custom application git repository or deploy the latest version from an archive provided by the developer with the new code version.
8. Remove old caches.
   ```none
   rm -rf var/cache/prod/
   ```
9. Install PHP and NodeJS dependencies, build assets and set up the right owner for the retrieved files.
   ```none
   composer install --prefer-dist --no-dev
   ```
10. Update the application state to correspond to the code changes.

> ```none
> php bin/console oro:platform:update --env=prod
> ```

> #### NOTE
> To speed up the update process, consider using `--schedule-search-reindexation` or the
> `--skip-search-reindexation` option:

> * `--schedule-search-reindexation` — postpone search reindexation process until
>   the message queue consumer is started (on step 12 below).
> * `--skip-search-reindexation` — skip search reindexation. Later, you can start it manually using commands
>   oro:search:reindex to update search index for the specified entities and oro:website-search:reindex to rebuild storefront search index.
>   See [Search Index: Indexation Process](../architecture/tech-stack/search/index.md#search-index-overview-indexation-process).
1. Remove the caches.
   ```none
   php bin/console cache:clear --env=prod
   ```

> Alternatively, run:

> > ```none
> > rm -rf var/cache/prod/
> > php bin/console cache:warmup --env=prod
> > ```
1. Enable cron.
   ```none
   crontab -e
   ```

   Uncomment the line below.
   ```text
   */1 * * * * /usr/bin/php <application-root-folder>/bin/console --env=prod oro:cron >> /dev/null
   ```
2. Switch your application back to the normal mode from the maintenance mode.
   ```none
   php bin/console lexik:maintenance:unlock --env=prod
   ```
3. Run the consumer(s) that you stopped at step 5.

   #### NOTE
   If PHP bytecode cache tools (e.g., opcache) are used, PHP-FPM (or Apache web server) should be restarted
   after the uprgade to flush cached bytecode from the previous installation.

**See Also**

* [Upgrade the Application to the Next Version](upgrade-to-new-version.md#upgrade-application).

#### NOTE
Business Tip

Looking to harness the new trend of digital commerce? Check out our guide on a <a href="https://oroinc.com/oromarketplace/b2b-marketplace/" target="_blank">B2B marketplace platform</a>.
<!-- Frontend -->
