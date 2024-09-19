# GitHub Action Example Two

Find the project Here: https://github.com/Teach2Give-Dotnet-Training/GA_second_Action
This is a simple React App with some test files, the goal will be to write an action that can run the test files.

```yaml
name: Example two
on: push
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - name: Get the Code from the repository
        uses: actions/checkout@v4

      - name: Install NodeJS
        uses: actions/setup-node@v4
        with:
          node-version: '20'

      - name: Install Dependencies
        run: npm ci

      - name: Run Tests
        run: npm run test
```

We will give it a name. 
Then run the program whenever we push to the repository.

## More on Events
![Events](image_4.png)
There are so many events that can trigger a workflow. We have repository related events and other events that don't depend on repositories.
Events also have variations. 
To get to know events and variations more you can check at this [link](https://docs.github.com/en/actions/writing-workflows/choosing-when-your-workflow-runs/events-that-trigger-workflows)


## Action
- A (custom) application that performs a (typically complex) frequently repeated task
- You can build your own actions, but you can also use official or community actions.

An alternative to an action is Command
## Command("run")
- A (typically simple) shell command that you define.
```yaml
   steps:
     - name: Get the Code from the repository
       uses: actions/checkout@v4
```
The ubuntu-latest runner by default does not have the code, the above step will make sure that 
the code is downloaded from the repository to the virtual machine.

```yaml
 - name: Install NodeJS
   uses: actions/setup-node@v4
   with:
      node-version: '20'
```

The above step is optional this is because the runner we are using has Node JS installed already.
Here is a List of software installed in it: [Link](https://github.com/actions/runner-images/blob/main/images/ubuntu/Ubuntu2204-Readme.md)

In case you want to use a different version of Node JS you can now use this action and give a different variation.

```yaml
 - name: Install Dependencies
   run: npm ci
```

This step will make sure that the runner installs the right dependencies needed to run the project.

```yaml
 - name: Run Tests
   run: npm run test
```

Now run the command to run the tests. This is also defined in package.json file.

Now push the changes and check your action (Its triggered by a push event)

![Success](image_5.png)

Now let's cause a Bug im the MainContent.test.jsx by negating the test.

```Javascript
describe('MainContent', () => {
  it('should render a button', () => {
    render(<MainContent />);

    expect(screen.getByRole('button')).not.toBeInTheDocument();
  });
```
Now push the changes and check the Workflow.
Now the workflow Fails:
![Failure](image_6.png)

Now let's correct the above and add another step then push.
The below step will mimic deploying an application

```yaml
name: Example two 
on: push
jobs: 
    test:
        runs-on: ubuntu-latest
        steps:
            - name: Get the Code from the repository
              uses: actions/checkout@v4

            - name: Install NodeJS
              uses: actions/setup-node@v4
              with:
                node-version: '20'

            - name: Install Dependencies
              run: npm ci
              
            - name: Run Tests
              run: npm run test
    deploy:
        runs-on: ubuntu-latest
        steps:
            - name: Get the Code from the repository
              uses: actions/checkout@v4

            - name: Install NodeJS
              uses: actions/setup-node@v4
              with:
                node-version: '20'

            - name: Install Dependencies
              run: npm ci
              
            - name: Build Projects
              run: npm run build
            
            - name: Deploy Project
              run: echo " Deploying Project..."

```

Now both steps are successful, but they run in parallel not sequential.
![Steps in Parallel](image_7.png)
## needs
To make this sequential, in the deploy code add:
```yaml
 deploy:
    needs: test
```
![sequential](image_8.png)

## Multi-Events
To trigger workflow based on multiple events 

```yaml
on: [push, workflow_dispatch]
```