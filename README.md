# Python CI Pipeline — AWS CodeCommit + CodeBuild + CodePipeline

## Project Structure

```
├── app.py            # Python application
├── test_app.py       # Unit tests
├── requirements.txt  # Python dependencies
├── buildspec.yml     # CodeBuild build instructions
├── pipeline.yml      # CloudFormation stack (full CI pipeline)
└── README.md
```

## CI Pipeline Flow

```
CodeCommit (push) → EventBridge → CodePipeline → CodeBuild (lint + test) → S3 Artifact
```

## Deploy the Pipeline

### Prerequisites
- AWS CLI configured (`aws configure`)
- IAM permissions for CloudFormation, CodeCommit, CodeBuild, CodePipeline, S3, IAM, Events

### Step 1 — Deploy the CloudFormation stack

```bash
aws cloudformation deploy \
  --template-file pipeline.yml \
  --stack-name python-ci-pipeline \
  --capabilities CAPABILITY_NAMED_IAM \
  --region us-east-1
```

### Step 2 — Push your code to CodeCommit

```bash
# Get the clone URL from stack outputs
aws cloudformation describe-stacks \
  --stack-name python-ci-pipeline \
  --query "Stacks[0].Outputs"

# Clone and push
git clone <RepoCloneUrl>
cd python-ci-app
cp ../app.py ../test_app.py ../requirements.txt ../buildspec.yml .
git add .
git commit -m "Initial commit"
git push origin main
```

### Step 3 — Watch the pipeline run

Open the `PipelineUrl` from the stack outputs in your browser, or run:

```bash
aws codepipeline get-pipeline-state --name python-ci-app-pipeline
```

## CodeBuild Phases

| Phase | What happens |
|------------|--------------------------------------|
| install | Installs pytest |
| pre_build | Runs flake8 linter on app.py |
| build | Runs pytest unit tests |
| post_build | Logs success message |

## Tear Down

```bash
aws cloudformation delete-stack --stack-name python-ci-pipeline
# Also empty and delete the S3 artifact bucket manually if needed
```
