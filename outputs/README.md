# Outputs

**outputs** in github actions allow us to use values of a runner machine in subsequent runner machines without the need to create artifacts.

```yaml


name: newbuildproject
on:
  # push:
  #   branches:
  #     - master
  #     - develop
  #     - feature/*
  # pull_request:
  #   types:
  #     - opened
  #   branches:
  #     - master
  #     - develop
  #     - feature/*
  workflow_dispatch:
  # paths_ignore:
  #   - 'README.md'
  #   - '.github/workflows/**'
jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - name: Check out repository
        uses: actions/checkout@v2
      - name: Build Project with node 18 version
        uses: actions/setup-node@v3
        with:
          node-version: '18'
      - name: Install dependencies
        run: npm ci
      - name: Run lint
        run: npm run lint
  build:
    needs: lint
    runs-on: ubuntu-latest
    outputs:
      script-file: ${{ steps.build.outputs.script-file }}
    steps:
      - name: Check out repository
        uses: actions/checkout@v3
      - name: Build Project with node 18 version
        uses: actions/setup-node@v2
        with:
          node-version: '18'
      - name: Install dependencies
        run: npm ci
      - name: Run tests
        run: npm test
      - name: Run build
        run: npm run build
      - name: Find JS file
        id: build
        run: find dist/assets -type f -name "*.js" -execdir echo 'script-file={}' >> $GITHUB_OUTPUT ';'
      - name: Upload build artifacts
        uses: actions/upload-artifact@v4
        with:
          name: dist-files
          path: |
            dist
  deploy:
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: Download build artifacts
        uses: actions/download-artifact@v4
        with:
          name: dist-files
      - name: Display build artifacts
        run: ls -la
      - name: Deploy to production
        run: echo "Deploying to production..."
      - name: Output filename
        run: echo "${{ needs.build.outputs.script-file }}"

```
