<a id="reference-format-jsmodules"></a>

# JS Modules

| Filename   | `jsmodules.yml`                                                                                                              |
|------------|------------------------------------------------------------------------------------------------------------------------------|
| Root Node  | none                                                                                                                         |
| Sections   | * [entry]()<br/>* [aliases]()<br/>* [app-modules]()<br/>* [configs]()<br/>* [dynamic-imports]()<br/>* [map]()<br/>* [shim]() |

## Location of jsmodules.yml

| Management Console   | [BUNDLE_NAME]/Resources/config/oro/jsmodules.yml                        |
|----------------------|-------------------------------------------------------------------------|
| Storefront           | [BUNDLE_NAME]/Resources/views/layouts/[THEME_NAME]/config/jsmodules.yml |

### `entry`

**type**: `map`

Webpack entry points configuration. For each new entry point, add the script tag
with the entry point file path to the twig template or layout update to load the entry point file on a page.

```yaml
entry:
    app:
        - oroui/js/app
        - oroui/js/app/services/app-ready-load-modules
```

### `aliases`

**type**: `map`

This section is used when a desired module name does not match the path to the module.
The keys of the map are module names with trailing $ which are mapped to the actual path of the file that contains the module’s source code.
Webpack provides the corresponding syntax and description in its documentation (see <a href="https://webpack.js.org/configuration/resolve/#resolvealias" target="_blank">Webpack Resolve Alias</a>).
Aliases can help import certain modules easily using short names.

```yaml
aliases:
    backbone$: backbone/backbone
    oro/block-widget$: oroui/js/widget/block-widget
```

### `app-modules`

**type**: `sequence`

Introduces a list of modules that should be initialized before application is launched. They can be used to track certain page events with bundle specific handlers, etc.

```yaml
app-modules:
    - oroui/js/app/modules/jstree-actions-module
    - oroui/js/app/modules/layout-module
```

### `configs`

**type**: `map`

Each module that should be configured at runtime (e.g., via twig templates) must be specified in this section where the key of the map is a module name, and the value is an empty object.

```yaml
configs:
    controllers/page-controller: {}
    oroui/js/app: {}
```

### `dynamic-imports`

**type**: `map`

Add a module name to this section to be able to import a module with the name that is determined at runtime.

```javascript
import loadModules from 'oroui/js/app/services/load-modules';

loadModules(moduleName).then(module => module.init());
```

Insert a module name to this section nested into the subsection with the name of webpack build chunk where modules have to be added.

```yaml
dynamic-imports:
    oroui:
        - jquery
        - oroui/js/app/components/view-component
```

#### NOTE
A chunk name should either be a new name or already exist in another bundle.
It is preferred to group modules that are used together or/and on specific pages for the maximum benefit of the webpack chunk concept.

<a id="reference-format-jsmodules-map"></a>

### `map`

**type**: `map`

The map option allows to substitute a module with the given ID with a different module. Such substitution is working for the given module prefix.

For example, <a href="https://github.com/oroinc/platform/tree/master/src/Oro/Bundle/UIBundle" target="_blank">OroUIBundle</a> is delivered with an extended version of the jQuery library. This means
that all modules should receive the extended jQuery library from the OroUIBundle. However, since
the bundle itself needs the original version of the library to be able to extend it, it must get
the original version when requiring it:

```yaml
map:
    '*':
        'jquery': 'oroui/js/jquery-extend'
    'oroui/js/jquery-extend':
        'jquery': 'jquery'
```

#### NOTE
\* is a special key that matches all module contexts.

### `shim`

**type**: `map`

Webpack places each module in its local scope, however, some third party libraries may expect global dependencies (e.g., $ for jQuery). The libraries may also create globals which need to be exported, so they can stop working. To solve this issue, webpack offers a shimming feature. See <a href="https://webpack.js.org/guides/shimming/" target="_blank">Webpack Shimming</a>
In our shim section, each key of the map is the name of a module to be created. For each module, a map that
configures the module must be specified. It can consist of the following keys:

`imports` (**type**: `sequence`)

> If the library depends on specific global variables, these dependencies can be listed here. See <a href="https://webpack.js.org/loaders/imports-loader" target="_blank">Webpack Imports Loader</a> for the detail.

`exports` (**type**: `string`)

> The name of a JavaScript symbol that is exposed to other parts of the system that use this symbol. See <a href="https://webpack.js.org/loaders/imports-loader" target="_blank">Webpack Exports Loader</a> for the details.

`expose` (**type**: `sequence`)

> Specify which local variables have to be exposed globally. See <a href="https://webpack.js.org/loaders/imports-loader" target="_blank">Webpack Expose Loader</a> for the details.
```yaml
shim:
    bootstrap-typeahead:
        imports:
            - single|jquery|jQuery
    jquery:
        expose:
            - $,jQuery
    jquery.select2:
        exports: single|Select2
        imports:
            - single|jquery|jQuery
    oroui/js/app/services/app-ready-load-modules:
        expose: loadModules
    '@oroinc/jsplumb/dist/js/jsPlumb-1.7.10':
        imports:
            - additionalCode: 'var define = false;'
              wrapper: window
```

<!-- Frontend -->
