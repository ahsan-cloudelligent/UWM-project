# CDK Constructs Development Standards

These standards define how we design, build, and maintain reusable AWS CDK constructs to ensure consistency, best practices, and maintainability across infrastructure code.

---

## 1. General Principles

- **Reusability first**: Design constructs for multiple use cases and projects
- **Best practices baked in**: Embed AWS best practices by default
- **Sensible defaults**: Provide good defaults, allow overrides
- **Composability**: Constructs should compose well with other constructs
- **Type safety**: Leverage TypeScript for compile-time safety
- **Documentation**: Clear API documentation and usage examples

---

## 2. Technology Standards

### Core Stack
- **Language**: TypeScript (strict mode enabled)
- **Framework**: AWS CDK v2
- **Testing**: Jest, CDK assertions
- **Documentation**: TSDoc comments, README with examples
- **Publishing**: npm packages for internal/external use

### Package Management
- Use `npm` or `pnpm` for dependency management
- Peer dependencies for CDK packages
- Lock dependency versions
- Regular security audits

---

## 3. Project Structure

Organize constructs library by domain:

```
constructs/
  lib/
    compute/
      api-lambda-function.ts
      api-lambda-function.test.ts
    database/
      single-table-dynamodb.ts
      single-table-dynamodb.test.ts
    api/
      rest-api-with-auth.ts
      rest-api-with-auth.test.ts
    monitoring/
      lambda-alarms.ts
      lambda-alarms.test.ts
    security/
      least-privilege-role.ts
      least-privilege-role.test.ts
  test/
    integ/
      api-lambda-function.integ.ts
  index.ts
  README.md
  package.json
```

**Principles**:
- One construct per file
- Co-locate tests with constructs
- Group by AWS service or domain
- Export all public constructs from index.ts

---

## 4. Construct Design Standards

### Construct Levels

**L1 Constructs (CloudFormation Resources)**:
- Direct mapping to CloudFormation
- Use only when L2/L3 don't exist
- Prefix with `Cfn` (e.g., `CfnBucket`)

**L2 Constructs (AWS Constructs)**:
- Higher-level abstractions with sensible defaults
- Built-in helper methods (e.g., `grantRead()`)
- Use these as building blocks

**L3 Constructs (Patterns/Custom)**:
- **This is what you build**: Opinionated, reusable patterns
- Combine multiple L2 constructs
- Encode organizational best practices
- Examples: `ApiLambdaFunction`, `RestApiWithAuth`, `MonitoredQueue`

### Construct Naming Conventions

- **Classes**: `PascalCase` with descriptive names (e.g., `ApiLambdaFunction`)
- **Props interfaces**: `<ConstructName>Props` (e.g., `ApiLambdaFunctionProps`)
- **Files**: `kebab-case.ts` matching class name (e.g., `api-lambda-function.ts`)
- **Avoid generic names**: Be specific about what the construct does

---

## 5. Props Interface Design

### Props Best Practices

```typescript
import * as cdk from 'aws-cdk-lib';
import * as lambda from 'aws-cdk-lib/aws-lambda';
import { Construct } from 'constructs';

/**
 * Properties for ApiLambdaFunction construct
 */
export interface ApiLambdaFunctionProps {
  /**
   * Path to Lambda function entry point
   * 
   * @example 'src/users/handler.ts'
   */
  readonly entry: string;

  /**
   * Environment variables for the function
   * 
   * @default - No environment variables
   */
  readonly environment?: { [key: string]: string };

  /**
   * Function timeout
   * 
   * @default Duration.seconds(30)
   */
  readonly timeout?: cdk.Duration;

  /**
   * Memory size in MB
   * 
   * @default 1024
   */
  readonly memorySize?: number;

  /**
   * Enable X-Ray tracing
   * 
   * @default true
   */
  readonly tracing?: boolean;

  /**
   * Log retention period
   * 
   * @default RetentionDays.ONE_WEEK
   */
  readonly logRetention?: logs.RetentionDays;
}
```

**Props Guidelines**:
- Use `readonly` for all properties
- Provide JSDoc comments with `@default` and `@example`
- Make required props explicit, optional props have defaults
- Use CDK types (Duration, Size) instead of primitives
- Group related props logically

---

## 6. Construct Implementation Standards

### Basic Construct Pattern

```typescript
import * as cdk from 'aws-cdk-lib';
import * as lambda from 'aws-cdk-lib/aws-lambda';
import * as logs from 'aws-cdk-lib/aws-logs';
import { Construct } from 'constructs';

/**
 * Lambda function optimized for API Gateway integration
 * 
 * Includes best practices:
 * - X-Ray tracing enabled
 * - Structured logging with log retention
 * - Appropriate timeout and memory defaults
 * - Environment variables support
 * 
 * @example
 * ```typescript
 * const userFunction = new ApiLambdaFunction(this, 'UserFunction', {
 *   entry: 'src/users/handler.ts',
 *   environment: {
 *     TABLE_NAME: table.tableName,
 *   },
 * });
 * ```
 */
export class ApiLambdaFunction extends Construct {
  /**
   * The underlying Lambda function
   */
  public readonly function: lambda.Function;

  /**
   * The function's execution role
   */
  public readonly role: iam.Role;

  constructor(scope: Construct, id: string, props: ApiLambdaFunctionProps) {
    super(scope, id);

    // Apply defaults
    const timeout = props.timeout ?? cdk.Duration.seconds(30);
    const memorySize = props.memorySize ?? 1024;
    const tracing = props.tracing ?? true;
    const logRetention = props.logRetention ?? logs.RetentionDays.ONE_WEEK;

    // Create Lambda function with best practices
    this.function = new lambda.Function(this, 'Function', {
      runtime: lambda.Runtime.NODEJS_20_X,
      handler: 'index.handler',
      code: lambda.Code.fromAsset(props.entry),
      timeout,
      memorySize,
      tracing: tracing ? lambda.Tracing.ACTIVE : lambda.Tracing.DISABLED,
      logRetention,
      environment: props.environment,
      architecture: lambda.Architecture.ARM_64, // Graviton2 for cost savings
    });

    this.role = this.function.role!;

    // Add tags
    cdk.Tags.of(this.function).add('ManagedBy', 'CDK');
    cdk.Tags.of(this.function).add('ConstructType', 'ApiLambdaFunction');
  }

  /**
   * Grant this function read access to a DynamoDB table
   */
  public grantReadData(table: dynamodb.ITable): void {
    table.grantReadData(this.function);
  }

  /**
   * Grant this function read/write access to a DynamoDB table
   */
  public grantReadWriteData(table: dynamodb.ITable): void {
    table.grantReadWriteData(this.function);
  }
}
```

### Construct Implementation Guidelines

1. **Extend `Construct` base class**
2. **Expose underlying resources** as public readonly properties
3. **Apply sensible defaults** in constructor
4. **Add helper methods** for common operations
5. **Tag resources** for tracking and cost allocation
6. **Use composition** over inheritance
7. **Validate props** early in constructor

---

## 7. Advanced Construct Patterns

### Composite Constructs

Combine multiple AWS resources into a cohesive pattern:

```typescript
/**
 * REST API with Lambda integration and Cognito authentication
 */
export class RestApiWithAuth extends Construct {
  public readonly api: apigateway.RestApi;
  public readonly authorizer: apigateway.CognitoUserPoolsAuthorizer;
  public readonly userPool: cognito.UserPool;

  constructor(scope: Construct, id: string, props: RestApiWithAuthProps) {
    super(scope, id);

    // Create User Pool
    this.userPool = new cognito.UserPool(this, 'UserPool', {
      userPoolName: props.userPoolName,
      selfSignUpEnabled: props.selfSignUpEnabled ?? false,
      signInAliases: { email: true },
      autoVerify: { email: true },
      passwordPolicy: {
        minLength: 8,
        requireLowercase: true,
        requireUppercase: true,
        requireDigits: true,
        requireSymbols: true,
      },
    });

    // Create API Gateway
    this.api = new apigateway.RestApi(this, 'Api', {
      restApiName: props.apiName,
      deployOptions: {
        stageName: props.stageName ?? 'prod',
        tracingEnabled: true,
        loggingLevel: apigateway.MethodLoggingLevel.INFO,
        metricsEnabled: true,
      },
      defaultCorsPreflightOptions: {
        allowOrigins: props.allowedOrigins ?? apigateway.Cors.ALL_ORIGINS,
        allowMethods: apigateway.Cors.ALL_METHODS,
      },
    });

    // Create Cognito authorizer
    this.authorizer = new apigateway.CognitoUserPoolsAuthorizer(
      this,
      'Authorizer',
      {
        cognitoUserPools: [this.userPool],
        authorizerName: `${props.apiName}-authorizer`,
      }
    );
  }

  /**
   * Add a Lambda-backed resource with authentication
   */
  public addAuthenticatedResource(
    path: string,
    method: string,
    handler: lambda.IFunction
  ): apigateway.Method {
    const resource = this.api.root.resourceForPath(path);
    return resource.addMethod(
      method,
      new apigateway.LambdaIntegration(handler),
      {
        authorizer: this.authorizer,
        authorizationType: apigateway.AuthorizationType.COGNITO,
      }
    );
  }
}
```

### Configurable Constructs

Allow customization while maintaining defaults:

```typescript
export interface MonitoredQueueProps {
  /**
   * Queue name
   */
  readonly queueName: string;

  /**
   * Visibility timeout
   * 
   * @default Duration.seconds(300)
   */
  readonly visibilityTimeout?: cdk.Duration;

  /**
   * Enable dead letter queue
   * 
   * @default true
   */
  readonly enableDlq?: boolean;

  /**
   * Max receive count before moving to DLQ
   * 
   * @default 3
   */
  readonly maxReceiveCount?: number;

  /**
   * Alarm email for DLQ messages
   */
  readonly alarmEmail?: string;
}

export class MonitoredQueue extends Construct {
  public readonly queue: sqs.Queue;
  public readonly dlq?: sqs.Queue;

  constructor(scope: Construct, id: string, props: MonitoredQueueProps) {
    super(scope, id);

    const enableDlq = props.enableDlq ?? true;
    const maxReceiveCount = props.maxReceiveCount ?? 3;

    // Create DLQ if enabled
    if (enableDlq) {
      this.dlq = new sqs.Queue(this, 'DLQ', {
        queueName: `${props.queueName}-dlq`,
        retentionPeriod: cdk.Duration.days(14),
      });

      // Alarm on DLQ messages
      const dlqAlarm = new cloudwatch.Alarm(this, 'DLQAlarm', {
        metric: this.dlq.metricApproximateNumberOfMessagesVisible(),
        threshold: 1,
        evaluationPeriods: 1,
        alarmDescription: `Messages in DLQ for ${props.queueName}`,
      });

      // Send email notification if configured
      if (props.alarmEmail) {
        const topic = new sns.Topic(this, 'AlarmTopic');
        topic.addSubscription(new subscriptions.EmailSubscription(props.alarmEmail));
        dlqAlarm.addAlarmAction(new actions.SnsAction(topic));
      }
    }

    // Create main queue
    this.queue = new sqs.Queue(this, 'Queue', {
      queueName: props.queueName,
      visibilityTimeout: props.visibilityTimeout ?? cdk.Duration.seconds(300),
      deadLetterQueue: this.dlq
        ? {
            queue: this.dlq,
            maxReceiveCount,
          }
        : undefined,
    });
  }
}
```

---

## 8. Testing Standards

### Unit Testing

Test construct synthesis and properties:

```typescript
import { App, Stack } from 'aws-cdk-lib';
import { Template } from 'aws-cdk-lib/assertions';
import { ApiLambdaFunction } from '../lib/compute/api-lambda-function';

describe('ApiLambdaFunction', () => {
  let app: App;
  let stack: Stack;

  beforeEach(() => {
    app = new App();
    stack = new Stack(app, 'TestStack');
  });

  test('creates Lambda function with defaults', () => {
    new ApiLambdaFunction(stack, 'TestFunction', {
      entry: 'src/handler.ts',
    });

    const template = Template.fromStack(stack);

    template.hasResourceProperties('AWS::Lambda::Function', {
      Runtime: 'nodejs20.x',
      Timeout: 30,
      MemorySize: 1024,
      TracingConfig: {
        Mode: 'Active',
      },
    });
  });

  test('applies custom timeout', () => {
    new ApiLambdaFunction(stack, 'TestFunction', {
      entry: 'src/handler.ts',
      timeout: cdk.Duration.seconds(60),
    });

    const template = Template.fromStack(stack);

    template.hasResourceProperties('AWS::Lambda::Function', {
      Timeout: 60,
    });
  });

  test('creates function with environment variables', () => {
    new ApiLambdaFunction(stack, 'TestFunction', {
      entry: 'src/handler.ts',
      environment: {
        TABLE_NAME: 'users-table',
        API_KEY: 'test-key',
      },
    });

    const template = Template.fromStack(stack);

    template.hasResourceProperties('AWS::Lambda::Function', {
      Environment: {
        Variables: {
          TABLE_NAME: 'users-table',
          API_KEY: 'test-key',
        },
      },
    });
  });

  test('exposes function and role', () => {
    const construct = new ApiLambdaFunction(stack, 'TestFunction', {
      entry: 'src/handler.ts',
    });

    expect(construct.function).toBeDefined();
    expect(construct.role).toBeDefined();
  });
});
```

### Integration Testing

Test constructs in realistic scenarios:

```typescript
import { App, Stack } from 'aws-cdk-lib';
import * as dynamodb from 'aws-cdk-lib/aws-dynamodb';
import { IntegTest } from '@aws-cdk/integ-tests-alpha';
import { ApiLambdaFunction } from '../lib/compute/api-lambda-function';

const app = new App();
const stack = new Stack(app, 'IntegTestStack');

// Create DynamoDB table
const table = new dynamodb.Table(stack, 'Table', {
  partitionKey: { name: 'PK', type: dynamodb.AttributeType.STRING },
});

// Create Lambda function
const fn = new ApiLambdaFunction(stack, 'Function', {
  entry: 'test/integ/handlers/test-handler.ts',
  environment: {
    TABLE_NAME: table.tableName,
  },
});

// Grant permissions
fn.grantReadWriteData(table);

// Run integration test
new IntegTest(app, 'ApiLambdaFunctionIntegTest', {
  testCases: [stack],
});
```

### Snapshot Testing

Capture CloudFormation template snapshots:

```typescript
test('matches snapshot', () => {
  const app = new App();
  const stack = new Stack(app, 'TestStack');

  new ApiLambdaFunction(stack, 'TestFunction', {
    entry: 'src/handler.ts',
  });

  const template = Template.fromStack(stack);
  expect(template.toJSON()).toMatchSnapshot();
});
```

---

## 9. Documentation Standards

### TSDoc Comments

```typescript
/**
 * Lambda function optimized for API Gateway integration
 * 
 * This construct creates a Lambda function with best practices baked in:
 * - X-Ray tracing enabled by default
 * - Structured logging with configurable retention
 * - ARM64 architecture for cost savings
 * - Appropriate timeout and memory defaults
 * 
 * @example
 * Basic usage:
 * ```typescript
 * const userFunction = new ApiLambdaFunction(this, 'UserFunction', {
 *   entry: 'src/users/handler.ts',
 * });
 * ```
 * 
 * @example
 * With custom configuration:
 * ```typescript
 * const userFunction = new ApiLambdaFunction(this, 'UserFunction', {
 *   entry: 'src/users/handler.ts',
 *   timeout: cdk.Duration.minutes(1),
 *   memorySize: 2048,
 *   environment: {
 *     TABLE_NAME: table.tableName,
 *   },
 * });
 * ```
 */
export class ApiLambdaFunction extends Construct {
  /**
   * The underlying Lambda function
   * 
   * Use this to access the function's ARN, name, or add event sources.
   */
  public readonly function: lambda.Function;

  /**
   * Grant this function read access to a DynamoDB table
   * 
   * @param table - The DynamoDB table to grant access to
   * 
   * @example
   * ```typescript
   * const fn = new ApiLambdaFunction(this, 'Function', { ... });
   * fn.grantReadData(table);
   * ```
   */
  public grantReadData(table: dynamodb.ITable): void {
    table.grantReadData(this.function);
  }
}
```

### README Documentation

Every construct library should have a comprehensive README:

```markdown
# Company CDK Constructs

Reusable AWS CDK constructs following organizational best practices.

## Installation

```bash
npm install @company/cdk-constructs
```

## Constructs

### ApiLambdaFunction

Lambda function optimized for API Gateway integration.

**Features:**
- X-Ray tracing enabled
- Structured logging with retention
- ARM64 architecture
- Sensible defaults for timeout and memory

**Usage:**

```typescript
import { ApiLambdaFunction } from '@company/cdk-constructs';

const userFunction = new ApiLambdaFunction(this, 'UserFunction', {
  entry: 'src/users/handler.ts',
  environment: {
    TABLE_NAME: table.tableName,
  },
});
```

**API Reference:** [Full documentation](./docs/api-lambda-function.md)

---

### RestApiWithAuth

REST API with Cognito authentication.

**Features:**
- Cognito User Pool with secure defaults
- API Gateway with CORS
- Built-in authorizer
- Helper methods for adding authenticated routes

**Usage:**

```typescript
import { RestApiWithAuth } from '@company/cdk-constructs';

const api = new RestApiWithAuth(this, 'Api', {
  apiName: 'my-api',
  userPoolName: 'my-users',
});

api.addAuthenticatedResource('/users', 'GET', userFunction);
```

## Contributing

See [CONTRIBUTING.md](./CONTRIBUTING.md)

## License

MIT
```

---

## 10. Versioning & Publishing

### Semantic Versioning

Follow semver for construct libraries:

- **Major (1.0.0 → 2.0.0)**: Breaking changes to construct API
- **Minor (1.0.0 → 1.1.0)**: New constructs or features, backward compatible
- **Patch (1.0.0 → 1.0.1)**: Bug fixes, no API changes

### Publishing Checklist

- [ ] All tests passing
- [ ] Documentation updated
- [ ] CHANGELOG.md updated
- [ ] Version bumped in package.json
- [ ] Git tag created
- [ ] Published to npm registry
- [ ] Release notes created

### Package.json Configuration

```json
{
  "name": "@company/cdk-constructs",
  "version": "1.0.0",
  "description": "Reusable AWS CDK constructs",
  "main": "lib/index.js",
  "types": "lib/index.d.ts",
  "scripts": {
    "build": "tsc",
    "test": "jest",
    "lint": "eslint . --ext .ts",
    "format": "prettier --write .",
    "docs": "typedoc"
  },
  "peerDependencies": {
    "aws-cdk-lib": "^2.0.0",
    "constructs": "^10.0.0"
  },
  "devDependencies": {
    "aws-cdk-lib": "^2.0.0",
    "constructs": "^10.0.0",
    "@types/jest": "^29.0.0",
    "jest": "^29.0.0",
    "typescript": "^5.0.0"
  },
  "keywords": ["aws", "cdk", "constructs", "infrastructure"],
  "license": "MIT"
}
```

---

## 11. Best Practices

### Do's ✅

- **Provide sensible defaults** for all optional props
- **Expose underlying resources** as public properties
- **Add helper methods** for common operations
- **Use TypeScript strict mode** for type safety
- **Write comprehensive tests** (unit + integration)
- **Document with TSDoc** and examples
- **Tag resources** for tracking
- **Follow AWS best practices** by default
- **Make constructs composable** with other constructs
- **Version constructs** semantically

### Don'ts ❌

- **Don't hardcode values** - use props or environment
- **Don't create overly complex constructs** - keep focused
- **Don't skip validation** - validate props early
- **Don't expose implementation details** - hide internals
- **Don't break backward compatibility** without major version bump
- **Don't skip documentation** - always document public API
- **Don't ignore testing** - test every construct
- **Don't use L1 constructs** unless necessary

---

## 12. Security Standards

### IAM Least Privilege

```typescript
export class SecureLambdaFunction extends Construct {
  public readonly function: lambda.Function;

  constructor(scope: Construct, id: string, props: SecureLambdaFunctionProps) {
    super(scope, id);

    // Create function with minimal permissions
    this.function = new lambda.Function(this, 'Function', {
      // ... config
    });

    // Grant only specific permissions
    if (props.tableName) {
      const table = dynamodb.Table.fromTableName(
        this,
        'Table',
        props.tableName
      );
      table.grantReadData(this.function);
    }

    // Deny dangerous actions
    this.function.addToRolePolicy(
      new iam.PolicyStatement({
        effect: iam.Effect.DENY,
        actions: ['iam:*', 'sts:AssumeRole'],
        resources: ['*'],
      })
    );
  }
}
```

### Encryption by Default

```typescript
export class SecureBucket extends Construct {
  public readonly bucket: s3.Bucket;

  constructor(scope: Construct, id: string, props: SecureBucketProps) {
    super(scope, id);

    this.bucket = new s3.Bucket(this, 'Bucket', {
      bucketName: props.bucketName,
      encryption: s3.BucketEncryption.S3_MANAGED, // Encryption by default
      blockPublicAccess: s3.BlockPublicAccess.BLOCK_ALL, // Block public access
      enforceSSL: true, // Require HTTPS
      versioned: true, // Enable versioning
      lifecycleRules: [
        {
          transitions: [
            {
              storageClass: s3.StorageClass.INFREQUENT_ACCESS,
              transitionAfter: cdk.Duration.days(30),
            },
          ],
        },
      ],
    });
  }
}
```

---

## 13. Performance Optimization

### Minimize Synthesis Time

- Avoid expensive operations in constructor
- Cache computed values
- Use lazy initialization when possible

### Optimize CloudFormation Templates

- Use CDK aspects for cross-cutting concerns
- Avoid creating unnecessary resources
- Leverage CloudFormation conditions

---

## 14. Code Quality & Formatting

### Required Tools
- **Prettier**: Code formatter (mandatory)
- **ESLint**: Code linter (mandatory)
- **Husky + lint-staged**: Pre-commit hooks

### Configuration
Same as backend standards - see backend-standards.md section 14.

---

## 15. Definition of Done

A CDK construct is complete when:

- [ ] Construct follows design standards
- [ ] TypeScript strict mode enabled
- [ ] Code formatted with Prettier and passes ESLint
- [ ] Props interface documented with TSDoc
- [ ] Sensible defaults provided
- [ ] Unit tests written and passing (>90% coverage)
- [ ] Integration tests validate real-world usage
- [ ] Snapshot tests capture CloudFormation template
- [ ] Public API documented with examples
- [ ] README updated with usage examples
- [ ] Helper methods for common operations
- [ ] Resources tagged appropriately
- [ ] Security best practices followed
- [ ] Versioned and published (if library)
- [ ] Code review approved
- [ ] All CI checks passing

---

**These standards ensure CDK constructs are reusable, maintainable, and follow AWS best practices.**
