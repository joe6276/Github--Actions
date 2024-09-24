# Custom Actions
We have been using public actions so far :
- actions/checkout@v3
- actions/cache@v3
- actions/upload-artifact@v3
- actions/download-artifact@v3

It's time to build custom actions now, but why?

## Why Custom Action
- Simplify workflow steps
Instead of writing multiple (possibly very complex) step definitions, you can build and use a single custom action.
Multiple steps can be grouped into a single custom action, which can be reused.

- No existing (community) action
Existing, public actions might not solve the specific problem you have in your workflow.
Custom actions can contain any logic you need to solve your specific workflow problems.

- You can publish it in the marketplace and ccontribute to the community.

## Different Types of custom Actions
- Javascript Actions
- Docker Actions
- Composite Actions
### Javascript Actions
- Execute a JavaScript file 
- Use JavaScript ( Node.js) + any packages of your choice. 
- Pretty Straightforward (If you know JavaScript)

### Docker Actions
- Create a Dockerfile with your required configuration.
- Perform any tasks of your choice with any Language
- Lots of flexibility but required Docker Knowledge.

### Composite Actions
- Combine multiple workflow steps in one single Action
- Combine run (commands) and use (Actions)
- Allows for reusing shared steps (without extra skills)

Here is out starting Workflow:

```yaml
name: Deployment
on:
  push:
    branches:
      - master
      
jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - name: Get code
        uses: actions/checkout@v3
      - name: Cache dependencies
        id: cache
        uses: actions/cache@v3
        with:
          path: node_modules
          key: deps-node-modules-${{ hashFiles('**/package-lock.json') }}
      - name: Install dependencies
        if: steps.cache.outputs.cache-hit != 'true'
        run: npm ci
      - name: Lint code
        run: npm run lint
  test:
    runs-on: ubuntu-latest
    steps:
      - name: Get code
        uses: actions/checkout@v3
      - name: Cache dependencies
        id: cache
        uses: actions/cache@v3
        with:
          path: node_modules
          key: deps-node-modules-${{ hashFiles('**/package-lock.json') }}
      - name: Install dependencies
        if: steps.cache.outputs.cache-hit != 'true'
        run: npm ci
      - name: Test code
        id: run-tests
        run: npm run test
      - name: Upload test report
        if: failure() && steps.run-tests.outcome == 'failure'
        uses: actions/upload-artifact@v3
        with:
          name: test-report
          path: test.json
  build:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - name: Get code
        uses: actions/checkout@v3
      - name: Cache dependencies
        id: cache
        uses: actions/cache@v3
        with:
          path: node_modules
          key: deps-node-modules-${{ hashFiles('**/package-lock.json') }}
      - name: Install dependencies
        if: steps.cache.outputs.cache-hit != 'true'
        run: npm ci
      - name: Build website
        run: npm run build
      - name: Upload artifacts
        uses: actions/upload-artifact@v3
        with:
          name: dist-files
          path: dist
  deploy:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - name: Get code
        uses: actions/checkout@v3
      - name: Get build artifacts
        uses: actions/download-artifact@v3
        with:
          name: dist-files
          path: ./dist
      - name: Output contents
        run: ls
      - name: Deploy site
        run: echo "Deploying..."
```

if you look at the workflow file, you will notice that there are repeated steps :
- Getting the code
- Caching dependencies

lets build a composite action

```yaml
name: 'Get  and Cache Dependencies'
description: ' Get the dependencies and Cache them'
runs:
  using: 'composite'
  steps:
    - name: Cache dependencies
      id: cache
      uses: actions/cache@v3
      with:
        path: node_modules
        key: deps-node-modules-${{ hashFiles('**/package-lock.json') }}
        
    - name: Install dependencies
      if: steps.cache.outputs.cache-hit != 'true'
      run: npm ci
      shell: bash

```

- Here, we will give it a name and a description first
- **`runs`:** Defines how the action will be executed. 
- This particular action uses a composite type, meaning it contains multiple steps within.
- For any step that uses the run command, we must pass the _shell: bash_

to use it, we need to specify a pathe of where the action file is located:

```yaml
 - name: Load & Cache Dependencies
   uses: ./.github/actions/cache
```

And now the workflow runs well, and it's streamlined.
![Workflow Output](image_35.png)

Composite actions can also have inputs and outputs too.

## JavaScript Action

Let create an S3 bucket
![S3 Creation](image_36.png)
![S3 Access](image_37.png)

Leave the rest as default and create it.
### Static website hosting

Go to the bucket> then properties > Static website hosting> Edit

Enable website hosting and also make sure the index.html is filled in the index document then save the changes.
![Static Website Hosting](image_40.png)

### Permission
We need permission to be able to protect the bucket.

```yaml
{
	"Version": "2012-10-17",
	"Statement": [
		{
			"Sid": "Statement1",
			"Principal": "*",
			"Effect": "Allow",
			"Action": [
				"s3:GetObject"
			],
			"Resource": [
				"arn:aws:s3:::gha-pipeline/*"
			]
		}
	]
}

```
While in AWS Create Access Key:

![Access Key](image_38.png)
![Output Security Credentials](image_39.png)


Now, back to my Workflow:

```yaml
name: 'Get  and Cache Dependencies'
description: ' Get the dependencies and Cache them'
inputs:
  bucket:
    description: ' The S3 Bucket Name'
    required: true

  bucket-region:
    description: ' The S3 Bucket region'
    required: true

  dist-folder:
    description: ' The folder containing artifacts'
    required: true
  
outputs:
  website-url:
    description: The Website URL

runs:
  using: 'node20'
  main: 'main.js'
```

The run part now will use Node.js version 20, and we will have to create a _main.js_ file for the JavaScript code

We will have some inputs that must be passed:
- bucket
- bucket region
- dist folder

And also output the website URL once it's uploaded to the S3 bucket.

Let's look at the main.js:

First, let's install some packages we need :

Inside the .github> actions> deploy-s3-javascript

run 
```Javascript
 npm init -y
```
then: 

```Javascript
npm install @actions/core @actions/github @actions/exec
```
and now the code :

```Javascript
const core = require('@actions/core')
const github = require('@actions/github')
const exec= require('@actions/exec')
async function run(){

    const bucket= core.getInput('bucket', {required:true})
    const bucketRegion= core.getInput('bucket-region', {required:true})
    const distFolder= core.getInput('dist-folder', {required:true})


    const s3Uri= `s3://${bucket}`
    exec.exec (` aws s3 sync ${distFolder} ${s3Uri} --region ${bucketRegion}`)

    const websiteUrl= `http://${bucket}.s3-website-${bucketRegion}.amazonaws.com`
    core.setOutput('website-url', websiteUrl)
}

run()
```


```Javascript
const core = require('@actions/core')
const github = require('@actions/github')
const exec= require('@actions/exec')
```

Here we will require the packages we have just installed.

```Javascript
 const bucket= core.getInput('bucket', {required:true})
 const bucketRegion= core.getInput('bucket-region', {required:true})
 const distFolder= core.getInput('dist-folder', {required:true})
```

then now we will read the inputs that will be passed and save then to the variables defined.

```Javascript
  const s3Uri= `s3://${bucket}`
  exec.exec (` aws s3 sync ${distFolder} ${s3Uri} --region ${bucketRegion}`)
```

Now let's construct the url then we will sync the dist folder received as input with the S3 bucket.


Then Lastly, we will construct the website URl too:

```Javascript

 const websiteUrl= `http://${bucket}.s3-website-${bucketRegion}.amazonaws.com`
 core.setOutput('website-url', websiteUrl)
```

then this will set the output.

Now in the Main Workflow:


```yaml
  deploy:
    needs:  build
    runs-on: ubuntu-latest
    steps:
      - name: Get code
        uses: actions/checkout@v3

      - name: Get Build artifacts
        uses:  actions/download-artifact@v3
        with:
          name: dist-files
          path: ./dist
      - name: Deploy site
        id: deploy
        uses: ./.github/actions/deploy-s3-javascript
        env:
          AWS_ACCESS_KEY_ID: ${{secrets.AWS_ACCESS_KEY_ID}}
          AWS_SECRET_ACCESS_KEY: ${{secrets.AWS_SECRET_ACCESS_KEY}}
        with:
          bucket: gha-pipeline
          dist-folder: ./dist

      - name: Output information
        run: |
          echo " Live URL ${{steps.deploy.outputs.website-url}}"
 
```

We will first get the code, then download the uploaded artifacts (from the build job), and now we will use the custom actions.

As part of the environment variables, we need to pass the access key ID and Access Secret Key for AWS authentication.

We will also pass the inputs:
- The bucket
- the Dist-folder 

The output the URL.


## Outputs
![Job Output](image_41.png)
![S3](image_43.png)
![Site](image_42.png)

