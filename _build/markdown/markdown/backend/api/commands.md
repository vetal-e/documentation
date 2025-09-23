<a id="web-api-commands"></a>

# CLI Commands

<a id="oroapidoccacheclear-command"></a>

## oro:api:cache:clear

This command clears the API cache.

Usually, you need to run this command when you add a new entity to Resources/config/oro/api.yml or a new processor that changes a list of available via the API.

```none
php bin/console oro:api:cache:clear
```

The `--no-warmup` option can be used to skip warming up the cache after cleaning:

```none
php bin/console oro:api:cache:clear --no-warmup
```

<a id="oroapidoccacheclear"></a>

## oro:api:doc:cache:clear

This clears or warms up the API documentation cache.

If this command is launched without parameters, it warm ups all API documentation caches:

```none
php bin/console oro:api:doc:cache:clear
```

To clear the cache without then warming it up, use the `--no-warmup` option:

```none
php bin/console oro:api:doc:cache:clear --no-warmup
```

To work only with the specified <a href="https://github.com/nelmio/NelmioApiDocBundle/blob/2.x/Resources/doc/multiple-api-doc.rst" target="_blank">API documentation views</a> use the `--view` option:

```none
php bin/console oro:api:doc:cache:clear --view=rest_json_api
```

<a id="oroapidocopenapidump-command"></a>

## oro:api:doc:open-api:dump

This command dumps API documentation in OpenAPI format.

The `--view` option is required and it is used to specify one of <a href="https://github.com/nelmio/NelmioApiDocBundle/blob/2.x/Resources/doc/multiple-api-doc.rst" target="_blank">API documentation views</a> for which OpenAPI specification should be dumped:

```none
php bin/console oro:api:doc:open-api:dump --view=rest_json_api
```

By default, OpenAPI specification is dumped in JSON. To dump it in another format, use the `--format` option:

```none
php bin/console oro:api:doc:open-api:dump --view=rest_json_api --format=json-pretty
php bin/console oro:api:doc:open-api:dump --view=rest_json_api --format=yaml
```

To skip validation of the generated OpenAPI specification, use the  `--no-validation` option:

```none
php bin/console oro:api:doc:open-api:dump --view=rest_json_api --no-validation
```

To generate OpenAPI specification only for the specified entities,  use the `--entity` option:

```none
php bin/console oro:api:doc:open-api:dump --view=rest_json_api --entity=countries --entity=regions
```

To provide a title of the OpenAPI specification, use the `--title` option:

```none
php bin/console oro:api:doc:open-api:dump --view=rest_json_api --title="My API"
```

To provide a URL where live API is served, use the `--server-url` option:

```none
php bin/console oro:api:doc:open-api:dump --view=rest_json_api --server-url=https://example.com
```

<a id="oroapidocopenapischedulerenew-command"></a>

## oro:api:doc:open-api:schedule-renew

This command schedules the renewal of all OpenAPI specifications.

```none
php bin/console oro:api:doc:open-api:schedule-renew
```

<a id="oroapidump-command"></a>

## oro:api:dump

This command shows all resources accessible via the API.

Run this command without parameters to see all available resources:

```none
php bin/console oro:api:dump
```

To display resources for a particular request type, specify the `--request-type` option:

```none
php bin/console oro:api:dump --request-type=rest --request-type=json_api
```

To show all available sub-resources, use the `--sub-resources` option:

```none
php bin/console oro:api:dump --sub-resources
```

If you are interested in information about a particular entity, specify an entity class or entity alias as an argument:

```none
php bin/console oro:api:dump "Oro\Bundle\UserBundle\Entity\User" --sub-resources
```

or

```none
php bin/console oro:api:dump users --sub-resources
```

To get all entities that are accessible via the API, use the `--accessible` option:

```none
php bin/console oro:api:dump --accessible
```

To get all entities that are not accessible via the API, use the `--not-accessible` option:

```none
php bin/console oro:api:dump --not-accessible
```

To get all entities that support a specific API action, use the `--action` option:

```none
php bin/console oro:api:dump --action=update_list
```

To get all entities that support [the upsert operation](../../api/upsert-operation.md#web-services-api-upsert-operation), use the `--upsert` option:

```none
php bin/console oro:api:dump --upsert
```

To get all entities that support [the validate operation](../../api/validate-operation.md#web-services-api-validate-operation), use the `--validate` option:

```none
php bin/console oro:api:dump --validate
```

<a id="oroapidebug"></a>

## oro:api:debug

This command shows details about registered API actions and processors.

To display all actions, run this command without parameters:

```none
php bin/console oro:api:debug
```

To see the processors registered for a given action, specify the action name as an argument:

```none
php bin/console oro:api:debug <action>
```

or

```none
php bin/console oro:api:debug --no-docs <action>
```

The list of the processors can be limited to some group specified as the second argument:

```none
php bin/console oro:api:debug <action> <group>
```

or

```none
php bin/console oro:api:debug --no-docs <action> <group>
```

You can use the `--attribute` option to show the processors that will be executed only when the context has a given attribute with the specified value.
Separate the attribute name and value by a colon, e.g., `--attribute=collection:true` for a scalar value, or `--attribute=extra:[definition,filters]` for an array value:

```none
php bin/console oro:api:debug --attribute=collection:true <action>
```

or

```none
php bin/console oro:api:debug --attribute=extra:[definition,filters] <action>
```

Use the `--processors` and ``--processors-without-description` options to display all processors and all processors without descriptions, respectively:

```none
php bin/console oro:api:debug --processors
```

or

```none
php bin/console oro:api:debug --processors-without-description
```

The `--request-type` option can limit the scope to the specified request type(s). Omitting this option is equivalent to `--request-type=rest --request-type=json_api`. The available types are `rest`, `json_api`, `batch`, or use `any` to include all request types:

```none
php bin/console oro:api:debug --request-type=rest other options and arguments
```

```none
php bin/console oro:api:debug --request-type=json_api other options and arguments
```

```none
php bin/console oro:api:debug --request-type=batch other options and arguments
```

```none
php bin/console oro:api:debug --request-type=any other options and arguments
```

<a id="oroapiconfigdump-command"></a>

## oro:api:config:dump

This command shows the configuration for a particular entity.

Execute this command with an entity class or entity alias specified as an argument:

```none
php bin/console oro:api:config:dump "Oro\Bundle\UserBundle\Entity\User"
```

or

```none
php bin/console oro:api:config:dump users
```

To display the configuration used for a particular action, use the `--action option` (please note that the default value for this option is `get`):

```none
php bin/console oro:api:config:dump users --action=update
```

To display the configuration for a particular request type, use the `request-type` option:

```none
php bin/console oro:api:config:dump users --request-type=rest --request-type=json_api
```

No extra configuration data are added to the output by default, but you can add them with the `--extra` option. The value for the `extra` option can be: actions, definition, filters, sorters, descriptions, or the full name of a class implements <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ApiBundle/Config/Extra/ConfigExtraInterface.php" target="_blank">ConfigExtraInterface</a>, e.g.

```none
php bin/console oro:api:config:dump users --extra=filters --extra=sorters
```

To display the human-readable representation of an entity and its fields, use:

```none
php bin/console oro:api:config:dump users --extra=descriptions
```

If you added a new extra section, pass the FQCN of a ConfigExtra:

```none
php bin/console oro:api:config:dump users --extra="Acme\Bundle\DemoBundle\Config\AcmeConfigExtra"
```

You can pass multiple options:

```none
php bin/console oro:api:config:dump users --extra=sorters --extra=descriptions --extra=filters --extra="Acme\Bundle\DemoBundle\Config\MyConfigExtra"
```

<a id="oroapimetadatadump-command"></a>

## oro:api:metadata:dump

This command shows metadata for a particular entity.

To display the metadata, run this command with an entity class or entity alias specified as an argument:

```none
php bin/console oro:api:metadata:dump "Oro\Bundle\UserBundle\Entity\User"
```

or

```none
php bin/console oro:api:metadata:dump users
```

To display the entity metadata used for a particular action, use the `--action` option (please note that the default value for this option is `get_list`):

```none
php bin/console oro:api:metadata:dump users --action=update
```

You can also use the `--parentAction` and the `--action` options together to display the entity metadata used for an action executed as part of another action. For example, to display the entity metadata used for the `get` action executed as part of the `create` action, use the following command:

```none
php bin/console oro:api:metadata:dump users --action=get --parentAction=create
```

To display the entity metadata used for a particular request type, use the `--request-type` option:

```none
php bin/console oro:api:metadata:dump users --request-type=rest --request-type=json_api
```

To include the HATEOAS links to the metadata, use the `--hateoas` option:

```none
php bin/console oro:api:metadata:dump --hateoas <entity>
```

<a id="oroapiconfigdumpreference-command"></a>

## oro:api:config:dump-reference

This command shows the structure of Resources/config/oro/api.yml.

```none
php bin/console oro:api:config:dump-reference
```

You can use the `--max-nesting-level` option to limit the depth of nesting target entities:

```none
php bin/console oro:api:config:dump-reference --max-nesting-level=<number>
```

<a id="web-api-commands-oro-cron-api-async-operations-cleanup"></a>

<a id="orocronapiasyncoperationscleanup-command"></a>

## oro:cron:api:async_operations:cleanup

This command deletes all obsolete asynchronous operations used by Batch API.

```none
php bin/console oro:cron:api:async_operations:cleanup
```

To show the number of obsolete asynchronous operations without the deletion of them, use the `--dry-run` option:

```none
php bin/console oro:cron:api:async_operations:cleanup --dry-run
```

<!-- Frontend -->
