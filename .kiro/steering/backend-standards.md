# Backend Development Standards

These standards define how we design, build, and maintain backend services to ensure consistency, quality, security, and long-term maintainability.

---

## 1. General Principles

- **Clarity over cleverness**: Write code that is easy to read and understand
- **Explicit over implicit**: Make behavior obvious, avoid magic
- **Fail fast and loud**: Errors should be visible and actionable
- **Security by default**: Never compromise on security
- **Scalable from day one**: Design for growth
- **Testable architecture**: Code should be easy to test

---

## 2. Technology Standards

### Core Stack
- **Runtime**: Node.js (LTS version)
- **Language**: TypeScript (strict mode enabled)
- **Framework**: Express.js, Fastify, NestJS
- **Infrastructure**: AWS CDK (TypeScript), Docker, Kubernetes
- **Database**: RDS (PostgreSQL, MySQL), MongoDB, Redis
- **Caching**: ElastiCache (Redis), Memcached
- **Message Queues**: SQS, RabbitMQ, Kafka

### Package Management
- Use `npm` or `pnpm` for dependency management
- Lock dependency versions (`package-lock.json` or `pnpm-lock.yaml`)
- Regular security audits (`npm audit`, `pnpm audit`)
- Minimize dependencies - evaluate necessity and maintenance status

---

## 3. Project Structure

Organize by feature/domain, following clean architecture principles:

```
src/
  features/
    users/
      domain/           # Business logic, entities
      application/      # Use cases, services
      infrastructure/   # DB, external APIs
      presentation/     # Lambda handlers, API routes
      users.test.ts
    orders/
      domain/
      application/
      infrastructure/
      presentation/
  shared/
    utils/
    types/
    middleware/
  config/
  tests/
```

**Principles**:
- Keep related code together
- Avoid deep nesting (max 3-4 levels)
- Clear separation of concerns (domain, application, infrastructure, presentation)
- Shared code only when truly reusable

---

## 4. Naming Conventions

- **Files**: `kebab-case.ts` (e.g., `user-service.ts`)
- **Classes**: `PascalCase` (e.g., `UserService`)
- **Functions/Variables**: `camelCase` (e.g., `getUserById`)
- **Constants**: `UPPER_SNAKE_CASE` (e.g., `MAX_RETRY_ATTEMPTS`)
- **Interfaces**: `PascalCase` with descriptive names (e.g., `UserRepository`, not `IUserRepository`)
- **Types**: `PascalCase` (e.g., `CreateUserRequest`)
- **Enums**: `PascalCase` for enum, `UPPER_SNAKE_CASE` for values

**Avoid**:
- Abbreviations unless universally understood (e.g., `id`, `url`)
- Generic names like `data`, `info`, `manager`, `handler` without context

---

## 5. TypeScript Standards

### Strict Mode (Required)
```json
{
  "compilerOptions": {
    "strict": true,
    "noImplicitAny": true,
    "strictNullChecks": true,
    "strictFunctionTypes": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noImplicitReturns": true,
    "noFallthroughCasesInSwitch": true
  }
}
```

### Type Safety
- Avoid `any` - use `unknown` if type is truly unknown
- Define explicit return types for functions
- Use discriminated unions for complex types
- Leverage type guards for runtime checks
- Use `readonly` for immutable data

### Example:
```typescript
// Good
interface User {
  readonly id: string;
  name: string;
  email: string;
  role: UserRole;
}

type UserRole = 'admin' | 'user' | 'guest';

function getUserById(id: string): Promise<User | null> {
  // Implementation
}

// Bad
function getUser(id: any): Promise<any> {
  // Implementation
}
```

---

## 6. Application Architecture Standards

### Service Design
- **Single responsibility**: Each service has one clear purpose
- **Stateless when possible**: Store state in databases, not in-memory
- **Graceful shutdown**: Handle SIGTERM for clean shutdowns
- **Health checks**: Implement `/health` and `/ready` endpoints
- **Configuration**: Use environment variables, never hardcode

### Express.js Application Pattern
```typescript
import express, { Request, Response, NextFunction } from 'express';
import { logger } from '@/shared/utils/logger';
import { errorHandler } from '@/shared/middleware/error-handler';

const app = express();

// Middleware
app.use(express.json());
app.use(logger.middleware);

// Routes
app.get('/users/:id', async (req: Request, res: Response, next: NextFunction) => {
  try {
    const user = await userService.getUserById(req.params.id);
    if (!user) {
      return res.status(404).json({ error: 'User not found' });
    }
    res.json({ success: true, data: user });
  } catch (error) {
    next(error);
  }
});

// Error handling
app.use(errorHandler);

// Health check
app.get('/health', (req, res) => {
  res.json({ status: 'healthy', timestamp: new Date().toISOString() });
});

export default app;
```

### Containerization Standards
- Use multi-stage Docker builds
- Minimize image size (Alpine base images)
- Run as non-root user
- Use .dockerignore to exclude unnecessary files
- Pin base image versions
- Scan images for vulnerabilities

### Message Queue Integration
- Use SQS for AWS-native queuing
- Use RabbitMQ for complex routing
- Use Kafka for high-throughput streaming
- Implement retry logic with exponential backoff
- Use dead letter queues for failed messages

---

## 7. AWS CDK Standards

### CDK Project Structure
```
infrastructure/
  lib/
    stacks/
      api-stack.ts
      database-stack.ts
      monitoring-stack.ts
    constructs/          # Handled by infrastructure agent
  bin/
    app.ts
  cdk.json
```

### Stack Design Principles
- **One stack per logical boundary** (API, Database, Monitoring)
- **Pass resources via props**, not by importing
- **Use CDK context** for environment-specific config
- **Tag all resources** for cost tracking and organization
- **Enable removal policies** appropriately (RETAIN for prod data)

### Example Stack:
```typescript
import * as cdk from 'aws-cdk-lib';
import * as lambda from 'aws-cdk-lib/aws-lambda';
import * as apigateway from 'aws-cdk-lib/aws-apigateway';
import { Construct } from 'constructs';

export interface ApiStackProps extends cdk.StackProps {
  environment: string;
}

export class ApiStack extends cdk.Stack {
  public readonly api: apigateway.RestApi;

  constructor(scope: Construct, id: string, props: ApiStackProps) {
    super(scope, id, props);

    // Lambda function
    const userHandler = new lambda.Function(this, 'UserHandler', {
      runtime: lambda.Runtime.NODEJS_20_X,
      handler: 'index.handler',
      code: lambda.Code.fromAsset('dist/users'),
      environment: {
        ENVIRONMENT: props.environment,
      },
      timeout: cdk.Duration.seconds(30),
      memorySize: 512,
    });

    // API Gateway
    this.api = new apigateway.RestApi(this, 'UserApi', {
      restApiName: `user-api-${props.environment}`,
      deployOptions: {
        stageName: props.environment,
        tracingEnabled: true,
        loggingLevel: apigateway.MethodLoggingLevel.INFO,
      },
    });

    const users = this.api.root.addResource('users');
    users.addMethod('GET', new apigateway.LambdaIntegration(userHandler));

    // Tags
    cdk.Tags.of(this).add('Environment', props.environment);
    cdk.Tags.of(this).add('Service', 'UserAPI');
  }
}
```

### CDK Best Practices
- Use L2 constructs (high-level) over L1 (low-level CloudFormation)
- Leverage CDK aspects for cross-cutting concerns
- Use CDK Pipelines for CI/CD
- Synthesize and diff before deploying
- Never hardcode account IDs or regions - use `cdk.Stack` properties

---

## 8. Database Standards

### DynamoDB
- **Single-table design** for related entities
- **Composite keys** for flexible queries (PK, SK)
- **GSI/LSI** for additional access patterns
- **Provisioned vs On-Demand**: Choose based on traffic patterns
- **TTL** for automatic data expiration
- **Point-in-time recovery** enabled for production

### RDS/Aurora
- **Connection pooling**: Reuse connections across requests
- **Secrets Manager**: Store credentials securely
- **Read replicas**: Scale read-heavy workloads
- **Automated backups**: Enabled with appropriate retention
- **Parameter groups**: Optimize for workload

### Data Access Layer
```typescript
// Repository pattern
export interface UserRepository {
  findById(id: string): Promise<User | null>;
  create(user: CreateUserInput): Promise<User>;
  update(id: string, updates: Partial<User>): Promise<User>;
  delete(id: string): Promise<void>;
}

// PostgreSQL implementation with connection pooling
export class PostgresUserRepository implements UserRepository {
  constructor(private readonly pool: Pool) {}

  async findById(id: string): Promise<User | null> {
    const result = await this.pool.query(
      'SELECT * FROM users WHERE id = $1',
      [id]
    );
    return result.rows[0] || null;
  }
  
  // Other methods...
}
```

---

## 9. API Design Standards

### RESTful Conventions
- **Resources**: Use nouns, not verbs (`/users`, not `/getUsers`)
- **HTTP Methods**: GET (read), POST (create), PUT (replace), PATCH (update), DELETE (remove)
- **Status Codes**: Use appropriate codes (200, 201, 400, 401, 403, 404, 500)
- **Versioning**: Use URL versioning (`/v1/users`) or header-based
- **Pagination**: Use `limit` and `cursor` for large datasets
- **Filtering**: Use query parameters (`?status=active&role=admin`)

### Request/Response Format
```typescript
// Request
interface CreateUserRequest {
  name: string;
  email: string;
  role: UserRole;
}

// Response
interface ApiResponse<T> {
  success: boolean;
  data?: T;
  error?: {
    code: string;
    message: string;
    details?: unknown;
  };
  metadata?: {
    timestamp: string;
    requestId: string;
  };
}

// Success response
{
  "success": true,
  "data": {
    "id": "user-123",
    "name": "John Doe",
    "email": "john@example.com"
  },
  "metadata": {
    "timestamp": "2026-02-19T02:34:27Z",
    "requestId": "abc-123"
  }
}

// Error response
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid email format",
    "details": { "field": "email" }
  },
  "metadata": {
    "timestamp": "2026-02-19T02:34:27Z",
    "requestId": "abc-123"
  }
}
```

### Input Validation
- Validate at API Gateway level (request models)
- Validate in application layer (business rules)
- Use validation libraries (Zod, Joi, class-validator)
- Return clear, actionable error messages

```typescript
import { z } from 'zod';

const CreateUserSchema = z.object({
  name: z.string().min(2).max(100),
  email: z.string().email(),
  role: z.enum(['admin', 'user', 'guest']),
});

export function validateCreateUser(input: unknown): CreateUserRequest {
  return CreateUserSchema.parse(input);
}
```

---

## 10. Error Handling

### Error Types
```typescript
export class AppError extends Error {
  constructor(
    public readonly code: string,
    public readonly message: string,
    public readonly statusCode: number,
    public readonly details?: unknown
  ) {
    super(message);
    this.name = this.constructor.name;
    Error.captureStackTrace(this, this.constructor);
  }
}

export class ValidationError extends AppError {
  constructor(message: string, details?: unknown) {
    super('VALIDATION_ERROR', message, 400, details);
  }
}

export class NotFoundError extends AppError {
  constructor(resource: string, id: string) {
    super('NOT_FOUND', `${resource} with id ${id} not found`, 404);
  }
}

export class UnauthorizedError extends AppError {
  constructor(message = 'Unauthorized') {
    super('UNAUTHORIZED', message, 401);
  }
}
```

### Error Handler Middleware
```typescript
export function errorHandler(error: unknown): APIGatewayProxyResult {
  logger.error('Error occurred', { error });

  if (error instanceof AppError) {
    return {
      statusCode: error.statusCode,
      body: JSON.stringify({
        success: false,
        error: {
          code: error.code,
          message: error.message,
          details: error.details,
        },
      }),
    };
  }

  // Unknown error - don't leak details
  return {
    statusCode: 500,
    body: JSON.stringify({
      success: false,
      error: {
        code: 'INTERNAL_ERROR',
        message: 'An unexpected error occurred',
      },
    }),
  };
}
```

---

## 11. Security Standards

### Authentication & Authorization
- **JWT tokens**: Use for stateless authentication
- **API keys**: For service-to-service communication
- **IAM roles**: For AWS service access
- **Cognito**: For user management and authentication
- **Token expiration**: Short-lived access tokens, refresh tokens for renewal

### Input Sanitization
- Validate all inputs at boundaries
- Sanitize user-provided data
- Use parameterized queries (prevent SQL injection)
- Escape output when rendering

### Secrets Management
- **AWS Secrets Manager**: For database credentials, API keys
- **Parameter Store**: For configuration values
- **Environment variables**: For non-sensitive config only
- **Never commit secrets** to version control

### HTTPS & Encryption
- Enforce HTTPS for all API endpoints
- Encrypt data at rest (DynamoDB, S3, RDS)
- Encrypt data in transit (TLS 1.2+)
- Use AWS KMS for encryption key management

### CORS Configuration
```typescript
const corsHeaders = {
  'Access-Control-Allow-Origin': process.env.ALLOWED_ORIGINS || '*',
  'Access-Control-Allow-Methods': 'GET,POST,PUT,DELETE,OPTIONS',
  'Access-Control-Allow-Headers': 'Content-Type,Authorization',
  'Access-Control-Max-Age': '86400',
};
```

---

## 12. Logging & Monitoring

### Structured Logging
```typescript
import { Logger } from '@aws-lambda-powertools/logger';

const logger = new Logger({
  serviceName: 'user-service',
  logLevel: process.env.LOG_LEVEL || 'INFO',
});

// Usage
logger.info('User created', { userId: user.id, email: user.email });
logger.error('Failed to create user', { error, input });
logger.warn('Rate limit approaching', { userId, requestCount });
```

### Log Levels
- **ERROR**: Failures requiring immediate attention
- **WARN**: Potential issues, degraded functionality
- **INFO**: Important business events (user created, order placed)
- **DEBUG**: Detailed diagnostic information (dev/staging only)

### Monitoring & Alerting
- **CloudWatch Metrics**: Lambda duration, errors, throttles
- **CloudWatch Alarms**: Alert on error rates, latency spikes
- **X-Ray Tracing**: Distributed tracing for request flows
- **CloudWatch Dashboards**: Visualize key metrics

### Key Metrics to Track
- Request count and error rate
- Lambda duration (p50, p95, p99)
- Cold start frequency
- DynamoDB throttles and consumed capacity
- API Gateway 4xx/5xx errors

---

## 13. Testing Standards

### Test Pyramid
- **Unit Tests (70%)**: Test business logic in isolation
- **Integration Tests (20%)**: Test component interactions
- **E2E Tests (10%)**: Test complete user flows

### Unit Testing
```typescript
import { describe, it, expect, beforeEach } from 'vitest';
import { UserService } from './user-service';
import { MockUserRepository } from './user-repository.mock';

describe('UserService', () => {
  let service: UserService;
  let repository: MockUserRepository;

  beforeEach(() => {
    repository = new MockUserRepository();
    service = new UserService(repository);
  });

  it('should create user with valid input', async () => {
    const input = { name: 'John', email: 'john@example.com', role: 'user' };
    const user = await service.createUser(input);

    expect(user.id).toBeDefined();
    expect(user.name).toBe(input.name);
    expect(repository.create).toHaveBeenCalledWith(input);
  });

  it('should throw ValidationError for invalid email', async () => {
    const input = { name: 'John', email: 'invalid', role: 'user' };
    
    await expect(service.createUser(input)).rejects.toThrow(ValidationError);
  });
});
```

### Integration Testing
- Test API endpoints with real HTTP requests
- Test database operations with test database
- Test message queue integration

### Mocking Strategy
- Mock external services (third-party APIs)
- Use real implementations for internal services when possible
- Mock AWS services using `aws-sdk-mock` or `localstack`

### Test Coverage
- Minimum 80% code coverage for business logic
- 100% coverage for critical paths (authentication, payments)
- Focus on meaningful tests, not just coverage numbers

---

## 14. Code Quality & Formatting (Mandatory)

### Required Tools
- **Prettier**: Code formatter (mandatory)
- **ESLint**: Code linter (mandatory)
- **Husky + lint-staged**: Pre-commit hooks (strict enforcement)

### Configuration Files

**.prettierrc**
```json
{
  "semi": true,
  "trailingComma": "es5",
  "singleQuote": true,
  "printWidth": 100,
  "tabWidth": 2
}
```

**.eslintrc.json**
```json
{
  "extends": [
    "eslint:recommended",
    "@typescript-eslint/recommended",
    "prettier"
  ],
  "parser": "@typescript-eslint/parser",
  "plugins": ["@typescript-eslint"],
  "rules": {
    "no-unused-vars": "off",
    "@typescript-eslint/no-unused-vars": ["error", { "argsIgnorePattern": "^_" }],
    "no-console": "warn",
    "@typescript-eslint/explicit-function-return-type": "warn",
    "@typescript-eslint/no-explicit-any": "error"
  }
}
```

**package.json**
```json
{
  "scripts": {
    "format": "prettier --write .",
    "format:check": "prettier --check .",
    "lint": "eslint . --fix",
    "lint:check": "eslint .",
    "prepare": "husky install"
  },
  "lint-staged": {
    "*.{ts,js}": ["prettier --write", "eslint --fix"],
    "*.{json,md}": ["prettier --write"]
  }
}
```

### CI/CD Requirements
- Pipeline must include `npm run format:check` and `npm run lint:check`
- Pipeline must fail if checks fail
- No exceptions - all code must pass before merge

---

## 15. Code Comments Standards

### When to Comment
- **Complex business logic**: Explain the "why", not the "what"
- **Non-obvious decisions**: Document trade-offs and rationale
- **Public APIs**: Document parameters, return values, errors
- **Workarounds**: Explain temporary solutions and link to tickets

### When NOT to Comment
- **Self-explanatory code**: Good naming eliminates most comments
- **Obvious operations**: Don't state what the code clearly does
- **Outdated comments**: Remove or update, never leave stale comments

### Good Comments
```typescript
// Calculate discount based on user tier and order value
// Business rule: VIP users get 20% off orders over $100
function calculateDiscount(user: User, orderValue: number): number {
  if (user.tier === 'VIP' && orderValue > 100) {
    return orderValue * 0.2;
  }
  return 0;
}

/**
 * Processes payment using Stripe API
 * 
 * @param orderId - Unique order identifier
 * @param amount - Payment amount in cents
 * @returns Payment confirmation with transaction ID
 * @throws PaymentError if payment fails
 */
async function processPayment(orderId: string, amount: number): Promise<PaymentResult> {
  // Implementation
}

// TODO: Refactor to use new payment provider (TICKET-123)
// Temporary workaround until migration is complete
```

### Bad Comments
```typescript
// Bad: States the obvious
// Increment counter by 1
counter++;

// Bad: Redundant with function name
// Get user by ID
function getUserById(id: string) { }

// Bad: Outdated comment
// Returns user email (actually returns full user object now)
function getUser(id: string): User { }
```

### JSDoc for Public APIs
```typescript
/**
 * Creates a new user in the system
 * 
 * @param input - User creation data
 * @param input.name - Full name of the user
 * @param input.email - Valid email address
 * @param input.role - User role (admin, user, guest)
 * @returns Created user with generated ID
 * @throws ValidationError if input is invalid
 * @throws ConflictError if email already exists
 * 
 * @example
 * const user = await createUser({
 *   name: 'John Doe',
 *   email: 'john@example.com',
 *   role: 'user'
 * });
 */
export async function createUser(input: CreateUserInput): Promise<User> {
  // Implementation
}
```

---

## 16. Performance Optimization

### Application Optimization
- **Connection pooling**: Reuse database connections across requests
- **Async operations**: Use Promise.all for parallel operations
- **Optimize queries**: Use indexes, avoid N+1 queries
- **Caching**: Use Redis/Memcached for hot data
- **Response compression**: Enable gzip compression
- **Rate limiting**: Protect against abuse

### Database Optimization
- **Batch operations**: Group database operations when possible
- **Connection pooling**: Reuse database connections
- **Indexes**: Create appropriate indexes for query patterns
- **Query optimization**: Use EXPLAIN to analyze queries
- **Caching**: Use ElastiCache for frequently accessed data

---

## 17. Deployment & CI/CD

### Deployment Strategy
- **Blue-Green**: Zero-downtime deployments
- **Canary**: Gradual rollout with monitoring
- **Rollback**: Automated rollback on errors

### CI/CD Pipeline Stages
1. **Build**: Compile TypeScript, run linters
2. **Test**: Unit, integration, E2E tests
3. **Security Scan**: Dependency vulnerabilities, SAST
4. **Deploy to Staging**: Automated deployment
5. **Smoke Tests**: Verify critical paths
6. **Deploy to Production**: Manual approval or automated
7. **Monitor**: Track metrics and errors

### Environment Management
- **Dev**: Frequent deployments, relaxed policies
- **Staging**: Production-like, pre-release testing
- **Production**: Strict policies, monitoring, alarms

---

## 18. Definition of Done

A backend task is complete when:

- [ ] Code meets these standards
- [ ] TypeScript strict mode enabled, no `any` types
- [ ] Code formatted with Prettier and passes ESLint
- [ ] Unit tests written and passing (>80% coverage)
- [ ] Integration tests cover key flows
- [ ] API contracts documented (OpenAPI/Swagger)
- [ ] Error handling implemented
- [ ] Logging and monitoring configured
- [ ] Security best practices followed
- [ ] CDK infrastructure code reviewed
- [ ] Deployment tested in staging
- [ ] Documentation updated
- [ ] Code review approved
- [ ] All CI checks passing

---

**These standards are living guidelines and should evolve with the product and team.**
