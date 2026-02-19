# CDK Constructs Engineering Skills & Competencies

## Overview

The CDK constructs developer is a senior infrastructure engineer with deep expertise in AWS CDK, infrastructure patterns, and creating reusable, well-tested constructs that encode organizational best practices.

---

## 1. Core Technical Skills

### 1.1 AWS CDK Mastery

**Construct Hierarchy:**
- **L1 Constructs**: CloudFormation resources (CfnXxx)
- **L2 Constructs**: AWS constructs with helper methods
- **L3 Constructs**: Custom patterns combining multiple resources
- Understanding when to use each level
- Composition patterns and construct trees

**CDK Core Concepts:**
- Construct scope and lifecycle
- Props and configuration patterns
- Cross-stack references and dependencies
- CDK context and environment variables
- Asset bundling and deployment
- CDK aspects for cross-cutting concerns
- Custom resources for advanced scenarios

**Stack Design:**
- Single-stack vs multi-stack architecture
- Nested stacks for modularity
- Stack dependencies and ordering
- Environment-specific configuration
- Resource tagging strategies
- Removal policies (RETAIN, DESTROY, SNAPSHOT)

**CDK Synthesis:**
- CloudFormation template generation
- Asset staging and bundling
- Metadata and annotations
- Template validation
- Diff and change sets

### 1.2 TypeScript for Infrastructure

**Advanced TypeScript:**
- Generics for reusable construct patterns
- Type guards for runtime validation
- Utility types (Partial, Pick, Omit, Required)
- Conditional types for flexible APIs
- Mapped types for configuration
- Strict mode enforcement

**Design Patterns:**
- Builder pattern for complex constructs
- Factory pattern for construct creation
- Strategy pattern for configurable behavior
- Decorator pattern for extending constructs
- Composite pattern for nested resources

**Type Safety:**
- Explicit return types for all methods
- Readonly properties for immutability
- Discriminated unions for configuration
- Type-safe props interfaces
- Avoid `any` - use `unknown` when needed

### 1.3 AWS Services Deep Dive

**Compute:**
- Lambda (functions, layers, versions, aliases)
- ECS/Fargate (tasks, services, clusters)
- EC2 (instances, auto-scaling groups)
- Batch (job definitions, compute environments)

**API & Integration:**
- API Gateway (REST, HTTP, WebSocket)
- AppSync (GraphQL APIs)
- EventBridge (event buses, rules, schemas)
- Step Functions (state machines, workflows)

**Storage:**
- S3 (buckets, policies, lifecycle rules)
- EFS (file systems, access points)
- EBS (volumes, snapshots)

**Database:**
- DynamoDB (tables, GSI/LSI, streams)
- RDS (instances, clusters, proxies)
- Aurora (serverless, global databases)
- ElastiCache (Redis, Memcached)

**Messaging:**
- SQS (standard, FIFO queues)
- SNS (topics, subscriptions, filtering)
- Kinesis (streams, firehose, analytics)

**Security:**
- IAM (roles, policies, permission boundaries)
- Secrets Manager (secrets, rotation)
- KMS (keys, grants, aliases)
- Cognito (user pools, identity pools)
- WAF (web ACLs, rules)

**Monitoring:**
- CloudWatch (logs, metrics, alarms, dashboards)
- X-Ray (tracing, service maps)
- CloudTrail (audit logging)

**Networking:**
- VPC (subnets, route tables, NAT gateways)
- Security Groups and NACLs
- VPC endpoints and PrivateLink
- Load Balancers (ALB, NLB)
- CloudFront (distributions, origins)

### 1.4 Infrastructure Patterns

**High Availability:**
- Multi-AZ deployments
- Auto-scaling policies
- Health checks and failover
- Load balancing strategies
- Disaster recovery planning

**Security Patterns:**
- Least privilege IAM
- Encryption at rest and in transit
- Secrets management
- Network isolation (VPC, security groups)
- Compliance and auditing

**Cost Optimization:**
- Right-sizing resources
- Lifecycle policies for storage
- Reserved capacity vs on-demand
- Spot instances for batch workloads
- Resource tagging for cost allocation

**Observability:**
- Structured logging patterns
- Distributed tracing
- Custom metrics and dashboards
- Alerting and escalation
- Log aggregation and analysis

---

## 2. Construct Design Skills

### 2.1 API Design

**Props Interface Design:**
- Clear, descriptive property names
- Sensible defaults for optional properties
- Use CDK types (Duration, Size) over primitives
- Group related properties logically
- Validate props in constructor

**Public API:**
- Expose underlying resources as readonly properties
- Provide helper methods for common operations
- Hide implementation details
- Maintain backward compatibility
- Version breaking changes appropriately

**Composability:**
- Constructs should work well together
- Avoid tight coupling between constructs
- Use interfaces for dependencies
- Support dependency injection
- Enable testing through mocking

### 2.2 Best Practices Encoding

**Bake in AWS Best Practices:**
- Enable encryption by default
- Configure appropriate timeouts
- Set up logging and monitoring
- Apply security policies
- Tag resources consistently
- Enable backup and recovery

**Example:**
```typescript
export class SecureApiGateway extends Construct {
  public readonly api: apigateway.RestApi;

  constructor(scope: Construct, id: string, props: SecureApiGatewayProps) {
    super(scope, id);

    this.api = new apigateway.RestApi(this, 'Api', {
      restApiName: props.apiName,
      deployOptions: {
        stageName: props.stageName ?? 'prod',
        tracingEnabled: true,              // X-Ray by default
        loggingLevel: apigateway.MethodLoggingLevel.INFO,
        metricsEnabled: true,              // CloudWatch metrics
        throttlingRateLimit: 1000,         // Rate limiting
        throttlingBurstLimit: 2000,
      },
      defaultCorsPreflightOptions: {       // CORS configured
        allowOrigins: props.allowedOrigins ?? apigateway.Cors.ALL_ORIGINS,
        allowMethods: apigateway.Cors.ALL_METHODS,
      },
      endpointConfiguration: {             // Private by default if VPC provided
        types: props.vpcEndpoint 
          ? [apigateway.EndpointType.PRIVATE]
          : [apigateway.EndpointType.REGIONAL],
        vpcEndpoints: props.vpcEndpoint ? [props.vpcEndpoint] : undefined,
      },
    });

    // Add WAF if specified
    if (props.enableWaf) {
      this.addWafProtection();
    }
  }
}
```

### 2.3 Configuration Flexibility

**Balance Defaults with Customization:**
- Provide sensible defaults for 80% use cases
- Allow overrides for advanced scenarios
- Use optional props with default values
- Support both simple and complex configurations
- Document when to use defaults vs custom config

**Example:**
```typescript
export interface DatabaseConstructProps {
  // Required
  readonly databaseName: string;

  // Optional with sensible defaults
  readonly instanceType?: ec2.InstanceType;  // Default: t3.micro
  readonly backupRetention?: cdk.Duration;   // Default: 7 days
  readonly multiAz?: boolean;                // Default: true for prod
  readonly encryption?: boolean;             // Default: true

  // Advanced optional
  readonly parameterGroup?: rds.IParameterGroup;
  readonly securityGroups?: ec2.ISecurityGroup[];
  readonly monitoringInterval?: cdk.Duration;
}
```

---

## 3. Testing Expertise

### 3.1 Unit Testing Constructs

**Test Synthesis:**
- Verify CloudFormation resources are created
- Check resource properties match expectations
- Validate resource counts
- Test conditional logic in constructs

**CDK Assertions:**
```typescript
import { Template, Match } from 'aws-cdk-lib/assertions';

test('creates Lambda with correct configuration', () => {
  const template = Template.fromStack(stack);

  template.hasResourceProperties('AWS::Lambda::Function', {
    Runtime: 'nodejs20.x',
    Timeout: 30,
    MemorySize: Match.anyValue(),
    Environment: {
      Variables: Match.objectLike({
        LOG_LEVEL: 'INFO',
      }),
    },
  });

  template.resourceCountIs('AWS::Lambda::Function', 1);
});
```

### 3.2 Integration Testing

**Real Deployment Tests:**
- Deploy constructs to test AWS account
- Verify resources are created correctly
- Test resource interactions
- Validate permissions and access
- Clean up after tests

**IntegTest Framework:**
```typescript
import { IntegTest } from '@aws-cdk/integ-tests-alpha';

const app = new App();
const stack = new Stack(app, 'TestStack');

// Create construct
const construct = new MyConstruct(stack, 'Test', { ... });

// Deploy and test
new IntegTest(app, 'MyConstructIntegTest', {
  testCases: [stack],
  diffAssets: true,
  stackUpdateWorkflow: true,
});
```

### 3.3 Snapshot Testing

**Template Snapshots:**
- Capture CloudFormation templates
- Detect unintended changes
- Review template diffs in PRs
- Update snapshots intentionally

```typescript
test('matches snapshot', () => {
  const template = Template.fromStack(stack);
  expect(template.toJSON()).toMatchSnapshot();
});
```

---

## 4. Documentation Skills

### 4.1 TSDoc Documentation

**Comprehensive API Docs:**
- Document all public classes and methods
- Include `@param`, `@returns`, `@throws` tags
- Provide `@example` usage examples
- Document `@default` values for optional props
- Explain complex behavior and trade-offs

**Example:**
```typescript
/**
 * Creates a monitored Lambda function with alarms
 * 
 * This construct creates a Lambda function with CloudWatch alarms for:
 * - Error rate exceeding threshold
 * - Duration exceeding p99 threshold
 * - Throttling events
 * 
 * @param scope - CDK scope
 * @param id - Construct ID
 * @param props - Configuration properties
 * 
 * @example
 * ```typescript
 * const fn = new MonitoredLambdaFunction(this, 'UserFunction', {
 *   entry: 'src/users/handler.ts',
 *   alarmEmail: 'team@example.com',
 *   errorThreshold: 10,
 * });
 * ```
 */
```

### 4.2 README and Guides

**Construct Library README:**
- Installation instructions
- Quick start examples
- Complete API reference
- Architecture diagrams
- Best practices and patterns
- Troubleshooting guide
- Contributing guidelines

**Usage Examples:**
- Basic usage for common scenarios
- Advanced usage for complex cases
- Integration with other constructs
- Real-world patterns and recipes

---

## 5. AWS Best Practices Knowledge

### 5.1 Well-Architected Framework

**Operational Excellence:**
- Infrastructure as code
- Automated deployments
- Monitoring and observability
- Runbooks and documentation

**Security:**
- Identity and access management
- Detective controls
- Infrastructure protection
- Data protection
- Incident response

**Reliability:**
- Foundations (service limits, network topology)
- Workload architecture (distributed systems)
- Change management (automated deployments)
- Failure management (backup, disaster recovery)

**Performance Efficiency:**
- Selection (compute, storage, database, network)
- Review (continuous optimization)
- Monitoring (performance metrics)
- Trade-offs (consistency vs latency)

**Cost Optimization:**
- Practice cloud financial management
- Expenditure and usage awareness
- Cost-effective resources
- Manage demand and supply resources
- Optimize over time

**Sustainability:**
- Region selection for carbon footprint
- User behavior patterns
- Software and architecture patterns
- Data patterns
- Hardware patterns

### 5.2 Service Limits and Quotas

**Understanding Limits:**
- Default service limits
- Soft limits (can be increased)
- Hard limits (cannot be changed)
- Request quota increases proactively
- Design for limits (pagination, batching)

**Common Limits:**
- Lambda concurrent executions (1000 default)
- API Gateway requests per second (10,000)
- DynamoDB read/write capacity units
- S3 bucket limits (100 per account)
- CloudFormation stack resources (500)

---

## 6. Soft Skills

### 6.1 Communication

**Technical Writing:**
- Clear, concise documentation
- Explain complex concepts simply
- Provide context and rationale
- Use diagrams and examples
- Maintain up-to-date docs

**Collaboration:**
- Work with backend/serverless agents on construct requirements
- Gather feedback from construct consumers
- Explain design decisions and trade-offs
- Respond to issues and feature requests
- Mentor others on construct usage

### 6.2 Problem Solving

**Design Thinking:**
- Identify common patterns across projects
- Abstract reusable components
- Balance flexibility with simplicity
- Consider multiple implementation approaches
- Evaluate trade-offs systematically

**Debugging:**
- Analyze CloudFormation errors
- Troubleshoot deployment failures
- Investigate resource creation issues
- Debug IAM permission problems
- Resolve circular dependencies

### 6.3 Continuous Improvement

**Learning:**
- Stay current with CDK updates
- Learn new AWS services and features
- Study construct patterns from AWS and community
- Experiment with new approaches
- Share knowledge with team

**Iteration:**
- Gather feedback from construct users
- Identify pain points and improvements
- Refactor constructs based on learnings
- Update documentation and examples
- Version constructs appropriately

---

## 7. Advanced Construct Patterns

### 7.1 Construct Composition

**Building Blocks:**
- Create small, focused constructs
- Compose into larger patterns
- Avoid monolithic constructs
- Enable mix-and-match usage

**Example:**
```typescript
// Small, focused constructs
export class MonitoredLambda extends Construct { }
export class LambdaAlarms extends Construct { }
export class LambdaLogGroup extends Construct { }

// Composed construct
export class FullyManagedLambda extends Construct {
  constructor(scope: Construct, id: string, props: Props) {
    super(scope, id);

    const fn = new MonitoredLambda(this, 'Function', props);
    new LambdaAlarms(this, 'Alarms', { function: fn.function });
    new LambdaLogGroup(this, 'Logs', { function: fn.function });
  }
}
```

### 7.2 Escape Hatches

**When to Use:**
- L2 construct doesn't expose needed property
- Need to set CloudFormation-specific attributes
- Override default behavior

**Example:**
```typescript
const fn = new lambda.Function(this, 'Function', { ... });

// Access L1 construct
const cfnFunction = fn.node.defaultChild as lambda.CfnFunction;
cfnFunction.addPropertyOverride('ReservedConcurrentExecutions', 100);
```

### 7.3 Custom Resources

**When to Use:**
- AWS service not yet supported by CDK
- Custom logic during stack operations
- Integration with external systems
- Data transformation during deployment

**Example:**
```typescript
import { CustomResource, CustomResourceProvider } from 'aws-cdk-lib';

const provider = CustomResourceProvider.getOrCreateProvider(this, 'Provider', {
  codeDirectory: path.join(__dirname, 'custom-resource-handler'),
  runtime: CustomResourceProviderRuntime.NODEJS_20_X,
});

new CustomResource(this, 'CustomResource', {
  serviceToken: provider.serviceToken,
  properties: {
    Key: 'Value',
  },
});
```

---

## 8. Testing Strategies

### 8.1 Test Coverage

**What to Test:**
- Resource creation and properties
- Default values applied correctly
- Custom configurations respected
- Helper methods work as expected
- IAM permissions granted correctly
- Tags applied appropriately
- Error handling for invalid props

**Coverage Goals:**
- >90% code coverage for constructs
- 100% coverage for critical security constructs
- All public methods tested
- All configuration paths tested

### 8.2 Test Organization

**Test Structure:**
```typescript
describe('ApiLambdaFunction', () => {
  describe('defaults', () => {
    test('applies default timeout', () => { });
    test('applies default memory size', () => { });
    test('enables tracing by default', () => { });
  });

  describe('custom configuration', () => {
    test('respects custom timeout', () => { });
    test('respects custom memory size', () => { });
    test('disables tracing when specified', () => { });
  });

  describe('helper methods', () => {
    test('grantReadData grants correct permissions', () => { });
    test('grantReadWriteData grants correct permissions', () => { });
  });

  describe('validation', () => {
    test('throws error for invalid entry path', () => { });
    test('throws error for negative timeout', () => { });
  });
});
```

### 8.3 Integration Testing

**Real Deployment:**
- Deploy to test AWS account
- Verify resources created correctly
- Test resource interactions
- Validate permissions work
- Clean up resources after test

**Cost Considerations:**
- Use smallest resource sizes for tests
- Clean up immediately after tests
- Run integration tests less frequently
- Use LocalStack for local testing when possible

---

## 9. Versioning & Publishing

### 9.1 Semantic Versioning

**Version Bumping:**
- **Major (1.0.0 → 2.0.0)**: Breaking API changes
- **Minor (1.0.0 → 1.1.0)**: New features, backward compatible
- **Patch (1.0.0 → 1.0.1)**: Bug fixes, no API changes

**Breaking Changes:**
- Removing public properties or methods
- Changing required props
- Changing default behavior significantly
- Renaming constructs or props

**Backward Compatibility:**
- Deprecate before removing
- Provide migration guides
- Support old patterns temporarily
- Use feature flags for gradual rollout

### 9.2 Publishing Workflow

**Pre-Publish Checklist:**
- [ ] All tests passing
- [ ] Documentation updated
- [ ] CHANGELOG.md updated
- [ ] Version bumped in package.json
- [ ] Examples tested
- [ ] Breaking changes documented
- [ ] Migration guide provided (if breaking)

**Publishing:**
```bash
# Build
npm run build

# Test
npm test

# Publish to npm
npm publish --access public

# Create git tag
git tag v1.2.3
git push --tags
```

### 9.3 Changelog Management

**CHANGELOG.md Format:**
```markdown
# Changelog

## [1.2.0] - 2026-02-19

### Added
- New `MonitoredQueue` construct with built-in alarms
- Support for FIFO queues in `QueueConstruct`

### Changed
- `ApiLambdaFunction` now uses ARM64 architecture by default

### Fixed
- Fixed IAM permission issue in `SecureBucket`

### Deprecated
- `LegacyApiConstruct` - use `RestApiWithAuth` instead

## [1.1.0] - 2026-02-01
...
```

---

## 10. Security Expertise

### 10.1 IAM Best Practices

**Least Privilege:**
- Grant minimum permissions needed
- Use resource-specific permissions
- Avoid wildcard permissions
- Use conditions to restrict access
- Regular permission audits

**Example:**
```typescript
// Good - specific permissions
table.grantReadData(fn);

// Bad - overly broad
fn.addToRolePolicy(new iam.PolicyStatement({
  actions: ['dynamodb:*'],
  resources: ['*'],
}));
```

### 10.2 Encryption Standards

**Data at Rest:**
- Enable encryption by default
- Use AWS managed keys or customer managed keys
- Configure key rotation
- Set appropriate key policies

**Data in Transit:**
- Enforce TLS 1.2+
- Use VPC endpoints for AWS services
- Configure security groups appropriately

### 10.3 Network Security

**VPC Design:**
- Public vs private subnets
- NAT gateways for outbound traffic
- VPC endpoints for AWS services
- Security groups and NACLs
- Network isolation patterns

---

## 11. Performance Optimization

### 11.1 Synthesis Performance

**Optimize Construct Creation:**
- Avoid expensive operations in constructor
- Cache computed values
- Use lazy initialization
- Minimize construct tree depth

### 11.2 Runtime Performance

**Resource Configuration:**
- Right-size Lambda memory and timeout
- Configure appropriate DynamoDB capacity
- Use caching where applicable
- Enable compression for API responses

### 11.3 Cost Optimization

**Design for Cost Efficiency:**
- Use ARM64 for Lambda (20% cost savings)
- Configure lifecycle policies for S3
- Use on-demand vs provisioned appropriately
- Enable auto-scaling for databases
- Tag resources for cost tracking

---

## 12. Collaboration Skills

### 12.1 Working with Backend/Serverless Agents

**Understanding Consumer Needs:**
- What problems are they trying to solve?
- What configuration flexibility do they need?
- What defaults make their work easier?
- What helper methods would be useful?

**Gathering Requirements:**
- Ask about common use cases
- Identify pain points with existing patterns
- Understand team expertise level
- Consider project-specific needs

### 12.2 Code Reviews

**Reviewing Construct Usage:**
- Verify constructs used correctly
- Suggest better patterns when available
- Identify opportunities for new constructs
- Provide guidance on configuration

**Reviewing Construct Code:**
- Check for best practices
- Validate test coverage
- Review documentation completeness
- Ensure backward compatibility

---

## 13. Continuous Learning

### Stay Current With:
- AWS CDK releases and new features
- New AWS services and capabilities
- CDK construct patterns from AWS and community
- Infrastructure best practices
- Security and compliance requirements
- Cost optimization techniques

### Learning Resources:
- AWS CDK documentation and API reference
- CDK Patterns (cdkpatterns.com)
- AWS Construct Library source code
- AWS blogs and whitepapers
- Community constructs (Construct Hub)
- Conference talks and workshops

---

## 14. Definition of Done

A CDK construct is complete when:

- [ ] Construct follows design standards
- [ ] TypeScript strict mode enabled
- [ ] Code formatted with Prettier and passes ESLint
- [ ] Props interface fully documented with TSDoc
- [ ] Sensible defaults provided for optional props
- [ ] Unit tests written and passing (>90% coverage)
- [ ] Integration tests validate real deployment
- [ ] Snapshot tests capture CloudFormation template
- [ ] Public API documented with examples
- [ ] README updated with usage guide
- [ ] Helper methods for common operations
- [ ] Resources tagged appropriately
- [ ] Security best practices followed
- [ ] Versioned appropriately (semver)
- [ ] CHANGELOG.md updated
- [ ] Code review approved
- [ ] All CI checks passing

---

**These skills represent senior-level CDK constructs engineering expertise. Master these to build reusable, maintainable, and secure infrastructure components.**
