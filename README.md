# Amazon Pinpoint custom channel in journeys for email attachments

This is the solution utilizes Amazon Pinpoint custom channel to support email attachments for Pinpoint Journeys. The attached file is stored in an S3 bucket

## High Level Architecture

![architecture](https://github.com/Pioank/pinpoint-custom-channel-attachment/blob/main/assets/architecture.PNG)

![attachment-scenarios](https://github.com/Pioank/pinpoint-custom-channel-attachment/blob/main/assets/attachment-scenarios.PNG)

## Solution implementation

### Prerequisites

To deploy this solution, you must have the following:

- An [AWS account](https://signin.aws.amazon.com/signin?redirect_uri=https%3A%2F%2Fportal.aws.amazon.com%2Fbilling%2Fsignup%2Fresume&client_id=signup)
- The AWS CLI on your local machine. For more information see [Installing the AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html) guide.
- The SAM CLI on your local machine. For more information, see [Install the SAM CLI](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/serverless-sam-cli-install.html) in the AWS Serverless Application Model Developer guide.
- The latest version of Boto3. For more information see [Installation](https://boto3.amazonaws.com/v1/documentation/api/latest/guide/quickstart.html) in the Boto3 Documentation. 
- An S3 bucket
- A Pinpoint Project with the email channel enabled

### Deploy the solution

#### Step 1: Deploy the SAM application

Clone the repository to your local machine, navigate there and execute the command below on your SAM CLI:

```sh
sam deploy --stack-name contextual-targeting --guided
```

Fill the fields below as displayed. Change the **AWS Region** to the AWS region of your preference, where Amazon Pinpoint is available.

```sh
Configuring SAM deploy
======================

        Looking for config file [samconfig.toml] :  Found
        Reading default arguments  :  Success

        Setting default arguments for 'sam deploy'
        =========================================
        Stack Name [pinpoint-custom-channel-attachments]:
        AWS Region [us-east-1]:
        Parameter PinpointAppId []: pinpoint-project-id
        Parameter AttachmentsBucketName []: s3-bucket-name
        Parameter S3URLExpiration []:3600
        Parameter FileType []: .pdf
        #Shows you resources changes to be deployed and require a 'Y' to initiate deploy
        Confirm changes before deploy [Y/n]:
        #SAM needs permission to be able to create roles to connect to the resources in your template
        Allow SAM CLI IAM role creation [Y/n]:
        #Preserves the state of previously provisioned resources when an operation fails
        Disable rollback [Y/n]:
        Save arguments to configuration file [Y/n]:
        SAM configuration file [samconfig.toml]:
        SAM configuration environment [default]:

        Looking for resources needed for deployment:
         Managed S3 bucket: aws-sam-cli-managed-default-samclisourcebucket-17iz89qh57gk7
         A different default S3 bucket can be set in samconfig.toml

```

#### Step 2: Add AWS Lambda resource base policy

To allow Amazon Pinpoint to invoke the AWS Lambda function, you will need to add an AWS Lambda resource base policy.

From the command below, replace the placeholder values for:
- **function-name** with the AWS Lambda function name from the SAM deployment output
- **principal** with the AWS region that your AWS Lambda function has been deployed
- **source-arn** with the AWS region and AWS account id
- **source-account** with the AWS account id

For Windows PowerShell use the command below:
```sh
aws lambda add-permission \
--function-name myFunction \ 
--statement-id sid0 \
--action lambda:InvokeFunction \
--principal pinpoint.us-east-1.amazonaws.com \
--source-arn arn:aws:mobiletargeting:us-east-1:111122223333:apps/*
--source-account 111122223333
```

## Testing



## Cost

The cost displayed below is only for AWS Lambda and doesn't include the cost per email nor the Amazon Pinpoint's monthly targeted audience cost.

![solution-cost](https://github.com/Pioank/pinpoint-custom-channel-attachment/blob/main/assets/custom-channel-cost-usd.PNG)

## Next steps

Contextual targeting is not bound to a single channel, like in this solution email. You can extend the **batch-segmentation-postprocessing** workflow to fit your engagement and targeting requirements.

For example, you could implement several branches based on the referenced endpoint channel types and create Amazon Pinpoint customer segments that can be engaged via Push Notifications, SMS, Voice Outbound and In-App.

## Cleanup

To delete the solution, run the following command in the AWS CLI.

The command below is compatible with Linux:

```sh
sam delete
```

For Windows PowerShell use the command below:
```sh
sam delete
```

