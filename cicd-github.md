# Setting up CI/CD with Github Actions

### Continuous integration and continuous deployment are must-haves for organizations that wish to scale and deliver high quality software at a high velocity. This article walks you through the process of using github actions to build CI/CD flows.

1. [What is CI/CD?]()
2. [Adding a github workflow to your project]()
3. [Building a CI workflow with Github actions]()
4. [Building a CD workflow with Github actions]()
5. [Optimizing the workflow]()


## Adding a github workflow to a project

Create a .github/workflows folder. We'll be adding our github actions within this folder. The actions are set in yaml files in a fairly straightforward structure that consists of three parts: 

**name**: the name of the workflow, in our case `Test and Build`
**on**: the trigger that starts the workflow when conditions are satisfied. In our case, we want the workflow to run when a pull request is created and pushed. We'll see examples of other triggers later in the tutorial. 
**jobs**: the actual commands that run in the flow. These can be multiple jobs that run in parallel or jobs that are dependent on one another. In this example there is one job called `build-test`that runs only on ubuntu-latest OS. 


```yaml
name: Test and Build
on:
 pull_request:
   branches:
     - main
jobs:
 build-test:
   runs-on: ubuntu-latest
   steps:
     - name: Checkout
       uses: actions/checkout@v2
       with:
         fetch-depth: 0
     - name: Setup NodeJS 14
       uses: actions/setup-node@v1
       with:
         node-version: 14
     - name: Install yarn
       run: npm install -g yarn
     - name: Install dependencies
       run: yarn install
     - name: Test
       run: yarn test
```

The first step checks out the repository, using a pre-made action `actions/checkout@v2` from the actions marketplace. We pass the `fetch-depth: 0` variable using the **with** property, which tells the chekout process will fetch the repositroy with its entire history. 
The following steps are for installations and running bash commands. 
