<a id="book-entities-database-schema-update"></a>

<a id="backend-entities-migrations"></a>

# Database Structure Migrations

Each bundle can have migration files that enable you to update the database schema.

To create a schema (database structure) migration, follow the steps below.

## Create Schema Migration

### Create Database Dump

It is required to create a database dump before any database changes.

### Synchronize Code with the Database

After you have modeled your entities, you need to update the database schema. To update the schema, use the `doctrine:schema:update command`. Use the `--dump-sql` option first to make sure that Doctrine makes the expected changes:

<!-- .. code-block:: none -->
<!-- php bin/console doctrine:schema:update --dump-sql -->

Double-check the configured mapping information and rerun the command if the command displays unexpected information.

When everything is displayed as expected, update the database schema by passing the `--force` option:

```none
php bin/console doctrine:schema:update --force
```

<a id="installer-generate"></a>

### Generate an Installer

#### Generate an Installer for a Bundle

When you have implemented new entities, ensure that the entities are added to the database on installing the application. For this, you need to create an installer [migration](#backend-entities-migrations). You can do it manually, however, it is more convenient to use a database dump as a template.

To create an installer for AcmeDemoBundle:

1. Clear the application cache:
   ```bash
   php bin/console cache:clear
   ```
2. Apply the changes that you defined in your code to the database:
   ```bash
   php bin/console doctrine:schema:update
   ```
3. Generate an installer and save it to the AcmeDemoBundleInstaller.php:
   ```bash
   php bin/console oro:migration:dump --bundle=AcmeDemoBundle
   ```

The generated AcmeDemoBundleInstaller.php will be placed into the AcmeDemoBundle/Migrations/Schema directory.

1. Reinstall your application instance.
2. Check that the database is synced with your code:
   ```bash
   php bin/console doctrine:schema:update --dump-sql
   ```

   If the database is successfully synchronized, you will see the following message:
   ```none
   Nothing to update - your database is already in sync with the current entity metadata.
   ```

### Migrations Dump Command

Use the **oro:migration:dump** command to help create installation files. This command outputs the current database structure as plain SQL or as `Doctrine\DBAL\Schema\Schema` queries.

This command supports the following additional options:

- **plain-sql** - Outputs schema as plain SQL queries
- **bundle** - The bundle name for which the migration is generated
- **migration-version** - Migration version number. This option sets the value returned by the getMigrationVersion method of the generated installation file.

Each bundle can have an **installation** file. This migration file replaces running multiple migration files. Install migration class must implement the <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/MigrationBundle/Migration/Installation.php" target="_blank">Installation</a> interface and the up and getMigrationVersion methods. The getMigrationVersion method must return the max migration version number that this installation file replaces.

When an install migration file is found during the install process (when you install the system from scratch), it is loaded first, followed by the migration files with versions greater than the version returned by the getMigrationVersion method.

For example, let’s assume we have migrations v1_0, v1_1, v1_2, v1_3 and installed the migration class. This class returns v1_2 as the migration version. That is why, during the installation process, the install migration file is loaded first, followed only by the migration file v1_3. In this case, migrations from v1_0 to v1_2 are not loaded.

Below is an example of an install migration file:

```php
namespace Acme\Bundle\DemoBundle\Migrations\Schema;

use Doctrine\DBAL\Schema\Schema;
use Oro\Bundle\MigrationBundle\Migration\Installation;
use Oro\Bundle\MigrationBundle\Migration\QueryBag;

/**
 * Creates all tables required for the bundle.
 */
class AcmeDemoBundleInstaller implements
    Installation
{
    #[\Override]
    public function getMigrationVersion()
    {
        return 'v1_0';
    }

    #[\Override]
    public function up(Schema $schema, QueryBag $queries)
    {
        /** Tables generation **/
        $this->createAcmeDemoPriorityTable($schema);
        $this->createAcmeDemoDocumentTable($schema);
        /** Foreign keys generation **/
        $this->addAcmeDemoPriorityForeignKeys($schema);
        $this->addAcmeDemoDocumentForeignKeys($schema);
        $this->addAcmeDemoNotManageableEntity($schema);
    }

    private function createAcmeDemoPriorityTable(Schema $schema): void
    {
        $table = $schema->createTable('acme_demo_priority');
        $table->addColumn('id', 'integer', ['autoincrement' => true]);
        $table->addColumn('label', 'string', ['length' => 255]);
        $table->setPrimaryKey(['id']);
        $table->addUniqueIndex(['label'], 'uidx_label_doc');
    }

    private function createAcmeDemoDocumentTable(Schema $schema): void
    {
        $table = $schema->createTable('acme_demo_document');
        $table->addColumn('id', 'integer', ['autoincrement' => true]);
        $table->addColumn('subject', 'string', ['length' => 255]);
        $table->addColumn('description', 'string', ['length' => 255]);
        $table->addColumn('organization_id', 'integer', ['notnull' => false]);
        $table->addColumn('priority_id', 'integer', ['notnull' => false]);
        $table->setPrimaryKey(['id']);
    }

    private function addAcmeDemoDocumentForeignKeys(Schema $schema): void
    {
        $table = $schema->getTable('acme_demo_document');
        $table->addForeignKeyConstraint(
            $schema->getTable('acme_demo_priority'),
            ['priority_id'],
            ['id'],
            ['onUpdate' => null, 'onDelete' => 'SET NULL']
        );
    }
}
```

### Create Versioned Schema Migrations

A good practice is for a bundle to have the installation file for the current version and migration files for migrating from the previous to the current version.

Migration files should be located in the `Migrations\Schema\version_number` folder. A version number must be a PHP-standardized version number string but with some limitations. This string must not contain “.” and “+” characters as a version parts separator. You can find more information about PHP-standardized version number string in the <a href="http://php.net/manual/en/function.version-compare.php" target="_blank">PHP manual</a>.

Each migration class must implement the <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/MigrationBundle/Migration/Migration.php" target="_blank">Migration</a> interface and the up method. This method receives a current database structure in the schema and queries parameters, adding additional queries.

With the schema parameter, you can create or update the database structure without fear of compatibility between database engines.
You can use the’ queries’ parameter if you want to execute additional SQL queries before or after applying a schema modification. This parameter represents a <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/MigrationBundle/Migration/QueryBag.php" target="_blank">query bag</a> and allows adding additional queries, which will be executed before (addPreQuery method) or after (addQuery or addPostQuery method). A query can be a string or an instance of a class that implements <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/MigrationBundle/Migration/MigrationQuery.php" target="_blank">MigrationQuery</a> interface. There are several ready-to-use implementations of this interface:

> - <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/MigrationBundle/Migration/SqlMigrationQuery.php" target="_blank">SqlMigrationQuery</a> - represents one or more SQL queries
> - <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/MigrationBundle/Migration/ParametrizedSqlMigrationQuery.php" target="_blank">ParametrizedSqlMigrationQuery</a> - similar to the previous class, but each query can have its own parameters.

If you need to create your own implementation of the <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/MigrationBundle/Migration/MigrationQuery.php" target="_blank">MigrationQuery</a>, implement <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/MigrationBundle/Migration/ConnectionAwareInterface.php" target="_blank">ConnectionAwareInterface</a> in your migration query class if you need a database connection. You can also use the <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/MigrationBundle/Migration/ParametrizedMigrationQuery.php" target="_blank">ParametrizedMigrationQuery</a> class as the base class for your migration query.

If you have several migration classes within the same version and you need to make sure that they are executed in a specific order, use <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/MigrationBundle/Migration/OrderedMigrationInterface.php" target="_blank">OrderedMigrationInterface</a>.

Below is an example of a migration file:

```php
 namespace Acme\Bundle\DemoBundle\Migrations\Schema\v1_1;

 use Doctrine\DBAL\Schema\Schema;
 use Oro\Bundle\MigrationBundle\Migration\Migration;
 use Oro\Bundle\MigrationBundle\Migration\QueryBag;

 class AddTmpTestTable implements Migration
 {
    #[\Override]
     public function up(Schema $schema, QueryBag $queries)
     {
         $table = $schema->createTable('tmp_test_table');
         $table->addColumn('id', 'integer', ['autoincrement' => true]);
         $table->addColumn('created', 'datetime', []);
         $table->addColumn('field', 'string', ['length' => 500]);
         $table->addColumn('another_field', 'string', ['length' => 255]);
         $table->addColumn('test_column', 'json', []);
         $table->setPrimaryKey(['id']);
     }
 }
```

<!-- You can use the following algorithm for new versions of your bundle:

- Create a new migration.
- Apply it with **oro:migration:load**.
- Generate a new installation file with **oro:migration:dump**.
- If required, add migration extension calls to the generated installation. -->

### Restore the Database Dump

In case of problems or a database crash, you can restore the database dump.

### Load Schema Migrations

To run migrations, use the **oro:migration:load** command. This command collects migration files from bundles, sorts them by their version number, and applies changes.

This command supports the following additional options:

- **force** - Causes the generated by migrations SQL statements to be physically executed against your database;
- **dry-run** - Outputs list of migrations without applying them;
- **show-queries** - Outputs list of database queries for each migration file;
- **bundles** - A list of bundles from which to load data. If the option is not set, migrations are taken from all bundles;
- **exclude** - A list of bundle names where migrations should be skipped.

## Examples of Database Structure Migrations

- <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/UserBundle/Migrations/Schema/v1_0/OroUserBundle.php" target="_blank">Simple migration</a>
- <a href="https://github.com/oroinc/platform/tree/master/src/Oro/Bundle/InstallerBundle/Migrations/Schema" target="_blank">Installer</a>
- <a href="https://github.com/oroinc/orocommerce/blob/master/src/Oro/Bundle/ProductBundle/Migrations/Schema/OroProductBundleInstaller.php" target="_blank">Complex migration</a>

## Extensions for Database Structure Migrations

You cannot always use standard Doctrine methods to modify the database structure. For example, `Schema::renameTable` does not work because it drops an existing table and then creates a new one. To help you manage such a case and enable you to add additional functionality to any migration, use the extensions mechanism. The following example illustrates how <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/MigrationBundle/Migration/Extension/RenameExtension.php" target="_blank">RenameExtension</a> can be used:

```php
 namespace Acme\Bundle\DemoBundle\Migrations\Schema\v1_2;

 use Doctrine\DBAL\Schema\Schema;
 use Oro\Bundle\MigrationBundle\Migration\Migration;
 use Oro\Bundle\MigrationBundle\Migration\QueryBag;
 use Oro\Bundle\MigrationBundle\Migration\Extension\RenameExtensionAwareInterface;
 use Oro\Bundle\MigrationBundle\Migration\Extension\RenameExtensionAwareTrait;

 class TestRenameTable implements Migration, RenameExtensionAwareInterface
 {
     use RenameExtensionAwareTrait;

    public function up(Schema $schema, QueryBag $queries)
    {
        $this->renameExtension->renameTable(
            $schema,
            $queries,
            'tmp_test_table',
            'new_test_table'
        );
    }
 }
```

As you can see from the example above, your migration class should implement <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/MigrationBundle/Migration/Extension/RenameExtensionAwareInterface.php" target="_blank">RenameExtensionAwareInterface</a> and setRenameExtension method in order to use the <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/MigrationBundle/Migration/Extension/RenameExtension.php" target="_blank">RenameExtension</a>.

Another example below illustrates how to use database-specific features in migration:

```php
namespace Acme\Bundle\DemoBundle\Migrations\Schema\v1_3;

use Doctrine\DBAL\Platforms\PostgreSQL94Platform;
use Doctrine\DBAL\Schema\Schema;
use Oro\Bundle\MigrationBundle\Migration\Extension\DatabasePlatformAwareInterface;
use Oro\Bundle\MigrationBundle\Migration\Extension\DatabasePlatformAwareTrait;
use Oro\Bundle\MigrationBundle\Migration\Migration;
use Oro\Bundle\MigrationBundle\Migration\QueryBag;
use Oro\Bundle\MigrationBundle\Migration\SqlMigrationQuery;

class CreateFunctionalIndex implements Migration, DatabasePlatformAwareInterface
{
    use DatabasePlatformAwareTrait;

    public function up(Schema $schema, QueryBag $queries)
    {

        if ($this->platform instanceof PostgreSQL94Platform) {
            $query = new SqlMigrationQuery(
                "CREATE INDEX test_idx1 ON new_test_table (LOWER(test_column->>'test_key'))"
            );
        } else {
            $query = new SqlMigrationQuery(
                "CREATE INDEX test_idx1 ON new_test_table ((LOWER(JSON_VALUE(test_column, '\$.test_key'))))"
            );
        }

        $queries->addPostQuery($query);
    }
}
```

You can also use the following additional interfaces in your migration class:

- ContainerAwareInterface - provides access to Symfony dependency container.
- <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/MigrationBundle/Migration/Extension/DatabasePlatformAwareInterface.php" target="_blank">DatabasePlatformAwareInterface</a> - allows to write database type independent migrations.
- <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/MigrationBundle/Migration/ConnectionAwareInterface.php" target="_blank">ConnectionAwareInterface</a> - provides access to the database connection.
- <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/MigrationBundle/Migration/Extension/NameGeneratorAwareInterface.php" target="_blank">NameGeneratorAwareInterface</a> - provides access to the <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/MigrationBundle/Tools/DbIdentifierNameGenerator.php" target="_blank">DbIdentifierNameGenerator</a> class used to generate names of indices, foreign key constraints, etc.

Here is a list of available extensions:

**Commerce**

* <a href="https://github.com/oroinc/orocommerce/blob/master/src/Oro/Bundle/PaymentTermBundle/Migration/Extension/PaymentTermExtension.php" target="_blank">PaymentTermExtension</a> - Adds payment term association to the entity.
* <a href="https://github.com/oroinc/orocommerce/blob/master/src/Oro/Bundle/RedirectBundle/Migration/Extension/SlugExtension.php" target="_blank">SlugExtension</a> - Adds slugs to the entity. More information is available in the RedirectBundle documentation.
* <a href="https://github.com/oroinc/crm/blob/master/src/Oro/Bundle/SalesBundle/Migration/Extension/CustomerExtension.php" target="_blank">CustomerExtension</a> - Adds association between the target customer table and the customer table. More information is available in the Migration Extension documentation.

**Platform**

* ActivityExtension - Adds association between the given table and the table that contains activity records.
* <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ActivityListBundle/Migration/Extension/ActivityListExtension.php" target="_blank">ActivityListExtension</a> - Adds association between the given table and the activity list table. See an example of usage in the [Activity List Inheritance Targets documentation](entity-activities.md#bundle-docs-platform-activity-list-bundle-inheritance).
* <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/AttachmentBundle/Migration/Extension/AttachmentExtension.php" target="_blank">AttachmentExtension</a> - Provides an ability to create file and attachment fields and attachment associations. More information is available in the Use Migration Extension Example in AttachmentBundle.
* <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/CommentBundle/Migration/Extension/CommentExtension.php" target="_blank">CommentExtension</a> - Adds comments association to the entity. More information is available in <a href="https://github.com/oroinc/platform/tree/master/src/Oro/Bundle/CommentBundle#how-to-enable-comment-association-with-new-activity-entity-using-migrations" target="_blank">Enable Comment Association with New Activity Entity</a>.
* <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/DataAuditBundle/Migration/Extension/AuditFieldExtension.php" target="_blank">AuditFieldExtension</a> - Add a possibility for developers to extend data types for DataAudit. More information is available in the [Add New Auditable Types](../entities-data-management/data-audit.md#bundle-docs-platform-data-audit-add-new-types) topic.
* <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/EntityBundle/Migrations/Extension/ChangeTypeExtension.php" target="_blank">ChangeTypeExtension</a> - Allows to change the type of entity primary column type.
* <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/EntityExtendBundle/Migration/Extension/ExtendExtension.php" target="_blank">ExtendExtension</a> - Provides the ability to create extended enum tables and fields and add relations between tables. More information is available in the [Create Custom Entities](create-custom-entities.md#backend-entities-create-custom-entities) topic.
* <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/EntityExtendBundle/Migration/Extension/ConvertToExtendExtension.php" target="_blank">ConvertToExtendExtension</a> - Allows to convert existing entity field to extended.
* <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/MigrationBundle/Migration/Extension/RenameExtension.php" target="_blank">RenameExtension</a> - Allows to rename an extended table or an extended column without losing data.
* <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/MigrationBundle/Migration/Extension/DataStorageExtension.php" target="_blank">DataStorageExtension</a>- Used ito exchange data between different migrations.
* <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ScopeBundle/Migration/Extension/ScopeExtension.php" target="_blank">ScopeExtension</a> - Adds association between the target table and the scope table.
* <a href="https://github.com/oroinc/OroEntitySerializedFieldsBundle/blob/master/Migration/Extension/SerializedFieldsExtension.php" target="_blank">SerializedFieldsExtension</a> - The migration extension that helps manage serialized fields of extended entities. More information is available in the [Serialized Fields](extend-entities/serialized-fields.md#book-entities-extended-entities-serialized-fields) topic.

<a id="backend-entities-migrations-create-extensions"></a>

## Create Extensions for Database Structure Migrations

To create your own extension:

1. Create an extension class in the `YourBundle/Migration/Extension` directory. Using the `YourBundle/Migration/Extension` directory is not mandatory but highly recommended. For example:
   ```php
     namespace Acme\Bundle\DemoBundle\Migration\Extension;

     use Doctrine\DBAL\Schema\Schema;
     use Oro\Bundle\MigrationBundle\Migration\QueryBag;

     class MyExtension
     {
         public function doSomething(Schema $schema, QueryBag $queries, /* other parameters, for example */ $tableName)
         {
             $table = $schema->getTable($tableName); // highly recommended to make sure that a table exists
             $query = 'SOME SQL'; /* or $query = new SqlMigrationQuery('SOME SQL'); */

             $queries->addQuery($query);
         }
     }
   ```
2. Create \*AwareInterface in the same namespace. It is important that the interface name is `{ExtensionClass}AwareInterface` and the set method is `set{ExtensionClass}({ExtensionClass} ${extensionName})`.    For example:
   ```php
     namespace Acme\Bundle\DemoBundle\Migration\Extension;

     /**
      * This interface should be implemented by migrations that depend on {@see MyExtension}.
      */
     interface MyExtensionAwareInterface
     {
         public function setMyExtension(MyExtension $myExtension): void;
     }
   ```
3. Register an extension in the dependency container. For example:
   ```yaml
     services:
         Acme\Bundle\DemoBundle\Migration\Extension\MyExtension:
             tags:
                 - { name: oro_migration.extension, extension_name: test /*, priority: -10 - priority attribute is optional and can be helpful if you need to override existing extension */ }
   ```

To access the database platform or the name generator, your extension class should implement <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/MigrationBundle/Migration/Extension/DatabasePlatformAwareInterface.php" target="_blank">DatabasePlatformAwareInterface</a> or <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/MigrationBundle/Migration/Extension/NameGeneratorAwareInterface.php" target="_blank">NameGeneratorAwareInterface</a> appropriately.

To use another extension in your extension, the extension class should implement `*AwareInterface` of the extension you need.

## Events During Migration

The `Oro\Bundle\MigrationBundle\Migration\Loader\MigrationsLoader` dispatches two events when migrations are being executed, *oro_migration.pre_up* and *oro_migration.post_up*. You can listen to either event and register your own migrations in your event listener. Use the `Oro\Bundle\MigrationBundle\Event\MigrationEvent::addMigration` method of the passed event instance to register your custom migrations:

```php
 namespace Acme\Bundle\DemoBundle\EventListener;

 use Acme\Bundle\DemoBundle\Migration\CustomMigration;
 use Oro\Bundle\MigrationBundle\Event\PostMigrationEvent;
 use Oro\Bundle\MigrationBundle\Event\PreMigrationEvent;

 class RegisterCustomMigrationListener
 {
     /**
     * Listening to the oro_migration.pre_up event
     *
     * @param PreMigrationEvent $event
     * @return void
     */
     public function onPreUp(PreMigrationEvent $event): void
     {
         $event->addMigration(new CustomMigration());
     }

     /**
      * Listening to the oro_migration.post_up event
      *
      * @param PostMigrationEvent $event
      * @return void
      */
     public function onPostUp(PostMigrationEvent $event): void
     {
         $event->addMigration(new CustomMigration());
     }
 }
```

Migrations registered in the *oro_migration.pre_up* event are executed before the *main* migrations, while migrations registered in the *oro_migration.post_up* event are executed after the *main* migrations have been processed.

<!-- Frontend -->
