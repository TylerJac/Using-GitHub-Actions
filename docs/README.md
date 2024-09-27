# Documentation of the CI/CD Workflow
- This GitHub Actions workflow automates the process of building, testing, and deploying your project to AWS S3. The workflow is triggered on any push to the main branch, following best practices of continuous integration and deployment (CI/CD). It ensures the application is properly built and tested before being deployed to an AWS S3 bucket.

## Workflow Breakdown:
### Trigger:
-The workflow runs whenever code is pushed to the main branch:

```
on:
push:
branches:
- main
```
- This ensures that each update to the main branch goes through the build, test, and deployment pipeline automatically.

## Build Job
The build job handles the following steps:

- Environment Setup: The job uses ubuntu-latest to run the tasks in a Linux environment.
- Code Checkout: It uses actions/checkout@v4 to fetch the latest code from the repository.
- Set Up Node.js: The workflow sets up Node.js using actions/setup-node@v4 and ensures that Node.js version 20 is used to maintain compatibility with the latest runtime.
- Install Dependencies: It installs the project dependencies using npm ci to ensure a clean, reproducible environment and runs the npm run build command to build the project.
- Run Tests: The npm test command runs any unit tests to ensure the application is functioning correctly.
- Generate Artifacts: If the build is successful, the job generates build artifacts, which can optionally be uploaded using actions/upload-artifact.
```
jobs:
build:
runs-on: ubuntu-latest
```

    steps:
      # Step 1: Checkout the code from the repository
      - name: Checkout code
        uses: actions/checkout@v4

      # Step 2: Set up Node.js environment
      - name: Set up Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'  # Ensures the latest Node.js version is used

      # Step 3: Install dependencies and build the project
      - name: Clean install dependencies and build
        run: |
          npm ci  # Clean install for reproducibility
          npm run build  # Builds the project

      # Step 4: Run tests
      - name: Run tests
        run: npm test  # Ensures the code passes all unit tests

      # Step 5: Upload build artifacts (optional)
      - name: Upload build artifacts
        uses: actions/upload-artifact@v3
        with:
          name: build
          path: build/
Deploy Job
The deploy job depends on the successful completion of the build job (needs: build). This ensures that deployment happens only if the build and tests pass.

Key steps include:

Checkout the Code: Ensures the latest version of the code is available before deploying.
Deploy to S3: Uses the AWS CLI to sync the contents of the dist directory (generated during the build process) to an S3 bucket. The AWS_ACCESS_KEY_ID and AWS_SECRET_ACCESS_KEY secrets are injected securely using GitHub Secrets.
yaml
Copy code
deploy:
runs-on: ubuntu-latest
needs: build

    env:
      FORCE_JAVASCRIPT_ACTIONS_TO_NODE20: true  # Ensures the Node.js runtime uses version 20

    steps:
      # Step 1: Checkout the code again
      - name: Checkout code
        uses: actions/checkout@v4

      # Step 2: Deploy to AWS S3
      - name: Deploy to S3
        env:
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        run: |
          aws s3 sync ./dist s3://coolbucketss --delete --region us-east-2
Key Aspects of the Workflow
Dependencies:

Node.js: Node.js is essential for building and testing JavaScript applications. The setup-node action ensures the latest stable version is used.
AWS CLI: The deployment process uses the AWS CLI to sync files to the S3 bucket. Make sure the IAM user associated with the AWS_ACCESS_KEY_ID and AWS_SECRET_ACCESS_KEY has the necessary S3 permissions (e.g., AmazonS3FullAccess).
Environment Variables and Secrets:

GitHub Secrets (AWS_ACCESS_KEY_ID and AWS_SECRET_ACCESS_KEY) securely store sensitive AWS credentials required for deployment.
FORCE_JAVASCRIPT_ACTIONS_TO_NODE20 ensures that the Node.js version for the runtime environment is forced to version 20, even if the action is using an older runtime.
