# WildRydes - Ride Sharing App

WildRydes is a fun, fictional ride-sharing application built using AWS services such as Cognito for authentication, Lambda for backend logic, DynamoDB for storage, and API Gateway for API management. This project demonstrates how to integrate these services for a seamless ride-request flow where users can register, log in, and request unicorn rides.

---

## Table of Contents
1. [Project Setup](#project-setup)
   - [Create and Clone GitHub Repo](#create-and-clone-github-repo)
   - [Connect Amplify for CI/CD](#connect-amplify-for-cicd)
2. [User Authentication](#user-authentication)
   - [Create Cognito User Pool](#create-cognito-user-pool)
   - [Update GitHub Repo with Cognito IDs](#update-github-repo-with-cognito-ids)
3. [Ride Request Functionality](#ride-request-functionality)
   - [Set Up DynamoDB Table](#set-up-dynamodb-table)
   - [Create Lambda Function](#create-lambda-function)
   -  [Create IAM Role](#create-iam-role)
   - [Test Lambda and DynamoDB Integration](#test-lambda-and-dynamodb-integration)
4. [API Gateway Integration](#api-gateway-integration)
   - [Create API Gateway](#create-api-gateway)
   - [Configure API Gateway Authorizer](#configure-api-gateway-authorizer)
5. [Deployment and Testing](#deployment-and-testing)
6. [Conclusion](#conclusion)

---

## Project Setup

### Create and Clone GitHub Repo

1. Create a new GitHub repository and clone the WildRydes site into your repo.
   ![Step 1](images/1.png)


  

### Connect Amplify for CI/CD

1. Connect the GitHub repository to AWS Amplify for continuous deployment and integration. Select the desired GitHub repository from the Amplify dashboard.
    ![Step 2](images/2.png)

2. If your repository does not appear, ensure your GitHub permissions are correctly updated.
   ![Step 3](images/3.png)
  
3. Click `Next` and review the configuration.
    ![Step 4](images/4.png)
   
4. Save and deploy the app.
   ![Step 5](images/5.png)
   
5. Once deployment is completed, click the domain URL to check the app.
   ![Step 6](images/6.png)

6. Now lets test the integration. Modify the index.html in the browser and commit.
Modifying this line 
```
<h2 class="section-title">How Does This Work?</h2>
 to 
<h2 class="section-title">How Does This Thing Work?</h2>
```
   ![Step 7](images/7.png)
   
7. This triggers the deployment in amplify
   
    ![Step 8](images/8.png)
   
8. Once the deployment completes, we can see the changes reflected same domain url application
    ![Step 9](images/9.png)
   
---

## User Authentication

### Create Cognito User Pool

 ![Step 10](images/10.png)

1. Go to AWS Cognito and create a new user pool.
   ![Step 11](images/11.png)

2. Select `Single-page application (SPA)` as the application type and name it `WildRydes`.
   ![Step 12](images/12.png)

3. Choose the required attributes for sign-up.
   ![Step 13](images/13.png)

4. User Pool is created
    ![Step 14](images/14.png)

6. After the user pool is created, click on it and copy the **User Pool ID**.
   ![Step 15](images/15.png)

7. Click on Set up your app : WildRydes and grab the client ID
   ![Step 16](images/16.png)

### Update GitHub Repo with Cognito IDs

1. Navigate to `wildrydes-site/js/config.js` in the GitHub repo and add the **User Pool ID** and **Client ID** you obtained from Cognito into the configuration.
   ![Step 17](images/17.png)

2. Commit the changes to trigger a new deployment in Amplify.
   ![Step 18](images/18.png)

3. We can click on the domain url and click on GIDDY UP!
   ![Step 19](images/19.png)
   
5. It navigates to the registration page
   ![Step 20](images/20.png)

6. Give the details and click on LET’S RYDE
    ![Step 21](images/21.png)
   
7. Verify the email address
   ![Step 22](images/22.png)

8. Login after verification
    ![Step 23](images/23.png)
   
9. The page is not the desired one. Just copy the token in if you'd like to test the Amazon Cognito user pool authorizer for your API, use the auth token below:
    
    ![Step 24](images/24.png)

---

## Ride Request Functionality

The next feature we’ll focus on is Ride Sharing, where users can request a unicorn, and one will be dispatched to their location.

To implement this, we will leverage AWS Lambda, which will be triggered whenever a user requests a unicorn ride. The user’s request will invoke a Lambda function that selects an available unicorn from the fleet and records the request in a DynamoDB table (a NoSQL database).

The process will look as follows:

User Request: When a user requests a unicorn, the Lambda function is triggered.
Unicorn Selection: The Lambda function selects the appropriate unicorn from the fleet based on availability and proximity.
Data Storage: The ride request details are then recorded in a DynamoDB table.
Response to Frontend: Finally, the Lambda function responds to the frontend with the details of the unicorn that has been dispatched, including the unicorn’s ID, location, and estimated arrival time.

### Set Up DynamoDB Table

1. Go to the DynamoDB console and create a new table.
   ![Step 25](images/25.png)

2. Set the table name (e.g., `Rides`) and use `RideId` as the Partition key. Leave other settings as default and create the table.
   ![Step 26](images/26.png)

3. Table will be created. 
   ![Step 27](images/27.png)

4. Once the table is created, click on it to open its details page. From there, copy the ARN, which can be found under the 'Additional Information' section.
   ![Step 28](images/28.png)

### Create IAM Role

Next, we need to create an execution role that allows the Lambda function to write to the DynamoDB table. Follow these steps:

1. Navigate to the IAM console and click on Create role.
![Step 29](images/29.png)

2. For the trusted entity, select AWS service, and under Use case, choose Lambda.
![Step 30](images/30.png)

3. In the next screen, select the AWSLambdaBasicExecutionRole policy, then click Next.
![Step 31](images/31.png)

4. Enter a name for the role, scroll down, and click Create role to finalize the creation.
![Step 32](images/32.png)

5. The role is now created. Click on the newly created role to open its details page.
![Step 33](images/33.png)

6. To add additional permissions, click on Add inline policy on the right side of the page.
![Step 34](images/34.png)

7. Select DynamoDB as the service, then filter the actions to include PutItem (for inserting items into the table). You’ll need to specify the table that the Lambda function will access. Use the ARN (Amazon Resource Name) that was copied earlier when you created the DynamoDB table. Click Add ARN to include the specific ARN for your table.
![Step 35](images/35.png)

8. Click Next, then give the policy a name (e.g., LambdaDynamoDBPolicy), and click Create policy.
![Step 36](images/36.png)

9. The role now has the necessary permissions to allow the Lambda function to write to the DynamoDB table.
![Step 37](images/37.png)


### Create Lambda Function

1. Create a new Lambda function from scratch.
   ![Step 18](images/18.png)

2. Select the runtime environment and set the execution role to the role you created earlier (e.g., `WildRydesLambda`).
   ![Step 19](images/19.png)

3. Paste the following code into the Lambda function:

```javascript
import { randomBytes } from 'crypto';
import { DynamoDBClient } from '@aws-sdk/client-dynamodb';
import { DynamoDBDocumentClient, PutCommand } from '@aws-sdk/lib-dynamodb';

const client = new DynamoDBClient({});
const ddb = DynamoDBDocumentClient.from(client);

const fleet = [
    { Name: 'Angel', Color: 'White', Gender: 'Female' },
    { Name: 'Gil', Color: 'White', Gender: 'Male' },
    { Name: 'Rocinante', Color: 'Yellow', Gender: 'Female' },
];

export const handler = async (event, context) => {
    if (!event.requestContext.authorizer) {
        return errorResponse('Authorization not configured', context.awsRequestId);
    }

    const rideId = toUrlString(randomBytes(16));
    console.log('Received event (', rideId, '): ', event);

    const username = event.requestContext.authorizer.claims['cognito:username'];
    const requestBody = JSON.parse(event.body);
    const pickupLocation = requestBody.PickupLocation;

    const unicorn = findUnicorn(pickupLocation);

    try {
        await recordRide(rideId, username, unicorn);
        return {
            statusCode: 201,
            body: JSON.stringify({
                RideId: rideId,
                Unicorn: unicorn,
                Eta: '30 seconds',
                Rider: username,
            }),
            headers: {
                'Access-Control-Allow-Origin': '*',
            },
        };
    } catch (err) {
        console.error(err);
        return errorResponse(err.message, context.awsRequestId);
    }
};

function findUnicorn(pickupLocation) {
    console.log('Finding unicorn for ', pickupLocation.Latitude, ', ', pickupLocation.Longitude);
    return fleet[Math.floor(Math.random() * fleet.length)];
}

async function recordRide(rideId, username, unicorn) {
    const params = {
        TableName: 'Rides',
        Item: {
            RideId: rideId,
            User: username,
            Unicorn: unicorn,
            RequestTime: new Date().toISOString(),
        },
    };
    await ddb.send(new PutCommand(params));
}

function toUrlString(buffer) {
    return buffer.toString('base64')
        .replace(/\+/g, '-')
        .replace(/\//g, '_')
        .replace(/=/g, '');
}

function errorResponse(errorMessage, awsRequestId) {
    return {
        statusCode: 500,
        body: JSON.stringify({
            Error: errorMessage,
            Reference: awsRequestId,
        }),
        headers: {
            'Access-Control-Allow-Origin': '*',
        },
    };
}
