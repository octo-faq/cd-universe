
# Demo Setup

## Pre Requirements:

- You are a member of the `pied-piper-inc` org on GitHub (https://github.com/pied-piper-inc). Otherwise, feel free to use https://github.com/affraes-organisation/affuniverse2020 or https://github.com/octo-faq/cd-universe/ as it is a public repository.
- You have a personal account that you have applied the MS Visual Studio Enterprise Subscription FTE Staff Benefit licence to. If you have not done so already, do the following:
  - Create a personal account at https://azure.microsoft.com/en-us/free/ - remember the email address for that account
  - Log in with your MSFT corporate credentials at https://my.visualstudio.com (eg dafiguci@microsoft.com)
  - Link your FTE allowance with your personal account created above


## Azure Setup

From the Azure side you will need to setup the following:
- An [Azure Resource Group](#create-a-resource-group) Resource Group: https://docs.microsoft.com/en-us/azure/azure-resource-manager/management/manage-resource-groups-portal#create-resource-groups
- An Azure Container registry: https://docs.microsoft.com/en-us/azure/container-registry/container-registry-get-started-portal
- An Azure App Service Plan: https://docs.microsoft.com/en-us/azure/app-service/app-service-plan-manage
- Registry login credentials: https://github.com/azure/login#configure-deployment-credentials

### Step By Step:

- Log in to https://portal.azure.com
- If you are prompted to Pick an account to continue to Microsoft Azure, choose the email address / personal account that has the MS Visual Studio Enterprise Subscription Staff Benefit applied

#### Create a Resource Group

We are creating a new Resource Group to group/hold all the resources we need for this demo together, so for Resource Group, create a new Resource Group called `popular-repos-resource-group`.
- Enter `popularReposRegistry` for the Name of the registry. 
- Note the Region to keep everything together with the App Service Plan you will soon create.
- Click on **Review + create**.
- Click on **Create**.
- You should see a message saying "Your deployment is complete".

#### Create a container registry

First you have to create a container registry (where we are going to host our container image), as well as create access keys that you can store in GitHub secrets to use during the workflow
- Click on **Create a resource**.
- Find and select **Container Registry**. 
- Choose the **Resource Group** created in the previous section
- Click on **Create**
- In the Settings section, select Access keys
- Turn on Admin user
- Note down the Login server, the Username and one of the two passwords (**password** or **password2**) for use later on in configuration and our secrets
- Click on **Home**

#### Create an App Service Plan

- Click on **Create a resource**.
- Find and select **App Service Plan**.
- Click on **Create**.
- For Resource Group, choose  `popular-repos-resource-group`, which you created in the previous section.
- For the Name use `popular-repos`.
- Use Linux for the Operating System.
- For Region use the region you chose when creating the container registry.
- Keep the default Sku and size.
- Click on **Review + create**.
- Click on **Create**.
- You should see a message saying "Your deployment is complete".
- Click on **Home**.

#### Create a WebApp

Here you will create a Web App with a production slot, give that production slot the credentials for pulling the container from the registry and create a staging slot with a copy of the production slot's configuration. Whilst the demo deploys to production by swapping slots, this way we ensure that both slots have the credentials required, so that staging will always have the credentials to pull the container.
- Click on Create a resource
- Find and select Web App
- Click on Create
- For Resource Group, use the same resource group you used to create the registry
- For the Name, use popular-repos-app
- For  Publish, select Docker Container
- Use Linux for the Operating System
- Select the region the App Service Plan you created above  is hosted in
- Select the App Server Plan you created above
- Keep the default Sku and size
- Click on **Review + create**.
- Click on **Create**.
- You should see a message saying "Your deployment is complete"
- Click on **Home**.
- Click on **App Services**.
- Click on the app you just created - `popular-repos-app`.
- Click on **Deployment Center**.
- Click on **Settings**.
- For Container Type, select Single Container.
- For Registry source select Private Registry.
- For Server URL, enter the value of Login server you noted down from when you created the Container Registry, above.
- For Username, enter the value of Username you noted down from when you created the Container Registry, above.
- For Password, enter the value of the password (password or password2) you noted down from when you created the Container Registry, above.
- For Full Image Name and Tag, use the name of the App Service you created (popular-repos). No tag needed.
- Leave Startup File blank and Continuous deployment Off.
- Click on Save.
- Click on Deployment slots.
- Click on + Add Slot.

	- Enter the Name staging.
	- Clone settings from popular-repos-app.
	- Click on Add.

#### Setup Credentials
- Open the Azure Cloud Shell at https://shell.azure.com. Again choose the email address / personal account that has the MS Visual Studio Enterprise Subscription Staff Benefit applied if prompted
- Choose **Bash** for the shell to run in if prompted
- Enter the following (this example has wrapped to fit on the page. Use two lines - the first starting with az and the second with --scopes) :

`az ad sp create-for-rbac --name "{sp-name}" --sdk-auth --role contributor \`

`--scopes /subscriptions/{subscription-id}/resourceGroups/{resource-group}`

- Replace fields in this command with the following:
	- **{sp-name}** with a suitable name for your service principal, such as the name of the app itself. The name must be unique within your organization.
	- **{subscription-id}** with the subscription ID you want to use (found in Subscriptions in portal)
	- **{resource-group}** the resource group containing the web app.
	- Copy the resultant JSON output into a file and keep it somewhere safe. You will use the contents of this for a secret a little later.



## GitHub Setup

### Mirror the repository

- Create a blank repository on GitHub.com
- Create a public repository in an organization that has the beta enabled (for these steps we will use https://github.com/affraes-organisation/affuniverse2020)
- Open Terminal.
- Create a temporary, bare clone of the repository:

`$ git clone --bare https://github.com/pied-piper-inc/universe-2020.git`

- Mirror-push to the new repository.

`$ cd old-repository.git`

`$ git push --mirror https://github.com/octodemo/affuniverse2020.git`

- Remove the temporary local repository you created earlier.

`$ cd ..`

`$ rm -rf old-repository.git`

### Modify the Repository
#### Change Workflows

Depending on the resource names you created above you may need to modify the .github/workflows/deploy.yml file in your copy of the repository. In that file:

1. In the **deploy_to_review**: job, on line 76, modify the --name, --resource-group and --configuration-source accordingly. Using our values created so far, 
 	az webapp deployment slot create --name popular-repos --resource-group pied-piper-inc --slot review-pr-${{ github.event.number }} --configuration-source popular-repos 
becomes
	az webapp deployment slot create --name popular-repos-app --resource-group popular-repos-resource-group --slot review-pr-${{ github.event.number }} --configuration-source popular-repos-app


2. In the **deploy_to_staging**: job, on line 82, modify the app-name: value accordingly. Using our values created so far, 
 	`app-name: 'popular-repos'`
 becomes
 	`app-name: 'popular-repos-app'`

3. In the deploy_to_staging: job, on line 104, modify the app-name: value accordingly. Using our values created so far, 
`app-name: 'popular-repos'`
becomes
`app-name: 'popular-repos-app'`

4. In the **deploy_to_production**: job, on line 114, modify the url: value accordingly. Using our values created so far, 
`url: 'https://popular-repos.azurewebsites.net'`
becomes
`url: 'https://popular-repos-app.azurewebsites.net'`

5. In the **deploy_to_production**: job, on line 124, modify the --name and --resource-group and accordingly. Using our values created so far, 
`az webapp deployment slot swap --name popular-repos --resource-group pied-piper-inc --slot staging --target-slot production`
becomes
`az webapp deployment slot swap --name popular-repos-app --resource-group popular-repos-resource-group --slot staging --target-slot production`

#### Setup Repository Secrets

In your copy of the repository, add the following secrets at the repository level:

- **AZURE_REGISTRY_NAME** - The value of **Login server** you noted down from step 12 of Creating a Container Registry, above
- **AZURE_REGISTRY_USER** -  The value of **Username** you noted down from step 12 of Creating a Container Registry, above
- **AZURE_REGISTRY_PASS** -  The password (**password** or **password2**) you noted down from step 12 of Creating a Container Registry, above
- **AZURE_SUBSCRIPTION_SECRET** - the contents of the JSON file you created in Step 4 of Create Credentials, above 

#### Setup Repository Secrets
In your copy of the repository, add the following environments:
- production: Set environment rules such that you are a required reviewer
- review-lab
- staging
