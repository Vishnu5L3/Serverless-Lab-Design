Lets start with high level design
![high-level-design](https://github.com/Vishnu5L3/Serverless-Lab-Design/assets/142379885/d4c9a960-7cca-4727-8dd1-b3b056db6b01)
In this Project, we will be setting up an Amazon API Gateway, which is essentially a collection of resources and methods that allow you to create a RESTful API. Our specific use case involves creating a resource called "DynamoDBManager" and defining a single method, which is the HTTP POST method. When you make a POST request to this API over HTTPS, Amazon API Gateway will invoke a Lambda function called "LambdaFunctionOverHttps."

The primary purpose of this API is to support various DynamoDB operations, including:

Create, update, and delete an item in DynamoDB.
Read an item from DynamoDB.
Scan DynamoDB to retrieve multiple items.
Additionally, we have some other operations (echo, ping) that are not directly related to DynamoDB. These additional operations can be useful for testing purposes.
When you make a POST request to this API, you need to provide a request payload in the form of JSON data. This payload should specify the DynamoDB operation you want to perform and include any necessary data related to that operation.

For example, here's a sample request payload for a DynamoDB "create item" operation:
```
{
    "operation": "create",
    "tableName": "lambda-apigateway",
    "payload": {
        "Item": {
            "id": "1",
            "name": "Bob"
        }
    }
}
```

The following is a sample request payload for a DynamoDB read item operation:
```
{
    "operation": "read",
    "tableName": "lambda-apigateway",
    "payload": {
        "Key": {
            "id": "1"
        }
    }
}
```

SETUP:
CREATING A LAMDA IAM ROLE:

To create an execution role for your Lambda function with permissions to access AWS resources such as DynamoDB and CloudWatch Logs, follow these steps:

Open the AWS IAM (Identity and Access Management) console.

In the IAM console, navigate to the "Roles" page. You can do this by clicking on "Roles" in the left-hand navigation pane.

Click the "Create role" button to start creating a new IAM role.

In the "Select type of trusted entity" section, choose "Lambda" since you want to create a role for your Lambda function.

In the "Permissions" section, you will create a custom policy. Click on the "Attach policies directly" button to attach permissions to the role. Here, you will create a custom policy that includes permissions for DynamoDB and CloudWatch Logs.

Click on "Create policy." This will open a new tab where you can create a custom policy.

In the "Create policy" page, choose the JSON tab to enter the policy document. You can use a policy like the following as a starting point. Be sure to customize the policy based on your specific needs and the AWS resources you want to access:
```
{
"Version": "2012-10-17",
"Statement": [
{
  "Sid": "Stmt1428341300017",
  "Action": [
    "dynamodb:DeleteItem",
    "dynamodb:GetItem",
    "dynamodb:PutItem",
    "dynamodb:Query",
    "dynamodb:Scan",
    "dynamodb:UpdateItem"
  ],
  "Effect": "Allow",
  "Resource": "*"
},
{
  "Sid": "",
  "Resource": "*",
  "Action": [
    "logs:CreateLogGroup",
    "logs:CreateLogStream",
    "logs:PutLogEvents"
  ],
  "Effect": "Allow"
}
]
}
```
*CREATING A LAMBDA FUNCTION*
To create the function

Click "Create function" in AWS Lambda Console

Select "Author from scratch". Use name LambdaFunctionOverHttps , select Python 3.7 as Runtime. Under Permissions, select "Use an existing role", and select lambda-apigateway-role that we created, from the drop down

Click "Create function"



<img width="1440" alt="Screenshot 2023-10-31 at 1 21 25 PM 6 41 03 PM" src="https://github.com/Vishnu5L3/Serverless-Lab-Design/assets/142379885/38e91abf-8cbf-4890-adf6-797b9247714b">
Replace the boilerplate coding with the following code snippet and click "Save"

*EXAMPLE PYTHON CODE*
```
from __future__ import print_function

import boto3
import json

print('Loading function')


def lambda_handler(event, context):
    '''Provide an event that contains the following keys:

      - operation: one of the operations in the operations dict below
      - tableName: required for operations that interact with DynamoDB
      - payload: a parameter to pass to the operation being performed
    '''
    #print("Received event: " + json.dumps(event, indent=2))

    operation = event['operation']

    if 'tableName' in event:
        dynamo = boto3.resource('dynamodb').Table(event['tableName'])

    operations = {
        'create': lambda x: dynamo.put_item(**x),
        'read': lambda x: dynamo.get_item(**x),
        'update': lambda x: dynamo.update_item(**x),
        'delete': lambda x: dynamo.delete_item(**x),
        'list': lambda x: dynamo.scan(**x),
        'echo': lambda x: x,
        'ping': lambda x: 'pong'
    }

    if operation in operations:
        return operations[operation](event.get('payload'))
    else:
        raise ValueError('Unrecognized operation "{}"'.format(operation))
```
<img width="1440" alt="Screenshot 2023-10-31 at 1 27 04 PM 6 41 02 PM" src="https://github.com/Vishnu5L3/Serverless-Lab-Design/assets/142379885/a030bd88-f974-4dc7-890c-6cf16d4953f5">

*TEST LAMBDA FUNCTION*

To configure a test event for your Lambda function on GitHub, follow these steps:

Open the AWS Lambda console.

Locate your Lambda function, "LambdaFunctionOverHttps," and click on its name.

In the function details page, click the "Test" button.

In the "Configure test event" dialog, click on "Create a new test event."

Give your test event a name, such as "SampleEchoTestEvent."

In the "Event template" section, define the event JSON payload you want to test your Lambda function with. For a sample echo operation, you can use this JSON payload:
```
{
    "operation": "echo",
    "payload": {
        "somekey1": "somevalue1",
        "somekey2": "somevalue2"
    }
}
```
Click "Test", and it will execute the test event. You should see the output in the console.

<img width="1440" alt="Screenshot 2023-10-31 at 1 38 39 PM 6 41 02 PM" src="https://github.com/Vishnu5L3/Serverless-Lab-Design/assets/142379885/893fe8ff-fa96-42a1-b587-b707e684c498">

To create the DynamoDB table that your Lambda function will utilize, follow these steps:

Access the DynamoDB console.

Click on the "Create table" option.

Set up a table with the specified configurations:

Table name: lambda-apigateway
Primary key: id (string)
Proceed by clicking the "Create" button.

<img width="1440" alt="Screenshot 2023-10-31 at 2 07 16 PM 6 41 02 PM" src="https://github.com/Vishnu5L3/Serverless-Lab-Design/assets/142379885/6869256b-5db5-4c35-932e-50714fd45586">

<img width="1430" alt="Screenshot 2023-10-31 at 2 21 47 PM 6 41 02 PM" src="https://github.com/Vishnu5L3/Serverless-Lab-Design/assets/142379885/2a18f569-a9a1-4ba2-be04-38cdde27757a">

<img width="1439" alt="Screenshot 2023-10-31 at 2 22 17 PM 6 41 02 PM" src="https://github.com/Vishnu5L3/Serverless-Lab-Design/assets/142379885/47eeca2c-e825-4e32-81c6-2cd37f77357b">

*Deploy the API*
To deploy the API you've created to a stage named "prod," follow these steps:

Click "Actions" in the API Gateway console.

Select "Deploy API."

Now it is going to ask you about a stage. Select "[New Stage]" for "Deployment stage". Give "Prod" as "Stage name". Click "Deploy

To obtain the endpoint URL for invoking your API, follow these steps:

Go to the "Stages" screen in the API Gateway console.

Expand the "Prod" stage.

Select the "POST" method.

Copy the "Invoke URL" from the screen.

You now have the endpoint URL that you can use to make requests to your API.

*Running our solution*

To perform the "create" operation using the Lambda function and request this operation, use the following JSON format:

```
{
    "operation": "create",
    "tableName": "lambda-apigateway",
    "payload": {
        "Item": {
            "id": "1234ABCD",
            "number": 5
        }
    }
}
```
To run this API from Postman, follow these steps:

Select "POST" as the HTTP method.

Paste the API invoke URL into the URL field.

Under the "Body" section, select "raw."

Paste the provided JSON payload.

Click "Send."

The API should execute, and you should receive an "HTTPStatusCode" of 200 as the response.

<img width="1440" alt="Screenshot 2023-10-31 at 4 00 54 PM 6 41 02 PM" src="https://github.com/Vishnu5L3/Serverless-Lab-Design/assets/142379885/9fb18677-7dd7-49fd-bc9b-c4a6ce4f9df7">

To run this from terminal using Curl, run the below

```
$ curl -X POST -d "{\"operation\":\"create\",\"tableName\":\"lambda-apigateway\",\"payload\":{\"Item\":{\"id\":\"1\",\"name\":\"Bob\"}}}" https://$API.execute-api.$REGION.amazonaws.com/prod/DynamoDBManager
```
To validate that the item has been successfully inserted into the DynamoDB table, follow these steps:

Access the DynamoDB console.

Select the "lambda-apigateway" table.

Navigate to the "Items" tab.

You should see the newly inserted item displayed in the table.

<img width="1440" alt="Screenshot 2023-10-31 at 4 03 58 PM 6 41 02 PM" src="https://github.com/Vishnu5L3/Serverless-Lab-Design/assets/142379885/88048c99-34e2-412a-b96a-bdc7dbef0d24">

To retrieve all the inserted items from the table using the "list" operation of Lambda through the API, you can pass the following JSON payload:

```
{
    "operation": "list",
    "tableName": "lambda-apigateway",
    "payload": {
    }
}
```
<img width="1440" alt="Screenshot 2023-10-31 at 4 06 32 PM 6 41 02 PM" src="https://github.com/Vishnu5L3/Serverless-Lab-Design/assets/142379885/ffee9f16-f932-4ce7-a387-e758a317330e">

We have successfully created a serverless API using API Gateway, Lambda, and DynamoDB!

*Cleanup*

Let's clean up the resources we have created for this lab.

*Cleaning up DynamoDB*

To delete the table, from DynamoDB console, select the table "lambda-apigateway", and click "Delete table

To delete the Lambda, from the Lambda console, select lambda "LambdaFunctionOverHttps", click "Actions", then click Delete

To delete the API we created, in API gateway console, under APIs, select "DynamoDBOperations" API, click "Actions", then "Delete"

<img width="1440" alt="Screenshot 2023-10-31 at 4 11 56 PM 6 41 02 PM" src="https://github.com/Vishnu5L3/Serverless-Lab-Design/assets/142379885/78ea0f55-ae4b-48f8-a1fd-c222b9ca49e1">

<img width="1432" alt="Screenshot 2023-10-31 at 4 14 54 PM 6 41 02 PM" src="https://github.com/Vishnu5L3/Serverless-Lab-Design/assets/142379885/1c1e63f6-f39d-4c8d-a121-b38c5f04be22">






















































# Serverless-Lab-Design
