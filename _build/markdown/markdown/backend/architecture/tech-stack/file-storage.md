<a id="backend-file-storage"></a>

# File Storage

This section describes the filesystem abstraction layer used in the Oro application to access data files.

The file storage abstraction is based on <a href="https://github.com/KnpLabs/KnpGaufretteBundle" target="_blank">KnpGaufretteBundle</a> with GaufretteBundle
that enables a filesystem abstraction layer in the Oro applications and provides a simplified file manager service for media files.

You can configure the file storage to use different types of filesystem adapters to store the data, for example, your local filesystem, GridFS storage, etc. For more details, see [File System Adapters Configuration](#file-system-adapters-configuration).

<a id="backend-file-storage-types-of-storage"></a>

## Types of Storage

By default, the file storage component provides two types of storage with <a href="https://github.com/KnpLabs/Gaufrette" target="_blank">Gaufrette</a> adapters:

- private
- public

The **private** file storage adapter is implemented to store data that should not be available via a direct link.
Examples of such data types are attachments’ data, import and export files, protected media cache files, etc.
If the local filesystem adapter is used by default, then data will be stored in the `/var/data` directory of the application.

The **public** file storage adapter is implemented to store data that can be available via a direct link without access checks.
Examples of such data are resized product images, sitemap files, etc. If the local filesystem adapter is used by default, data will be stored in the `public/media` directory of the application.

<a id="backend-file-storage-adapters-configuration"></a>

## File System Adapters Configuration

By default, the **public** and **private** Gaufrette adapters are configured to use the local filesystem adapter.
However, you can reconfigure them to use another storage adapter.

Oro applications support the local filesystem adapter and <a href="https://www.mongodb.com/docs/manual/core/gridfs/" target="_blank">GridFS</a> by GridFSConfigBundle.

There are two ways to change the configuration of adapters:

1. The usual way that requires the configuration of <a href="https://github.com/KnpLabs/KnpGaufretteBundle" target="_blank">KnpGaufretteBundle</a>.
2. A simplified way where you can reconfigure the already configured adapters with the config/parameters.yml file.

#### NOTE
Be aware that old data uploaded to the local storage will not be migrated to the new storage.

<a id="backend-file-storage-adapters-configuration-with-knpgaufrettebundle"></a>

## File System Adapters Configuration with KnpGaufretteBundle

As storage types are the <a href="https://github.com/KnpLabs/KnpGaufretteBundle" target="_blank">KnpGaufretteBundle</a> adapters, you can configure them with the manual configuration of this bundle.

For example, to use the Oro GridFS Gaufrette adapter, use the `oro_gridfs` adapter type. To configure a new or reconfigure an
existing adapter, add the <a href="https://github.com/KnpLabs/KnpGaufretteBundle" target="_blank">KnpGaufretteBundle</a> configuration to the Resources/config/oro/app.yml file of any bundle
or the config/config.yml file of your application:

```yaml
knp_gaufrette:
    adapters:
        public: # the adapter name
            oro_gridfs:
                mongodb_gridfs_dsn: 'mongodb://127.0.0.1:27017/media'
```

As you can see from the example, the configuration of the `oro_gridfs` adapter has the `mongodb_gridfs_dsn` parameter with the configuration of the MongoDB DSN string. The format of this string is the following:
`[protocol]://[username]:[password]@[host]:[port]/[database]`, where:

- **protocol** is mongodb
- **username** is the username that has access to the MongoDB database
- **password** is the user’s password
- **host** is the hostname or IP address of the MongoDB server
- **database** is the MongoDB database name that should be used as GridGS storage

<a id="backend-file-storage-adapters-configuration-with-parameters-yml"></a>

## File System Adapters Configuration with parameters.yml

To simplify the configuration of the already existing Gaufrette adapters or filesystems, you can use your application’s config/parameters.yml file.

To reconfigure an adapter, add the parameter with the `gaufrette_adapter.[adapter_name]` name,
where `adapter_name` is the name of an existing adapter.

The parameter’s value is the MongoDB DSN string described in the previous chapter, which starts with the `gridfs:` prefix.

The following example shows the reconfiguration of the **public** adapter:

```yaml
gaufrette_adapter.public: 'gridfs:mongodb://user:password@host:27017/media'
```

To get the list of existing Gaufrette adapters, use the following command:

```bash
bin/console debug:config knp_gaufrette adapters
```

To reconfigure a filesystem, add the parameter with the name `gaufrette_filesystem.[filesystem_name]`,
where `filesystem_name` is the name of an existing filesystem.

As for the adapter configuration, the parameter’s value is the MongoDB DSN string that starts with the `gridfs:` prefix.

The following example shows the reconfiguration of the `attachments` filesystem:

```yaml
gaufrette_filesystem.attachments: 'gridfs:mongodb://user:password@host:27017/attachments'
```

To get the list of existing Gaufrette filesystems, use the following command:

```none
bin/console debug:config knp_gaufrette filesystems
```

<a id="backend-file-storage-configuration-for-cluster-mongodb-gridfs-setup"></a>

## Configuration for Cluster MongoDB GridFS Setup

If you have installed MongoDB cluster, it can also be used as the GridFS storage.

In this case, the MongoDB DSN string has the following format:
`[protocol]://[user]:[password]@[host1]:[port],[host2]:[port]/[database]`

Example:

```yaml
gaufrette_adapter.public: 'gridfs:mongodb://user:password@host1:27017,host2:27017/media'
```

#### NOTE
`mongodb://user:password@host1:27017,host2:27017` part of the value pointed above can be replaced with container parameter that relies on environment variable. This provides an ability to switch between MongoDB GridFS storages in runtime. See an example in the `config/parameters.yml.dist` file in the application.

<a id="backend-file-storage-add-ability-to-configure-adapter-with-parameters-yml"></a>

## Add Ability To Configure File System Adapter with parameters.yml

You can also enable additional Gaufrette adapter types to support configuration via the config/parameters.yml file.

To do this, create a configuration factory class that implements <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/GaufretteBundle/DependencyInjection/Factory/ConfigurationFactoryInterface.php" target="_blank">ConfigurationFactoryInterface</a> and register it as the configuration factory for the oro_gaufrette bundle in your bundle class:

```php
namespace Acme\Bundle\DemoBundle\DependencyInjection\Factory;

use Oro\Bundle\GaufretteBundle\DependencyInjection\Factory\ConfigurationFactoryInterface;

class SomeAdapterConfigurationFactory implements ConfigurationFactoryInterface
{
    #[\Override]
    public function getAdapterConfiguration(string $configString): array
    {
        // implement logic here
    }

    #[\Override]
    public function getKey(): string
    {
        // implement logic here
    }

    #[\Override]
    public function getHint(): string
    {
        // implement logic here
    }
}
```

```php
namespace Acme\Bundle\DemoBundle;

use Acme\Bundle\DemoBundle\DependencyInjection\Factory\SomeAdapterConfigurationFactory;
use Oro\Bundle\GaufretteBundle\DependencyInjection\OroGaufretteExtension;
use Symfony\Component\DependencyInjection\ContainerBuilder;
use Symfony\Component\HttpKernel\Bundle\Bundle;

class AcmeDemoBundle extends Bundle
{
    #[\Override]
    public function build(ContainerBuilder $container): void
    {
        parent::build($container);

        /** @var OroGaufretteExtension $gaufretteExtension */
        $gaufretteExtension = $container->getExtension('oro_gaufrette');
        $gaufretteExtension->addConfigurationFactory(new SomeAdapterConfigurationFactory());
    }
}
```

<a id="backend-access-the-data-in-storage"></a>

## Access to Data in Storage

The main access point to the files in storage is the <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/GaufretteBundle/FileManager.php" target="_blank">FileManager</a> service.

To implement the access point to your type of data in the storage:

- Add a new Gaufrette filesystem configuration into the Resources/config/oro/app.yml file of your bundle and set **private** or **public** as the filesystem adapter:

```yaml
knp_gaufrette:
    filesystems:
        some_storage:
            adapter: private # the type of storage (public or private)
            alias: some_storage_filesystem
```

- Register the <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/GaufretteBundle/FileManager.php" target="_blank">FileManager</a> service as the child of the `oro_gaufrette.file_manager` service:

```yaml
acme.file_manager:
    parent: oro_gaufrette.file_manager
    arguments:
        - 'some_storage' # the name of Gaufrette filesystem should be used as the storage.
```

As a result, the **acme.file_manager** service will be the entry point of your new private file storage.

By default, if the Gaufrette local filesystem adapter is used to store files for a filesystem, the data will be saved to the
sub-directory with the name of the Gaufrette filesystem used for this file storage.

For the previous example, this default path is the `var/data/some_storage` directory of the application.

For the public filesystem, it is `public/media/your_filesystem_name`.

You can change this sub-directory name by setting them as the second parameter of the file manager service declaration:

```yaml
acme.file_manager:
    parent: oro_gaufrette.file_manager
    arguments:
        - 'some_storage' # the name of the Gaufrette filesystem should be used as the storage.
        - 'another_sub_directory' # data will save to another_sub_directory subdirectory.
```

<a id="backend-simple-access-to-the-public-storage-files"></a>

## Simple Access to Public Storage Files

If the Gaufrette local filesystem adapter is used to store files for the public filesystem, all the files
stored in this filesystem will be available via direct URI `http://your_domain/media/sub_directory/filename`.

But if the storage uses another adapter type, for example, the <a href="https://www.mongodb.com/docs/manual/core/gridfs/" target="_blank">GridFS</a> storage type
by GridFSConfigBundle, this URIs will not work
and you will have to implement access points to the files manually.

To simplify this case, configure the file manager service with the `oro_gaufrette.public_filesystem_manager` tag.
In this case, the files of such storage will be available via the `http://your_domain/media/sub_directory/filename`
path, regardless of the adapter configuration used for the public type file storage.

For example, if you have configured `my_public` Gaufrette filesystem that uses the **public** adapter, the configuration of the file manager service can be the following:

```yaml
acme.public_file_manager:
    parent: oro_gaufrette.file_manager
    arguments:
        - 'my_public'
    tags:
        - { name: oro_gaufrette.public_filesystem_manager }
```

In this example, the files are available via the `http://your_domain/media/my_public/filename` URIs.

<a id="backend-file-storage-access-to-data-via-stream-wrappers"></a>

## Access to Data via Stream Wrappers

Files in the filesystem can be accessed via stream wrappers and with the help of the <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/GaufretteBundle/FileManager.php" target="_blank">FileManager</a> service.

The application has two stream wrappers configured to be used with Gaufrette filesystems:

- Common wrapper by the <a href="https://github.com/KnpLabs/KnpGaufretteBundle" target="_blank">KnpGaufretteBundle</a>;
- Read-only wrapper by the OroGaufretteBundle.

The standard stream wrapper allows full access to the files stored in the filesystem. By default, the wrapper is configured
to use the `gaufrette` protocol. To get the full URL of a file use the getFilePath() method of the <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/GaufretteBundle/FileManager.php" target="_blank">FileManager</a> service.

You can use the read-only stream wrapper if you need to read data but do not know if the data to be written is available.
For example, this case can be figured if the local adapter was used and the files were uploaded by someone other than the user that runs
the application. By default, the wrapper is configured to use the `gaufrette-readonly` protocol.
To get the full URL of a file use getReadonlyFilePath() method of the <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/GaufretteBundle/FileManager.php" target="_blank">FileManager</a> service.

<a id="backend-file-storage-migrate-data-command"></a>

## Migrate Data Command

You need to migrate data from the previous location during the upgrade from local filesystem storage to another storage type or location. You can also use the command to upload some data to the file storage.

To migrate the data, you can use the console command `oro:gaufrette:migrate-filestorages` which moves the application files from old storage to the proper Gaufrette file storage.

The command can work in 2 modes: Automatic and Manual.

In the **Automatic** mode, the data is migrated to the current structure by a predefined list of paths used in the application before v.4.2.

In the **Manual** mode, a user is asked for a path to be migrated and the Gaufrette file system name where the data should migrate.

The command has a list of pre-configured default paths from which the data is moved in the automatic mode and a list
of <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/GaufretteBundle/FileManager.php" target="_blank">FileManager</a> services where data can be uploaded.

You can set the path to be migrated with the `--migration-path` option.

You can set the Gaufrette file system name with the `--gaufrette-filesystem` option.

To get the list of available file systems, run the command with the `--mode=filesystems-list` option.

To add an additional path from which the data is going to be moved or an additional <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/GaufretteBundle/FileManager.php" target="_blank">FileManager</a>, add a new CompilerPass
in your bundle and add it into the Bundle class:

```php
namespace Acme\Bundle\DemoBundle\DependencyInjection\Compiler;

use Oro\Bundle\GaufretteBundle\Command\MigrateFileStorageCommand;
use Symfony\Component\DependencyInjection\Compiler\CompilerPassInterface;
use Symfony\Component\DependencyInjection\ContainerBuilder;
use Symfony\Component\DependencyInjection\Reference;

/**
 * Adds file storage config to the oro:gaufrette:migrate-filestorages migration command.
 */
class MigrateFileStorageCommandCompilerPass implements CompilerPassInterface
{
    #[\Override]
    public function process(ContainerBuilder $container)
    {
        $container->getDefinition(MigrateFileStorageCommand::class)
            // adds the mapping to migrate path /path/to/application/var/some_path
            // to the filesystem with name 'some_filesystem'
            ->addMethodCall(
                'addMapping',
                ['/var/some_path', 'some_filesystem']
            )
            // adds the file manager service as 'some_filesystem' filesystem
            ->addMethodCall(
                'addFileManager',
                ['some_filesystem', new Reference('acme_your_bundle.file_manager')]
            );
    }
}
```

<a id="backend-file-storage-cleanup-lost-attachment-files-command"></a>

## Cleanup Lost Attachment Files Command

When GridFSConfigBundle is installed, and <a href="https://www.mongodb.com/docs/manual/core/gridfs/" target="_blank">GridFS</a> is configured
to store attachment files, you can use the `oro:attachment:cleanup-gridfs-files`
console command to check if any attachment file is left unlinked in the storage and to purge such files.

To see the list of lost files, use this command with the `--dry-run` option:

```bash
php bin/console oro:attachment:cleanup-gridfs-files --dry-run
```

To purge lost files, use this command with the `--force` option:

```bash
php bin/console oro:attachment:cleanup-gridfs-files --force
```

#### IMPORTANT
The `oro:attachment:cleanup-gridfs-files --force` command removes files in batches, not all at once. This behavior is intentional and prevents issues caused by large bulk deletions, such as MongoDB or PostgreSQL failures. To completely clear all unlinked files, re-run the command multiple times until the output shows that no files remain to delete.

<!-- Frontend -->
