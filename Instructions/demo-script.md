# Demonstration Script:

## Summary

In this demonstration, we are trying to highlight 4 (ish) new capabilities added to GitHub Actions:

#### Environments
- Keep track of history of deployments, eg staging, production, etc.
- Allows for activity logs that show environments
- Already existed in deployments UI, but is being surfaced now in Actions.

#### Protected Environments
- Prevent unauthorized deployments.
- Required reviewers (similar to code review for branch protection).
- Configured outside of workflow.
- If you try to deploy to a protected environment, the system will pause the job before it goes to a runner and send notifications to the required reviewers.
- They give the ok and the job proceeds.
Separation of concerns - read access to repo is required - this means that the people who can approve a deployment do not necessarily have write access to code, which may be mandatory in some orgs.

#### Scope secrets to specific environments
- Repo level secrets are great but anyone with write level access to repo can change or use those secrets.
- So a environment only scope for secrets that can only be used for that environment's jobs, can only be managed by repository owner or admin (if organization owns the repository), eg a deploy key for the production environment.
- A job cannot access a secret until the protection rules have been passed - approvals, etc.

#### Visualizer
- Track progress of complex workflows
- Pipeline style deployments, matrices, etc

## Known Issues

Currently there is a reproducible issue with Chrome and Safari, in that once:
- a deployment to a the Web App Deployment slot in Azure has occurred and
- you have opened that slot's site in a non-private/incognito window

if you try to reload in that window, or try and open it in another non-private/incognito window - you are getting the previous cached version of the site from somewhere.

If you open an updated slot's site in a private/incognito window, then you see the updated versions.

You also cannot reload a deployment slot's site in a private/incognito window - you have to close that and open another one to see the updated slot's site.
This happens even though a deploy to production is actually a swap slot action. You still have to open the production slot's sites in private/incognito windows to see the changes.

After a period of time, the non-private windows will finally load the new version of the slot's site - but we do not know how long that timeout is.

This does not occur with Firefox, with changes displaying when reloading the window.


## Tour the demo application

- Switch to the Browser tab showing the application in production
- Lists popular repos by language
- Note that **Rust** is not covered - we want to change that

## Useful Browser Tabs

- The application in **production**
- Repository's **Code** Tab
- Repository's **Actions** Tab
- Your notifications page in GitHub to show when you have been requested to review a environment deployment (eg **Staging**)

## Make a quick code change to kick off the process

- Go to Code tab of the repository
- Go to src ➡️ components ➡️ home.js
- Edit the file:

      In the function `SelectLanguages(props){..}`, 

    replace 

      `const languages = ['All', 'JavaScript', 'Java', 'CSharp', 'Python', 'Go'];`

    with

      `const languages = ['All', 'JavaScript', 'Java', 'CSharp', 'Python', 'Go', 'Rust'];`

     If `Rust` is already on the list, delete it.

- Commit with:
**Message:** `adding Rust (or removing Rust)`
- Select the option to create a new branch and start a pull request
- you can call the branch anything, `adding-rust` would be a good example
- Create the pull request: You can use the same title `adding Rust (or removing Rust)`. 

https://user-images.githubusercontent.com/5361987/119584925-fee15f80-be0c-11eb-989f-274493821d41.mp4

## Demo Narrative

1. Firstly the **Deploy Site / Build (pull_request)** workflow will execute to build the application.
2. Show the workflow executing
 
      a. Go to the browser tab showing the **Actions** tab of the repository and select the running job (you may need to refresh) - it will have the same name as the title of your pull request (in this demo script assuming `adding Rust`) and should be running the Deploy Site flow
      
     b. Show the visualization as the job starts executing, state that the job will take a branch depending on how things are configured for the workflow
     
     c. Go to the browser tab showing the **Code** tab of the repository and drill down to  .github ➡️ workflows ➡️ deploy.yml
     
      - Scroll down to the **build** job and point out that it just builds the site and container - does not deploy the application
            - Show that we are using the community of actions - here **docker** to do the heavy lifting for us:
               
           - Login to **Container Registry**
           - **Build and push** the container with caching
      - Show the **Deploy Review** Job
           - Show the **environmen**t keyword. This includes: the **name** of the environment (`review-lab`) and the notification that the **url** for looking at the application built in this environment by this specific execution of the workflow will be generated by the `popular-repos-web` step 
           - specifically the `webapp-url` output.

      - Show that we have a dependency (**needs**) on the **build** job. This means this job will not run until the **build** job succeeds.


     d. Go to the browser tab that has the Pull Request view.

      - Note that the **Deploy Site / Build (pull_request)** has executed and the **Deploy Site / Deploy Review (pull_request)**  is executing

      - Click on **Show environments** to demonstrate that the deployment to the review-lab environment is occurring (or is complete) and has been done by this particular execution of the workflow.

      - Go back to the **Actions** tab and show the execution visualizer. Both **Build** and **Deploy Review** should be complete.

      - Back in the Pull Request view (refresh if necessary) show the environments again and click on **View deployment** - display the site in a private/incognito window (See Issues at the top of this document)

      - That will take you to the web page for that particular deployment and describe that users can see the Rust option showing.

     e. Go back to the Pull Request view and click on **Merge Pull Request**
      - Note that the workflow will kick off again, but will take a different path
      - Show the settings tab and show the Environment secret
     
           - Note that it has the same name (**AZURE_SUBSCRIPTION_SECRET**) as the repository secret, but is only accessed when the workflow is running in the context of the particular environment - in this case, Production.
           -  Click on **Manage Environment** - show the secret and the protection rules.
           -  Explain the wait timer

      - By now the deployment to staging should have taken place. Go to the **Actions** tab and show the visualization for the current workflow.
      
           - Discuss the useful info - like how much time has passed since deployment to staging, how much time is left on a wait timer, etc.
           - A link to the Staging URL that was just published - display the site in a private/incognito window (See Issues at the top of this document)
           - Point out the **Review deployments** button on the visualizer (if you were requested)
           - Also show your **Notifications** browser tab - there are also notifications coming to the mobile app and the Slack integration
           - Go back to the **Actions** workflow visualization and click on **Review deployment**
                   
               - Click on **Production** environment checkbox, add a comment and click on **Approve and deploy**
               - Wait for the **Deploy Production** job to start. Highlight when it does
               - Scroll down and show the Audit log for Events
                   
           - Show the app in production  - display the site in a private/incognito window.

## Conclusion/Final Remarks
- We feel this brings the capability to allow for more rich and flexible continuous delivery (CD) workflows for GitHub Actions
- We have a lot of plans to enrich this in future, such as more protection rules for environments.
- Looking for comments and feedback as to how folks take advantage of these new features and what they would like to see in future
- When is this available? GA in the next couple of months as per our public [roadmap](https://github.com/github/roadmap/projects/1): Actions: [Deployment branches Environment protection rule](https://github.com/github/roadmap/issues/164).
