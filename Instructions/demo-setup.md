
# Demo Setup

## Pre Requirements:

- You are a member of the `pied-piper-inc` org on GitHub (https://github.com/pied-piper-inc). Otherwise, feel free to use https://github.com/affraes-organisation/affuniverse2020 or https://github.com/octo-faq/cd-universe/ as it is a public repository.
- You have a personal account that you have applied the MS Visual Studio Enterprise Subscription FTE Staff Benefit licence to. If you have not done so already, do the following:
  - Create a personal account at https://azure.microsoft.com/en-us/free/ - remember the email address for that account
  - Log in with your MSFT corporate credentials at https://my.visualstudio.com (eg dafiguci@microsoft.com)
  - Link your FTE allowance with your personal account created above


## Azure Setup


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
