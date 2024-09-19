# Contextual Information
Contexts are a way to access information about workflow runs, variables, runner environments, jobs, and steps. Each context is an object that contains properties, which can be strings or other objects.
To Learn more about contextual information. Here is a [Link](https://docs.github.com/en/actions/writing-workflows/choosing-what-your-workflow-does/accessing-contextual-information-about-workflow-runs)

For example, to output the information about a repository, let's use the GitHub contextual object.
```yaml
name: Output information
on: workflow_dispatch
jobs: 
    output:
        runs-on: ubuntu-latest
        steps:
            - name: Output information
              run: echo "${{toJson(github)}}"
```
Below is the output:
![Github_Context](image_9.png)
