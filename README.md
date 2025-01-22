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

