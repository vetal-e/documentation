<a id="setup-dev-env-docker-symfony-mac"></a>

# Set up Environment for OroPlatform Based Application on Mac OS X

This guide demonstrates how to set up [Docker and Symfony Server development stack](docker-and-symfony/index.md#setup-dev-env-docker-symfony) for Oro applications on Mac OS X.

## Environment Setup

1. Install Homebrew package manager to manage all the required software from CLI:
   ```none
   /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install.sh)"
   ```
2. Install Docker with Docker Compose and start the Docker daemon:
   ```none
   brew cask install docker
   open /Applications/Docker.app
   ```
3. Install php 8.4, Composer and Node.js 22:
   ```none
   brew install php@8.4 composer node
   echo 'export PATH="/usr/local/opt/php@8.4/bin:$PATH" \nexport PATH="/usr/local/opt/php@8.4/sbin:$PATH" \nexport PATH="/usr/local/opt/node@20/bin:$PATH"' >> ~/.bash_profile
   ```
4. Install PNPM 10 Using NPM:
   ```none
   npm install -g pnpm@latest-10
   ```
5. If you are going to use an Enterprise Edition of the application, install and enable the mongodb php extension:
   ```none
   pecl install mongodb
   echo "extension=\"mongodb.so\"" >> /usr/local/etc/php/8.4/php.ini
   ```
6. Configure PHP:
   ```none
   echo "memory_limit = 2048M \nmax_input_time = 600 \nmax_execution_time = 600 \nrealpath_cache_size=4096K \nrealpath_cache_ttl=600 \nopcache.enable=1 \nopcache.enable_cli=0 \nopcache.memory_consumption=512 \nopcache.interned_strings_buffer=32 \nopcache.max_accelerated_files=32531 \nopcache.save_comments=1" >> /usr/local/etc/php/8.4/php.ini
   ```
7. Install Symfony Server and enable TLS:
   ```none
   curl -sS https://get.symfony.com/cli/installer | bash
   echo 'export PATH="$HOME/.symfony/bin:$PATH"' >> ~/.bash_profile
   source ~/.bash_profile
   symfony local:server:ca:install
   ```
8. Restart the terminal and web browser to get them ready.

## What’s Next

* [Follow the recommendations](docker-and-symfony/index.md#setup-dev-env-docker-symfony-recommendations)
* [Install the Oro Application via the Command-Line Interface](docker-and-symfony/index.md#setup-dev-env-docker-symfony-install-application)
