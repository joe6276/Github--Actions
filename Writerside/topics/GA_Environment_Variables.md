# Environment Variables & Secrets 

Environment Variables are certain variables that are dynamic e.g. password to connect server to your database might depend if you are on development environment 
or Production environment. 
These two environments might have different databases, hence different passwords.

Below are some JS variables set from environment variables

```Javascript

const clusterAddress = process.env.MONGODB_CLUSTER_ADDRESS;
const dbUser = process.env.MONGODB_USERNAME;
const dbPassword = process.env.MONGODB_PASSWORD;
const dbName = process.env.MONGODB_DB_NAME;

```


You can define environment variables in different levels:
- Workflow level
```yaml
name: Environment Variables
on:
  push:
    branches:
      - master
env:
  MONGO_DB_NAME: githubExample

```
- Job Level (Available for that step only)
```yaml
jobs:
  test:
    env:
      MONGODB_USERNAME: test
      MONGODB_PASSWORD: Test@123
      MONGODB_CLUSTER_ADDRESS: cluster0.15...
      PORT: 80
```

- Step Level (Available for that step only)

To output/ use the environment variables, there are two ways:

```yaml
  - name: Run server
    run: npm start & npx wait-on http://127.0.0.1:$PORT
```

or
```yaml
   - name: Output Environment Variables
     run: |
         echo "MONGO DB UserName : ${{env.MONGODB_USERNAME}}"
         echo " At Port $PORT"
```
Output:

![ENV output](image_17.png)

But the Environment value is still part of the workflow file, which means anyone who can access the workflow file can access the environment variables. So we need **_Secrets_**
## Secrets
Secrets can be stored on :
- Organizational Level
- Repository Level
### Repository Level
 Under Settings find Secrets & Variables > Actions 
![Setting](image_18.png)
![Action](image_19.png)

Now add new repository secret:
![Secret](image_20.png)

Once a secret is stored, it can never be viewed only deleted or updated.

![Secret 1g](image_21.png)

Now, how can we access the secrets:
```yaml
  env:
    MONGODB_USERNAME: ${{secrets.MONGODB_USERNAME}}
```

And Now even when you try to output, GitHub will hide the secret.
![Secret Output](image_22.png)

The above secrets are available for the whole workflow file, hence every job. This might work well, but sometimes you want to use different databases for different Jobs 
e.g., you want to use a database Named "Gh-Dev" for development and "GH-Testing" for testing. This is where environments can play a role in .

Now from your settings> Secrets & variables > Actions
![Enviroment ](image_24.png)

Create a new Environment:
![Create Environment](image_23.png)

Then Add Secrets:

![Add Secrets ](image_25.png)

Now in the Workflow File

```yaml
jobs:
  test:
    environment: testing
    env:
      MONGODB_USERNAME: ${{secrets.MONGODB_USERNAME}}


```

This Job will use the variable defined in the 'testing' environment, and 
you can now configure other environment and give different values for the environment variables.

## Summary

![Summary](image_26.png)