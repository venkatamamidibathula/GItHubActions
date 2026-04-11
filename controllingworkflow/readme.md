# Special Conditions
===========================
- failure() : If previous condition failed
- success() : If previous step is successful
- always(): Step always executes
- cancelled() : If a step is cancelled


```yaml

name: Workflow
on: [push,workflow_dispatch]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Run a one-line script
        run: echo Hello, world!
      - uses: actions/setup-node@v3
        with:
          node-version: '16'
      - name: Run a multi-line script
        id: build-and-test
        run: |
          npm ci
          npm run build
          npm test
      - name: Upload test reports
        if: failure() && steps.build-and-test.outcome == 'failure'
        uses: actions/upload-artifact@v3
        with:
          name: test-reports
  deploy:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Deploy to production
        run: echo Deploying to production...    

```
