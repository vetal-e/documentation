<a id="dev-integrations-import-export-gaufrette"></a>

# Gaufrette

The ImportExport bundle uses <a href="https://github.com/KnpLabs/Gaufrette" target="_blank">Gaufrette</a> for the file storage.
The Gaufrette configuration is stored in `Resources/config/oro/app.yml`.

By default, the ImportExport bundle configured to use the private [File Storage](../../architecture/tech-stack/file-storage.md#backend-file-storage).

## Gaufrette Configuration for Amazon S3 Storage

This configuration allows to use Amazon S3 cloud service for the importing and exporting. It is applicable if consumers run on different servers.

**Example**

```yaml
services:
    aws_s3.client:
        class: AmazonS3
        arguments:
            -
                key: {your amazon s3 key}
                secret: {your amazon s3 secret}

knp_gaufrette:
    adapters:
        importexport:
            amazon_s3:
                amazon_s3_id: aws_s3.client
                bucket_name: {your bucket name}
                options:
                    directory: 'import_export'
                    create: true
                    region: {your amazon s3 bucket region}
```

#### HINT
See also the <a href="http://knplabs.github.io/Gaufrette/" target="_blank">official Gaufrette documentation</a>.

<!-- Frontend -->
