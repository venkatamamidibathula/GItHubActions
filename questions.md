1. How do you ensure only master branch with secrets can be used during build phase?
2. What is the difference between npm ci and npm build?
3. Given a requirement , create a build pipeline for the repo where you create an artifact and use it another stage in workflow? (REPO URL: https://github.com/venkatamamidibathula/nodejs)
4. Outline the structure of github actions?
5. Create a build and deploy script for the nodejs project and in terms of issues output the information to console?
6. Given a requirement , create a pipeline where it uploads test report only if the npm test fails? (REPO URL: https://github.com/venkatamamidibathula/nodejs)
7. What is the difference between steps.id.outcome and steps.id.conclusion?
   steps.id.outcome yields false when continue_on_error is set to true for previous failed step whereas steps.id.conclusion yields true if continue_on_error is set to true in previous step.
8. What is a resuable workflow and how its called? Give me an example where you used resuable workflow?
   Reusable worklow can be synonymous to a function call in a program. we define a resuable workflow with a trigger workflow_call and define the steps and it can callled across multiple workflows whenever needed. I specifically use it for download artifact where i pass the artifact name as input value across deployment jobs.
9. What is reusable input and where is it passed?
   Reusable input is an input parameter which we use in workflow_call in resuable workflow.


