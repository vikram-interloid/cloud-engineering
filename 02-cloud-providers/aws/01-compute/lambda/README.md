# AWS Lambda for Web Development

> **Focus**: Serverless web APIs, HTTP payload format 2.0, cold start optimization, RDS Proxy database pooling, and Lambda Function URLs for webhooks.

---

## 1. Serverless Web API Architecture with RDS Proxy

When hundreds of concurrent Lambda functions open direct database connections, traditional relational databases (PostgreSQL/MySQL) run out of connection slots. **Amazon RDS Proxy** pools and shares DB connections across serverless functions.

```mermaid
flowchart LR
    BROWSER["Web Users"] --> APIGW["Amazon API Gateway"]
    
    subgraph LAMBDA_CONCURRENCY["Auto-Scaling Lambda Fleet"]
        L1["Lambda Exec 1"]
        L2["Lambda Exec 2"]
        L3["Lambda Exec N... (e.g. 500)"]
    end

    APIGW --> L1
    APIGW --> L2
    APIGW --> L3

    subgraph CONNECTION_MGMT["Connection Management Layer"]
        PROXY["<b>Amazon RDS Proxy</b><br/>Multiplexes 500 Lambda connections<br/>into 20 warm persistent DB connections"]
    end

    L1 --> PROXY
    L2 --> PROXY
    L3 --> PROXY

    PROXY --> RDS[("Amazon Aurora PostgreSQL<br/>(Protected from Connection Exhaustion)")]

    style BROWSER fill:#0284c7,fill-opacity:0.12,stroke:#0284c7,stroke-width:1.5px,color:inherit
    style APIGW fill:#0284c7,fill-opacity:0.15,stroke:#0284c7,stroke-width:2px,color:inherit
    style LAMBDA_CONCURRENCY fill:#10b981,fill-opacity:0.06,stroke:#10b981,stroke-width:1.5px,color:inherit
    style PROXY fill:#f59e0b,fill-opacity:0.2,stroke:#f59e0b,stroke-width:2px,color:inherit
    style RDS fill:#3b82f6,fill-opacity:0.15,stroke:#3b82f6,stroke-width:2px,color:inherit
```

---

## 2. Cold Start Elimination for Interactive Web APIs

```mermaid
flowchart TD
    subgraph COLD["Standard On-Demand (Cold Start: 300ms - 2s)"]
        direction LR
        C_BOOT["Download Container/Zip"] --> C_INIT["Runtime Init (Node/Python)"] --> C_EXEC["Function Execution"]
    end

    subgraph WARM["Provisioned Concurrency (Instant Response: &lt;15ms)"]
        direction LR
        P_POOL["Pre-Initialized Warm Pool Always Kept Hot in Memory"] --> P_EXEC["Immediate Handler Execution"]
    end

    style COLD fill:#ef4444,fill-opacity:0.1,stroke:#ef4444,stroke-width:1.5px,color:inherit
    style WARM fill:#10b981,fill-opacity:0.15,stroke:#10b981,stroke-width:2px,color:inherit
```

> [!TIP] Web Developer Cold Start Optimization Rules
> 1. **Keep Packages Lean**: Tree-shake imports (e.g., `import { S3Client } from '@aws-sdk/client-s3'` instead of the entire `aws-sdk`).
> 2. **Global Connection Reuse**: Initialize DB clients, HTTP clients, and AWS SDKs **outside** the handler function so warm microVMs reuse open sockets.
> 3. **ARM64 Architecture (Graviton2/3)**: Select ARM64 for 20% lower cost and faster execution times.

---

## 3. Lambda Function URLs for Webhooks (Stripe / GitHub)

For third-party webhook receivers that do not require API Gateway features, **Lambda Function URLs** provide a direct HTTPS endpoint with built-in CORS.

```mermaid
flowchart LR
    STRIPE["Stripe Webhook Event<br/>(payment_intent.succeeded)"] -->|HTTPS POST| FURL["<b>Lambda Function URL</b><br/>https://abc123xyz.lambda-url.us-east-1.on.aws/"]
    FURL --> HANDLER["Lambda Handler Function"]
    HANDLER --> SQS["Amazon SQS Queue (Async Fulfillment)"]
    HANDLER -->> STRIPE["200 OK (Instant Receipt)"]

    style STRIPE fill:#0284c7,fill-opacity:0.12,stroke:#0284c7,stroke-width:1.5px,color:inherit
    style FURL fill:#0284c7,fill-opacity:0.15,stroke:#0284c7,stroke-width:2px,color:inherit
    style HANDLER fill:#10b981,fill-opacity:0.15,stroke:#10b981,stroke-width:2px,color:inherit
```

---

## 4. Production Web Handler (Payload Format 2.0)

```javascript
// Node.js 20 - API Gateway HTTP API Proxy Handler
export const handler = async (event) => {
  const method = event.requestContext.http.method;
  const path = event.requestContext.http.path;

  // Read Claims from Cognito / JWT Authorizer
  const userId = event.requestContext.authorizer?.jwt?.claims?.sub;

  if (method === 'GET' && path === '/api/user') {
    return {
      statusCode: 200,
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ userId, status: 'active' }),
    };
  }

  return { statusCode: 404, body: JSON.stringify({ error: 'Not Found' }) };
};
```

---

## 5. Infrastructure as Code & SDK Implementations

### CDK (TypeScript)

```typescript
import * as cdk from 'aws-cdk-lib';
import * as lambda from 'aws-cdk-lib/aws-lambda';
import * as sqs from 'aws-cdk-lib/aws-sqs';
import * as eventSources from 'aws-cdk-lib/aws-lambda-event-sources';
import { Construct } from 'constructs';

export class FaaSStack extends cdk.Stack {
  constructor(scope: Construct, id: string, props?: cdk.StackProps) {
    super(scope, id, props);

    const queue = new sqs.Queue(this, 'OrdersQueue', {
      visibilityTimeout: cdk.Duration.seconds(300),
    });

    const handler = new lambda.Function(this, 'OrderProcessor', {
      runtime: lambda.Runtime.NODEJS_20_X,
      handler: 'index.handler',
      code: lambda.Code.fromInline(`
        exports.handler = async (event) => {
          for (const record of event.Records) {
            console.log("Processed order payload:", record.body);
          }
          return { statusCode: 200 };
        };
      `),
      memorySize: 256,
      timeout: cdk.Duration.seconds(30),
      architecture: lambda.Architecture.ARM_64,
    });

    handler.addEventSource(new eventSources.SqsEventSource(queue, { batchSize: 10 }));
  }
}
```

### CDK (Python)

```python
import aws_cdk as cdk
from aws_cdk import (
    aws_lambda as _lambda,
    aws_sqs as sqs,
    aws_lambda_event_sources as event_sources,
    Duration
)
from constructs import Construct

class FaaSStack(cdk.Stack):
    def __init__(self, scope: Construct, construct_id: str, **kwargs) -> None:
        super().__init__(scope, construct_id, **kwargs)

        queue = sqs.Queue(self, "OrdersQueue", visibility_timeout=Duration.seconds(300))

        handler = _lambda.Function(
            self, "OrderProcessor",
            runtime=_lambda.Runtime.PYTHON_3_12,
            handler="index.handler",
            code=_lambda.Code.from_inline(
                "def handler(event, context):\n"
                "    for record in event.get('Records', []):\n"
                "        print(f'Processed order: {record[\"body\"]}')\n"
                "    return {'statusCode': 200}\n"
            ),
            memory_size=256,
            timeout=Duration.seconds(30),
            architecture=_lambda.Architecture.ARM_64
        )

        handler.add_event_source(event_sources.SqsEventSource(queue, batch_size=10))
```

### YAML (AWS SAM)

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Transform: AWS::Serverless-2016-10-31
Description: AWS Lambda Function with SQS Trigger

Resources:
  OrdersQueue:
    Type: AWS::SQS::Queue
    Properties:
      VisibilityTimeout: 300

  OrderProcessorFunction:
    Type: AWS::Serverless::Function
    Properties:
      Handler: index.handler
      Runtime: nodejs20.x
      Architectures:
        - arm64
      MemorySize: 256
      Timeout: 30
      InlineCode: |
        exports.handler = async (event) => {
          for (const record of event.Records) {
            console.log("Processed:", record.body);
          }
          return { statusCode: 200 };
        };
      Events:
        SQSTrigger:
          Type: SQS
          Properties:
            Queue: !GetAtt OrdersQueue.Arn
            BatchSize: 10
```

### SDK (TypeScript)

```typescript
import { LambdaClient, InvokeCommand } from '@aws-sdk/client-lambda';

const client = new LambdaClient({ region: 'us-east-1' });

async function invokeFunction() {
  const payload = { orderId: 'ord-8832', amount: 49.99, currency: 'USD' };

  const command = new InvokeCommand({
    FunctionName: 'OrderProcessorFunction',
    InvocationType: 'RequestResponse',
    Payload: Buffer.from(JSON.stringify(payload)),
  });

  const response = await client.send(command);
  const result = Buffer.from(response.Payload ?? []).toString('utf-8');
  console.log('Execution Result:', JSON.parse(result));
}

invokeFunction();
```

### SDK (Python)

```python
import json
import boto3

lambda_client = boto3.client('lambda', region_name='us-east-1')

def invoke_function():
    payload = {"orderId": "ord-8832", "amount": 49.99, "currency": "USD"}
    
    response = lambda_client.invoke(
        FunctionName='OrderProcessorFunction',
        InvocationType='RequestResponse',
        Payload=json.dumps(payload).encode('utf-8')
    )
    
    result = json.loads(response['Payload'].read().decode('utf-8'))
    print("Execution Result:", result)

invoke_function()
```
