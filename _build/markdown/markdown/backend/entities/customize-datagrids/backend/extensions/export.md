<a id="customize-datagrids-extensions-export"></a>

# Export Extension

This extension provides the functionality to export grid rows. This allows you to export rows from all pages. The exported data will be the same as on a grid, including filters and sorting.

## Configuration

To enable export functionality, add the export option to the configuration of your grid. For example:

```yaml
datagrids:
    accounts-grid:
        ...
        options:
            export: true
```

The Export button should be displayed in the top left corner of a grid. To export grid data, users need to click this button and select the format of exporting data (currently, only CSV format is implemented).

Configure your grid properly if you need to allow the export of grid data in other formats. For example, to allow export data in CSV and PDF formats, use the following configuration:

```yaml
datagrids:
    my-grid:
        ...
        options:
            export:
                csv: { label: oro.grid.export.csv }
                pdf: { label: acme.grid.export.pdf }
```

Implement and register a writer for the new export format. To register a writer in the dependency container, use the following naming convention: `oro_importexport.writer.echo.[format]`. So, a writer for PDF should be registered as `oro_importexport.writer.echo.pdf`.

You can use <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/ImportExportBundle/Writer/CsvEchoWriter.php" target="_blank">existing CSV writer</a> as an example for your writer.

The configuration can also influence performance by changing the value of the grid export page size in the configuration. This will allow you to change the number of queries for the database. But keep in mind that increasing the batch size increases memory consumption.

```yaml
datagrids:
    my-grid:
        ...
        options:
            export:
                csv:
                    label: oro.grid.export.csv
                    page_size: 500
```

**Related Articles**

* [Datagrids](../../../data-grids/index.md#data-grids)
* [Datagrid Configuration Reference](../../../../configuration/yaml/datagrids.md#reference-format-datagrids)

<!-- Frontend -->
