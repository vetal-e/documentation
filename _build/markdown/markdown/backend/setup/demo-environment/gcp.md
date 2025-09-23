<a id="demo-environment-gcp"></a>

# Google Cloud Platform

Google Cloud Platform enables you to deploy your Oro application instance in one click without manually configuring the software and settings. OroCommerce VM images comes with demo data and provides all the necessary information for you to test the application, such as a preconfigured list of customers, products, submitted orders, quotes, the structured master, and web catalogs. You can also explore the storefront using one of the pre-configured demo user roles. Sign in either as a guest user, a buyer (use *BrandaJSanborn@example.org* as your login and password), or a manager (use *AmandaRCole@example.org* as your login and password).

## Deploy the Solution

1. Navigate to <a href="https://cloud.google.com/marketplace" target="_blank">Google Cloud Marketplace</a>, click **Explore the marketplace** and then search for your solution provided by Oro Inc.
   ![A page of the Oro solution](img/backend/setup/gcp/oro_solution.png)
2. Click **Launch**.
3. The Oro solution deployment page displays the default settings (e.g., name, zone, machine type, boot disk type, networking interfaces, etc.). You can accept or customize them if necessary.
   ![The details page of the Oro solution settings](img/backend/setup/gcp/oro_solution_settings.png)
4. When complete, click **Deploy** on the bottom left to launch the deployment process. Once the deployment is finished, you should see the following information:
   ![The details page of the deployed Oro solution](img/backend/setup/gcp/deployed_oro_solution.png)

## Access the Oro Application

Use the generated credentials to access your Oro application:

* **Site Address** — a link to your Oro application storefront (only for OroCommerce).
* **Admin URL** — a link to your Oro application back-office.
* **Admin user** — a username used to log into the admin panel (back-office).
* **Admin password** - a password used to log into the admin panel (back-office).

Also, you can access the VM instance using SSH by clicking SSH and selecting the required option from the dropdown.

![Access the Oro application using SSH](img/backend/setup/gcp/oro_solution_via_ssh.png)

You can delete the deployment by clicking **Delete** on the upper left, next to the solution name. Resources created by this deployment, including VM instances, disks, and firewalls will be deleted as well.

![Delete the deployment](img/backend/setup/gcp/oro_solution_delete.png)
<!-- Frontend -->
