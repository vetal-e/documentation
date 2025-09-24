<a id="demo-environment-docker"></a>

# Docker

## Prerequisite

<a href="https://docs.docker.com/engine/install/" target="_blank">Install Docker</a> with <a href="https://docs.docker.com/compose/install" target="_blank">Docker Compose</a>.

#### NOTE
The application uses port 80, so make sure that other services do not use it.

## Run Application

1. Download the repository with a `docker-compose.yml` configuration file.

   Check out the git repository
   ```none
   git clone https://github.com/oroinc/docker-demo.git
   cd docker-demo
   ```

   or download the archive file and extract it
   ```none
   wget https://github.com/oroinc/docker-demo/archive/master.tar.gz -O - | tar -xzf -
   cd docker-demo
   ```
2. Run application containers

   The configuration is entirely predefined, and you can only change the name of the domain where the application will be located.
   By default, it is oro.demo. If you need to change the domain, create the .env file with content ORO_APP_DOMAIN=my-custom-domain.demo.

   Run init service
   ```none
   docker compose up restore
   ```

   Alternatively, you can install the application from scratch. This will require more time and resources.

   Run install service
   ```none
   docker compose up install
   ```

   You can run the application as soon as it is installed or initialized:
   ```none
   docker compose up application
   ```

   The docker compose will download the required images, create networks and run containers. Application commerce-crm-application is used by default.

   If you want to get the application in a different locale, add the contents of the file .env-locale-de_DE or .env-locale-fr_FR to .env and restart the restore service and application.
   ```none
   cat .env-locale-de_DE >> .env
   ```

   To track the logs from the php-fpm-app container, run docker compose logs -f php-fpm-app. To get the list of containers, run docker compose ps.
3. Add a record to file /etc/hosts
   ```none
   127.0.0.1 oro.demo
   ```
4. Open the application in a browser: `http://oro.demo`.

   To access the back-office, use admin as both login and password. To access the storefront, use the credentials of the predefined demo user roles. To log in as a buyer, use BrandaJSanborn@example.org both as your login and password. To log in as a manager, use AmandaRCole@example.org both as your login and password.

## Access the Mail Catcher

<a href="https://github.com/mailhog/MailHog" target="_blank">Smtp service</a> is additionally launched so you can send emails from the application. It receives all mail and has a web interface that enables you to view it and perform the required actions. The web interface for the mail catcher is available at the address [http://oro.demo/mailcatcher](http://oro.demo/mailcatcher).

## Stop the Application

- To stop and remove all containers, run docker compose down.
- To stop and remove all containers with the data saved in volumes, run docker compose down -v.

## Troubleshooting

If you deployed the application before, pull up fresh images with docker-compose pull.

## About This Project

This repository provides a Docker Compose configuration file (compose.yaml) and demonstrates how to run different applications and required services in containers. Oro Inc. provides images with applications Community Edition in public Docker Hub.

#### IMPORTANT
This deployment is NOT intended for a production environment.

One image is used to run containers in several roles: web server, php-fpm, consumer, WebSocket server, and cron service.
All these services must be running, and the PostgreSQL database must be prepared for a full-fledged application.

#### NOTE
Business Tip

Technologies are at the forefront of digital change in important sectors like manufacturing. Discover the benefits of <a href="https://oroinc.com/b2b-ecommerce/blog/digital-transformation-in-manufacturing/" target="_blank">digital transformation for manufacturing companies</a>.
<!-- Frontend -->
