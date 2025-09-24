<a id="index-0"></a>

<a id="system-requirements"></a>

# System Requirements

OroCommerce is a web application which runs on a server. Users interact with the application via a web browser on any computer or mobile device that have access to the internet or the network where the server is hosted.

## Server-side Requirements

### Resources

Resources configuration depends on the data size and number of active users and integrations. Typical setup could be done on a single server with a minimum of 2 CPU cores, 2GB RAM, and a fast hard drive (SSD is recommended). The application could scale to multiple servers and a separate database server based on the expected load.

### Operating Systems

Linux distributions (RedHat, Ubuntu, Debian, Oracle Linux) are recommended for the production setup.
Windows 7 and above and Mac OS X can be used for the development environment.

### Software

Oro applications are compatible with most web servers with PHP support, but the following configuration is recommended:

| *Web Server*      | * <a href="https://httpd.apache.org/" target="_blank">Apache</a> 2.2.x or 2.4.x<br/>* <a href="https://www.nginx.com/" target="_blank">Nginx</a> latest mainline or stable version<br/><br/>Web server configuration recommendations are well<br/>described in <a href="https://symfony.com/doc/6.4/setup/web_server_configuration.html" target="_blank">Symfony web server documentation</a>                                                                                                                                                                                                                                                                                                                                                                                                          |
|-------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| *PHP*             | * <a href="https://secure.php.net/" target="_blank">PHP</a> >=8.4<br/>* PHP CLI, the same version as for the web server                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| *PHP Settings*    | Few updates to default PHP configuration settings<br/>should be done in php.ini for the web server and<br/>CLI:<br/><br/>* date.timezone must be set<br/>* detect_unicode must be disabled<br/>* memory_limit should be 1Gb or above<br/><br/>Depending on the debugging tools and extensions<br/>used during development, the memory_limit may be<br/>even higher in the development environment.<br/><br/>If the xdebug is installed (which is not<br/>recommended in the production setup):<br/><br/>* xdebug.scream must be disabled<br/>* xdebug.show_exception_trace must be disabled<br/>* xdebug.max_nesting_level above 100<br/><br/>By default, max_execution_time value equals 30<br/>seconds. In case of using the **Schema update**<br/>option, it is recommended to increase this value. |
| *PHP Extensions*  | * ctype<br/>* curl<br/>* fileinfo<br/>* gd<br/>* intl (ICU library 4.4 and above)<br/>* json<br/>* mbstring<br/>* sodium<br/>* openssl<br/>* pgsql<br/>* pcre<br/>* simplexml<br/>* tokenizer<br/>* xml<br/>* zip<br/>* imap<br/>* soap<br/>* bcmath<br/>* ldap<br/>* mongodb (to use OroGridFSConfigBundle)                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| *Database*        | * <a href="https://www.postgresql.org/" target="_blank">PostgreSQL</a> >=16.1                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| *Process Control* | * <a href="http://supervisord.org/" target="_blank">Supervisor</a>  or alternative                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| *Assets*          | * <a href="https://nodejs.org/en/" target="_blank">Node.js</a> >=22<br/>* <a href="https://pnpm.io/" target="_blank">PNPM</a> >10<br/><br/>Used for JS assets minification and SCSS assets<br/>build.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |

#### NOTE
It is recommended to disable phar PHP extension to reduce the risk of PHP unserialization vulnerability.

### Enterprise Edition Software

Enterprise edition is built to support better scale and performance. It is compatible with additional software configuration that enables to achieve these goals.

| *PHP Extensions*   | * pgsql                                                                                                                                                  |
|--------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------|
| *Database*         | * <a href="https://www.postgresql.org/" target="_blank">PostgreSQL</a> / <a href="https://www.enterprisedb.com/" target="_blank">EnterpriseDB</a> >=16.1 |
| *Search Index*     | * <a href="https://www.elastic.co/products/elasticsearch" target="_blank">Elasticsearch</a> >=8.4.1, <9.0                                                |
| *Job Queue*        | * <a href="https://www.rabbitmq.com/" target="_blank">RabbitMQ</a> 3.12.x                                                                                |

<a id="sys-requirements-postgre-config"></a>

**PostgreSQL Configuration**

PostgreSQL uuid-ossp extension should be loaded for proper doctrine’s guid type handling. In order to enable it, one can connect to the database server and run the following sql query:

```sql
CREATE EXTENSION "uuid-ossp";
```

### Optional recommendations

* <a href="http://php.net/manual/en/book.tidy.php" target="_blank">Tidy PHP extension</a> should be installed to make sure that HTML is correctly converted into a text representation
* <a href="https://redis.io/" target="_blank">Redis</a> - could be used for more efficient application caching. Supported versions of Redis: 7.2.x
* <a href="https://pngquant.org" target="_blank">pngquant</a> and <a href="https://github.com/tjko/jpegoptim" target="_blank">jpegoptim</a> are used if it is necessary to optimize the image size in storage
* <a href="https://gotenberg.dev" target="_blank">Gotenberg</a> is used for PDF generation. Supported versions of Gotenberg: 8.0.x

## Client-side Requirements

On the client side, Oro applications could be used with most of the graphical browsers on any operating system.
Recommended and supported browsers are:

> * <a href="https://www.mozilla.org/en-US/firefox/new/" target="_blank">Mozilla Firefox</a> (latest)
> * <a href="https://www.google.com/chrome/" target="_blank">Google Chrome</a> (latest)
> * <a href="https://www.microsoft.com/en-us/edge?form=MA13FJ" target="_blank">Microsoft Edge</a> (latest)
> * <a href="http://www.apple.com/safari/" target="_blank">Safari</a> (latest)

#### NOTE
Any browser needs to have cookies and JavaScript turned on.

#### NOTE
Business Tip

Want to take advantage of the latest trend in digital commerce? Explore the benefits of a <a href="https://oroinc.com/oromarketplace/b2b-marketplace/" target="_blank">B2B online marketplace</a> and successful marketplace examples.
<!-- Frontend -->

**See Also**

* [Performance Optimization](performance-optimization.md)
