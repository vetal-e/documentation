<a id="installation-web-server-configuration"></a>

<a id="step-3-configure-the-webserver"></a>

# Web Server Configuration

<!-- begin_web_server_configuration -->

The following sections contain the recommended configuration for supported web server types and versions.

## Apache 2.4

```text
<VirtualHost *:80>
    ServerName <your-domain-name>
    ServerAlias www.<your-domain-name>

    DirectoryIndex index.php
    DocumentRoot <application-root-folder>/public
    <Directory  <application-root-folder>/public>
        # enable the .htaccess rewrites
        AllowOverride All
        Require all granted
    </Directory>

    ErrorLog /var/log/apache2/<your-domain-name>_error.log
    CustomLog /var/log/apache2/<your-domain-name>_access.log combined
</VirtualHost>
```

* Replace **<application-root-folder>** with the absolute path to the Oro application.
* Replace **<your-domain-name>** with the configured domain name used for the Oro application.

#### NOTE
Please make sure mod_rewrite and mod_headers are enabled.

## Nginx

```text
server {
    server_name <your-domain-name> www.<your-domain-name>;
    root <application-root-folder>/public;

    location / {
        # try to serve file directly, fallback to index.php
        try_files $uri /index.php$is_args$args;
    }

    location ~ ^/(index|index_dev|config|install)\.php(/|$) {
        fastcgi_pass 127.0.0.1:9000;
        # or
        # fastcgi_pass unix:/var/run/php/php7-fpm.sock;
        fastcgi_split_path_info ^(.+\.php)(/.*)$;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param HTTPS off;
    }

    location ~* ^[^(\.php)]+\.(jpg|jpeg|gif|png|ico|css|pdf|ppt|txt|bmp|rtf|js)$ {
       access_log off;
       expires 1h;
       add_header Cache-Control public;
    }

    error_log /var/log/nginx/<your-domain-name>_error.log;
    access_log /var/log/nginx/<your-domain-name>_access.log;
}
```

* Replace **<application-root-folder>** with the absolute path to the Oro application.
* Replace **<your-domain-name>** with the configured domain name used for the Oro application.

#### NOTE
Make sure that the web server user has permissions for the log directory of the application.

More details on the file permissions configuration are available
<a href="https://symfony.com/doc/6.4/setup/file_permissions.html" target="_blank">in the official Symfony documentation</a>.

#### NOTE
Business Tip

Thinking of going digital? Explore our thorough platform comparison guide to narrow down your selection of <a href="https://oroinc.com/b2b-ecommerce/b2b-ecommerce-comparison" target="_blank">B2B eCommerce solutions</a>.
<!-- Frontend -->
