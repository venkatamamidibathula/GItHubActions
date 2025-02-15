# GItHubActions
GitHub Actions Repository

**Upload and Deploy Artifacts**



```yaml

name: Deploy website
on:
  push:
    branches:
      - main
  workflow_dispatch:
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - name: Get code
        uses: actions/checkout@v3
      - name: Install dependencies
        run: npm ci
      - name: Lint code
        run: npm run lint
      - name: Test code
        run: npm run test
  build:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - name: Get code
        uses: actions/checkout@v3
      - name: Install dependencies
        run: npm ci
      - name: Build website
        run: npm run build
      - name: upload to artifactory
        uses: actions/upload-artifact@v4
        with:
          name: dist-files
          path: |
            dist
  deploy:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - name: Downloading artifacts
        uses: actions/download-artifact@v4
        with:
          name: dist-files
      - name: Output contents
        run: ls
      - name: Deploy
        run: echo "Deploying..."


```
---

**Outputs**

Outputs are published values that can be used for other jobs.


**$GITHUB_OUTPUT**: It targets a special file created by GITHUB in environment which job runs where output key value pair is written to.


```yaml

name: Deploy website
on:
  push:
    branches:
      - main
  workflow_dispatch:
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - name: Get code
        uses: actions/checkout@v3
      - name: Install dependencies
        run: npm ci
      - name: Lint code
        run: npm run lint
      - name: Test code
        run: npm run test
  build:
    needs: test
    runs-on: ubuntu-latest
    outputs:
      script-file: ${{ steps.publish.outputs.filename }} #steps.id.outputs.outputname
    steps:
      - name: Get code
        uses: actions/checkout@v3
      - name: Install dependencies
        run: npm ci
      - name: Build website
        run: npm run build
      - name: Publish JS filename
        id: publish
        run: find dist/assets/*.js -type f -execdir echo 'filename={}' >> $GITHUB_OUTPUT ';' #It targets a special file created by GITHUB in environment which job runs where output key value pair is written to.
      - name: upload to artifactory
        uses: actions/upload-artifact@v4
        with:
          name: dist-files
          path: |
            dist
  deploy:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - name: Downloading artifacts
        uses: actions/download-artifact@v4
        with:
          name: dist-files
      - name: Output contents
        run: ls
      - name: Deploy
        run: echo "Deploying..."

```

**Using output variables in other jobs**

**Syntax**: needs.build.output.script-file

```yaml
name: Deploy website
on:
  push:
    branches:
      - main
  workflow_dispatch:
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - name: Get code
        uses: actions/checkout@v3
      - name: Install dependencies
        run: npm ci
      - name: Lint code
        run: npm run lint
      - name: Test code
        run: npm run test
  build:
    needs: test
    runs-on: ubuntu-latest
    outputs:
      script-file: ${{ steps.publish.outputs.filename }} #steps.id.outputs.outputname
    steps:
      - name: Get code
        uses: actions/checkout@v3
      - name: Install dependencies
        run: npm ci
      - name: Build website
        run: npm run build
      - name: Publish JS filename
        id: publish
        run: find dist/assets/*.js -type f -execdir echo 'filename={}' >> $GITHUB_OUTPUT ';' #It targets a special file created by GITHUB in environment which job runs where output key value pair is written to.
      - name: upload to artifactory
        uses: actions/upload-artifact@v4
        with:
          name: dist-files
          path: |
            dist
  deploy:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - name: Downloading artifacts
        uses: actions/download-artifact@v4
        with:
          name: dist-files
      - name: Output contents
        run: ls
      - name: Deploy
        run: echo "Deploying..."
      - name: Output filename
        run: echo "${{ needs.build.outputs.script-file }}"
```

---

**Caching Dependencies**

- Dependencies can be centrally cached which aid in fast job executions.
- If dependencies are not cached the jobs will have reinstall all dependencies potentially causing delays.

- Action to be used **actions/cache@v3** and key should be a dynamic value which uses a special variable in github called **deps-node-modules-{{ hashFiles(****'/package-lock.json') }}**

```yaml


name: Deploy website
on:
  push:
    branches:
      - main
  workflow_dispatch:
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - name: Get code
        uses: actions/checkout@v3
      - name: cache dependencies
        uses: actions/cache@v3
        with:
          path: ~/.npm
          key: deps-node-modules-${{ hashFiles('**/package-lock.json') }}
          #used for retrieving the cache and be used by the runner machine and also for discarding and the value is a dynamic value
      - name: Install dependencies
        run: npm ci
      - name: Lint code
        run: npm run lint
      - name: Test code
        run: npm run test
  build:
    needs: test
    runs-on: ubuntu-latest
    outputs:
      script-file: ${{ steps.publish.outputs.filename }} #steps.id.outputs.outputname
    steps:
      - name: Get code
        uses: actions/checkout@v3
      - name: cache dependencies
        uses: actions/cache@v3
        with:
          path: ~/.npm
          key: deps-node-modules-${{ hashFiles('**/package-lock.json') }}
      - name: Install dependencies
        run: npm ci
      - name: Build website
        run: npm run build
      - name: Publish JS filename
        id: publish
        run: find dist/assets/*.js -type f -execdir echo 'filename={}' >> $GITHUB_OUTPUT ';' #It targets a special file created by GITHUB in environment which job runs where output key value pair is written to.
      - name: upload to artifactory
        uses: actions/upload-artifact@v4
        with:
          name: dist-files
          path: |
            dist
  deploy:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - name: Downloading artifacts
        uses: actions/download-artifact@v4
        with:
          name: dist-files
      - name: Output contents
        run: ls
      - name: Deploy
        run: echo "Deploying..."
      - name: Output filename
        run: echo "${{ needs.build.outputs.script-file }}"

```

---

**Environment Variables**

You can define environment variables at **job** level or **workflow level** and reference them in your github yaml using the parameter ${{ env.variable }}

However, this approach is not advisable as it can lead to exposing them on repo and for that reason we have to use the secrets configuration.

```yaml


name: Deployment
on:
  push:
    branches:
      - develop
      - master
jobs:
  test:
    env:
      MONGODB_CLUSTER_ADDRESS: cluster0.kka04.mongodb.net
      MONGODB_USERNAME: mongouser
      MONGODB_PASSWORD: FMUW0JGcqtDcD8dM
      PORT: 8080
    runs-on: ubuntu-latest
    steps:
      - name: Get Code
        uses: actions/checkout@v3
      - name: Cache dependencies
        uses: actions/cache@v3
        with:
          path: ~/.npm
          key: npm-deps-${{ hashFiles('**/package-lock.json') }}
      - name: Install dependencies
        run: npm ci
      - name: Run server
        run: npm start & npx wait-on http://127.0.0.1:$PORT
      - name: Run tests
        run: npm test
      - name: Output information
        run: echo "MongoDB username is ${{ env.MONGODB_USERNAME }}"
  deploy:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - name: Output information
        run: |
          echo "Output Mongo user name : ${{ env.MONGODB_USERNAME }}"
```
---

**Secrets**

![Secrets Image](secrets.jpg)


```yaml


name: Deployment
on:
  push:
    branches:
      - develop
      - master
jobs:
  test:
    env:
      MONGODB_CLUSTER_ADDRESS: cluster0.kka04.mongodb.net
      MONGODB_USERNAME: ${{ secrets.MONGODB_USERNAME }}
      MONGODB_PASSWORD: ${{ secrets.MONGODB_PASSWORD }}
      PORT: 8080
    runs-on: ubuntu-latest
    steps:
      - name: Get Code
        uses: actions/checkout@v3
      - name: Cache dependencies
        uses: actions/cache@v3
        with:
          path: ~/.npm
          key: npm-deps-${{ hashFiles('**/package-lock.json') }}
      - name: Install dependencies
        run: npm ci
      - name: Run server
        run: npm start & npx wait-on http://127.0.0.1:$PORT
      - name: Run tests
        run: npm test
      - name: Output information
        run: echo "MongoDB username is ${{ env.MONGODB_USERNAME }}"
  deploy:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - name: Output information
        run: |
          echo "Output Mongo user name : ${{ env.MONGODB_USERNAME }}"

```
---

**Repository Environments**

![githubenvs](githubenvs.png)

- Most importantly we need this if we have environment specific secrets.
- It can be specified using environment key. Its similar to develop branch only triggers for dev and test environment and release branch triggers for stage.

```yaml

name: Deployment
on:
  push:
    branches:
      - develop
      - master
jobs:
  test:
    environment: testing
    env:
      MONGODB_CLUSTER_ADDRESS: cluster0.kka04.mongodb.net
      MONGODB_USERNAME: ${{ secrets.MONGODB_USERNAME }}
      MONGODB_PASSWORD: ${{ secrets.MONGODB_PASSWORD }}
      PORT: 8080
    runs-on: ubuntu-latest
    steps:
      - name: Get Code
        uses: actions/checkout@v3
      - name: Cache dependencies
        uses: actions/cache@v3
        with:
          path: ~/.npm
          key: npm-deps-${{ hashFiles('**/package-lock.json') }}
      - name: Install dependencies
        run: npm ci
      - name: Run server
        run: npm start & npx wait-on http://127.0.0.1:$PORT
      - name: Run tests
        run: npm test
      - name: Output information
        run: echo "MongoDB username is ${{ env.MONGODB_USERNAME }}"
  deploy:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - name: Output information
        run: |
          echo "Output Mongo user name : ${{ env.MONGODB_USERNAME }}"
```

![Secret Variables](secvars.jpg)


---

**Conditional Flow Executions**

**URL For Reference** : https://docs.github.com/en/actions/writing-workflows/choosing-what-your-workflow-does/accessing-contextual-information-about-workflow-runs


![Secret Variables](cond1.jpg)

```yaml


name: Website Deployment
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
          path: ~/.npm
          key: deps-node-modules-${{ hashFiles('**/package-lock.json') }}
      - name: Install dependencies
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
          path: ~/.npm
          key: deps-node-modules-${{ hashFiles('**/package-lock.json') }}
      - name: Install dependencies
        run: npm ci
      - name: Test code
        id: testfail
        run: npm run test
      - name: Upload test report
        if: failure() && steps.testfail.outcome == 'failure'
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
          path: ~/.npm
          key: deps-node-modules-${{ hashFiles('**/package-lock.json') }}
      - name: Install dependencies
        run: npm ci
      - name: Build website
        id: build-website
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
      - name: Get build artifacts
        uses: actions/download-artifact@v3
        with:
          name: dist-files
      - name: Output contents
        run: ls
      - name: Deploy
        run: echo "Deploying..."

```

**Failure condition at job level**

We can apply failure condition at job level.

```yaml

name: Website Deployment
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
          path: ~/.npm
          key: deps-node-modules-${{ hashFiles('**/package-lock.json') }}
      - name: Install dependencies
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
          path: ~/.npm
          key: deps-node-modules-${{ hashFiles('**/package-lock.json') }}
      - name: Install dependencies
        run: npm ci
      - name: Test code
        id: testfail
        run: npm run test
      - name: Upload test report
        if: failure() && steps.testfail.outcome == 'failure'
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
          path: ~/.npm
          key: deps-node-modules-${{ hashFiles('**/package-lock.json') }}
      - name: Install dependencies
        run: npm ci
      - name: Build website
        id: build-website
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
      - name: Get build artifacts
        uses: actions/download-artifact@v3
        with:
          name: dist-files
      - name: Output contents
        run: ls
      - name: Deploy
        run: echo "Deploying..."
  report:
    needs: [lint,deploy]
    if: failure()
    runs-on: ubuntu-latest
    steps:
      - name: Output Information
        run: |
          echo "Something went wrong"
          echo "$ {{ toJSON(github) }}"
```
**continue-on-error**: This flag lets the job to proceed with successive despite errors.

**cache-hit** is a variable output that can be used for comparison.
**steps.id.outputs.param**

```yaml


name: Website Deployment
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
          path: ~/.npm
          key: deps-node-modules-${{ hashFiles('**/package-lock.json') }}
      - name: Install dependencies
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
        if: steps.cache.output.cache-hit != 'true'
        run: npm ci
      - name: Test code
        id: testfail
        run: npm run test
      - name: Upload test report
        continue-on-error: true
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
          path: ~/.npm
          key: deps-node-modules-${{ hashFiles('**/package-lock.json') }}
      - name: Install dependencies
        run: npm ci
      - name: Build website
        id: build-website
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
      - name: Get build artifacts
        uses: actions/download-artifact@v3
        with:
          name: dist-files
      - name: Output contents
        run: ls
      - name: Deploy
        run: echo "Deploying..."
  report:
    needs: [lint,deploy]
    if: failure()
    runs-on: ubuntu-latest
    steps:
      - name: Output Information
        run: |
          echo "Something went wrong"
          echo "$ {{ toJSON(github) }}"
```

---

**Matrix Jobs**: A matrix job in GitHub Actions allows you to run multiple job configurations in parallel with different parameters. This is particularly useful for testing your code across different environments, such as multiple versions of a programming language, operating systems, or dependency versions.

```yaml

name: matrixjob
on: push
jobs:
  build:
    continue-on-error: true
    strategy:
      matrix:
        node-version: [12,14,16]
        operating-system: [ubuntu-latest,windows-latest]
    runs-on: ${{ matrix.operating-system }}
    steps:
      - name: Get code
        uses: actions/checkout@v3
      - name: Cache dependencies
        id: cache
        uses: actions/cache@v3
        with:
          path: ~/.npm
          key: deps-node-modules-${{ hashFiles('**/package-lock.json') }}
      - name: Install dependencies
        run: npm ci
      - name: Build code
        run: npm build

```

---

**Include and Exclude** in matrix jobs.


In GitHub Actions, you can use the include and exclude keywords within a matrix strategy to fine-tune the combinations of parameters that are tested. This allows you to add specific combinations that are not covered by the general matrix or to exclude certain combinations that you don't want to test.

```yaml

name: matrixjob
on: push
jobs:
  build:
    continue-on-error: true
    strategy:
      matrix:
        node-version: [12,14,16]
        operating-system: [ubuntu-latest,windows-latest]
        include:
          - node-version: 18
            operating-system: ubuntu-latest
        exclude:
          - node-version: 12
            operating-system: windows-latest
    runs-on: ${{ matrix.operating-system }}
    steps:
      - name: Get code
        uses: actions/checkout@v3
      - name: Cache dependencies
        id: cache
        uses: actions/cache@v3
        with:
          path: ~/.npm
          key: deps-node-modules-${{ hashFiles('**/package-lock.json') }}
      - name: Install dependencies
        run: npm ci
      - name: Build code
        run: npm build

```
---

**Resuable workflows**


Reusable workflows in GitHub Actions allow you to define a set of steps once and reuse them across multiple workflows. This helps in maintaining consistency and reducing duplication in your CI/CD pipelines.

### Creating a Reusable Workflow

1. **Define the reusable workflow** in a YAML file.
2. **Call the reusable workflow** from another workflow.

```yaml


name: Website Deployment
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
          path: ~/.npm
          key: deps-node-modules-${{ hashFiles('**/package-lock.json') }}
      - name: Install dependencies
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
        if: steps.cache.output.cache-hit != 'true'
        run: npm ci
      - name: Test code
        id: testfail
        run: npm run test
      - name: Upload test report
        continue-on-error: true
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
          path: ~/.npm
          key: deps-node-modules-${{ hashFiles('**/package-lock.json') }}
      - name: Install dependencies
        run: npm ci
      - name: Build website
        id: build-website
        run: npm run build
      - name: Upload artifacts
        uses: actions/upload-artifact@v3
        with:
          name: dist-files
          path: dist
  deploy:
    needs: build
    uses: ./.github/workflows/reusable.yml
    #runs-on: ubuntu-latest
    #steps:
    #  - name: Get build artifacts
    #    uses: actions/download-artifact@v3
    #    with:
    #      name: dist-files
    #  - name: Output contents
    #    run: ls
    #  - name: Deploy
    #    run: echo "Deploying..."
  report:
    needs: [lint,deploy]
    if: failure()
    runs-on: ubuntu-latest
    steps:
      - name: Output Information
        run: |
          echo "Something went wrong"
          echo "$ {{ toJSON(github) }}"
```

**reusable.yml**

```yaml
name: Reusable Deploy
on: workflow_call
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Output information
        run: echo "Deploying & uploading...."

```
---

**Reusable Inputs**

In GitHub Actions, reusable workflows can accept inputs that allow you to customize the behavior of the workflow when it is called. These inputs are defined in the reusable workflow using the workflow_call event. You can specify the type, description, and whether the input is required.


```yaml

name: Reusable Deploy
on: 
  workflow_call:
    inputs:
      artifact-name:
        description: The name of deployable artifact
        required: false
        default: dist
        type: string
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Output information
        run: echo "Deploying & uploading...."

```


```yaml


name: Website Deployment
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
          path: ~/.npm
          key: deps-node-modules-${{ hashFiles('**/package-lock.json') }}
      - name: Install dependencies
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
        if: steps.cache.output.cache-hit != 'true'
        run: npm ci
      - name: Test code
        id: testfail
        run: npm run test
      - name: Upload test report
        continue-on-error: true
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
          path: ~/.npm
          key: deps-node-modules-${{ hashFiles('**/package-lock.json') }}
      - name: Install dependencies
        run: npm ci
      - name: Build website
        id: build-website
        run: npm run build
      - name: Upload artifacts
        uses: actions/upload-artifact@v3
        with:
          name: dist-files
          path: dist
  deploy:
    needs: build
    uses: ./.github/workflows/reusable.yml
    with:
      artifact-name: dist-file
  report:
    needs: [lint,deploy]
    if: failure()
    runs-on: ubuntu-latest
    steps:
      - name: Output Information
        run: |
          echo "Something went wrong"
          echo "$ {{ toJSON(github) }}"
```

