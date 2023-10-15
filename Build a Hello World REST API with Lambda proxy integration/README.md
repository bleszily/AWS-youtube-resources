Lambda proxy integration is a lightweight, flexible API Gateway API integration type that allows you to integrate an API method – or an entire API – with a Lambda function. The Lambda function can be written in any language that Lambda supports. Because it's a proxy integration, you can change the Lambda function implementation at any time without needing to redeploy your API.

In this tutorial, you do the following:

    Create a "Hello, World!" Lambda function to be the backend for the API.

    Create and test a "Hello, World!" API with Lambda proxy integration.

Spet 1: Create a "Hello, World!" Lambda function

This function returns a greeting to the caller as a JSON object in the following format:

{
    "greeting": "Good {time}, {name} of {city}.[ Happy {day}!]"
}

Create a "Hello, World!" Lambda function in the Lambda console

    Sign in to the Lambda console at https://console.aws.amazon.com/lambda


On the AWS navigation bar, choose a region (for example, US East (N. Virginia)).
Note

Note the region where you create the Lambda function. You'll need it when you create the API.

Choose Functions in the navigation pane.

Choose Create function.

Choose Author from scratch.

Under Basic information, do the following:

    In Function name, enter GetStartedLambdaProxyIntegration.

    Under Permissions, expand Choose or create an execution role. From the Execution role dropdown list, choose Create new role from AWS policy templates.

    In Role name, enter GetStartedLambdaBasicExecutionRole.

    Leave the Policy templates field blank.

    Choose Create function.

Under Function code, in the inline code editor, copy/paste the following code:

Choose Deploy.


Sept 2: Create a "Hello, World!" API

Now create an API for your "Hello, World!" Lambda function by using the API Gateway console.
Build a "Hello, World!" API

    Sign in to the API Gateway console at https://console.aws.amazon.com/apigateway


If this is your first time using API Gateway, you see a page that introduces you to the features of the service. Under REST API, choose Build. When the Create Example API popup appears, choose OK.

If this is not your first time using API Gateway, choose Create API. Under REST API, choose Build.

Create an empty API as follows:

    Under Create new API, choose New API.

    Under Settings:

        For API name, enter LambdaSimpleProxy.

        If desired, enter a description in the Description field; otherwise, leave it empty.

        Leave Endpoint Type set to Regional.

    Choose Create API.

Create the helloworld resource as follows:

    Choose the root resource (/) in the Resources tree.

    Choose Create Resource from the Actions dropdown menu.

    Leave Configure as proxy resource unchecked.

    For Resource Name, enter helloworld.

    Leave Resource Path set to /helloworld.

    Leave Enable API Gateway CORS unchecked.

    Choose Create Resource.

In a proxy integration, the entire request is sent to the backend Lambda function as-is, via a catch-all ANY method that represents any HTTP method. The actual HTTP method is specified by the client at run time. The ANY method allows you to use a single API method setup for all of the supported HTTP methods: DELETE, GET, HEAD, OPTIONS, PATCH, POST, and PUT.

To set up the ANY method, do the following:

    In the Resources list, choose /helloworld.

    In the Actions menu, choose Create method.

    Choose ANY from the dropdown menu, and choose the checkmark icon

    Leave the Integration type set to Lambda Function.

    Choose Use Lambda Proxy integration.

    From the Lambda Region dropdown menu, choose the region where you created the GetStartedLambdaProxyIntegration Lambda function.

    In the Lambda Function field, type any character and choose GetStartedLambdaProxyIntegration from the dropdown menu.

    Leave Use Default Timeout checked.

    Choose Save.

    Choose OK when prompted with Add Permission to Lambda Function.

Step 3: Deploy and test the API
Deploy the API in the API Gateway console

    Choose Deploy API from the Actions dropdown menu.

    For Deployment stage, choose [new stage].

    For Stage name, enter test.

    If desired, enter a Stage description.

    If desired, enter a Deployment description.

    Choose Deploy.

    Note the API's Invoke URL.

Use browser and cURL to test an API with Lambda proxy integration

You can use a browser or cURL to test your API.

To test GET requests using only query string parameters, you can type the URL for the API's helloworld resource into a browser address bar. For example: https://r275xc9bmd.execute-api.us-east-1.amazonaws.com/test/helloworld?name=John&city=Seattle

For other methods, you must use more advanced REST API testing utilities, such as POSTMAN
or cURL

This tutorial uses cURL. The cURL command examples below assume that cURL is installed on your computer.
To test the deployed API using cURL on Mac or linux:

    Open a terminal window.

    Copy the following cURL command and paste it into the terminal window, replacing r275xc9bmd with your API's API ID and us-east-1 with the region where your API is deployed.

curl -v -X POST \
  'https://r275xc9bmd.execute-api.us-east-1.amazonaws.com/test/helloworld?name=John&city=Seattle' \
  -H 'content-type: application/json' \
  -H 'day: Thursday' \
  -d '{ "time": "evening" }'

To test the deployed API using cURL on Windows:
curl -v -X POST "https://r275xc9bmd.execute-api.us-east-1.amazonaws.com/test/helloworld?name=John&city=Seattle" -H "content-type: application/json" -H "day: Thursday" -d "{ \"time\": \"evening\" }"