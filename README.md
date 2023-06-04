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

DONE

### Step 3 - Create Lambda Function

**To create the function**
1. Go to the Lambda console
2. Click "Create function" in AWS Lambda Console

<img width="941" alt="image" src="https://github.com/ValarioWhite/create-a-basic-search-engine-website-using-AWS/assets/126350373/98ff6774-c903-4c4f-92bb-c5582cc2f395">


2. Select "Author from scratch". Use name **GamingSearchEngine** , select **Python 3.8** as Runtime. Under Permissions, select "Use an existing role", and select **lambda-apigateway-dynamodb-role** that we created, from the drop down

3. Click "Create function"

<img width="919" alt="image" src="https://github.com/ValarioWhite/create-a-basic-search-engine-website-using-AWS/assets/126350373/b5943d7b-cf16-4b22-bdef-ef94109b6c10">

4. Replace the existing sample code with the following code snippet and click "Deploy"

**Python CRUD Operation Code**

```python
import boto3
import json

def lambda_handler(event, context):
    # Create a DynamoDB client
    dynamodb = boto3.resource('dynamodb')

    # Retrieve the partition key value from the event
    partition_key_value = event['Name of Game']

    # Get the DynamoDB table
    table = dynamodb.Table('GamingSearchEngine')

    # Call the get_item operation
    response = table.get_item(
        Key={
            'Name of Game': partition_key_value
        }
    )

    # Process the response
    if 'Item' in response:
        item = response['Item']
        # Do something with the item
        item_data = json.loads(json.dumps(item))  # Convert item to JSON-compatible format
        return {
            'statusCode': 200,
            'body': item_data
        }
    else:
        return {
            'statusCode': 404,
            'body': 'Item not found'
        }

        
```
<img width="608" alt="image" src="https://github.com/ValarioWhite/create-a-basic-search-engine-website-using-AWS/assets/126350373/ebc0decd-f1e9-4dca-9f35-f6b30e0e0303">

DONE

### Step 4 - Test Lambda Function

Let's test our newly created function. Testing this function will make sure we will receive gaming information back when we search for the game.

**Get Information for Minecraft Game**
1. Click the arrow on the "Test" button and click "Configure test events"

![Configure test events](https://user-images.githubusercontent.com/126350373/221981236-6632840f-3f45-4373-9e2c-f1644699b67a.png)

2. Name the event "Test" then Paste the following JSON into the event. Afterwards, click "Save" to create.
```json
{
  "Name of Game" : "Minecraft"
}
```
<img width="308" alt="image" src="https://github.com/ValarioWhite/create-a-basic-search-engine-website-using-AWS/assets/126350373/8e953640-2095-4c76-97eb-5884a9211445">


3. Click "Test", and it will execute the test event. You should see the output in the console

<img width="611" alt="image" src="https://github.com/ValarioWhite/create-a-basic-search-engine-website-using-AWS/assets/126350373/b468e0a8-1ee9-453f-9873-edca8fe61f70">

DONE

### Step 5 - Create API and Deploy API

**To create the API**
1. Go to API Gateway console
2. Click "Create API" 
3. Scroll down to REST API and click "Build"

![Build API](https://user-images.githubusercontent.com/126350373/221991046-d4a3f54c-5f82-48c9-9941-4cf100559302.png)

3. Make sure to select "New API" and Give the API name as **GamingSearchEngine**, keep everything as is, click "Create API"

<img width="972" alt="image" src="https://github.com/ValarioWhite/create-a-basic-search-engine-website-using-AWS/assets/126350373/34725232-2623-4dfd-890b-8b4fd5d1aa27">

4. Each API is collection of resources and methods that are integrated with backend HTTP endpoints, Lambda functions, or other AWS services. Typically, API resources are organized in a resource tree according to the application logic. At this time you only have the root resource, but let's add a resource next 

Click "Actions", then click "Method"

<img width="343" alt="image" src="https://github.com/ValarioWhite/create-a-basic-search-engine-website-using-AWS/assets/126350373/eb9cae2c-ce40-483d-bb1e-3a7e0f8b4236">

5. Select "POST" in the dropdown and click the check button

<img width="293" alt="image" src="https://github.com/ValarioWhite/create-a-basic-search-engine-website-using-AWS/assets/126350373/d99078ac-3e44-403e-b86e-ef830567fa85">

6. Keep the integration type as Lambda Function. Make sure the Region is the same as your Lambda Function and DynamoDB (It should be because of the default settings). 

7. In Lambda Function text bar type in "GamingSearchEngine" and select that lambda function we created in the previous step. Click Save then OK.

<img width="740" alt="image" src="https://github.com/ValarioWhite/create-a-basic-search-engine-website-using-AWS/assets/126350373/cb4e5dab-f662-420a-ba76-a070b2ba78ec">

8. Click the "Action" dropdown and click Enable CORS.

<img width="314" alt="image" src="https://github.com/ValarioWhite/create-a-basic-search-engine-website-using-AWS/assets/126350373/28bd7b5a-687f-4603-b5e3-f7ae93191e9a">

9. Make sure POST is checked under "Methods" then click the "Enable CORS and replace existing CORS headers" button at the bottom right. Then click "Yes, replace existin values"

<img width="802" alt="image" src="https://github.com/ValarioWhite/create-a-basic-search-engine-website-using-AWS/assets/126350373/a061ada3-d98f-48bd-be02-5886d7bf5b65">

<img width="340" alt="image" src="https://github.com/ValarioWhite/create-a-basic-search-engine-website-using-AWS/assets/126350373/a94845ba-7289-4b5b-a987-d6ebfb282625">

10. Click the "Action" dropdown and click Deploy API. Click deployment stage

<img width="244" alt="image" src="https://github.com/ValarioWhite/create-a-basic-search-engine-website-using-AWS/assets/126350373/27519321-e00c-444f-b8a3-e97309d61958">

<img width="226" alt="image" src="https://github.com/ValarioWhite/create-a-basic-search-engine-website-using-AWS/assets/126350373/c7487aca-f7a2-43cd-b977-0459d0d151f6">

<img width="226" alt="image" src="https://github.com/ValarioWhite/create-a-basic-search-engine-website-using-AWS/assets/126350373/a3e1e871-dc31-4b0c-91a4-d2ec4cccd30d">

11. Keep your Invoke URL handy. We will use this in the next step.

<img width="881" alt="image" src="https://github.com/ValarioWhite/create-a-basic-search-engine-website-using-AWS/assets/126350373/394198be-b26c-4d34-a9d4-42e2e353ab43">

### Step 6 - Update the HTML Code and Save It

1. Copy the HTML code shown below
```html
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

2. Paste it to Notepad (for windows), TextEdit (for apple), or a source code editor like Visual Studio Code (download the software from a Google Search) 

3. Update the "INVOKE URL" in the code with your REST API Invoke URL from API Gateway in the previous step.

Insert your Invoke URL here. Keep the quotations around your link as well.

<img width="470" alt="image" src="https://github.com/ValarioWhite/create-a-basic-search-engine-website-using-AWS/assets/126350373/16b6daea-cdd7-4efc-abb4-c79001ddbe69">

4. Save the file as "index" of html type. Thus, you may have to save it as "index.html". 

Verify: Right click the saved file and view the properties. It should be of HTML type of file with the name "index"
<img width="271" alt="image" src="https://github.com/ValarioWhite/create-a-basic-search-engine-website-using-AWS/assets/126350373/6ed2d8fe-79d7-4541-a9c6-f55d1db91aa1">

5. Turn the file into a "compressed (zipped) folder". Again, keep the name as "index"

***Windows: Right Click, Send to, "Compressed (zipped) folder"***
***IOS: Right Click, Choose "Compress"***



### Step 7 - Create Website using Amplify

1. Go to the AWS Amplify console
2. Click "GET STARTED" and then click get started for "Host your web app" under "Amplify Hosting"

<img width="374" alt="image" src="https://github.com/ValarioWhite/create-a-basic-search-engine-website-using-AWS/assets/126350373/ac77b331-f131-4c98-91e9-78a9ce8a8839">

<img width="406" alt="image" src="https://github.com/ValarioWhite/create-a-basic-search-engine-website-using-AWS/assets/126350373/0f5e7504-20f6-4ae8-8ff5-41022f256c61">

3. Click "Deploy without Git provider" option then click "Continue"
<img width="670" alt="image" src="https://github.com/ValarioWhite/create-a-basic-search-engine-website-using-AWS/assets/126350373/ddbb458e-a8e7-4c63-984c-4385022f7866">

4. Name the app "GamingSearchEngine", Name the environment "dev", then drag and drop the zipped "index" file that was saved in the previous step. Then click "Save and deploy".
<img width="428" alt="image" src="https://github.com/ValarioWhite/create-a-basic-search-engine-website-using-AWS/assets/126350373/f623bec5-3997-4c64-b074-c250bd288ca9">

5. Once deployed, click the domain URL. You should be directed to the website we created via the HTML code.
<img width="836" alt="image" src="https://github.com/ValarioWhite/create-a-basic-search-engine-website-using-AWS/assets/126350373/e2e92244-6647-430c-8f2b-85b8a7101905">

HTML Website we created
<img width="943" alt="image" src="https://github.com/ValarioWhite/create-a-basic-search-engine-website-using-AWS/assets/126350373/f5d07ad4-d224-4d33-9f97-f4558e1679ea">


### Step 8 - Test the Webpage

1. Type in "Minecraft" and you should receive a response back. Feel free to test the other game names as well (Must type the names of them exactly how they are in DynamoDB - Yes, Case Sensitive).
<img width="947" alt="image" src="https://github.com/ValarioWhite/create-a-basic-search-engine-website-using-AWS/assets/126350373/28799a92-fc31-4f33-9dd5-d79f561b1d34">

Congradulations, you've created a basic search engine.

## Clean Up

### Step 9 - Delete the Web Hosting App - Amplify Console

<img width="844" alt="image" src="https://github.com/ValarioWhite/create-a-basic-search-engine-website-using-AWS/assets/126350373/46392828-0693-4d31-aadf-88e8343a3add">

### Step 10 - Delete the REST API - API Gateway Console

<img width="845" alt="image" src="https://github.com/ValarioWhite/create-a-basic-search-engine-website-using-AWS/assets/126350373/2852dd5f-e476-4a93-8dfb-d6343e0a6f11">

### Step 11 - Delete the Lambda Function - Lambda Console

<img width="814" alt="image" src="https://github.com/ValarioWhite/create-a-basic-search-engine-website-using-AWS/assets/126350373/239c3afe-8520-4556-9a70-204ccd4af7c0">

### Step 12 - Delete the Database - DynamoDB Console

<img width="817" alt="image" src="https://github.com/ValarioWhite/create-a-basic-search-engine-website-using-AWS/assets/126350373/a749e627-8981-4862-a898-bf47a2d4b439">
