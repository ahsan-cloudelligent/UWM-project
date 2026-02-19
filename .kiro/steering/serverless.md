# Serverless Development Standards

These standards define how we design, build, and maintain serverless applications on AWS to ensure scalability, cost-efficiency, security, and maintainability.

---

## 1. General Principles

- **Event-driven by default**: Design for asynchronous, event-based communication
- **Stateless functions**: No reliance on local state between invocations
- **Pay-per-use mindset**: Optimize for cost efficiency
- **Cold start awareness**: Design with cold starts in mind
- **Fail fast and retry**: Embrace eventual consistency and idempotency
- **Security by default**: Least privilege IAM, encryption everywhere

---

## 2. Technology Standards

### Core Stack
- **Runtime**: Node.js (LTS version) with TypeScript
- **Compute**: AWS Lambda
- **API**: API Gateway (REST API, HTTP API, WebSocket API)
- **Events**: EventBridge, SQS, SNS, DynamoDB Streams
- **Orchestration**: Step Functions
- **Database**: DynamoDB (primary), Aurora Serverless (when needed)
- **Storage**: S3
- **Infrastructure**: AWS CDK (TypeScript)

### Package Management
- Use `npm` or `pnpm` for dependency management
- Minimize dependencies to reduce cold starts
- Use Lambda layers for shared dependencies
- Lock dependency versions
- Regular security audits

---

## 3. Project Structure

Organize by feature with serverless-specific patterns:

```
src/
  functions/
    users/
      create-user/
        handler.ts
        handler.test.ts
        schema.ts
      get-user/
        handler.ts
        handler.test.ts
    orders/
      create-order/
      process-order/
  layers/
    common/
      utils/
      middleware/
      types/
  events/
    schemas/
    handlers/
  step-functions/
    workflows/
  shared/
    repositories/
    services/
infrastructure/
  lib/
    stacks/
    constructs/
```

**Principles**:
- One directory per Lambda function
- Shared code in layers
- Event schemas separate from handlers
- Step Functions workflows isolated

---

## 4. Lambda Function Standards

### Function Design
- **Single responsibility**: One function, one purpose
- **Small deployment packages**: < 50MB unzipped, < 10MB zipped
- **Fast initialization**: Minimize code outside handler
- **Timeout configuration**: Set realistic timeouts (default: 30s)
- **Memory allocation**: Right-size based on profiling (min 512MB for production)
- **Environment variables**: Use for configuration, never hardcode

### Handler Pattern
```typescript
import { APIGatewayProxyEvent, APIGatewayProxyResult } from 'aws-lambda';
import { logger } from '/opt/nodejs/logger';
import { errorHandler } from '/opt/nodejs/error-handler';

// Initialize outside handler (reused across invocations)
const dynamoClient = new DynamoDBClient({});

export const handler = async (
  event: APIGatewayProxyEvent
): Promise<APIGatewayProxyResult> => {
  try {
    logger.info('Processing request', { 
      path: event.path,
      requestId: event.requestContext.requestId 
    });
    
    // Business logic
    const result = await processRequest(event);
    
    return {
      statusCode: 200,
      headers: {
        'Content-Type': 'application/json',
        'X-Request-Id': event.requestContext.requestId,
      },
      body: JSON.stringify(result),
    };
  } catch (error) {
    return errorHandler(error, event.requestContext.requestId);
  }
};
```

### Cold Start Optimization
- **Minimize dependencies**: Only import what you need
- **Lazy load heavy modules**: Import inside handler if rarely used
- **Use Lambda layers**: Share common code across functions
- **Provisioned concurrency**: For latency-sensitive functions
- **Keep functions warm**: Use EventBridge scheduled rules (if cost-justified)
- **Optimize initialization**: Move SDK client creation outside handler

### Memory and Timeout Configuration
```typescript
// CDK configuration
new lambda.Function(this, 'UserFunction', {
  runtime: lambda.Runtime.NODEJS_20_X,
  handler: 'index.handler',
  code: lambda.Code.fromAsset('dist/users'),
  memorySize: 1024,        // Right-size based on profiling
  timeout: cdk.Duration.seconds(30),
  reservedConcurrentExecutions: 100,  // Prevent runaway costs
});
```

---

## 5. API Gateway Standards

### REST API vs HTTP API
- **Use HTTP API** for: Lower latency, lower cost, simple APIs
- **Use REST API** for: Request validation, API keys, usage plans, caching

### API Design
```typescript
// CDK API Gateway configuration
const api = new apigateway.RestApi(this, 'ServerlessApi', {
  restApiName: 'serverless-api',
  deployOptions: {
    stageName: 'prod',
    tracingEnabled: true,
    loggingLevel: apigateway.MethodLoggingLevel.INFO,
    metricsEnabled: true,
    throttlingRateLimit: 1000,
    throttlingBurstLimit: 2000,
  },
  defaultCorsPreflightOptions: {
    allowOrigins: apigateway.Cors.ALL_ORIGINS,
    allowMethods: apigateway.Cors.ALL_METHODS,
  },
});
```

### Request Validation
```typescript
// Define request model
const createUserModel = api.addModel('CreateUserModel', {
  contentType: 'application/json',
  schema: {
    type: apigateway.JsonSchemaType.OBJECT,
    required: ['name', 'email'],
    properties: {
      name: { type: apigateway.JsonSchemaType.STRING, minLength: 2 },
      email: { type: apigateway.JsonSchemaType.STRING, format: 'email' },
    },
  },
});

// Apply validation
resource.addMethod('POST', integration, {
  requestValidator: new apigateway.RequestValidator(this, 'Validator', {
    api,
    validateRequestBody: true,
  }),
  requestModels: {
    'application/json': createUserModel,
  },
});
```

### WebSocket API
```typescript
const webSocketApi = new apigatewayv2.WebSocketApi(this, 'WebSocketApi', {
  connectRouteOptions: { integration: connectIntegration },
  disconnectRouteOptions: { integration: disconnectIntegration },
  defaultRouteOptions: { integration: defaultIntegration },
});
```

---

## 6. Event-Driven Architecture

### EventBridge Standards
```typescript
// Event schema
interface UserCreatedEvent {
  version: '1.0';
  source: 'user-service';
  detailType: 'UserCreated';
  detail: {
    userId: string;
    email: string;
    timestamp: string;
  };
}

// Publish event
await eventBridgeClient.send(new PutEventsCommand({
  Entries: [{
    Source: 'user-service',
    DetailType: 'UserCreated',
    Detail: JSON.stringify({
      userId: user.id,
      email: user.email,
      timestamp: new Date().toISOString(),
    }),
    EventBusName: 'default',
  }],
}));

// CDK event rule
new events.Rule(this, 'UserCreatedRule', {
  eventBus: bus,
  eventPattern: {
    source: ['user-service'],
    detailType: ['UserCreated'],
  },
  targets: [new targets.LambdaFunction(handler)],
});
```

### SQS Standards
```typescript
// Standard queue for high throughput
const queue = new sqs.Queue(this, 'OrderQueue', {
  visibilityTimeout: cdk.Duration.seconds(300),
  retentionPeriod: cdk.Duration.days(14),
  deadLetterQueue: {
    queue: dlq,
    maxReceiveCount: 3,
  },
});

// FIFO queue for ordering guarantees
const fifoQueue = new sqs.Queue(this, 'OrderFifoQueue', {
  fifo: true,
  contentBasedDeduplication: true,
  deduplicationScope: sqs.DeduplicationScope.MESSAGE_GROUP,
});

// Lambda SQS event source
handler.addEventSource(new lambdaEventSources.SqsEventSource(queue, {
  batchSize: 10,
  maxBatchingWindow: cdk.Duration.seconds(5),
  reportBatchItemFailures: true,  // Partial batch failures
}));
```

### SNS Standards
```typescript
// Topic for fan-out
const topic = new sns.Topic(this, 'OrderTopic', {
  displayName: 'Order Events',
});

// Subscribe Lambda
topic.addSubscription(new subscriptions.LambdaSubscription(handler, {
  filterPolicy: {
    orderStatus: sns.SubscriptionFilter.stringFilter({
      allowlist: ['COMPLETED', 'FAILED'],
    }),
  },
}));
```

### DynamoDB Streams
```typescript
// Enable streams on table
const table = new dynamodb.Table(this, 'UsersTable', {
  partitionKey: { name: 'PK', type: dynamodb.AttributeType.STRING },
  sortKey: { name: 'SK', type: dynamodb.AttributeType.STRING },
  stream: dynamodb.StreamViewType.NEW_AND_OLD_IMAGES,
});

// Lambda stream event source
handler.addEventSource(new lambdaEventSources.DynamoEventSource(table, {
  startingPosition: lambda.StartingPosition.LATEST,
  batchSize: 100,
  maxBatchingWindow: cdk.Duration.seconds(10),
  retryAttempts: 3,
  bisectBatchOnError: true,
}));
```

---

## 7. Step Functions Standards

### Workflow Design
```typescript
// Define tasks
const validateTask = new tasks.LambdaInvoke(this, 'Validate', {
  lambdaFunction: validateFn,
  outputPath: '$.Payload',
});

const processTask = new tasks.LambdaInvoke(this, 'Process', {
  lambdaFunction: processFn,
  outputPath: '$.Payload',
});

const notifyTask = new tasks.LambdaInvoke(this, 'Notify', {
  lambdaFunction: notifyFn,
});

// Error handling
const errorHandler = new sfn.Pass(this, 'HandleError', {
  result: sfn.Result.fromObject({ error: 'Processing failed' }),
});

// Workflow definition
const definition = validateTask
  .next(processTask)
  .next(notifyTask)
  .addCatch(errorHandler, {
    errors: ['States.ALL'],
    resultPath: '$.error',
  });

// State machine
new sfn.StateMachine(this, 'OrderWorkflow', {
  definition,
  timeout: cdk.Duration.minutes(5),
  tracingEnabled: true,
});
```

### Workflow Patterns
- **Sequential**: Chain tasks with `.next()`
- **Parallel**: Use `sfn.Parallel` for concurrent execution
- **Choice**: Use `sfn.Choice` for conditional branching
- **Map**: Use `sfn.Map` for iterating over arrays
- **Wait**: Use `sfn.Wait` for delays
- **Retry**: Configure retry policies for transient failures

---

## 8. DynamoDB Standards

### Single-Table Design
```typescript
// Entity structure
interface UserEntity {
  PK: string;        // USER#<userId>
  SK: string;        // USER#<userId>
  GSI1PK: string;    // EMAIL#<email>
  GSI1SK: string;    // USER#<userId>
  type: 'USER';
  userId: string;
  email: string;
  name: string;
  createdAt: string;
}

interface OrderEntity {
  PK: string;        // USER#<userId>
  SK: string;        // ORDER#<orderId>
  GSI1PK: string;    // ORDER#<orderId>
  GSI1SK: string;    // STATUS#<status>#<timestamp>
  type: 'ORDER';
  orderId: string;
  userId: string;
  status: string;
  total: number;
  createdAt: string;
}
```

### Access Patterns
```typescript
// Get user by ID
const user = await dynamoClient.send(new GetItemCommand({
  TableName: tableName,
  Key: {
    PK: { S: `USER#${userId}` },
    SK: { S: `USER#${userId}` },
  },
}));

// Get user by email (GSI)
const userByEmail = await dynamoClient.send(new QueryCommand({
  TableName: tableName,
  IndexName: 'GSI1',
  KeyConditionExpression: 'GSI1PK = :email',
  ExpressionAttributeValues: {
    ':email': { S: `EMAIL#${email}` },
  },
}));

// Get user's orders
const orders = await dynamoClient.send(new QueryCommand({
  TableName: tableName,
  KeyConditionExpression: 'PK = :userId AND begins_with(SK, :orderPrefix)',
  ExpressionAttributeValues: {
    ':userId': { S: `USER#${userId}` },
    ':orderPrefix': { S: 'ORDER#' },
  },
}));
```

### Batch Operations
```typescript
// Batch write (up to 25 items)
await dynamoClient.send(new BatchWriteItemCommand({
  RequestItems: {
    [tableName]: items.map(item => ({
      PutRequest: { Item: item },
    })),
  },
}));

// Batch get (up to 100 items)
await dynamoClient.send(new BatchGetItemCommand({
  RequestItems: {
    [tableName]: {
      Keys: keys.map(key => ({ PK: { S: key.pk }, SK: { S: key.sk } })),
    },
  },
}));
```

### Transactions
```typescript
// Transact write (up to 100 items)
await dynamoClient.send(new TransactWriteItemsCommand({
  TransactItems: [
    {
      Put: {
        TableName: tableName,
        Item: orderItem,
        ConditionExpression: 'attribute_not_exists(PK)',
      },
    },
    {
      Update: {
        TableName: tableName,
        Key: { PK: { S: userPK }, SK: { S: userSK } },
        UpdateExpression: 'SET orderCount = orderCount + :inc',
        ExpressionAttributeValues: { ':inc': { N: '1' } },
      },
    },
  ],
}));
```

---

## 9. Error Handling & Resilience

### Error Types
```typescript
export class ServerlessError extends Error {
  constructor(
    public readonly code: string,
    public readonly message: string,
    public readonly statusCode: number,
    public readonly retryable: boolean = false,
    public readonly details?: unknown
  ) {
    super(message);
    this.name = this.constructor.name;
  }
}

export class ThrottlingError extends ServerlessError {
  constructor(message = 'Request throttled') {
    super('THROTTLING_ERROR', message, 429, true);
  }
}

export class TimeoutError extends ServerlessError {
  constructor(message = 'Request timeout') {
    super('TIMEOUT_ERROR', message, 504, true);
  }
}
```

### Retry Strategy
```typescript
// Exponential backoff with jitter
async function retryWithBackoff<T>(
  fn: () => Promise<T>,
  maxRetries: number = 3,
  baseDelay: number = 100
): Promise<T> {
  for (let attempt = 0; attempt <= maxRetries; attempt++) {
    try {
      return await fn();
    } catch (error) {
      if (attempt === maxRetries || !isRetryable(error)) {
        throw error;
      }
      
      const delay = baseDelay * Math.pow(2, attempt);
      const jitter = Math.random() * delay * 0.1;
      await sleep(delay + jitter);
    }
  }
  throw new Error('Max retries exceeded');
}
```

### Idempotency
```typescript
// Idempotency key in DynamoDB
async function processIdempotent(
  idempotencyKey: string,
  fn: () => Promise<any>
): Promise<any> {
  // Check if already processed
  const existing = await getIdempotencyRecord(idempotencyKey);
  if (existing) {
    return existing.result;
  }
  
  // Process and store result
  const result = await fn();
  await storeIdempotencyRecord(idempotencyKey, result);
  
  return result;
}
```

### Dead Letter Queues
```typescript
// DLQ for failed messages
const dlq = new sqs.Queue(this, 'DLQ', {
  retentionPeriod: cdk.Duration.days(14),
});

const queue = new sqs.Queue(this, 'Queue', {
  deadLetterQueue: {
    queue: dlq,
    maxReceiveCount: 3,
  },
});

// Monitor DLQ
new cloudwatch.Alarm(this, 'DLQAlarm', {
  metric: dlq.metricApproximateNumberOfMessagesVisible(),
  threshold: 1,
  evaluationPeriods: 1,
  alarmDescription: 'Messages in DLQ',
});
```

---

## 10. Security Standards

### IAM Least Privilege
```typescript
// Function-specific permissions
const userFunction = new lambda.Function(this, 'UserFunction', {
  // ... config
});

// Grant only necessary permissions
usersTable.grantReadWriteData(userFunction);
bucket.grantRead(userFunction);

// Custom policy for specific actions
userFunction.addToRolePolicy(new iam.PolicyStatement({
  actions: ['ses:SendEmail'],
  resources: ['*'],
  conditions: {
    StringEquals: {
      'ses:FromAddress': 'noreply@example.com',
    },
  },
}));
```

### Secrets Management
```typescript
// Store secrets in Secrets Manager
const dbSecret = new secretsmanager.Secret(this, 'DBSecret', {
  generateSecretString: {
    secretStringTemplate: JSON.stringify({ username: 'admin' }),
    generateStringKey: 'password',
  },
});

// Access in Lambda
const secret = await secretsClient.send(new GetSecretValueCommand({
  SecretId: process.env.SECRET_ARN,
}));
const credentials = JSON.parse(secret.SecretString);
```

### API Authentication
```typescript
// Cognito authorizer
const authorizer = new apigateway.CognitoUserPoolsAuthorizer(this, 'Authorizer', {
  cognitoUserPools: [userPool],
});

resource.addMethod('GET', integration, {
  authorizer,
  authorizationType: apigateway.AuthorizationType.COGNITO,
});

// Lambda authorizer
const lambdaAuthorizer = new apigateway.TokenAuthorizer(this, 'LambdaAuthorizer', {
  handler: authFunction,
  resultsCacheTtl: cdk.Duration.minutes(5),
});
```

---

## 11. Monitoring & Observability

### CloudWatch Metrics
```typescript
// Custom metrics
await cloudwatchClient.send(new PutMetricDataCommand({
  Namespace: 'MyApp',
  MetricData: [{
    MetricName: 'OrderProcessed',
    Value: 1,
    Unit: 'Count',
    Timestamp: new Date(),
    Dimensions: [
      { Name: 'Environment', Value: 'prod' },
      { Name: 'OrderType', Value: 'premium' },
    ],
  }],
}));
```

### X-Ray Tracing
```typescript
import { captureAWSv3Client } from 'aws-xray-sdk-core';

// Instrument AWS SDK clients
const dynamoClient = captureAWSv3Client(new DynamoDBClient({}));

// Custom segments
const segment = AWSXRay.getSegment();
const subsegment = segment.addNewSubsegment('ProcessOrder');
try {
  await processOrder();
  subsegment.close();
} catch (error) {
  subsegment.addError(error);
  subsegment.close();
  throw error;
}
```

### Structured Logging
```typescript
import { Logger } from '@aws-lambda-powertools/logger';

const logger = new Logger({
  serviceName: 'order-service',
  logLevel: 'INFO',
});

// Log with context
logger.info('Order created', {
  orderId: order.id,
  userId: order.userId,
  total: order.total,
});

// Log errors
logger.error('Order processing failed', {
  orderId: order.id,
  error: error.message,
  stack: error.stack,
});
```

### Alarms
```typescript
// Lambda error alarm
new cloudwatch.Alarm(this, 'ErrorAlarm', {
  metric: fn.metricErrors(),
  threshold: 10,
  evaluationPeriods: 2,
  alarmDescription: 'Lambda function errors',
});

// Duration alarm
new cloudwatch.Alarm(this, 'DurationAlarm', {
  metric: fn.metricDuration(),
  threshold: 5000,
  evaluationPeriods: 3,
  statistic: 'p99',
  alarmDescription: 'Lambda p99 duration > 5s',
});
```

---

## 12. Cost Optimization

### Right-Sizing
- Profile Lambda memory usage and adjust
- Use Compute Optimizer recommendations
- Monitor unused provisioned concurrency
- Review DynamoDB capacity modes

### Efficient Data Access
```typescript
// Use projection expressions to fetch only needed attributes
await dynamoClient.send(new GetItemCommand({
  TableName: tableName,
  Key: { PK: { S: pk }, SK: { S: sk } },
  ProjectionExpression: 'userId, email, #name',
  ExpressionAttributeNames: { '#name': 'name' },
}));

// Use sparse indexes to reduce storage
const table = new dynamodb.Table(this, 'Table', {
  // ... config
});

table.addGlobalSecondaryIndex({
  indexName: 'ActiveUsersIndex',
  partitionKey: { name: 'isActive', type: dynamodb.AttributeType.STRING },
  projectionType: dynamodb.ProjectionType.KEYS_ONLY,
});
```

### Caching
```typescript
// API Gateway caching
api.deploymentStage.methodSettings = [{
  resourcePath: '/users',
  httpMethod: 'GET',
  cachingEnabled: true,
  cacheTtl: cdk.Duration.minutes(5),
}];

// Lambda@Edge for CloudFront
const edgeFn = new cloudfront.experimental.EdgeFunction(this, 'EdgeFn', {
  runtime: lambda.Runtime.NODEJS_20_X,
  handler: 'index.handler',
  code: lambda.Code.fromAsset('dist/edge'),
});
```

---

## 13. Testing Standards

### Unit Testing
```typescript
import { mockClient } from 'aws-sdk-client-mock';
import { DynamoDBDocumentClient, GetCommand } from '@aws-sdk/lib-dynamodb';

const dynamoMock = mockClient(DynamoDBDocumentClient);

describe('getUserById', () => {
  beforeEach(() => {
    dynamoMock.reset();
  });

  it('should return user when found', async () => {
    dynamoMock.on(GetCommand).resolves({
      Item: { userId: '123', name: 'John' },
    });

    const user = await getUserById('123');
    expect(user).toEqual({ userId: '123', name: 'John' });
  });
});
```

### Integration Testing
```typescript
// Test with LocalStack or real AWS resources
import { DynamoDBClient } from '@aws-sdk/client-dynamodb';

const client = new DynamoDBClient({
  endpoint: 'http://localhost:4566',  // LocalStack
});

describe('User Repository Integration', () => {
  beforeAll(async () => {
    await createTestTable();
  });

  it('should create and retrieve user', async () => {
    const user = await repository.create({ name: 'John', email: 'john@example.com' });
    const retrieved = await repository.findById(user.id);
    expect(retrieved).toEqual(user);
  });
});
```

### E2E Testing
```typescript
// Test complete workflows
describe('Order Workflow E2E', () => {
  it('should process order end-to-end', async () => {
    // Create order via API
    const response = await fetch(`${API_URL}/orders`, {
      method: 'POST',
      body: JSON.stringify({ items: [...] }),
    });
    const order = await response.json();

    // Wait for async processing
    await waitFor(() => getOrderStatus(order.id) === 'COMPLETED');

    // Verify side effects
    const notifications = await getNotifications(order.userId);
    expect(notifications).toContainEqual(
      expect.objectContaining({ type: 'ORDER_COMPLETED' })
    );
  });
});
```

---

## 14. Code Quality & Formatting

### Required Tools
- **Prettier**: Code formatter (mandatory)
- **ESLint**: Code linter (mandatory)
- **Husky + lint-staged**: Pre-commit hooks

### Configuration
Same as backend standards - see backend-standards.md section 14.

---

## 15. Deployment & CI/CD

### CDK Deployment
```bash
# Synthesize CloudFormation
cdk synth

# Diff changes
cdk diff

# Deploy to environment
cdk deploy --context environment=prod

# Rollback
cdk deploy --rollback
```

### CI/CD Pipeline
```typescript
import { pipelines } from 'aws-cdk-lib';

const pipeline = new pipelines.CodePipeline(this, 'Pipeline', {
  synth: new pipelines.ShellStep('Synth', {
    input: pipelines.CodePipelineSource.gitHub('owner/repo', 'main'),
    commands: [
      'npm ci',
      'npm run build',
      'npm test',
      'npx cdk synth',
    ],
  }),
});

// Add stages
pipeline.addStage(new AppStage(this, 'Dev', { env: devEnv }));
pipeline.addStage(new AppStage(this, 'Prod', { env: prodEnv }), {
  pre: [new pipelines.ManualApprovalStep('PromoteToProd')],
});
```

---

## 16. Definition of Done

A serverless task is complete when:

- [ ] Code meets serverless standards
- [ ] TypeScript strict mode enabled
- [ ] Code formatted with Prettier and passes ESLint
- [ ] Unit tests written and passing (>80% coverage)
- [ ] Integration tests cover AWS service interactions
- [ ] E2E tests cover critical workflows
- [ ] Lambda functions optimized for cold starts
- [ ] IAM permissions follow least privilege
- [ ] Error handling and retries implemented
- [ ] Idempotency ensured for critical operations
- [ ] CloudWatch logs and metrics configured
- [ ] X-Ray tracing enabled
- [ ] Alarms configured for errors and performance
- [ ] Cost optimization reviewed
- [ ] CDK infrastructure code reviewed
- [ ] Deployment tested in staging
- [ ] Documentation updated
- [ ] All CI checks passing

---

**These standards ensure serverless applications are scalable, cost-efficient, resilient, and maintainable on AWS.**
