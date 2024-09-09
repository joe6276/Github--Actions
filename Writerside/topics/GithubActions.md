# GitHub Actions

## What is GitHub Actions
GitHub Actions is a workflow automation service offered by GitHub, to automate all kinds of repository related processes and actions.

### Repository
Is a bucket that contains the code.

## Use Cases for GitHub Actions
### Code Deployment (CI/CD)
- Automate code testing, building & deployment.
- Combined, they group methods for automating app development and deployment.
- Code changes are automatically built, tested and merged with existing code (CI)
- After integration, new app or package versions are published automatically.
### Code & Repository Management 
- Automate Code Reviews, issues management 
 
## GitHub Actions Key Element
There are three main key elements in GitHub:
- Workflows
- Jobs
- Steps


### Workflows
- Workflows are attached to a GitHub repository 
- They Contain one or more jobs
- They are triggered upon Events

### Jobs
- Jobs define a runner( execution environment)
- Contain one or more steps 
- Run in parallel( default ) or sequential.
- Can be conditional 

### Steps
- Execute a shell script or an action.
- Can use custom or third party actions
- Steps are executed in order 
- Can be conditional
