<a id="setup-dev-env-docker-symfony"></a>

# Set up Environment for OroPlatform Based Application with Docker and Symfony Server

During development, you can use Docker to run various application
services (Postgres, ElasticSearch, RabbitMQ, Redis and MailCatcher), but for
simplicity, performance and reliability have PHP and NodeJS installed
locally on a host machine.

## Set Up the Environment

#### HINT
There are quick guides to set up Docker and Symfony Server development stack:

- [Setup on Mac OS X](../mac.md#setup-dev-env-docker-symfony-mac)
- [Setup on Ubuntu 24.04 LTS](../ubuntu.md#setup-dev-env-docker-symfony-ubuntu)
- [Setup on Windows Subsystem for Linux (WSL) 2](../windows.md#setup-dev-env-docker-symfony-windows)

**Development Stack**

- PHP, Composer, Node.js, NPM and PNPM should be installed locally for a better development experience.
- <a href="https://symfony.com/doc/current/setup/symfony_server.html" target="_blank">Symfony Local Web Server</a> is used to make you more productive while
  developing applications. This server is not intended for production
  use. It supports HTTP/2, TLS/SSL, automatic generation of security
  certificates, local domains, and many other features.
- <a href="https://www.docker.com/" target="_blank">Docker</a> is used to run application services.
- <a href="https://docs.docker.com/compose/" target="_blank">Docker Compose</a> is used to manage them all with a single command.

#### NOTE
PHP and NodeJS should meet the [System Requirements](../../system-requirements/index.md#system-requirements).

<a id="setup-dev-env-docker-symfony-recommendations"></a>

**Recommendations**

To avoid reaching composer API rate limit and to work with enterprise applications, configure <a href="https://getcomposer.org/doc/06-config.md#github-oauth" target="_blank">GitHub OAuth token</a>:

```none
composer config -g github-oauth.github.com <oauthtoken>
```

<a id="setup-dev-env-docker-symfony-install-application"></a>

## Install the Application

1. [Get the Oro application source code](../../get-source-files.md#installation-get-files) .
2. Run application services in the folder with your application:

```none
docker compose up -d
```

#### NOTE
On Linux, it may not work if you use Docker as a root user. In this case, consider adding your user to the “docker” group with:

```none
sudo usermod -aG docker your-user
```

1. Install application dependencies:

```none
symfony composer install -n
```

1. If you are using an Enterprise edition application, update the parameters.yml file. Skip this step if you are installing a Community edition application.

```none
composer set-parameters redis_dsn_cache='%env(ORO_REDIS_CACHE_DSN)%' redis_dsn_doctrine='%env(ORO_REDIS_DOCTRINE_DSN)%' redis_dsn_layout='%env(ORO_REDIS_LAYOUT_DSN)%'
```

1. Install Oro application. This may take up to several minutes:

```none
symfony console oro:install -vvv --sample-data=y --application-url=https://127.0.0.1:8000 --user-name=admin --user-email=admin@example.com --user-firstname=John --user-lastname=Doe --user-password=admin --organization-name=Oro --timeout=0 --env=prod -n
```

![Oro application installation in progress](img/backend/setup/wsl/app-installation-in-progress.png)

<a id="setup-dev-env-docker-symfony-services"></a>

## Use a Symfony Server

To automatically apply environment variables exposed by Symfony Server
from Docker Compose and to use the proper PHP version, you should run
all the symfony application commands using `symfony console` instead
of `php bin/console`. Use `symfony php` to run php binaries
using proper PHP version and expose environment variables from the application services defined with Docker Compose.

#### NOTE
On Windows with WSL2 the website is accessible using `https://localhost:8000`, instead of `https://127.0.0.1:8000`.

### Run Symfony Server in a `Dev` Environment

```none
symfony server:start -d
```

### Run Symfony Server in a `Prod` Environment

```none
symfony server:start -d --passthru=index.php
```

### Open the Application in a Browser

```none
symfony open:local
```

### Check Application Logs

```none
symfony server:log
```

### Switch PHP version

You can have multiple versions of PHP versions locally. To use a
specific PHP version for the project, go to the project root folder and run:

```none
echo 8.4 > .php-version
```

This will switch the php version to 8.4 for Symfony Server and all the
console commands wrapped with `symfony`.

### Run Message Consumer in the Background

```none
symfony run -d php bin/console oro:message-queue:consume -vv
```

You can also ask symfony to restart the message consumer when changes happen in `src/` folder:

```none
symfony run -d --watch=src php bin/console oro:message-queue:consume -vv
```

#### NOTE
You can kill Symfony server processes with command `killall <PID>`, you can get PID  from the `symfony server:status` output.

### Check Symfony Server Status

```none
symfony server:status
```

For more details, see: <a href="https://symfony.com/doc/current/setup/symfony_server.html" target="_blank">Symfony Local Web Server</a>.

<a id="optional-local-domain-names"></a>

### Enable Local Domain Names

By default, projects are accessible at a random port of the 127.0.0.1
local IP.

You can enable local domains by <a href="https://symfony.com/doc/current/setup/symfony_server.html#setting-up-the-local-proxy" target="_blank">setting up the Local Proxy</a>.

<!-- Frontend -->

## Manage Application Services

All application services are defined in the `docker-compose.yml` file.
By default, the `docker-compose.yml` file shipped with an application has a
set of recommended services for each application:

* For community edition applications: **Postgres** and **MailCatcher**.
* For enterprise edition applications: **Postgres**, **ElasticSearch**, **RabbitMQ**, **Redis** and **MailCatcher**.

### Override Docker Compose Configuration Locally

You can use `docker-compose.override.yml` file to override Docker
Compose configuration locally. By default, the file is in `.gitignore`.

#### NOTE
For an enterprise application, you first have to update the parameters.yml file to start working with the application services.

### Run Application Services

```none
docker compose up -d
```

### Check Services Logs

```none
docker compose logs -f
```

### Check Application Services Status

```none
docker compose ps
```

### Stop Application Services (No Data Loss)

```none
docker compose stop
```

### Destroy Application Services with all Volumes (Data Loss)

```none
docker compose down -v
```

For more details, see <a href="https://docs.docker.com/compose/" target="_blank">Overview of Docker Compose</a>.

## Store Sessions in Redis

It is not recommended to store sessions on the same redis server as the
cache, but for testing purpose, you can enable it with the following
command:

```none
composer set-parameters 'env(ORO_SESSION_DSN)'='%env(ORO_REDIS_SESSION_DSN)%'
```

## Troubleshooting

**Environment variable not found: “ORO_REDIS_URL”**

This error appears when the Symfony server does not pass environment variables from the Docker Compose to the application.

Make sure that all the application services are up and healthy with `docker-compose ps`. There should be a `redis` service in the list.

If the list is empty, run `docker compose up -d` to start all the services.

**An exception occurred while establishing a connection to figure out your platform version**

Make sure all the application services are up and healthy with `docker compose ps`. There should be `pgsql` service in the list.

If the list is empty, run `docker compose up -d` to start all the services.
