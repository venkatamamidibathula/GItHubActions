**Composite Actions**: Whenver you find steps in workflows across multiple jobs thats a use case for composite action.

```yaml
name: 'Get & Cache Dependencies'
description: 'Get the dependencies and cache them'
inputs:
  caching:
    description: Whether to cache them or not
    required: true
runs:
  using: 'composite'
  steps:
    - name: Get dependencies
      run: |
        echo "Getting dependencies..."
        # Add your commands to get dependencies here
    - name: Cache dependencies
      if: ${{ inputs.caching == 'true' }}
      uses: actions/cache@v3
      with:
        path: |
          **/node_modules
        key: deps-node-modules-${{ hashFiles('**/package-lock.json') }}
    - name: Install dependencies
      if: ${{ inputs.caching == 'true' && steps.cache.outputs.cache-hit != 'true' }}
      run: npm ci

```

**using: composite** is the keyword.

```yaml

name: Deployment
on:
  push:
    branches:
      - main
jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - name: Get code
        uses: actions/checkout@v3
      - name: Get & Cache Dependencies
        uses: ./.github/actions/cache-deps
        with:
          caching: true
      - name: Lint code
        run: npm run lint
  test:
    runs-on: ubuntu-latest
    steps:
      - name: Get code
        uses: actions/checkout@v3
      - name: Get & Cache Dependencies
        uses: ./.github/actions/cache-deps
        with:
          caching: true
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
      - name: Get & Cache Dependencies
        uses: ./.github/actions/cache-deps
        with:
          caching: true
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
