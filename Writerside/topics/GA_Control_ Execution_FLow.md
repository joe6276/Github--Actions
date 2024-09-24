# Control Execution Flow

At this point, we have jobs executed in a series, and if one fails, the rest are not executed. That's the default behavior.
![Execution](image_27.png)

But sometimes you want to keep executing even if one previous step fails, or even trigger another step if it fails or even control your execution flow.

## Conditional Jobs & steps

![Control Flow](image_28.png)

```yaml
   - name: Test code
     run: npm run test
     - name: Upload test report
       uses: actions/upload-artifact@v3
        with:
          name: test-report
          path: test.json
```
For the above case, it would make sense if we upload a test report if the  tests fail. So we should run the Upload test report conditionally and only if when the test fail.

```yaml
     - name: Test code
        id: steps-test
        run: npm run test
      - name: Upload test report
        if: failure() && steps.steps-test.outcome =='failure'
        uses: actions/upload-artifact@v3
        with:
          name: test-report
          path: test.json
```
We will first start by assigning the step an ID so that we can know the outcome of the test step.
On the upload test report step
We execute the failure() which should return true if a previous step fails,
and also the outcome of the previous step is checked.
If both are true, then 
we allow the upload step to execute, and it will upload the necessary files.


## Special Condition Functions
![Special Condition Function](image_29.png)


Now when the tests fail, we get a report:
![Report](image_30.png)

## Condition on Jobs


```yaml
  report: 
    needs: [lint,deploy]
    runs-on: ubuntu-latest
    if: failure()
    steps:
      - name: Output Text
        run: echo "Either Lint or test failed"
```

The above job will only run when the lint or deploy a j ob fails

Output:
![Conditional Jobs](image_31.png)


## Continue
Instead of using an if statement,
we can use _continue-on-error_ which is going to make sure the workflow still executes even if there is an error.

We can make our test a failing test then modify the workflow as follows:

```yaml
      - name: Test code
        id: steps-test
        continue-on-error: true
        run: npm run test
      - name: Upload test report
        uses: actions/upload-artifact@v3
        with:
          name: test-report
          path: test.json
```

Output is:
Even through the Test Step Fails:
![Test Fails](image_32.png)
The Workflow succeeds:
![Workflow Success](image_33.png)

## Matrix

Sometimes you have a workflow :
```yaml
name: Matrix -Demo 
on: push
jobs:
    build:
        runs-on: ubuntu-latest
        steps:
            - name: Get Code
              uses: actions/checkout@v4
              
            - name: Install NodeJS
              uses: actions/setup-node@v4
              with:
                node-version: 14
            
            - name: Install Dependencies
              run: npm ci
            
            - name: Build Project
              run: npm run build
```
And you want to see how the workflow would behave in different environments.E.g. How would it work on node version 18, 20  or even on 
different runners. You need to use a matrix in that case:

```yaml
name: Matrix -Demo 
on: push
jobs:
    build:
        continue-on-error: true
        strategy:
            matrix:
                node-version: [12,14,16]
                operating-system: [ubuntu-latest, windows-latest]
        runs-on: ${{matrix.operating-system}}
        steps:
            - name: Get Code
              uses: actions/checkout@v4
              
            - name: Install NodeJS
              uses: actions/setup-node@v4
              with:
                node-version: ${{matrix.node-version}}
            
            - name: Install Dependencies
              run: npm ci
            
            - name: Build Project
              run: npm run build

```
Now in this case, we will run the job with different operating systems and also in different node versions.

![matrix](image_34.png)

But assuming you need Node-version 18 and ubuntu-latest operating system and not Node-version 18 and Windows-latest operating system, 
you can use the include keyword. You can also exclude a certain combination.


```yaml
   strategy:
            matrix:
                node-version: [12,14,16]
                operating-system: [ubuntu-latest, windows-latest]
                include:
                    - node-version: 18
                      operating-system: ubuntu-latest 
                exclude:
                    - node-version: 12
                      operating-system: windows-latest 
  runs-on: ${{matrix.operating-system}}
  steps:
```

## Reusable Workflows
Sometimes you have workflows that others workflows can reuse, for example, lets 
have the deployment job in a separate file.

in a separate file write:

```yaml
name: Reusable Workflow
on: workflow_call
jobs:
    deploy:
        runs-on: ubuntu-latest
        steps:
            - name: Output information
              run: echo " Deploying & Uploading Site..."
```

The only thing that is different is this will be called on _workflow_call_, so that is an event that means another workflow must call it.

on the target workflow write:
```yaml
  Deploy:
        needs: build
        uses: ./.github/workflows/reusable.yaml
```

you must give the relative path to the workflow we want to reuse. 

Now separating the two files brings a problem. The problem is that the deploy job needs the artifacts
produced by the build job.
So we need to pass the file from the Main workflow to the reusable deploy job.

on the deploy job, we are going to accept inputs:
```yaml
name: Reusable Workflow
on: 
    workflow_call:
        inputs:
            artifacts:
                description: The name of the deployable artifact files
                required: false
                default: dist
                type: string
jobs:
    deploy:
        runs-on: ubuntu-latest
        steps:
            - name: Get Code
              uses: actions/download-artifact@v4
              with:
                name: ${{inputs.artifacts}}

            - name: List Files
              run: ls

            - name: Output information
              run: echo " Deploying & Uploading Site..."
```

Here we will define an input that has a description, a required boolean to show if the value must be passed,
a default value for any default you would like to pass in case the required is false and the type of the value 
you will pass.

On the other workflow we need to pass the artifacts:

```yaml
  Deploy:
        needs: build
        uses: ./.github/workflows/reusable.yaml
        with: 
          artifacts: dist-files

```

besides inputs you can also pass secrets. and also outputs.

### Outputs
Below is the reusable workflow:

```yaml
name: Reusable Workflow
on: 
    workflow_call:
        inputs:
            artifacts:
                description: The name of the deployable artifact files
                required: false
                default: dist
                type: string
       
        outputs:
            result:
                description: The result of the deployment operation
                value: ${{jobs.deploy.outputs.outcome}}

jobs:
    deploy:
        outputs:
            outcome: ${{steps.output-step.outputs.step-result}}
        runs-on: ubuntu-latest
        steps:
            - name: Get Code
              uses: actions/download-artifact@v4
              with:
                name: ${{inputs.artifacts}}

            - name: List Files
              run: ls

            - name: Output information
              run: echo " Deploying & Uploading Site..."
            - name: Set Result Output
              id: output-step
              run: echo "::set-output name=step-result::success"
```

```yaml
   - name: Set Result Output
     id: output-step
     run: echo "::set-output name=step-result::success"
```

- name: "Set Result Output."
- id: The ID is output-step, used to reference this step’s output.
run: Sets the output variable step-result to success using echo "::set-output name=step-result::success". 
- outputs: Defines the output outcome for this job, which is taken from the result of the step with ID output-step.

```yaml
 outputs:
    outcome: ${{steps.output-step.outputs.step-result}}
```
- jobs: Defines a job named deploy.
- outputs: Defines the output outcome for this job, which is taken from the result of the step with ID output-step.