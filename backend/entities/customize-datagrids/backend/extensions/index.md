<a id="customize-datagrid-extensions"></a>

# Extensions

A datagrid object only takes care of converting datasource to the result set. All other operations are performed by extensions (e.g., pagination, filtering, etc.).

Here is a list of already implemented extensions:

- [Formatter](formatter.md#customize-datagrids-extensions-formatter) - responsible for backend field formatting(e.g., generating URL using router, translation using Symfony translator, etc.). This extension also takes care of passing column configuration to the view layer.
- [Pager](pager.md#customize-datagrid-extensions-pager) - responsible for pagination
- [Sorter](sorter.md#customize-datagrids-extensions-sorters) - responsible for sorting
- [Action](action.md#customize-datagrids-extensions-action) - provides actions configurations for grid
- [Mass Action](mass-action.md#customize-datagrid-extensions-mass-action) - provides mass actions configurations for grid
- [Toolbar](toolbar.md#customize-datagrid-extensions-toolbar) - provides toolbar configuration for view
- [Grid Views](grid-views.md#customize-datagrids-extensions-grid-views) - provides configuration for grid views toolbar
- [Export](export.md#customize-datagrids-extensions-export) - responsible for export grid data
- [Field ACL](field-acl.md#customize-datagrids-extensions-acl) - allow to protect entity fields with ACL
- [Board](board.md#customize-datagrids-extensions-board) - responsible for adding Kanban board views for datagrids
- Filter - responsible for adding filtering and filter widgets to grid
- [Organization Column](organization-column.md#customize-datagrids-extensions-organization-column) - responsible for adding the organization column if a user works in the global organization

## Customization

To implement your extension:

- Develop a class that implements ExtensionVisitorInterface (there is also a basic implementation in AbstractExtension class)
- Register you extension as service with tag { name: oro_datagrid.extension }
