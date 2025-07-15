# SNS Integration Guide

This guide explains how the Super Deals application integrates with AWS SNS for event-driven communication.

## Overview

The application uses AWS SNS (Simple Notification Service) to implement event-driven architecture for certain workflows, such as sending welcome emails after successful user sign-up verification.

## Implementation Details

### 1. SNS Topics

The following SNS topics are used:

| Topic Name | Purpose | CDK Construct |
|------------|---------|---------------|
| SignUpCompleted | Triggered when a user successfully verifies their email | `backend/lib/sns/accounts/sign-up-completed/construct.js` |

### 2. Publishing to SNS

The application publishes to SNS directly from the SvelteKit server-side code using the AWS SDK:

```javascript
// Example from accounts.svelte.js
import { publishToTopic } from '$lib/services/aws/sns.svelte.js';
import { AWS_CONFIG } from '$lib/config/aws.js';

// After successful verification
await publishToTopic(
  AWS_CONFIG.signUpCompletedTopicArn,
  {
    email,
    userType,
    verificationTime: new Date().toISOString()
  },
  `User verification completed: ${email}`
);
```

### 3. SNS Topic ARN Configuration

The SNS topic ARN is exported from the CDK stack using `CfnOutput`:

```javascript
// In backend/lib/sns/accounts/sign-up-completed/construct.js
new CfnOutput(this, "SignUpCompletedTopicArn", {
  value: this.topic.topicArn,
  description: "ARN of the SNS topic for sign-up completed events",
  exportName: "SignUpCompletedTopicArn"
});
```

This ARN is then imported into the SvelteKit application's environment variables using the `update-env.js` script.

### 4. SNS Message Consumers

The following services consume messages from the SNS topics:

| Service | Topic | Purpose |
|---------|-------|---------|
| SendWelcomeEmail | SignUpCompleted | Sends a welcome email to newly verified users |

### 5. IAM Permissions

The Lambda functions that run the SvelteKit server-side code need permissions to publish to SNS. These permissions are configured in the IAM role attached to the Lambda functions.

Required permissions:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "sns:Publish",
      "Resource": "arn:aws:sns:us-east-1:*:SignUpCompleted"
    }
  ]
}
```

## Adding New SNS Integrations

To add a new SNS integration:

1. Create a new SNS topic construct in the backend CDK code
2. Add a `CfnOutput` to export the topic ARN
3. Update the `update-env.js` script to include the new topic ARN
4. Add the topic ARN to the `AWS_CONFIG` object in `src/lib/config/aws.js`
5. Use the `publishToTopic` function to publish messages to the topic
6. Create consumers (Lambda functions) that subscribe to the topic
7. Update IAM permissions for the Lambda functions that need to publish to the topic

## Troubleshooting

Common issues and solutions:

1. **Missing SNS topic ARN**: Ensure the backend stack has been deployed and the `update-env.js` script has been run.
2. **Permission denied errors**: Check that the Lambda function has the necessary IAM permissions to publish to SNS.
3. **Message not received by consumers**: Check CloudWatch logs for both the publisher and consumer to identify any issues.
