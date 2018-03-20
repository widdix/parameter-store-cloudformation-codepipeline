# Configure your CloudFormation managed infrastructure with Parameter Store and CodePipeline

Code for my blog post: [Configure your CloudFormation managed infrastructure with Parameter Store and CodePipeline](https://cloudonaut.io/configure-your-cloudformation-managed-infrastructure-with-parameter-store-and-codepipeline/)

## Setup instructions

> Have you [installed](https://docs.aws.amazon.com/cli/latest/userguide/installing.html) and [configured](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-getting-started.html) the AWS CLI?

1. Clone this repository `git clone https://github.com/widdix/parameter-store-cloudformation-codepipeline.git` download the ZIP file](https://github.com/widdix/parameter-store-cloudformation-codepipeline/archive/master.zip)
1. `cd parameter-store-cloudformation-codepipeline/`
1. Create parameter in Parameter Store: `aws ssm put-parameter --name '/application/stage/instancetype' --value 't2.micro' --type String`
1. Create pipeline stack with CloudFormation: `aws cloudformation create-stack --stack-name cloudonaut --template-body file://pipeline.yaml --capabilities CAPABILITY_IAM`
1. Wait until CloudFormation stack is created: `aws cloudformation wait stack-create-complete --stack-name cloudonaut`
1. Push files to the CodeCommit repository created by the pipeline stack (I don't use `git push` here to skip the git configuration):
    1. `COMMIT_ID="$(aws codecommit put-file --repository-name cloudonaut --branch-name master --file-content file://infrastructure.yaml --file-path infrastructure.yaml --query commitId --output text)"`
    1. `COMMIT_ID="$(aws codecommit put-file --repository-name cloudonaut --branch-name master --parent-commit-id $COMMIT_ID --file-content file://infrastructure.json --file-path infrastructure.json --query commitId --output text)"`
    1. `COMMIT_ID="$(aws codecommit put-file --repository-name cloudonaut --branch-name master --parent-commit-id $COMMIT_ID --file-content file://vpc-2azs.yaml --file-path vpc-2azs.yaml --query commitId --output text)"`
1. Wait until the first pipeline run is finished: `open 'https://console.aws.amazon.com/codepipeline/home#/view/cloudonaut'`
1. Visit the website exposed by the EC2 instance: `open "http://$(aws cloudformation describe-stacks --stack-name cloudonaut-infrastructure --query "Stacks[0].Outputs[0].OutputValue" --output text)"`
1. Update the parameter value: `aws ssm put-parameter --name '/application/stage/instancetype' --value 't2.nano' --type String --overwrite` (`t2.nano` is outside th Free Tier, expect charges of a few cents)
1. Wait until the second pipeline run is finished: `open 'https://console.aws.amazon.com/codepipeline/home#/view/cloudonaut'`
1. Visit the website exposed by the EC2 instance: `open "http://$(aws cloudformation describe-stacks --stack-name cloudonaut-infrastructure --query "Stacks[0].Outputs[0].OutputValue" --output text)"`

## Clean up instructions

1. Remove CloudFormation stacks
    1. `aws cloudformation delete-stack --stack-name cloudonaut-infrastructure`
    1. `aws cloudformation wait stack-delete-complete --stack-name cloudonaut-infrastructure`
    1. `aws cloudformation delete-stack --stack-name cloudonaut-vpc`
    1. `aws cloudformation wait stack-delete-complete --stack-name cloudonaut-vpc`
    1. `aws cloudformation delete-stack --stack-name cloudonaut`
    1. `aws cloudformation wait stack-delete-complete --stack-name cloudonaut`
1. Remove S3 bucket prefixed with `cloudonaut-artifactsbucket-` including all files: `open "https://s3.console.aws.amazon.com/s3/home"`
1. Remove Parameter Store parameter: `aws ssm delete-parameter --name '/application/stage/instancetype'`
