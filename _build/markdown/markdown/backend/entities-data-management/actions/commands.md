<a id="bundle-docs-platform-action-bundle-commands"></a>

# Operations (Actions) Console Commands

## oro:debug:action

This command displays available actions for an application.

```none
php bin/console oro:debug:action
```

To get information about a specific action, specify its name:

```none
php bin/console oro:debug:action <action>
```

```none
php bin/console oro:debug:action flash_message
```

### Usage

- Displays a list of current actions php bin/console oro:debug:action;
- Shows a full description php bin/console oro:debug:action [<name>].

## oro:debug:condition

This command displays available conditions for an application.

```none
php bin/console oro:debug:condition
```

To get information about a specific condition, specify its name:

```none
php bin/console oro:debug:condition <condition>
```

```none
php bin/console oro:debug:condition instanceof
```

## oro:debug:operation

This command displays available operations and action groups.

```none
php bin/console oro:debug:operation
```

Use the `--action-group` option to see action groups instead of operations:

```none
php bin/console oro:debug:operation --action-group
```

To get information about a specific operation or action group, specify its name:

```none
php bin/console oro:debug:operation <operation-name>
```

```none
php bin/console oro:debug:operation --action-group <action-group-name>
```

```none
php bin/console oro:debug:operation DELETE
```

```none
php bin/console oro:debug:operation --action-group DELETE
```

The `--assemble` option can be used to display instantiated objects instead of plain data:

```none
php bin/console oro:debug:operation --assemble <operation-name>
```

```none
php bin/console oro:debug:operation --assemble --action-group <action-group-name>
```

```none
php bin/console oro:debug:operation --assemble DELETE
```

```none
php bin/console oro:debug:operation --assemble --action-group DELETE
```

## oro:action:configuration:validate

This command validates action configuration and displays the encountered actions.

```none
php bin/console oro:action:configuration:validate
```

<!-- Frontend -->
