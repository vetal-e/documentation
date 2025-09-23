<a id="web-api-batch-api"></a>

# Batch API

The Batch API provides a way to create or update a list of entities of the same type via one API request.

For detailed information how this API works, see the [update_list](actions.md#update-list-action),
[batch_update](actions.md#batch-update-action) and [batch_update_item](actions.md#batch-update-item-action) actions.

<a id="web-api-batch-api-enable"></a>

## Enable Batch API for Entity

By default, Batch API is disabled for all API resources. To enable it for an API resource,
the [update_list](actions.md#update-list-action) action should be enabled for this resource.
You can do this via `actions` section for an entity in Resources/config/oro/api.yml.

Example:

```yaml
api:
    entities:
        Oro\Bundle\UserBundle\Entity\User:
            actions:
                update_list: true
```

<a id="web-api-batch-api-config"></a>

## Batch API Configuration

All configuration options related to Batch API are grouped in the `batch_api` section of ApiBundle configuration:

```yaml
oro_api:
    batch_api:

        # The default maximum number of entities that can be saved in a chunk. The default value is 100.
        chunk_size: 100

        # The maximum number of entities of a specific type that can be saved in a chunk.
        chunk_size_per_entity:
            Oro\Bundle\UserBundle\Entity\User: 10 # example

        # The default maximum number of included entities that can be saved in a chunk. The default value is 50.
        included_data_chunk_size: 50

        # The maximum number of included entities that can be saved in a chunk for a specific primary entity type.
        included_data_chunk_size_per_entity:
            Oro\Bundle\UserBundle\Entity\User: 20 # example

        # The maximum number of seconds that API waits for a synchronous batch API operation finished. If the operation is not finished within this time interval it is processed as an asynchronous operation.
        sync_processing_wait_timeout: 25

        # The default maximum number of entities that can be processed by synchronous batch API.
        sync_processing_limit: 100

        # The maximum number of entities of a specific type that can be processed by synchronous batch API.
        # The null value can be used to revert already configured limit for a specific entity type and use the default limit for it.
        sync_processing_limit_per_entity:
            Oro\Bundle\UserBundle\Entity\User: 10 # example

        # The default maximum number of included entities that can be processed by synchronous batch API.
        sync_processing_included_data_limit: 50

        # The maximum number of included entities that can be processed by synchronous batch API for a specific primary entity type.
        # The null value can be used to revert already configured limit for a specific entity type and use the default limit for it.
        sync_processing_included_data_limit_per_entity:
            Oro\Bundle\UserBundle\Entity\User: 20 # example
```

Parameters `chunk_size_per_entity` and `included_data_chunk_size_per_entity` can be used to tuning of
an API engine to have maximum performance.

To get maximum performance in requests with included entities, you can change the value of the `included_data_chunk_size_per_entity`
parameter to the medium number of related entities that will be set to one primary entity when processing the request.

<a id="web-api-batch-api-async-operation-config"></a>

## Asynchronous Batch Operations Cleanup Configuration

Obsolete asynchronous batch operations are removed once a day by a cron job.

The default configuration of this cron job is illustrated below:

```yaml
oro_api:
    batch_api:
        async_operation:

            # The number of days async operations are stored in the system.
            lifetime: 30

            # The maximum number of seconds that the cron job can spend in one run.
            cleanup_process_timeout: 3600 # 1 hour

            # The maximum number of seconds after which an operation will be removed regardless of status.
            operation_timeout: 3600 # 1 hour
```

<a id="web-api-batch-api-storage-config"></a>

## Storage Configuration

The <a href="https://github.com/KnpLabs/KnpGaufretteBundle" target="_blank">KnpGaufretteBundle</a> is used to configure storages for source data files of Batch API requests and all files created when processing asynchronous batch operations (e.g., chunk files, error files, etc.).

Here is the default configuration of these storages:

```yaml
knp_gaufrette:
    adapters:
        api:
            doctrine_dbal:
                connection_name: batch
                table: oro_api_async_data
                columns:
                    key: name
                    content: content
                    mtime: updated_at
                    checksum: checksum
    filesystems:
        # a storage for source data files
        api_source_data:
            adapter: private
            alias: api_source_data_filesystem
        # a storage for files created when processing asynchronous batch operations
        api:
            adapter: api
            alias: api_filesystem
```

To change the adapter configuration, use Resources/config/oro/app.yml in any application bundle or config/config.yml.
The following example shows how to reconfigure this adapter to use a local filesystem:

```yaml
knp_gaufrette:
    adapters:
        api:
            local:
                directory: '%kernel.project_dir%/var/api_files'
```

You can find more examples in <a href="https://github.com/KnpLabs/KnpGaufretteBundle/blob/master/README.md" target="_blank">KnpGaufretteBundle documentation</a>.

<!-- Frontend -->
