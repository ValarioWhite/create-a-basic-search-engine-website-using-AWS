# create-a-basic-search-engine-website-using-AWS

***Click Here for Video Demonstration - insert here***

## Lab Overview and Highlevel

In this lab we will create a Gaming Search Engine from scratch in the AWS console using services/components like DynamoDB, Amplify, and API gateway.

**VPC Architecture Design**

INSERT ARCHITECTURAL DESIGN

## Setup

### Step 1 - Create and Populate our DynamoDB table

**Create DynamoDB table**
1. Go to the DynamoDB console
2. Click "Create Table"

<img width="809" alt="image" src="https://github.com/ValarioWhite/create-a-basic-search-engine-website-using-AWS/assets/126350373/f1fc63d1-ab19-44a8-949b-1377d8984349">

3. Name the table "GamingSearchEngine" and name the partition key as "Name of Game". Keep everything else default
<img width="341" alt="image" src="https://github.com/ValarioWhite/create-a-basic-search-engine-website-using-AWS/assets/126350373/b7bd291f-a62b-4e48-8c04-d0da455cb4c7">

4. Click Create table at the bottom of the screen (may take a few minutes to create)

DONE

**Populate DynamoDB table**
1. Click the check box for the newly created database, click "Actions" dropdown menu, then click "Explore items"

<img width="815" alt="image" src="https://github.com/ValarioWhite/create-a-basic-search-engine-website-using-AWS/assets/126350373/8fb442b6-12d9-47b0-8a83-4fd05bfe51b9">

2. Click "Create item" button
3. Fill out the Attribute value as shown in the screenshot below. To add the Category and ESRB Rating Attribute, click "Add new attribute" and select "String"

<img width="563" alt="image" src="https://github.com/ValarioWhite/create-a-basic-search-engine-website-using-AWS/assets/126350373/116c7c4d-9c96-4a5a-bae2-12b496e0b37c">

4. Repeat step for all 5 games shown below -  

| Name of Game | Category | ESRB Rating |
|--------------|----------|------------|
| Grand Theft Auto V | Open World - Shooter | Mature - 17+ |
| Fortnite | Battle Royal Game | Teen - 13+ |
| God of War | Action-Adventure | Mature - 17+ |
| Call of Duty Warzone | First-Person Shooter | Mature - 17+ |
| Minecraft | Survival Game | Everyone - 10+ |

***Note: You can duplicate an item to help expedite the process - shown in screenshot below***
<img width="689" alt="image" src="https://github.com/ValarioWhite/create-a-basic-search-engine-website-using-AWS/assets/126350373/6c265b16-4e49-4030-a28c-b80690defebc">


5. Once complete, the items in your table should look like what's shown below
<img width="680" alt="image" src="https://github.com/ValarioWhite/create-a-basic-search-engine-website-using-AWS/assets/126350373/cf5cd878-85bb-497b-a08b-50cab8278bc9">

DONE 

### Step 2 - Create Lambda IAM Role 
Create the execution role that gives your function permission to access AWS resources.

To create an execution role

1. Open the roles page in the IAM console.
2. Choose Create role.
3. Create a role with the following properties.
    * Trusted entity – Lambda.
    * Role name – **lambda-apigateway-dynamodb-role**.
    * Permissions – Custom policy with permission to DynamoDB and CloudWatch Logs. This custom policy has the permissions that  the function needs to write data to DynamoDB and upload logs. 
    ```json
    {
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VisualEditor0",
            "Effect": "Allow",
            "Action": [
                "logs:CreateLogStream",
                "dynamodb:PutItem",
                "dynamodb:DeleteItem",
                "dynamodb:GetItem",
                "dynamodb:Scan",
                "dynamodb:Query",
                "dynamodb:UpdateItem",
                "logs:PutLogEvents",
                "logs:CreateLogGroup"
            ],
            "Resource": "*"
        }
    ]
    }
    ```


### Step 3 - Create Lambda Function

**To create the function**
1. Go to the Lambda console
2. Click "Create function" in AWS Lambda Console

![Create function](https://user-images.githubusercontent.com/126350373/221980515-da86c483-004e-449b-93f6-5f9388e959e8.png)


2. Select "Author from scratch". Use name **LambdaCRUDOverHTTPS** , select **Python 3.7** as Runtime. Under Permissions, select "Use an existing role", and select **lambda-apigateway-dynamodb-role** that we created, from the drop down

3. Click "Create function"

![Lambda basic information](https://user-images.githubusercontent.com/126350373/221984406-df5c15dc-1c36-4bb8-81e5-ea9986025a9d.png)

4. Replace the existing sample code with the following code snippet and click "Deploy"

**Python CRUD Operation Code**
```python
import boto3
import json

print('Loading function')

def lambda_handler(event,context):
    '''Provide an event that contains the following keys:

      - operation: one of the operations in the operations dict below
      - tableName: required for operations that interact with DynamoDB
      - payload: a parameter to pass to the operation being performed
    '''
    
    print("Received event: " + json.dumps(event, indent=1))
     
    operation = event['operation']
    

    if 'tableName' in event:
        dynamo = boto3.resource('dynamodb').Table(event['tableName'])

    operations = {
    #CRUD operations shown below:
        'create': lambda x: dynamo.put_item(**x),
        'read': lambda x: dynamo.get_item(**x),
        'update': lambda x: dynamo.update_item(**x),
        'delete': lambda x: dynamo.delete_item(**x),
        'echo': lambda x: x
    }

    if operation in operations:
        return operations[operation](event.get('payload'))
    else:
        raise ValueError('Unrecognized operation "{}"'.format(operation))
        
```
![Lambda Code](https://user-images.githubusercontent.com/126350373/221973523-c46347aa-2dfb-47d0-874e-a8c55d1c2456.png)

### Step 4 - Test Lambda Function

Let's test our newly created function. We will test our **"echo"** operation AND our **"create"** operation. **"echo"** is an operation that should output whatever the request/input/event submitted. **"create"** is an operation that will create a real record into the DynamoDB table (*apigateway-lambda-crud*) we created in the first step.  

**"echo" Operation TEST**
1. Click the arrow on the "Test" button and click "Configure test events"

![Configure test events](https://user-images.githubusercontent.com/126350373/221981236-6632840f-3f45-4373-9e2c-f1644699b67a.png)

2. Paste the following JSON into the event. The field "operation" dictates what the lambda function will perform. In this case, it'd simply return the payload from input event as output. Click "Create" to save.
```json
{
  "operation": "echo",
  "payload": {
    "testkey1": "testvalue1",
    "testkey2": "testvalue2"
  }
}
```
![Save test event](https://user-images.githubusercontent.com/126350373/221981831-b5ad9171-fb46-4747-a90e-a6ec0f5f168c.png)


3. Click "Test", and it will execute the test event. You should see the output in the console

![Successful Test](https://user-images.githubusercontent.com/126350373/221988503-418d8ea0-a8b1-452c-badb-379bf3456d3c.png)


### Step 4 - Create NAT Gateway (NATGW)

**For high availability we will create 2 NATGW. One in each AZ (One in Public Subnet A and One in Public Subnet B)**
1. Open the VPC Console
2. Click on "NAT gateways" on the left-hand panel 
3. Click "Create NAT gateway" button in the top right hand corner
4. Name the 1st NATGW "NATGW A", Select "Public Subnet A" that we created in a previous step, click "Allocate Elastic IP" button, and click "Create NAT gateway" button in the bottom right hand corner

![image](https://user-images.githubusercontent.com/126350373/228589852-71cb71e1-3edb-4149-af80-899e10deed72.png)

5. Go back to the NATGW dashboard and click "Create NAT gateway" button in the top right hand corner
6. Name the 2nd NATGW "NATGW B", Select "Public Subnet B" that we created in a previous step, click "Allocate Elastic IP" button, and click "Create NAT gateway" button in the bottom right hand corner
DONE

![image](https://user-images.githubusercontent.com/126350373/228591148-56c2e540-32b6-42cf-93c3-b36fc178eb65.png)

**Connect Route Tables to NATGW**
1. Open the VPC Console
2. Click on "Route tables" on the left-hand panel 
3. Select "Private Route Table A"
4. Click the "Actions" dropdown button at the top right hand corner and click "Edit routes"
5. Click "Add route", for Destination route select "0.0.0.0/0", for Target select "NAT Gateway" and select "NATGW A", click "Save changes" button at the bottom right hand corner.

![image](https://user-images.githubusercontent.com/126350373/228594577-b4956203-547e-4761-8d27-cd6ad636d462.png)

6. Go back to the Route tables dashboard
7. Select "Private Route Table B"
8. Click "Actions" dropdown button at the top right hand corner and click "Edit routes"
9. Click "Add route", for Destination route select "0.0.0.0/0", for Target select "NAT Gateway" and select "NATGW B", click "Save changes" button at the bottom right hand corner.
DONE

![image](https://user-images.githubusercontent.com/126350373/228595882-30cdb64f-ee69-427b-beb9-b2c67e6c7df9.png)

### NOTE: The default NACL allows all inbound and all outbound. We will not change the default NACL settings in this lab. Found in VPC console.

**NACL Inbound Rules**

![image](https://user-images.githubusercontent.com/126350373/228600720-8b890a6d-c46c-4625-967f-b6a6d439e411.png)

**NACL Outbound Rules**

![image](https://user-images.githubusercontent.com/126350373/228600965-4d9aef6e-e43e-409b-bbdc-e8f98bb7e3ce.png)



### Step 5 - Create API and Deploy API

**To create the API**
1. Go to API Gateway console
2. Scroll down to REST API and click "Build"

![Build API](https://user-images.githubusercontent.com/126350373/221991046-d4a3f54c-5f82-48c9-9941-4cf100559302.png)

3. Make sure to select "New API" and Give the API name as **DynamoOperations**, keep everything as is, click "Create API"

![Create REST API](https://user-images.githubusercontent.com/126350373/221992297-49184253-380d-4ee7-afb3-cec597b67dd7.png)

4. Each API is collection of resources and methods that are integrated with backend HTTP endpoints, Lambda functions, or other AWS services. Typically, API resources are organized in a resource tree according to the application logic. At this time you only have the root resource, but let's add a resource next 

Click "Actions", then click "Create Resource"

![Create API resource](https://user-images.githubusercontent.com/126350373/222020359-db68ce6e-65ca-4eee-ae8b-b6d65351ee21.png)

5. Input **DynamoOperations** in the Resource Name, Resource Path will get populated. Click "Create Resource"

![Create resource](https://user-images.githubusercontent.com/126350373/221992928-d7646703-bda8-477e-bb92-a71a9834db7d.png)

6. Let's create a POST Method for our API. With the "/dynamoooperations" resource selected, Click "Actions" again and click "Create Method". 

![Create resource method](https://user-images.githubusercontent.com/126350373/221993094-1fa244ec-91c9-4c74-a308-d0c927604681.png)

7. Select "POST" from drop down , then click checkmark

![Create resource method](https://user-images.githubusercontent.com/126350373/221993423-c955c2e1-7f6c-40ce-82aa-7062359c5679.png)

8. The integration will come up automatically with "Lambda Function" option selected. Select **LambdaCRUDOverHTTPS** function that we created earlier. As you start typing the name, your function name will show up. Select and click "Save". A popup window will come up to add resource policy to the lambda to be invoked by this API. Click "Ok"

![Create lambda integration](https://user-images.githubusercontent.com/126350373/221993769-f5291680-0647-492b-883b-0fdbe2a13891.png)

Our API-Lambda integration is done!

**Deploy the API**

In this step, you deploy the API that you created to a stage called prod.

1. Click "Actions", select "Deploy API"

![Deploy API](https://user-images.githubusercontent.com/126350373/222020622-f3317982-5d0a-492b-a418-235350ee05d4.png)

2. Now it is going to ask you about a stage. Select "[New Stage]" for "Deployment stage". Give "Prod" as "Stage name". Click "Deploy"

![Deploy API to Prod Stage](https://user-images.githubusercontent.com/126350373/221994238-db3d8c5a-8102-4a9d-bb6d-34a6e6cca617.png)

3. We're all set to run our solution! To invoke our API endpoint, we need the endpoint url. In the "Stages" screen, expand the stage "Prod", select "POST" method, and copy the "Invoke URL" from screen (we are going to need it in Step 7)

![Copy Invoke URL](https://user-images.githubusercontent.com/126350373/222017833-9a7e2780-dd44-4ce9-afa2-17c6bc5b2fdd.png)

### Step 7 - Update the HTML Code and Save It

1. Copy the HTML code shown below, 
2. Paste it to Notepad (for windows), TextEdit (for apple), or a source code editor like Visual Studio Code (download the software from a Google Search), and 
3. Udate the "INVOKE URL" in the code with your REST API Invoke URL from API Gateway in the previous step.
4. Save the file as "index"

```html
<!DOCTYPE html>
<html>
<head>
  <title>Gaming Search Engine</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      background-color: #f2f2f2;
      margin: 0;
      padding: 20px;
    }

    #container {
      max-width: 600px;
      margin: 0 auto;
      background-color: #fff;
      padding: 20px;
      box-shadow: 0 2px 5px rgba(0, 0, 0, 0.1);
    }

    h1 {
      font-size: 32px;
      text-align: center;
      margin: 0;
    }

    form {
      text-align: center;
      margin-top: 40px;
    }

    input[type="text"] {
      width: 400px;
      height: 36px;
      padding: 8px;
      font-size: 16px;
      border: 1px solid #ccc;
      border-radius: 4px;
    }

    input[type="submit"] {
      background-color: #FFA500;
      border: none;
      padding: 8px 16px;
      font-size: 16px;
      cursor: pointer;
    }

    pre {
      margin-top: 40px;
      background-color: #f2f2f2;
      padding: 20px;
      white-space: pre-wrap;
      word-wrap: break-word;
    }
  </style>
</head>
<body>
  <div id="container">
    <h1>Gaming Search Engine</h1>
    <form id="apiForm" action="INVOKE URL" method="POST">
      <input type="text" id="nameInput" name="Name of Game" value="GameName">
      <br><br>
      <input type="submit" value="Submit">
    </form>
    <pre id="response"></pre>
  </div>

  <script>
    const form = document.getElementById('apiForm');
    const responsePre = document.getElementById('response');

    form.addEventListener('submit', function(event) {
      event.preventDefault();
      const formData = new FormData(form);

      fetch(form.action, {
        method: form.method,
        body: JSON.stringify(Object.fromEntries(formData)),
        headers: {
          'Content-Type': 'application/json'
        }
      })
      .then(response => response.json())
      .then(data => {
        responsePre.textContent = JSON.stringify(data, null, 2);
      })
      .catch(error => {
        responsePre.textContent = 'An error occurred: ' + error.message;
      });
    });
  </script>
</body>
</html>

```



### Step 8 - Create Website using Amplify

1. Insert Step here

### Step 9 - Test the Webpage

1. Insert step here


## Clean Up

### Step 8 - Delete the Web Hosting App - Amplify Console

<img width="844" alt="image" src="https://github.com/ValarioWhite/create-a-basic-search-engine-website-using-AWS/assets/126350373/46392828-0693-4d31-aadf-88e8343a3add">

### Step 9 - Delete the REST API - API Gateway Console

<img width="845" alt="image" src="https://github.com/ValarioWhite/create-a-basic-search-engine-website-using-AWS/assets/126350373/2852dd5f-e476-4a93-8dfb-d6343e0a6f11">

### Step 10 - Delete the Lambda Function - Lambda Console

<img width="814" alt="image" src="https://github.com/ValarioWhite/create-a-basic-search-engine-website-using-AWS/assets/126350373/239c3afe-8520-4556-9a70-204ccd4af7c0">

### Step 12 - Delete the Database - DynamoDB Console

<img width="817" alt="image" src="https://github.com/ValarioWhite/create-a-basic-search-engine-website-using-AWS/assets/126350373/a749e627-8981-4862-a898-bf47a2d4b439">
