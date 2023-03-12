---
title: '5. Deploy CRUD using AWS API Gateway, Serverless Lambda and DynamoDB'
date: 2021-01-02
weight: 5
layout: 'list'
---
---
> Specification : AWS, API Gateway, Serverless, Lambda, RDS, Postman

### **Lab Topology**
![aws-lambda-diagram](./aws-lambda-diagram.png)

![aws-lambda-detail](./aws-lambda-detail.png)

&nbsp;


#### Step-by-step
- Create IAM role for allow configuration
- Create Database table with DynamoDB
- Create AWS API Gateway service
- Create Lambda function
- Testing CRUD with postman
- Verify

#### A. Create IAM role

First step is create IAM role to allow Lambda function to call AWS services, for it you can follow guide bellow :

1. Login to your AWS console, search and chose IAM menu

![aws1](./docs/aws1.png)

2. Choose `Roles` and create `role`

![aws2](./docs/aws2.png)

3. Choose AWS Services -> Lambda and Next
   
![aws3](./docs/aws3.png)

4. And you can see menu bellow
![aws4](./docs/aws4.png)

5. For integrate lambda with RDS & cloudwatch, wee need filter & checklist `cloudwatchfullaccess`
![aws5](./docs/aws5.png)

6. And the last one, search `dynamodb` and checklist full access permissions
![aws6](./docs/aws6.png)

7. Add rolename
![aws7](./docs/aws7.png)

8. Verify you already added two roles for it, and create role
![aws8](./docs/aws8.png)

---
&nbsp;

#### B. Create RDS DynamoDB Table
Next step we need create database for store data, with step bellow : 

1. Choose DynamoDB services

![aws9](./docs/aws9.png)

2. Choose Dashboard and create `Table`
   
![aws10](./docs/aws10.png)

3. Put table name, you can same with our tutorial use `food-aws-serverless` and put `Partition Key` with `foodId` for indexing like bellow, and create table  
    
![aws11](./docs/aws11.png)

4. You can see table after created
   
![aws12](./docs/aws12.png)

---
&nbsp;


#### C. Create Lambda Function

1. After create RDS, next create Lambda function to integrate between API Gateway to RDS DynamoDB

![aws13](./docs/aws13.png)

2. Choose `Functions` and `create function`

![aws14](./docs/aws14.png)

3. And follow function like bellow and create function

![aws15](./docs/aws15.png)

![aws16](./docs/aws16.png)

4. And you can see lambda function has been created, after that we need increase specs of lambda with step bellow. Click `configuration > Edit`, Increase value of Memory and Storage and save.
   
![aws17](./docs/aws17.png)
![aws18](./docs/aws18.png)

5. And `Important` step is create `lambda function` for CRUD, choose code > edit lambda_function.py > copas code from bellow > and last deploy
   
![aws19](./docs/aws19.png)

`lambda_function.py`

```python
import boto3
import json
from botocore.vendored import requests

from custom_encoder import CustomEncoder
import logging
logger = logging.getLogger()
logger.setLevel(logging.INFO)

dynamodbTableName = 'food-aws-serverless'
dynamodb = boto3.resource('dynamodb')
table = dynamodb.Table(dynamodbTableName)

getMethod = 'GET'
postMethod = 'POST'
patchMethod = 'PATCH'
deleteMethod = 'DELETE'
healthPath = '/health'
foodPath = '/food'

def lambda_handler(event, context):
    logger.info(event)
    httpMethod = event['httpMethod']
    path = event['path']
    if httpMethod == getMethod and path == healthPath:
        response = buildResponse(200)
    elif httpMethod == getMethod and path == foodPath:
        response = getFood(event['queryStringParameters']['foodId'])
    elif httpMethod == postMethod and path == foodPath:
        response = saveFood(json.loads(event['body']))
    elif httpMethod == patchMethod and path == foodPath:
        requestBody = json.loads(event['body'])
        response = modifyFood(requestBody['foodId'], requestBody['updateKey'], requestBody['updateValue'])
    elif httpMethod == deleteMethod and path == foodPath:
        requestBody = json.loads(event['body'])
        response = deleteFood(requestBody['foodId'])
    else:
        response = buildResponse(404, 'Sorry, Not Found')
    
    return response

def getFood(foodId):
    try:
        response = table.get_item(
            Key={
                'foodId': foodId
            }
        )
        if 'Item' in response:
            return buildResponse(200, response['Item'])
        else:
            return buildResponse(404, {'Message' : 'FoodId: %s not found' % foodId})
    except:
        logger.exception('Do your custom error handling here, I am just gonna log it out there!')

def saveFood(requestBody):
    try:
        table.put_item(Item=requestBody)
        body = {
            'Operation' : 'SAVE',
            'Message' : 'SUCCESS',
            'Item' : requestBody
        }
        return buildResponse(200, body)
    except:
        logger.exception('Do your custom error handling here, I am just gonna log it out there!')

def modifyFood(foodId, updateKey, updateValue):
    try:
        response = table.update_item(
            Key={
                'foodId': foodId
            },
            UpdateExpression='set %s = :value' % updateKey,
            ExpressionAttributeValues={
                ':value': updateValue
            },
            ReturnValues='UPDATED_NEW'
        )
        body = {
            'Operation': 'UPDATE',
            'Message': 'SUCCESS',
            'UpdatedAttrebutes': response
        }
        return buildResponse(200, body)
    except:
        logger.exception('Do your custom error handling here, I am just gonna log it out there!')

def deleteFood(foodId):
    try:
        response = table.delete_item(
            Key={
                'foodId': foodId
            },
            ReturnValues='ALL_OLD'
        )
        body = {
            'Operation': 'DELETE',
            'Message': 'SUCCESS',
            'UpdatedAttrebutes': response
        }
        return buildResponse(200, body)
    except:
        logger.exception('Do your custom error handling here, I am just gonna log it out there!')



def buildResponse(statusCode, body=None):
    response = {
        'statusCode' : statusCode,
        'headers' : {
            'Content-Type': 'application/json',
            'Access-Controll-Allow-Origin': '*'
        }
    }
    if body is not None:
        response['body'] = json.dumps(body, cls=CustomEncoder)
    return response

```


6. And last on is create new file named `custom_encoder.py` > put code bellow > and `deploy` again.

![aws20](./docs/aws20.png)

`custom_encoder.py`

```
import json
from decimal import Decimal

class CustomEncoder(json.JSONEncoder):
    def default(self, obj):
        if isinstance(obj, Decimal):
            return float(obj)

        return json.JSONEncoder.default(self, obj)
```

&nbsp;

---
&nbsp;

#### D. Create AWS API Gateway
Next is create Rest API gateway, to interact between and user & AWS services, for more you can follow bellow step :

1. Choose `AWS API Gateway` menu

![aws21](./docs/aws21.png)

2. Choose `Public REST API` and `build`

![aws22](./docs/aws22.png)

3. Click REST > New API and put API name
   
![aws23](./docs/aws23.png)

4. And next `create Resource` for place are API Method, bellow for detail structure :

```
1. Resource /health
Method GET

2. Resource /food
Method GET, POST, PATCH, DELETE

3. Resource /foods (optional, not important)
Method GET, POST, PATCH, DELETE
```

![aws24](./docs/aws24.png)

4. Input `Resource name`, `Resource path` and checklist Enable API Gateway

![aws25](./docs/aws25.png)

5. And create also for /food resource like above

![aws27](./docs/aws27.png)

6. After create `Resource`, next create `Method` for every resource. Click Resource name, for example `/health` > Create Method 

```
1. Resource /health
Method GET

2. Resource /food
Method GET, POST, PATCH, DELETE

3. Resource /foods (optional, not important)
Method GET, POST, PATCH, DELETE
```

![aws29](./docs/aws29.png)

7. Chose Method, for example `GET` and click OK

![aws30](./docs/aws30.png)

8. Checklist `Lambda Function` > Proxy integration > and chose `Lambda Function` previously created > save

![aws31](./docs/aws31.png)

9. Click OK for continue, and REPEAT create Method for `every resource` like it and follow structure 

![aws32](./docs/aws32.png)

10. After all `Method and Resource` has been created, last step is `Deploy API` like bellow

![aws33](./docs/aws33.png)

11. Create new stage name, for example: `prod`, and finish you already created API Gateway

![aws34](./docs/aws34.png)

---

&nbsp;

#### E. Testing & Verify CRUD

1. Before testing, you need download `Postman Application` for testing API, download on : https://www.postman.com/downloads/ 
2. First step is copy API Endpoint in APIs menu to access from public internet 

![aws36](./docs/aws36.png)

##### GET Option (Health Check)
To check API status reached or not, with status 200 OK

![aws37](./docs/aws37.png)

##### POST Option 
To Put new database to RDS via Json format file (key-value), bellow example to add data :

```
{
    "foodId": "001",
    "name": "Banana",
    "price": "500"
}
```

![aws38](./docs/aws38.png)
![aws39](./docs/aws39.png)


And you can see result success POST in DynamoDB table menu :

![aws40](./docs/aws40.png)

##### DELETE Option
For delete database on RDS you can use delete option with index ID (foodId) like bellow 

![aws41](./docs/aws41.png)

And you can see after deleted foodId = 002 on RDS Table 

![aws42](./docs/aws42.png)

##### GET Option
To check detail table by foodId

![aws43](./docs/aws43.png)

##### PATCH Option
Patch you can use for update about value/key, bellow example for update banana price, from 500 to 99999

![aws44](./docs/aws44.png)
![aws45](./docs/aws45.png)

---

&nbsp;

#### Reference 
- Felix Yu Channel : https://www.youtube.com/watch?v=9eHh946qTIk&list=LL&index=32&t=159s