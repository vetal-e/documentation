<a id="demo-environment-azure"></a>

# Azure Cloud Platform

Azure Marketplace enables you to deploy pre-configured demo instances of the latest LTS releases of OroCommerce Community Edition with demo data.

To proceed with the installation, make sure you have registered with Azure marketplace and have a valid <a href="https://portal.azure.com" target="_blank">Azure portal account</a>.

## Deploy the Solution

To deploy the solution, follow the steps below:

1. Navigate to <a href="https://azuremarketplace.microsoft.com/en-us/marketplace/" target="_blank">Azure Marketplace</a> and search for OroCommerce VM in the search bar.
   ![OroCRM VM or OroCommerce VM in Azure Marketplace search dropdown](img/backend/setup/azure/search.png)
2. Once you select the application type to deploy, click **GET IT NOW** under the application logo on your left.
3. In the pop-up dialog, select the software plan with demo data. This provides all the necessary information for you to test the application, such as a preconfigured list of customers, products, submitted orders, quotes, the structured master, and web catalogs.
4. Click **Continue**.

   You are redirected to Azure Portal to complete the installation.
   At this point, you are asked to log into Azure Portal if you have not already done so.
5. Once again, select the type of application to deploy.
6. Click **Create**.
   ![Select the software plan pop-up dialog](img/backend/setup/azure/software-plan.png)
7. Complete the required fields in the **Basics** tab:
   > ![Creating a VM - Basics tab](img/backend/setup/azure/basics-1.png)
   * *Subscription* - Select your subscription type.
   * *Resource Group* - Create a new resource group, or select an existing one.

     A resource group is a container that holds related resources for an Azure solution. Resource group names only allow alphanumeric characters, periods, underscores, hyphens, and parentheses and cannot end in a period.
   * *Region* - Choose your region, e.g., (US) East US
   * *Virtual Machine Name* - Provide a virtual machine name.

     Virtual machines in Azure have two distinct names: a virtual machine name used as the Azure resource identifier and the guest hostname. When you create a VM in the portal, the same name is used for both the virtual machine and host names. You cannot change the virtual machine name after the VM is created. You can change the hostname when you log into the virtual machine.
   * *Username* - Provide a username for the virtual machine.
   * *Authentication type* - Choose whether to use a password or SSH public key for authentication.
     * For *password*, complete fields Username, Password, Confirm password
     * For *SSH public key*, complete fields Username and SSH public key
   * *Password* - Provide a password to the virtual machine.
   * *Confirm password* - Confirm the password to the virtual machine.
   * *Password for admin part* - Provide a password to log in to the web interface.
   * *Confirm password for admin part* - Confirm the password for the web interface.
8. Click **Virtual Machine Settings**. The fields are predefined with demo data.
   ![Predefined Virtual Machine Settings](img/backend/setup/azure/basic-2.png)
9. Click **Review+Create** to check the provided information. You should see **Running Final Validation**, followed by **Validation Passed**.
   ![Review and validate](img/backend/setup/azure/basics-3.png)
10. Click **Create** at the bottom.
11. Make sure the deployment is completed successfully.
    ![Deployment complete notification](img/backend/setup/azure/deployment-complete.png)
12. Click **Go to resource group**. DNS is already configured.
    ![Configure DNS](img/backend/setup/azure/dns.png)

1. Paste the DNS into the address bar of a new browser window, and add  */admin* to the URL (e.g., http://<DNSprefix>.cloudapp.azure.com/admin).

   The Oro application interface should now be displayed.
   ![OroCommerce startup page](img/backend/setup/azure/admin-startup.png)

   #### IMPORTANT
   **OroCommerce VM Demo Data**: Use *AmandaRCole@example.org* both as your login and password to access the storefront (http://<DNSprefix>.cloudapp.azure.com). To access the back-office of the application (http://<DNSprefix>.cloudapp.azure.com/admin/admin), use *admin* as login and password. To access the application via SSH, specify your username and password or a public key, and restart services (systemctl restart oro\*).

## Configure Application URL

For the demo application to work correctly, you need to configure the application URL, Secure URL, and URL.

1. Navigate to **System Configuration > General Setup > Application Settings** in the application back-office, and provide the *Application URL* (e.g., http://<DNSprefix>.cloudapp.azure.com, or a different domain).
   ![Change application URL in the configuration settings](img/backend/setup/azure/change-app-url.png)

1. (*For OroCommerce only*) Navigate to **System Configuration > Websites > Routing** in the application back-office, and provide *Secure URL* and *URL*.
   ![Change secure url and url in the website routing configuration](img/backend/setup/azure/secure-url.png)
2. Return to the Azure Portal, and click **Restart** to reboot the virtual machine.
   ![Restart VM](img/backend/setup/azure/restart-vm.png)

   Follow the restart progress in the notification bar on your top right.
3. Copy and paste the DNS into the address bar of a new browser window and press Enter.

   The storefront should now be displayed if you have deployed the OroCommerce application.

   #### NOTE
   Due to <a href="https://docs.microsoft.com/en-us/archive/blogs/azuresecurity/pro-tip-on-sending-email-from-azure-virtual-machines-to-external-domains" target="_blank">Azure blocking port 25</a>, we recommend configuring SMTP settings once you install the application if you would like to send messages from the Oro application you have deployed.

<!-- Frontend -->
