# Amazon Pinpoint custom channel in journeys for email attachments

Amazon Pinpoint currently doesn't support attachments when sending emails via Campaigns or Journeys. Customers need to use the Amazon SES SendRawMessage API operation to include attachments, which lacks other features such as customer segmentation and scheduling. Another approach for attachments is to host the file in Amazon S3 and include a pre-signed URL to the emails send. That way you don't need to think about the file size or deliverability.

Email attachments are key for many customers / use cases, some of them are:
- Monthly bills (specific to the recipient)
- New terms & conditions (same for all)
- Contracts (specific to the recipient)
- Booking confirmation (specific to the recipient)
- e-Tickets (specific to the recipient)

## High Level Architecture

This is the solution utilizes Amazon Pinpoint custom channel (AWS Lambda function) to support email attachments for Pinpoint Journeys. The AWS Lambda function calls Pinpoint & SES API operation for sending emails. The attached file is stored in an S3 bucket and can be either attached to the email send or accessed via an Amazon S3 pre-signed URL.

![architecture](https://github.com/Pioank/pinpoint-custom-channel-attachment/blob/main/assets/architecture-n.PNG)

![attachment-scenarios](https://github.com/Pioank/pinpoint-custom-channel-attachment/blob/main/assets/attachment-scenarios.PNG)

**Note:** Amazon Pinpoint custom channel processes endpoints in batches of 50 per AWS Lambda invokation.

**Attahced files mechanism:** 
- **One file per recipient:** In this case each recipient (endpoint) should receive a different file e.g. monthly bill. To achieve this, the files stored in S3 should follow the following naming convention prefix_endpointid.file e.g. OctoberBill_111.pdf. You can specify the file prefix in the Amazon Pinpoint journeys custom channel **Custom Data** field. The AWS Lambda function will append the endpoint id and file type. To select this method, specify **ONEPER** in the Pinpoint journey custom data.
- **One file for the whole journey:** In this case the attachment file name should be the same as the Pinpoint journey custom data file prefix. To select this method, specify **ONEALL** in the Pinpoint journey custom data.

**Sending mechanism for file attachments:**
- **Attachment per recipient**: To attach and send a file that is specific to a recipient the solution calls the Amazon S3 GetObject API operation per recipient and then the Amazon SES SendRawEmail API operation to send the email with the attachment. To follow that approach specify **ONEPER** in the Pinpoint journey custom data.
- **Attachment per journey**: To attach the same file for all the recipients the solution calls once the Amazon S3 GetObject API operation, creates a list of all the recipients per AWS Lambda invokation (max 50) and then calls the Amazon SES SendRawEmail API operation to send the email with the attachment.To follow that approach specify **ONEALL** in the Pinpoint journey custom data.

## Considerations

1. **Events' attribution:** Engagement events from the emails sent won't be attributed automatically back to the journey. These events can still be accessed and analysed if streamed using Amazon Kinesis Firehose. To reconsile the events back to the journey, the solution includes the journey id as trace_id when sent via the SendMessage Pinpoint API operation and as a tag **JourneyId** when sent via SES SendRawEmail.
3. **No email personalization when attaching a file:** Emails with attached files sent via the SES SendRawEmail API operation won't support message helpers for personalisation. The message template is expected to not have any user or endpoint attributes.

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

Download the email HTML templates below and create them in Pinpoint:

FriendlySenderName,EmailTemplateName,example@example.com,YES,Oct-Statement,URL


## Cost

The cost displayed below is only for AWS Lambda and doesn't include the cost per email, Amazon S3 or the Amazon Pinpoint's monthly targeted audience cost.

![solution-cost](https://github.com/Pioank/pinpoint-custom-channel-attachment/blob/main/assets/custom-channel-cost-usd-nn.PNG)

## Next steps

Contextual

## Cleanup

To delete the solution, run the following command in the AWS CLI (compatible for both Linux & Windows PowerShell).

```sh
sam delete
```

