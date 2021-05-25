# Demonstration Script:


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

## Demo Narrative

1. Firstly the **Deploy Site / Build (pull_request)** workflow will execute to build the application
2. Show the workflow executing
 
      a. Go to the browser tab showing the Actions tab of the repository and select the running job (you may need to refresh) - it will have the same name as the title of your pull request (in this demo script assuming adding Rust) and should be running the Deploy Site flow
      
     b. Show the visualization as the job starts executing, state that the job will take a branch depending on how things are configured for the workflow
     
     c. Go to the browser tab showing the Code tab of the repository and drill down to .github ➡️ workflows ➡️ deploy.yml




## Conclusion/Final Remarks
- We feel this brings the capability to allow for more rich and flexible continuous delivery (CD) workflows for GitHub Actions
- We have a lot of plans to enrich this in future, such as more protection rules for environments.
- Looking for comments and feedback as to how folks take advantage of these new features and what they would like to see in future
- When is this available? GA in the next couple of months as per our public [roadmap](https://github.com/github/roadmap/projects/1): Actions: [Deployment branches Environment protection rule](https://github.com/github/roadmap/issues/164).
