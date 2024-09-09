# First GitHub Action

To create a GitHub Action, Let's first Create a repository.
![Create Repository](image.png)

Then click on the Actions tab> then configure.

![Action](image_1.png)

Now delete the steps on the .yml file and paste the following:

```yaml
name: First Action
# Above is the name of the workflow
on: workflow_dispatch
# This states the event that will trigger the workflow
# in this case "workflow_dispatch" means it will be manually triggered.
jobs:
  ## Now list the job to be executed 
  first-job:
    # Give a name (depend on you)
    runs-on: ubuntu-latest
    # This is the virtual machine that will run the workflow
    steps:
    # Now the steps that will be taken 
      - name: First hello World
        run: echo " Hello World "
        # Here we give a name, to uniquely identify the step 
        # Then we use run to run a shell Command
      - name: Say GoodBye
        run: echo " GoodBye!"
        # Another step 
```

Now commit the changes.


## Run the workflow

Under actions > All Workflows You will find the 'First Action' work flow:
![First Workflow](image_2.png)

Click on run workflow.
![Run Workflow](image_3.png)