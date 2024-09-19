 # GitHub Events


## Event Activity Types and Filters
some events have Activity Types, others have filters.

### Activity Types 
More detailed control over when a workflow will be triggered.
E.g., pull_request Event Activities include:
- opened
- closed
- edited

```yaml
name: Events Demo 1
on: 
  pull_request:
    types: [opened, closed]
  workflow_dispatch:
    
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Output event data
        run: echo "${{ toJSON(github.event) }}"
      - name: Get code
        uses: actions/checkout@v3
      - name: Install dependencies
        run: npm ci
      - name: Test code
        run: npm run test
      - name: Build code
        run: npm run build
      - name: Deploy project
        run: echo "Deploying..."
```

```yaml
on: 
  pull_request:
    types: [opened, closed]
  workflow_dispatch:
    
```

This part will make sure that when a pull request is opened or closed, the workflow will be triggered. 
The workflow can also be triggered manually using the workflow_dispatch( note that you have to add the semicolon at the end)

Most events with variation have defaults in case there is no type specified. E.g., for pull-request, the event's activity type is opened, synchronize, or reopened.


### Filters
More detailed control over when a workflow will be triggered.
E.g., push Event filter based on target branch.


```yaml
  
  push:
    branches:
      - master
    paths-ignore:
      - ".github/workflows/*"

```

The above actions will filter any push that targets the “master” branch. Any change made to any .yml file located in the .github>workflows directory.



## Cancelling and Skipping Workflow
### Cancelling
- Workflows get cancelled automatically when jobs fail
- You can manually cancel workflows

### Skipping a Workflow

You can Skip via [Skip ci] etc. in the commit message. If the command is part of your commit message, the Workflow won't be executed.

