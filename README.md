# Deploy a Serverless Function using AWS Lambda

Develop and deploy a serverless function using AWS Lambda, leveraging the AWS CLI for automated deployment. Then, integrate it with API Gateway to expose the function as a secure and scalable HTTP endpoint.

## Prerequisites
1. **AWS Account** – You need an active AWS account. Sign up at [AWS Console](https://aws.amazon.com/).
2. **IAM Role with Lambda, API Permissions** – Create an AWS IAM role with the `AWSLambdaBasicExecutionRole` policy.
3. **AWS CLI Installed & Configured** – To interact with AWS services via the command line.

## Setup
### **1. Create a directory**
```sh
mkdir lambda-api && cd lambda-api
```

### **2. Install and configure AWS CLI**
```sh
aws configure
```

## Step 1: Write a Simple Function (`handler.py`)
This is your actual serverless function that AWS Lambda will execute.
Create a file named `handler.py` with the following code:

```python
import json

def lambda_handler(event, context):
    return {
        "statusCode": 200,
        "body": json.dumps({"message": "Hello from AWS Lambda!"})
    }
```

## Step 2: Deploy as an AWS Lambda Function
Before deploying to AWS, package your function as a ZIP file.

### **1. Create a Deployment Package**
```sh
zip function.zip handler.py
```

### **2. Create an AWS Lambda Function**
Upload the ZIP file to AWS Lambda using AWS CLI:
```sh
aws lambda create-function \
    --function-name myServerlessFunction \
    --runtime python3.9 \
    --role arn:aws:iam::<AWS_ACCOUNT_ID>:role/<ROLE_NAME> \
    --handler handler.lambda_handler \
    --zip-file fileb://function.zip
```
![Screenshot (42)](https://github.com/user-attachments/assets/0932839b-64d1-44b7-bd30-da3ea93295e3)

## Step 3: Configure API Gateway
### **1. Create a REST API**
```sh
aws apigateway create-rest-api --name "MyServerlessAPI"
```
👉 This creates an API Gateway and returns an **API_ID**.

### **2. Create a Resource**
Retrieve the root resource ID:
```sh
aws apigateway get-resources --rest-api-id <API_ID>
```
👉 This gives you a **PARENT_RESOURCE_ID**.

Create a new resource:
```sh
aws apigateway create-resource --rest-api-id <API_ID> --parent-id <PARENT_RESOURCE_ID> --path-part myresource
```

### **3. Create a GET Method and Integration**
Create a GET method:
```sh
aws apigateway put-method --rest-api-id <API_ID> --resource-id <RESOURCE_ID> --http-method GET --authorization-type "NONE"
```
![Screenshot (45)](https://github.com/user-attachments/assets/21277fb0-324a-4e6e-bd68-0b96045bea45)

Link the method to Lambda:
```sh
aws apigateway put-integration --rest-api-id <API_ID> --resource-id <RESOURCE_ID> --http-method GET --type AWS_PROXY --integration-http-method POST --uri arn:aws:apigateway:<REGION>:lambda:path/2015-03-31/functions/arn:aws:lambda:<REGION>:<AWS_ACCOUNT_ID>:function:myServerlessFunction/invocations
```

### **4. Deploy the API**
```sh
aws apigateway create-deployment --rest-api-id <API_ID> --stage-name dev
```

## Step 4: Grant API Gateway Permission to Invoke Lambda
```sh
aws lambda add-permission \
    --function-name myServerlessFunction \
    --statement-id apigateway-invoke \
    --action lambda:InvokeFunction \
    --principal apigateway.amazonaws.com
```

## Step 5: Get the API Endpoint
Retrieve your API URL:
```sh
echo "https://<API_ID>.execute-api.<REGION>.amazonaws.com/dev/myresource"
```

Test it using:
```sh
curl https://<API_ID>.execute-api.<REGION>.amazonaws.com/dev/myresource
```
![Screenshot (49)](https://github.com/user-attachments/assets/d686d47d-e1cf-4aed-84a7-523faf3e3073)

![Screenshot (48)](https://github.com/user-attachments/assets/25e39f5c-687c-4a9a-9ad3-5a9d6a28a4c0)


## 🎯 Summary of the Process
1️⃣ Write the function (`handler.py`).
2️⃣ Package it into a ZIP file.
3️⃣ Upload to AWS Lambda using AWS CLI.
4️⃣ Create API Gateway to expose it as an HTTP endpoint.
5️⃣ Deploy API Gateway and link it to Lambda.
6️⃣ Allow API Gateway to call Lambda (IAM permissions).
7️⃣ Get API endpoint and test it.

Your serverless API is now live and scales automatically! 🚀

