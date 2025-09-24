<a id="datagrid-references-configuration"></a>

# References in YAML Configuration

You can use a static method call, a service method call, and class constant access in the YAML datagrid configuration.
These references will be called by SystemAwareResolver while building the datagrid in the datagrid manager.

## References types

## Service Call

```none
@oro_email.grid.query_builder->getChoicesQuery
```

Call method getChoicesQuery with datagrid name and YAML configuration key as arguments from oro_email.grid.query_builder service.

## Static Method Call

```none
Acme\Bundle\DemoBundle\SomeClass::testStaticCall
```

The class name can be defined in the container’s parameters or specified directly.

## Constant

```none
Acme\Bundle\DemoBundle\SomeClass::TEST
```

PHP is_callable is used to determine if it is callable or should be treated as constant.

The value becomes unchanged if it is not callable and no constant exists with such a name in the class.

## Service Injection

```none
some_key: @some.serviceID
```
