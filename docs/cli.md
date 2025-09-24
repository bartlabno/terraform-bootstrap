## Deploying with the AWS CLI

This guide provides instructions on how to deploy and manage the `terraform-bootstrap.yaml` CloudFormation stack using the AWS Command Line Interface (CLI). This method is ideal for scripting, automation, and for users who prefer the command line over the AWS Management Console.

### Prerequisites

- AWS CLI Installed and Configured: You must have the AWS CLI installed and configured with credentials that have sufficient permissions to create the resources in the template. You can test your configuration by running aws sts get-caller-identity.

- `terraform-bootstrap.yaml` File: You need the CloudFormation template file saved locally on the machine where you will be running the CLI commands.

### Deployment Command

The `aws cloudformation deploy` command is the recommended way to deploy stacks. It simplifies the process by creating a change set and then executing it, which allows you to see the proposed changes before they are applied. It can be used for both initial creation and subsequent updates.

#### Step 1: Prepare Your Parameters

Before running the command, gather the values for the required parameters. You will need:

- A globally unique name for your Terraform state S3 bucket.

- A globally unique name for your CloudTrail log S3 bucket.

- The comma-separated ARNs of the IAM roles you wish to grant access to.

#### Step 2: Run the Deployment Command

Open your terminal, navigate to the directory containing the `terraform-bootstrap.yaml` file, and execute the following command. Be sure to replace the placeholder values with your actual values.

##### Basic Deployment (Notifications Disabled):

```
aws cloudformation deploy \
  --template-file `terraform-bootstrap.yaml` \
  --stack-name "terraform-state-backend-stack" \
  --parameter-overrides \
    S3BucketName="your-unique-terraform-state-bucket-name" \
    CloudTrailLogBucketName="your-unique-cloudtrail-log-bucket-name" \
    AllowedAdminRoleArns="arn:aws:iam::123456789012:role/AdminRole,arn:aws:iam::123456789012:role/AnotherRole" \
  --capabilities CAPABILITY_IAM CAPABILITY_NAMED_IAM
```

#### Deployment with Notifications Enabled:

If you want to enable the optional SNS notifications, add the EnableNotifications and NotificationSnsTopicArn parameters.

```
aws cloudformation deploy \
  --template-file `terraform-bootstrap.yaml` \
  --stack-name "terraform-state-backend-stack" \
  --parameter-overrides \
    S3BucketName="your-unique-terraform-state-bucket-name" \
    CloudTrailLogBucketName="your-unique-cloudtrail-log-bucket-name" \
    AllowedAdminRoleArns="arn:aws:iam::123456789012:role/AdminRole" \
    EnableNotifications="true" \
    NotificationSnsTopicArn="arn:aws:sns:us-east-1:123456789012:MyAlarmsTopic" \
  --capabilities CAPABILITY_IAM CAPABILITY_NAMED_IAM
```

Command Breakdown:

- `--template-file`: The local path to your CloudFormation template.

- `--stack-name:` A unique name for your CloudFormation stack.

- `--parameter-overrides`: A space-separated list of Key=Value pairs for the parameters defined in the template.

- `--capabilities`: Acknowledges that the stack will create IAM resources (a requirement for any stack that creates IAM roles or policies).

#### Step 3: Monitor the Deployment

The AWS CLI will show the progress of the stack creation events in your terminal. You can also monitor the status in the AWS CloudFormation console.

### Updating the Stack

To update the stack (for example, to change the log retention period or add a new admin role), you can run the exact same `aws cloudformation deploy` command with the new parameter values. CloudFormation will automatically detect the changes and create a change set to update only the necessary resources.

### Retrieving Stack Outputs

After a successful deployment, you can retrieve the values from the `Outputs` section of the stack (like the KMS Key ARN and S3 Bucket Name) using the following command:

```
aws cloudformation describe-stacks \
  --stack-name "terraform-state-backend-stack" \
  --query "Stacks[0].Outputs"
```

This will return a JSON object containing the outputs, which you can then use to configure your Terraform backend.