<a id="setup-dev-env-docker-symfony-windows"></a>

# Set up Environment for OroPlatform Based Application on Windows Subsystem for Linux (WSL) 2

This guide demonstrates how to set up [Docker and Symfony Server development stack](docker-and-symfony/index.md#setup-dev-env-docker-symfony) for Oro applications on Windows 10, version 1903 or higher. Please make sure you have the latest version of the Windows OS before you start.

## Environment Setup

1. Install <a href="https://www.microsoft.com/en-us/p/ubuntu-2004-lts/9n6svws3rx71" target="_blank">Ubuntu 20.04 LTS from the Microsoft Store</a> to use with WSL 2. Alternatively, you can install it with a WSL command `wsl --install -d Ubuntu`, followed by `wsl --set-version Ubuntu 2`
2. Install <a href="https://www.microsoft.com/en-us/p/windows-terminal/9n0dx20hk701" target="_blank">Windows Terminal</a>. It is not required to use a Windows Terminal/Powershell but we recommend using it as it comes with the built-in WSL integration. Please make sure that you run your windows terminal as an administrator. You may be prompted to reboot your PC after installation.
   ![An example of a successful installation of Windows Terminal](img/backend/setup/wsl/terminal-successfull-installation.png)

   If you encounter an error during installation, please follow the link provided in the terminal to troubleshoot the issue or refer to the <a href="https://docs.microsoft.com/en-us/windows/wsl/install" target="_blank">official Microsoft WSL documentation</a>:
   ![An example of an error during terminal installation](img/backend/setup/wsl/terminal-error.png)

   Once rebooted, create a new NIX username and password to log into Ubuntu.
   ![An example of terminal messages displayed once you log into ubuntu](img/backend/setup/wsl/logged-in-ubuntu.png)

   To switch to Ubuntu on your Windows Powershell, click on the dropdown next to the **+** tab and select Ubuntu from the list.
   ![Ubuntu option in the Powershell dropdown](img/backend/setup/wsl/powershell-ubuntu-dropdown-list.png)

   To avoid switching to Ubuntu manually every time, you can set up your Windows Powershell to run Ubuntu by default on startup. For this, navigate to your Windows settings > Startup and change the **Default Profile** to *Ubuntu*, as illustrated in the screenshot below:
   ![Change default terminal profile to Ubuntu](img/backend/setup/wsl/ubuntu-on-powershell.png)

   As WSL integration does not always work well with the Windows file system, go to the Linux file system by typing in `cd` in the terminal:
   ![An example of switching to the Linux file system](img/backend/setup/wsl/switch-to-linux-filesystem.png)
3. Install <a href="https://docs.docker.com/docker-for-windows/install/" target="_blank">Docker Desktop for Windows</a>. During installation, make sure that the checkbox for *Install required Windows components for WSL 2* is selected. Reboot your PC once the installation is finished.
   ![Checkbox for *Install required Windows components for WSL 2* is selected](img/backend/setup/wsl/docker-installation-wsl2.png)
4. Enable <a href="https://docs.docker.com/docker-for-windows/wsl/" target="_blank">Docker Desktop WSL 2 backend</a> for the Ubuntu 20.04 LTS distribution that you installed at step 1.
   * In the **General Settings** of the Docker application, make sure that *Use the WSL 2 based engine* option is selected.
   * In the **Resources > WSL Integration** settings, enable option *Ubuntu*. Apply all changes and restart Docker.

   ![Configure WSL 2 on the docker side](img/backend/setup/wsl/docker-wsl2-config.png)
5. Log into Ubuntu 20.04 LTS using Windows Terminal. All the below commands will be executed in it.
6. Install php 8.4 with all required extensions to Ubuntu 20.04 LTS:

   #### HINT
   It is recommended to run all commands one by one to make sure they exit successfully and avoid missing potential warnings. If you have unreliable connection leading to command failure, please rerun it.

   ```none
   sudo apt install software-properties-common
   sudo add-apt-repository -y ppa:ondrej/php
   sudo apt update
   sudo apt -y install php8.4 php8.4-fpm php8.4-cli php8.4-pdo php8.4-mysqlnd php8.4-xml php8.4-soap php8.4-gd php8.4-zip php8.4-intl php8.4-mbstring php8.4-opcache php8.4-curl php8.4-bcmath php8.4-ldap php8.4-pgsql php8.4-dev php8.4-mongodb
   ```

> You will be prompted to type in your password as you are running the commands as a sudo user.
1. Configure PHP:
   ```none
   echo -e "memory_limit = 2048M \nmax_input_time = 600 \nmax_execution_time = 600 \nrealpath_cache_size=4096K \nrealpath_cache_ttl=600 \nopcache.enable=1 \nopcache.enable_cli=0 \nopcache.memory_consumption=512 \nopcache.interned_strings_buffer=32 \nopcache.max_accelerated_files=32531 \nopcache.save_comments=1" | sudo tee -a  /etc/php/8.4/fpm/php.ini
   echo -e "memory_limit = 2048M" | sudo tee -a  /etc/php/8.4/cli/php.ini
   ```
2. Install Node.js 22:
   ```none
   sudo apt -y install curl dirmngr apt-transport-https lsb-release ca-certificates
   curl -sL https://deb.nodesource.com/setup_16.x | sudo -E bash -
   sudo apt -y install nodejs
   ```
3. Install PNPM 10 Using NPM:
   ```none
   npm install -g pnpm@latest-10
   ```
4. Install Composer:

> ```none
> php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');" && php composer-setup.php
> php -r "unlink('composer-setup.php');"
> sudo mv composer.phar /usr/bin/composer
> ```
1. Install Symfony Server:
   ```none
   sudo apt -y install libnss3-tools
   wget https://get.symfony.com/cli/installer -O - | bash
   echo 'PATH="$HOME/.symfony/bin:$PATH"' >> ~/.bashrc
   source ~/.bashrc
   symfony server:ca:install
   ```

   You can also enable TLS, but as Symfony Server does not automate certificate installation for WSL on Windows, you have to copy the generated certificate manually from the `/usr/local/share/ca-certificates/` folder to the host filesystem and install it manually to your web browser:
   ![An illustration of copying the generated certificate manually from the ``/usr/local/share/ca-certificates/`` folder to the host filesystem](img/backend/setup/wsl/symfony-certificate-1.png)

   An example of importing a certificate in Chrome:
   ![Opening certificates in Chrome settings](img/backend/setup/wsl/chrome-certificates-2.png)![Importing certificate to Chrome](img/backend/setup/wsl/import-certificate-3.png)
2. Configure the network. WSL 2 changes the way networking is configured compared to WSL 1. You need to enable proxy of traffic to permit the traffic through the Windows firewall.

   Run in Ubuntu `ip addr | grep eth0` to see the IP address of the WSL 2 virtual machine.
   ![IP address of WSL 2 virtual machine](img/backend/setup/wsl/ip-addr-ubuntu.png)

   Map WSL 2 port to the internal host `netsh interface portproxy add v4tov4 listenport=8000 listenaddress=0.0.0.0 connectport=8000 connectaddress=172.22.33.170`.

   Configure Windows Defender Firewall, as illustrated below:
   ![Configure Windows Defender Firewall step 1](img/backend/setup/wsl/firewall-1.png)![Configure Windows Defender Firewall step 2](img/backend/setup/wsl/firewall-2.png)![Configure Windows Defender Firewall step 3](img/backend/setup/wsl/firewall-3.png)![Configure Windows Defender Firewall step 4](img/backend/setup/wsl/firewall-4.png)![Configure Windows Defender Firewall step 5](img/backend/setup/wsl/firewall-5.png)![Configure Windows Defender Firewall step 6](img/backend/setup/wsl/firewall-6.png)
3. Restart the terminal and the web browser to get them ready.

## What’s Next

* [Tips and Recommendations](docker-and-symfony/index.md#setup-dev-env-docker-symfony-recommendations)
* [Installation of the Oro Application via the Command-Line Interface](docker-and-symfony/index.md#setup-dev-env-docker-symfony-install-application)
* Consider using the Visual Studio Code or PhpStorm with the built-in WSL integration for development.

<!-- Frontend -->
