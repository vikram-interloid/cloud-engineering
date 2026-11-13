# Lab 03 · Deploy a Serverless CRUD API

> **Objective**: Build an end-to-end serverless REST API using API Gateway (HTTP API), AWS Lambda (Python), and Amazon DynamoDB.

---

## Architecture Flow

```mermaid
flowchart LR
    CLIENT["HTTP Client (curl / Postman)"] --> APIGW["Amazon API Gateway<br/>(HTTP API)"]
    APIGW -->|Proxy Integration| LAMBDA["AWS Lambda<br/>(Python Handler)"]
    LAMBDA -->|boto3 PutItem / GetItem| DDB[("Amazon DynamoDB<br/>(Orders Table)")]

    style CLIENT fill:#0284c7,fill-opacity:0.12,stroke:#0284c7,stroke-width:1.5px,color:inherit
    style APIGW fill:#0284c7,fill-opacity:0.15,stroke:#0284c7,stroke-width:2px,color:inherit
    style LAMBDA fill:#10b981,fill-opacity:0.15,stroke:#10b981,stroke-width:2px,color:inherit
    style DDB fill:#f59e0b,fill-opacity:0.15,stroke:#f59e0b,stroke-width:2px,color:inherit
```

---

## Lambda Handler Code (`lambda_function.py`)

```python
import json
import boto3
import os

dynamodb = boto3.resource('dynamodb')
table = dynamodb.Table(os.environ.get('TABLE_NAME', 'Orders'))

def handler(event, context):
    method = event.get('requestContext', {}).get('http', {}).get('method', 'GET')
    
    if method == 'POST':
        body = json.loads(event.get('body', '{}'))
        table.put_item(Item={'order_id': body['order_id'], 'amount': body['amount']})
        return {'statusCode': 201, 'body': json.dumps({'message': 'Order created'})}
    
    return {'statusCode': 200, 'body': json.dumps({'status': 'Serverless API healthy'})}
```
