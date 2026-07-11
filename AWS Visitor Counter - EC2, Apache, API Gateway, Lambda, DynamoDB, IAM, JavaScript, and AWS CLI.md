This is an excellent practice project because it covers **EC2, Apache, API Gateway, Lambda, DynamoDB, IAM, JavaScript, and AWS CLI** in one end-to-end implementation.

## Project Overview

**Project Name:** AWS Visitor Counter

### Services Used

* Amazon EC2
* Apache HTTP Server
* Amazon API Gateway
* AWS Lambda (Python)
* Amazon DynamoDB
* IAM
* JavaScript (Fetch API)
* AWS CLI

---

# Architecture

```text
                +----------------------+
                |     User Browser     |
                +----------+-----------+
                           |
                           |
                    HTTP Request
                           |
                           ▼
              +-------------------------+
              | EC2 + Apache Web Server |
              |     Static Website      |
              +-----------+-------------+
                          |
               JavaScript fetch()
                          |
                          ▼
                 Amazon API Gateway
                          |
                          ▼
                 AWS Lambda Function
                          |
          Read/Update Visitor Count
                          |
                          ▼
                   Amazon DynamoDB
```

---

# Project Structure

```
visitor-counter/

│
├── website/
│   ├── index.html
│   ├── style.css
│   └── script.js
│
├── lambda/
│   ├── lambda_function.py
│   └── requirements.txt
│
├── screenshots/
│
└── README.md
```

---

# Step 1

## Create DynamoDB Table

Table Name

```
VisitorCounter
```

Partition Key

```
id
```

Type

```
String
```

Insert one item

```json
{
"id":"visitors",
"count":0
}
```

---

# Step 2

## Create IAM Role for Lambda

Attach

* AWSLambdaBasicExecutionRole

Create Inline Policy

```json
{
    "Version":"2012-10-17",
    "Statement":[
        {
            "Effect":"Allow",
            "Action":[
                "dynamodb:GetItem",
                "dynamodb:UpdateItem"
            ],
            "Resource":"*"
        }
    ]
}
```

For production, replace `"Resource":"*"` with your DynamoDB table ARN.

---

# Step 3

## Create Lambda Function

Runtime

```
Python 3.12
```

Function Name

```
visitor-counter
```

### lambda_function.py

```python
import json
import boto3

dynamodb = boto3.client('dynamodb')

TABLE_NAME = "VisitorCounter"

def lambda_handler(event, context):

    response = dynamodb.update_item(
        TableName=TABLE_NAME,
        Key={
            'id':{
                'S':'visitors'
            }
        },
        UpdateExpression="SET #c = if_not_exists(#c, :start) + :inc",
        ExpressionAttributeNames={
            "#c":"count"
        },
        ExpressionAttributeValues={
            ":inc":{"N":"1"},
            ":start":{"N":"0"}
        },
        ReturnValues="UPDATED_NEW"
    )

    count = response["Attributes"]["count"]["N"]

    return {
        "statusCode":200,
        "headers":{
            "Access-Control-Allow-Origin":"*",
            "Content-Type":"application/json"
        },
        "body":json.dumps({
            "visitors":count
        })
    }
```

Deploy the function after saving.

---

# Step 4

## Test Lambda

Expected Output

```json
{
  "statusCode":200,
  "body":"{\"visitors\":\"1\"}"
}
```

Running it again:

```json
{
  "visitors":"2"
}
```

---

# Step 5

## Create API Gateway

Choose:

```
HTTP API
```

Integration

```
Lambda
```

Select

```
visitor-counter
```

Route

```
GET /
```

Deploy.

Example endpoint:

```
https://abcd123.execute-api.us-east-1.amazonaws.com/
```

---

# Step 6

## Enable CORS

Allow

```
GET
```

Origins

```
*
```

Headers

```
*
```

Deploy again after updating CORS settings.

---

# Step 7

## Create Website

### index.html

```html
<!DOCTYPE html>
<html>

<head>
    <title>Visitor Counter</title>
    <link rel="stylesheet" href="style.css">
</head>

<body>

<h1>AWS Visitor Counter</h1>

<h2>
Visitors:
<span id="count">Loading...</span>
</h2>

<script src="script.js"></script>

</body>

</html>
```

---

### style.css

```css
body{

font-family:Arial;
text-align:center;
margin-top:100px;
background:#f4f4f4;

}

h1{

color:#333;

}

h2{

font-size:40px;

}
```

---

### script.js

Replace the API URL with your deployed API Gateway endpoint.

```javascript
const API_URL = "YOUR_API_GATEWAY_URL";

fetch(API_URL)
.then(response => response.json())
.then(data => {

const body = JSON.parse(data.body);

document.getElementById("count").innerHTML = body.visitors;

});
```

If you configure API Gateway to return the JSON body directly (rather than Lambda proxy format), you can simplify the parsing accordingly.

---

# Step 8

## Upload Website to EC2

Install Apache

```bash
sudo yum install httpd -y
```

Start Apache

```bash
sudo systemctl enable httpd
sudo systemctl start httpd
```

Copy files

```bash
sudo cp -r website/* /var/www/html/
```

Permissions

```bash
sudo chmod -R 755 /var/www/html
```

Visit

```
http://<EC2-Public-IP>
```

---

# Step 9

## Expected Output

```
AWS Visitor Counter

Visitors

1
```

Refresh

```
2

3

4

5
```

Every refresh increments the value stored in DynamoDB.

---

# Step 10

## Verify DynamoDB

The item will look like:

```json
{
"id":"visitors",
"count":57
}
```

---

# Common Errors

| Error                        | Cause                        | Solution                                   |
| ---------------------------- | ---------------------------- | ------------------------------------------ |
| 500 Internal Server Error    | Lambda exception             | Check CloudWatch Logs                      |
| Missing Authentication Token | Wrong API endpoint or route  | Verify the deployed API URL                |
| CORS error                   | CORS not enabled             | Enable CORS and redeploy the API           |
| AccessDeniedException        | IAM permissions              | Ensure the Lambda role has DynamoDB access |
| ResourceNotFoundException    | Wrong table or function name | Verify resource names and Region           |
| Visitor count doesn't update | Incorrect key or item        | Confirm the `id` value is `visitors`       |

---

## Skills Demonstrated

By completing this project, you'll gain hands-on experience with:

* AWS Lambda
* Amazon API Gateway
* Amazon DynamoDB
* IAM Roles and Policies
* EC2 and Apache
* JavaScript Fetch API
* Python with `boto3`
* Serverless application architecture
* CORS configuration
* End-to-end AWS integration

This project is also a strong addition to a DevOps or Cloud portfolio, as it demonstrates integrating serverless services with a web frontend and persistent storage.
