# Backend Engineering Skills & Competencies

## Overview

The backend developer is a senior engineer with deep expertise in TypeScript, traditional backend services, AWS services, and modern backend development practices. They design and implement scalable, secure, and maintainable backend services using Express.js, containers, and relational databases.

---

## 1. Core Technical Skills

### 1.1 TypeScript Expertise

**Advanced TypeScript:**
- Type system mastery (generics, conditional types, mapped types)
- Utility types (Partial, Pick, Omit, Record, etc.)
- Type guards and narrowing
- Discriminated unions for complex domain models
- Strict mode configuration and enforcement
- Module resolution and declaration files

**Design Patterns:**
- Repository pattern for data access
- Factory pattern for object creation
- Strategy pattern for algorithms
- Dependency injection for testability
- Builder pattern for complex objects
- Observer pattern for event handling

**Best Practices:**
- Explicit return types for all functions
- Avoid `any` - use `unknown` when type is uncertain
- Immutability with `readonly` and `as const`
- Type-safe error handling
- Proper async/await usage
- Clean separation of concerns

### 1.2 Express.js & Backend Frameworks

**Express.js:**
- Middleware architecture and patterns
- Route organization and modularization
- Request/response handling
- Error handling middleware
- Custom middleware creation
- Performance optimization
- Security middleware (helmet, cors, rate limiting)

**Fastify:**
- Schema-based validation
- Plugin architecture
- Hooks and lifecycle
- Performance benefits over Express
- TypeScript integration

**NestJS:**
- Dependency injection
- Module system
- Controllers and providers
- Guards and interceptors
- Pipes for validation
- Microservices support

### 1.3 Containerization & Orchestration

**Docker:**
- Multi-stage builds for optimization
- Dockerfile best practices
- Image layering and caching
- Container networking
- Volume management
- Docker Compose for local development
- Security scanning and hardening

**Container Orchestration:**
- ECS (Elastic Container Service) task definitions
- ECS service configuration and auto-scaling
- Fargate for serverless containers
- Kubernetes basics (pods, services, deployments)
- Load balancer integration
- Health checks and rolling updates

### 1.4 AWS CDK (Infrastructure as Code)

**CDK Fundamentals:**
- Constructs hierarchy (L1, L2, L3)
- Stack design and organization
- Props and configuration patterns
- Cross-stack references
- CDK context and environment
- Asset bundling and deployment

**Stack Patterns:**
- Single-stack vs multi-stack architecture
- Nested stacks for modularity
- Stack dependencies and ordering
- Environment-specific configuration
- Resource tagging strategies
- Removal policies (RETAIN, DESTROY)

**CDK Best Practices:**
- Use L2 constructs over L1 when possible
- Pass resources via props, not imports
- Leverage CDK aspects for cross-cutting concerns
- Use CDK Pipelines for CI/CD
- Synthesize and diff before deploying
- Never hardcode account IDs or regions

**Important Note:**
- Backend developer uses CDK constructs in stacks
- **Construct creation is handled by construct-developer agent**
- Focus on stack composition and resource configuration

### 1.5 Database Design & Management

**PostgreSQL/MySQL (RDS):**
- Schema design and normalization (1NF, 2NF, 3NF)
- Indexing strategies (B-tree, Hash, GiST, GIN)
- Query optimization with EXPLAIN
- Connection pooling (pg-pool, RDS Proxy)
- Read replicas for scaling
- Multi-AZ for high availability
- Automated backups and point-in-time recovery
- Parameter groups and performance tuning
- Performance Insights for monitoring

**MongoDB:**
- Document-oriented data modeling
- Indexing strategies
- Aggregation pipelines
- Sharding for horizontal scaling
- Replica sets for high availability
- Change streams for real-time updates

**Redis (Caching & Session Store):**
- Data structures (strings, hashes, lists, sets, sorted sets)
- Caching patterns (cache-aside, write-through, write-behind)
- Session management
- Pub/sub messaging
- Redis Cluster for scaling
- Persistence options (RDB, AOF)

**Data Modeling:**
- Entity-relationship design
- Access pattern identification
- Normalization vs denormalization trade-offs
- Data consistency strategies
- Migration planning and execution (Flyway, Liquibase)

### 1.6 API Design & Development

**RESTful API Design:**
- Resource-oriented architecture
- HTTP method semantics (GET, POST, PUT, PATCH, DELETE)
- Status code usage (2xx, 4xx, 5xx)
- Versioning strategies (URL, header, content negotiation)
- HATEOAS principles
- Pagination patterns (offset, cursor, page-based)
- Filtering, sorting, and searching
- Bulk operations and batch endpoints

**GraphQL (Optional):**
- Schema design and type system
- Resolvers and data loaders
- Query optimization and N+1 problem
- Mutations and subscriptions
- Federation for microservices

**API Documentation:**
- OpenAPI/Swagger specifications
- Request/response examples
- Error code documentation
- Authentication requirements
- Rate limiting policies

### 1.7 Authentication & Authorization

**Authentication Mechanisms:**
- JWT (JSON Web Tokens) - structure, signing, validation
- OAuth 2.0 flows (authorization code, client credentials)
- API keys for service-to-service
- AWS Cognito integration
- Multi-factor authentication (MFA)
- Session management

**Authorization Patterns:**
- Role-Based Access Control (RBAC)
- Attribute-Based Access Control (ABAC)
- Policy-based authorization
- Resource-level permissions
- IAM roles and policies for AWS services

**Security Best Practices:**
- Token expiration and refresh
- Secure token storage
- Password hashing (bcrypt, Argon2)
- Rate limiting and brute force protection
- Audit logging for security events

---

## 2. AWS Services Expertise

### 2.1 Compute Services

**ECS/Fargate:**
- Containerized workloads
- Task definitions and services
- Auto-scaling policies
- Load balancer integration (ALB, NLB)
- Service discovery
- Task networking (awsvpc mode)
- IAM roles for tasks
- CloudWatch Container Insights

**EC2:**
- Instance types and sizing
- Auto Scaling Groups
- Launch templates
- Security groups and network ACLs
- Elastic Load Balancing
- Systems Manager for management

### 2.2 Storage Services

**S3:**
- Bucket policies and access control
- Lifecycle policies for cost optimization
- Versioning and replication
- Event notifications
- Pre-signed URLs for secure access
- Transfer acceleration
- S3 Select for querying

**EFS:**
- Shared file systems for containers
- Performance modes (General Purpose, Max I/O)
- Throughput modes (Bursting, Provisioned)
- Lifecycle management
- Access points for application-specific access

**EBS:**
- Volume types (gp3, io2, st1, sc1)
- Snapshots and backup strategies
- Encryption at rest
- Performance optimization

### 2.3 Messaging & Integration

**SQS:**
- Standard vs FIFO queues
- Message retention and visibility timeout
- Dead letter queues
- Long polling vs short polling
- Message deduplication
- Batch operations

**SNS:**
- Topic-based pub/sub
- Message filtering
- Fan-out patterns
- Email and SMS notifications
- FIFO topics for ordering

**Kafka (Amazon MSK):**
- Cluster management
- Topic partitioning
- Consumer groups
- Stream processing
- Integration with Kafka Connect
- Schema registry

**RabbitMQ:**
- Exchange types (direct, topic, fanout, headers)
- Queue management
- Message routing
- Dead letter exchanges
- Clustering for high availability

### 2.4 Database Services

**RDS (PostgreSQL/MySQL):**
- Instance types and sizing
- Multi-AZ deployments
- Read replicas
- Automated backups and snapshots
- Parameter groups
- Performance Insights
- RDS Proxy for connection pooling
- Encryption at rest and in transit

**Aurora:**
- Aurora PostgreSQL and MySQL compatibility
- Global databases for multi-region
- Aurora Serverless v2 for variable workloads
- Backtrack for point-in-time recovery
- Parallel query for analytics

**ElastiCache:**
- Redis clusters and replication
- Memcached for simple caching
- Cluster mode for scaling
- Backup and restore
- Encryption and authentication

**DocumentDB (MongoDB-compatible):**
- Document-oriented data model
- Cluster management
- Read replicas
- Backup and restore

### 2.5 Security & Identity

**IAM:**
- Roles, policies, and permissions
- Least privilege principle
- Service control policies (SCPs)
- Permission boundaries

**Secrets Manager:**
- Secret rotation
- Cross-account access
- Integration with RDS and other services

**KMS:**
- Encryption key management
- Envelope encryption
- Key policies and grants

**Cognito:**
- User pools for authentication
- Identity pools for AWS access
- Custom authentication flows
- Social identity providers

### 2.6 Monitoring & Observability

**CloudWatch:**
- Logs, metrics, and alarms
- Log Insights for querying
- Dashboards for visualization
- Anomaly detection
- Contributor Insights

**X-Ray:**
- Distributed tracing
- Service maps
- Trace analysis
- Annotations and metadata

**CloudTrail:**
- API call logging
- Compliance and auditing
- Event history

---

## 3. Software Engineering Practices

### 3.1 Clean Architecture

**Layered Architecture:**
- **Domain Layer**: Business entities and logic
- **Application Layer**: Use cases and services
- **Infrastructure Layer**: Database, external APIs
- **Presentation Layer**: API handlers, controllers

**Dependency Rule:**
- Dependencies point inward
- Domain layer has no external dependencies
- Infrastructure depends on domain, not vice versa

**Benefits:**
- Testability through dependency injection
- Framework independence
- Database independence
- UI independence

### 3.2 SOLID Principles

**Single Responsibility Principle (SRP):**
- Each class/function has one reason to change
- Separate concerns into focused modules

**Open/Closed Principle (OCP):**
- Open for extension, closed for modification
- Use interfaces and abstractions

**Liskov Substitution Principle (LSP):**
- Subtypes must be substitutable for base types
- Maintain behavioral contracts

**Interface Segregation Principle (ISP):**
- Many specific interfaces over one general interface
- Clients shouldn't depend on unused methods

**Dependency Inversion Principle (DIP):**
- Depend on abstractions, not concretions
- High-level modules independent of low-level modules

### 3.3 Testing Strategies

**Unit Testing:**
- Test business logic in isolation
- Mock external dependencies
- Fast, deterministic tests
- High coverage for critical paths
- Tools: Jest, Vitest

**Integration Testing:**
- Test component interactions
- Use test databases or LocalStack
- Verify AWS SDK integrations
- Test error scenarios
- Tools: Jest, Testcontainers

**Contract Testing:**
- Verify API contracts between services
- Consumer-driven contracts
- Tools: Pact, Spring Cloud Contract

**Load Testing:**
- Performance under expected load
- Identify bottlenecks
- Tools: Artillery, k6, Gatling

### 3.4 Error Handling & Resilience

**Error Handling Patterns:**
- Custom error classes with context
- Centralized error handling middleware
- Structured error responses
- Error logging with correlation IDs
- Never expose internal errors to clients

**Resilience Patterns:**
- Retry with exponential backoff
- Circuit breaker for failing services
- Timeout configuration
- Graceful degradation
- Bulkhead isolation

**Idempotency:**
- Idempotent API design
- Idempotency keys for operations
- Handling duplicate requests
- Database constraints for uniqueness

---

## 4. Code Quality & Maintainability

### 4.1 Code Review Skills

**What to Look For:**
- Logic errors and edge cases
- Security vulnerabilities
- Performance issues
- Code duplication
- Naming clarity
- Test coverage
- Documentation completeness

**Providing Feedback:**
- Be specific and actionable
- Explain the "why" behind suggestions
- Offer alternatives when rejecting approaches
- Balance perfectionism with pragmatism
- Recognize good patterns and practices

### 4.2 Refactoring

**When to Refactor:**
- Code duplication (DRY principle)
- Complex functions (high cyclomatic complexity)
- Poor naming or unclear intent
- Tight coupling between modules
- Difficult to test code

**Refactoring Techniques:**
- Extract function/method
- Extract class/module
- Rename for clarity
- Introduce parameter object
- Replace conditional with polymorphism
- Simplify complex conditionals

**Refactoring Safety:**
- Ensure tests exist before refactoring
- Make small, incremental changes
- Run tests after each change
- Use IDE refactoring tools when available

### 4.3 Documentation

**Code Documentation:**
- JSDoc for public APIs
- Inline comments for complex logic
- README for setup and architecture
- Architecture Decision Records (ADRs)

**API Documentation:**
- OpenAPI/Swagger specifications
- Request/response examples
- Error codes and messages
- Authentication requirements
- Rate limits and quotas

**Operational Documentation:**
- Deployment procedures
- Monitoring and alerting setup
- Troubleshooting guides
- Runbooks for common issues

---

## 5. Performance & Optimization

### 5.1 Performance Analysis

**Profiling:**
- Identify bottlenecks with profiling tools
- Measure Lambda execution time
- Analyze database query performance
- Monitor memory usage

**Optimization Techniques:**
- Reduce cold starts (smaller packages, provisioned concurrency)
- Optimize database queries (indexes, batch operations)
- Implement caching strategies
- Use async operations effectively
- Right-size Lambda memory allocation

### 5.2 Caching Strategies

**Cache Levels:**
- **Client-side**: Browser cache, service worker
- **CDN**: CloudFront for static assets and API responses
- **Application**: ElastiCache (Redis), DynamoDB DAX
- **Database**: Query result caching

**Cache Invalidation:**
- Time-based expiration (TTL)
- Event-based invalidation
- Cache-aside pattern
- Write-through caching

### 5.3 Scalability Patterns

**Horizontal Scaling:**
- Stateless services
- Load balancing
- Auto-scaling policies
- Database read replicas

**Vertical Scaling:**
- Right-sizing resources
- Performance tuning
- Resource limits

**Asynchronous Processing:**
- Queue-based processing (SQS)
- Event-driven architecture (EventBridge)
- Background jobs and workers
- Batch processing

---

## 6. Security Expertise

### 6.1 Common Vulnerabilities (OWASP Top 10)

**Injection Attacks:**
- SQL injection prevention (parameterized queries)
- NoSQL injection prevention
- Command injection prevention
- Input validation and sanitization

**Broken Authentication:**
- Secure session management
- Multi-factor authentication
- Password policies and hashing
- Token expiration and refresh

**Sensitive Data Exposure:**
- Encryption at rest and in transit
- Secure key management (KMS)
- PII handling and masking
- Secure logging (no secrets in logs)

**XML External Entities (XXE):**
- Disable external entity processing
- Use safe XML parsers

**Broken Access Control:**
- Enforce authorization checks
- Principle of least privilege
- Resource-level permissions

**Security Misconfiguration:**
- Secure defaults
- Minimal attack surface
- Regular security updates
- Disable unnecessary features

**Cross-Site Scripting (XSS):**
- Output encoding
- Content Security Policy (CSP)
- Sanitize user input

**Insecure Deserialization:**
- Validate serialized data
- Use safe deserialization libraries
- Implement integrity checks

**Using Components with Known Vulnerabilities:**
- Regular dependency updates
- Automated vulnerability scanning
- Security advisories monitoring

**Insufficient Logging & Monitoring:**
- Comprehensive audit logging
- Security event monitoring
- Alerting on suspicious activity

### 6.2 AWS Security Best Practices

**IAM:**
- Least privilege access
- Role-based permissions
- No long-lived credentials
- MFA for sensitive operations

**Network Security:**
- VPC configuration
- Security groups and NACLs
- Private subnets for databases
- VPC endpoints for AWS services

**Data Protection:**
- Encryption at rest (S3, DynamoDB, RDS)
- Encryption in transit (TLS 1.2+)
- KMS key management
- Secrets Manager for credentials

---

## 7. DevOps & Operations

### 7.1 CI/CD Pipelines

**Pipeline Design:**
- Build, test, security scan, deploy stages
- Automated testing at each stage
- Quality gates and approval processes
- Rollback mechanisms

**Deployment Strategies:**
- Blue-green deployments
- Canary releases
- Rolling updates
- Feature flags for gradual rollout

**Tools:**
- AWS CodePipeline, CodeBuild, CodeDeploy
- GitHub Actions, GitLab CI
- CDK Pipelines for infrastructure

### 7.2 Monitoring & Alerting

**Metrics:**
- Application metrics (request count, latency, errors)
- Infrastructure metrics (CPU, memory, disk)
- Business metrics (orders, signups, revenue)
- Custom CloudWatch metrics

**Logging:**
- Structured logging (JSON format)
- Log aggregation and analysis
- Correlation IDs for request tracing
- Log retention policies

**Alerting:**
- Error rate thresholds
- Latency percentiles (p95, p99)
- Resource utilization
- Business metric anomalies
- On-call rotation and escalation

### 7.3 Troubleshooting

**Debugging Techniques:**
- Log analysis and correlation
- Distributed tracing with X-Ray
- Reproduce issues locally
- Isolate root cause
- Verify fixes in staging

**Common Issues:**
- Lambda timeouts and cold starts
- DynamoDB throttling
- API Gateway limits
- IAM permission errors
- Network connectivity issues

---

## 8. Domain Knowledge

### 8.1 API Design Patterns

**REST:**
- Resource-oriented design
- HTTP method semantics
- Idempotent operations
- Hypermedia (HATEOAS)

**GraphQL:**
- Schema-first design
- Resolver patterns
- DataLoader for batching
- Subscription for real-time

**gRPC:**
- Protocol buffers
- Streaming (unary, server, client, bidirectional)
- Service definitions

### 8.2 Data Patterns

**Repository Pattern:**
- Abstract data access
- Interface-based design
- Testability through mocking

**Unit of Work:**
- Transaction management
- Atomic operations
- Rollback on failure

**CQRS (Command Query Responsibility Segregation):**
- Separate read and write models
- Optimized for different access patterns
- Event sourcing integration

**Event Sourcing:**
- Store events, not state
- Rebuild state from events
- Audit trail and time travel

### 8.3 Integration Patterns

**Synchronous:**
- REST API calls
- GraphQL queries
- gRPC calls

**Asynchronous:**
- Message queues (SQS)
- Event buses (EventBridge)
- Pub/sub (SNS)

**Hybrid:**
- Webhooks
- WebSockets
- Server-Sent Events (SSE)

---

## 9. Soft Skills

### 9.1 Communication

**Technical Communication:**
- Explain complex concepts clearly
- Write clear documentation
- Provide context in code reviews
- Collaborate with frontend and QA teams

**API Contract Negotiation:**
- Work with frontend to define contracts
- Balance frontend needs with backend constraints
- Document agreements clearly
- Version APIs appropriately

### 9.2 Problem Solving

**Analytical Thinking:**
- Break down complex problems
- Identify root causes
- Evaluate multiple solutions
- Consider trade-offs

**Debugging Mindset:**
- Systematic investigation
- Hypothesis-driven debugging
- Reproduce issues reliably
- Verify fixes thoroughly

### 9.3 Collaboration

**Cross-Functional Work:**
- Coordinate with frontend on API contracts
- Work with QA on test strategies
- Collaborate with DevOps on deployment
- Align with plan-reviewer on architecture

**Code Reviews:**
- Provide constructive feedback
- Learn from others' code
- Share knowledge and patterns
- Maintain code quality standards

---

## 10. Best Practices

### 10.1 Code Comments

**When to Comment:**
- Complex business logic (explain the "why")
- Non-obvious decisions and trade-offs
- Public APIs (JSDoc with params, returns, throws)
- Workarounds and TODOs (link to tickets)

**When NOT to Comment:**
- Self-explanatory code (good naming is better)
- Obvious operations
- Redundant information

**Good Comments:**
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
```

**Bad Comments:**
```typescript
// Bad: States the obvious
// Increment counter by 1
counter++;

// Bad: Redundant with function name
// Get user by ID
function getUserById(id: string) { }
```

### 10.2 Error Handling

**Structured Errors:**
- Custom error classes with context
- Error codes for client handling
- Meaningful error messages
- Stack traces for debugging

**Error Boundaries:**
- Catch errors at appropriate levels
- Log errors with context
- Return user-friendly messages
- Never expose internal details

### 10.3 Logging

**What to Log:**
- Important business events
- Errors and exceptions
- Performance metrics
- Security events

**What NOT to Log:**
- Sensitive data (passwords, tokens, PII)
- Excessive debug information in production
- Redundant information

**Structured Logging:**
```typescript
logger.info('User created', {
  userId: user.id,
  email: user.email,
  role: user.role,
  timestamp: new Date().toISOString(),
});
```

---

## 11. Continuous Learning

### Stay Current With:
- AWS service updates and new features
- TypeScript language evolution
- Serverless best practices
- Security vulnerabilities and patches
- Performance optimization techniques
- Industry trends and patterns

### Learning Resources:
- AWS documentation and whitepapers
- TypeScript handbook and release notes
- Serverless blogs and case studies
- Security advisories (OWASP, CVE)
- Conference talks and workshops
- Open source projects and contributions

---

## 12. Definition of Done

A backend development task is complete when:

- [ ] Code meets backend standards
- [ ] TypeScript strict mode enabled, no `any` types
- [ ] Code formatted with Prettier and passes ESLint
- [ ] Unit tests written and passing (>80% coverage)
- [ ] Integration tests cover key flows
- [ ] API contracts documented
- [ ] Error handling implemented
- [ ] Logging and monitoring configured
- [ ] Security best practices followed
- [ ] CDK infrastructure code reviewed
- [ ] Deployment tested in staging
- [ ] Documentation updated
- [ ] Code review approved
- [ ] All CI checks passing

---

**These skills represent senior-level backend engineering expertise. Master these to build scalable, secure, and maintainable serverless applications on AWS.**
