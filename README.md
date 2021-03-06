# S3BucketBatch resource

This S3BucketBatch Lambda function (backing a CloudFormation custom resource) is (mostly) a working example of a custom resource implementation.
As focus was mainly on creating a functional working example, implementing best-practices on code level (e.g. error handling) are left as an excercise for the reader :wink:.

This setup serves as an example for:

- Lambda backed CloudFormation Custom Resource, implementing the `Create` and `Delete` events (implicitly `Update`!).
- Deployment of aforementioned Lambda function using the [AWS Serverless Application Model cli](https://github.com/awslabs/aws-sam-cli)
- Using exported values between CloudFormation stacks
- Python unit test for aforementioned Lambda function, including event mocks
- Python unit test with mocking of S3 actions (creating, deleting S3 buckets)

## Usage

Have a look at the [setup section](#usage) below. After configuring an S3 Bucket, deployment is done using a simple bash script, executing three [AWS SAM CLI](https://github.com/awslabs/aws-sam-cli) commands.

Once implemented, it provides a CloudFormation Custom Resource for creating `Count` number of S3 buckets. As an example you can use the CloudFormation template in this repo: [`ten_s3_buckets.yaml`](./ten_s3_buckets.yaml).
It basically comes down to:

```YAML
AWSTemplateFormatVersion: "2010-09-09"

Resources:
  S3BucketBatchResource:
    Type: Custom::S3BucketBatch
    Properties:
      ServiceToken: !ImportValue "s3bucketbatch-S3BucketBatchFunction"
      Count: "10"
      BucketName: "many-buckets-project"
```

The Lambda function backing the custom resource, `ARN`, needs to be specified by `ServiceToken`. The SAM deployment used in this repository, exposes the ARN of the Lambda function from it's own stack.

## AWS Serverless Application Model

The initial code sources and repository setup have been generated by the AWS Serverless Application Model (`sam init --runtime python3.6 --name s3bucketbatch`), thereby allowing deployment using the AWS-SAM Cli.

```bash
.
├── README.md                   <-- This file
├── s3bucketbatch               <-- Source code of the lambda function backing the custom resource
│   ├── __init__.py
│   ├── app.py                  <-- Lambda function code
│   └── requirements.txt        <-- Python dependencies
├── template.yaml               <-- SAM Template
├── ten_s3_buckets.yaml         <-- Example CloudFormation template
└── tests                       <-- Unit tests
    └── unit
        ├── __init__.py
        └── test_handler.py
```

## Requirements

- AWS CLI already configured with Administrator permission
- [AWS SAM CLI](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/serverless-sam-cli-install.html)
- An S3 bucket (used by AWS SAM to store the ZIP file for Lambda deployment)
- [Python 3.6 installed](https://www.python.org/downloads/), but only when using the unit tests (`pytest`)

(Python 3.6 has been configured as runtime in the [deploment template](./template.yaml))

## Setup

Firstly, we need an `S3 bucket` where AWS SAM can upload our Lambda functions packaged as ZIP before we deploy anything - If you don't have a S3 bucket to store code artifacts then this is a good time to create one:

```bash
aws s3 mb s3://BUCKET_NAME
```

Next, edit the [build_package_deploy.sh](./build_package_deploy.sh) to configure the respective bucket name:

```bash
# Configure your S3 bucket here:
S3_DEPLOYMENT_BUCKET=my-deployment-bucket-rand2ut79hlcns6qy
```

Perhaps it doesn't come as a surprise that `my-deployment-bucket-rand2ut79hlcns6qy` is already taken by someone :wink:.

Next, execute the [build_package_deploy.sh](./build_package_deploy.sh) script.

After deployment is complete you can run the following command to have a look at the `Outputs` and `Exports`:

```bash
aws cloudformation describe-stacks \
    --stack-name s3bucketbatch \
    --query 'Stacks[].Outputs'
```

To actually use the resource, you can use the CloudFormation template [`ten_s3_buckets.yaml`](./ten_s3_buckets.yaml):

```bash
aws cloudformation create-stack \
  --template-body "file://`pwd`/ten_s3_buckets.yaml" \
  --stack-name S3BucketBatch-Test
```

This will create ten buckets, by using the single resource. Removal, by removing the respective stack, is also supported! :sweat_smile:
