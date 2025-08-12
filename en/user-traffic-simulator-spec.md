# User Service Traffic Simulator - Technical Specification

## Executive Summary

This document outlines the comprehensive technical specification for implementing a Hyperf command that simulates realistic API traffic to generate meaningful metrics for Grafana visualization. The simulator will create diverse user interactions across all service endpoints using coroutines for optimal performance.

**Command Signature:**
```bash
php bin/hyperf.php user:populate --users=100 --min-request-count=10 --max-request-count=1000
```

## Project Context Analysis

### Current Architecture
- **Framework**: Hyperf with Swoole coroutines
- **Authentication**: JWT-based with token caching (Redis)
- **Metrics System**: Prometheus integration with Grafana dashboards
- **Domain Structure**: Clean Architecture with CQRS pattern
- **API Endpoints**: REST (HTTP) and gRPC interfaces

### Existing Metrics Infrastructure
The project already has robust metrics collection:
- **Auth Metrics**: Login attempts, token generation, validation
- **User Metrics**: User counts, activity tracking
- **Performance Metrics**: Request duration, system performance
- **System Metrics**: Memory usage, resource utilization

### Available API Endpoints for Simulation

#### Authentication Endpoints (`/api/v1/auth`)
- `POST /login` - User authentication
- `POST /logout` - Session termination
- `POST /refresh` - Token refresh
- `POST /me` - Current user info

#### User Management (`/api/v1/users`)
- `GET /` - List users (paginated)
- `GET /search` - Search users
- `GET /{id}` - Get user by ID
- `GET /email/{email}` - Get user by email
- `GET /username/{username}` - Get user by username
- `POST /` - Create user
- `PUT /{id}` - Update user
- `DELETE /{id}` - Delete user
- `PUT /{id}/activate` - Activate user
- `PUT /{id}/deactivate` - Deactivate user
- `PUT /{id}/change-password` - Change password
- `PUT /{id}/reset-password` - Reset password

#### User Profiles (`/api/v1/users/{userId}/profile`)
- `POST /` - Create profile
- `GET /` - Get profile
- `PUT /` - Update profile
- `DELETE /` - Delete profile

#### Role Management (`/api/v1/roles`)
- `GET /` - List roles
- `POST /` - Create role
- `GET /{id}` - Get role
- `PUT /{id}` - Update role
- `DELETE /{id}` - Delete role

#### Permission Management (`/api/v1/permissions`)
- `GET /` - List permissions
- `POST /` - Create permission
- `GET /{id}` - Get permission
- `PUT /{id}` - Update permission
- `DELETE /{id}` - Delete permission

#### User Role Assignments (`/api/v1/users/{userId}/roles`)
- `GET /` - List user roles
- `POST /{roleId}` - Assign role
- `DELETE /{roleId}` - Revoke role

## Technical Architecture

### 1. Command Structure

```php
<?php

namespace App\Infrastructure\Command;

use Hyperf\Command\Annotation\Command;
use Hyperf\Command\Command as HyperfCommand;
use Symfony\Component\Console\Input\InputInterface;
use Symfony\Component\Console\Input\InputOption;
use Symfony\Component\Console\Output\OutputInterface;

#[Command]
class UserTrafficSimulatorCommand extends HyperfCommand
{
    protected ?string $name = 'user:populate';
    
    protected string $description = 'Simulate realistic API traffic for metrics generation';
}
```

#### Command Parameters
- `--users` (required): Number of concurrent virtual users (default: 100)
- `--min-request-count` (required): Minimum requests per user (default: 10)
- `--max-request-count` (required): Maximum requests per user (default: 1000)
- `--duration`: Optional time limit in seconds
- `--base-url`: Target service base URL (default: http://localhost:9501)
- `--delay-min`: Minimum delay between requests in ms (default: 100)
- `--delay-max`: Maximum delay between requests in ms (default: 2000)
- `--scenario`: Specific scenario to run (default: mixed)

### 2. Core Components

#### A. Traffic Orchestrator
**Responsibility**: Coordinate all simulation activities
```php
class TrafficOrchestrator
{
    public function orchestrate(SimulationConfig $config): void;
    public function createUserPools(int $userCount): array;
    public function distributeWorkload(array $users, array $scenarios): void;
}
```

#### B. User Behavior Engine
**Responsibility**: Define realistic user interaction patterns
```php
class UserBehaviorEngine
{
    public function generateUserSession(VirtualUser $user): UserSession;
    public function selectNextAction(UserSession $session): ApiAction;
    public function calculateDelay(ApiAction $action): int;
}
```

#### C. HTTP Client Manager
**Responsibility**: Manage HTTP connections and requests
```php
class HttpClientManager
{
    private Client $client;
    private TokenManager $tokenManager;
    
    public function executeRequest(ApiRequest $request): ApiResponse;
    public function handleAuthentication(VirtualUser $user): AuthResult;
}
```

#### D. Metrics Collector
**Responsibility**: Real-time metrics collection and reporting
```php
class TrafficMetricsCollector
{
    public function recordRequest(ApiRequest $request, ApiResponse $response): void;
    public function recordError(ApiRequest $request, Exception $error): void;
    public function generateReport(): TrafficReport;
}
```

### 3. Coroutine Architecture

#### Coroutine Pool Design
```php
class CoroutinePool
{
    private int $maxConcurrency;
    private array $activeCoroutines = [];
    
    public function spawn(callable $task): int;
    public function manage(): void;
    public function waitAll(): void;
}
```

#### User Session Management
Each virtual user runs in its own coroutine:
```php
go(function() use ($user, $scenarios) {
    $session = new UserSession($user);
    
    while ($session->shouldContinue()) {
        $action = $behaviorEngine->selectNextAction($session);
        $delay = $behaviorEngine->calculateDelay($action);
        
        Coroutine::sleep($delay / 1000);
        
        try {
            $response = $httpClient->executeRequest($action->toRequest());
            $session->recordSuccess($action, $response);
        } catch (Exception $e) {
            $session->recordError($action, $e);
        }
    }
});
```

### 4. User Simulation Strategies

#### A. User Archetypes

**New User Journey**
1. User registration
2. Profile creation
3. Basic exploration (list users, search)
4. Profile updates

**Active User Journey**
1. Login
2. User management operations
3. Role/permission queries
4. Profile management
5. Logout

**Administrative User Journey**
1. Login
2. User CRUD operations
3. Role/permission management
4. Bulk operations
5. System monitoring queries

**Power User Journey**
1. Login
2. Complex search operations
3. Batch user operations
4. Advanced filtering
5. Multiple sessions

#### B. Realistic Behavior Patterns

**Request Distribution** (based on typical API usage):
- Authentication: 20%
- User queries: 35%
- User mutations: 15%
- Profile operations: 15%
- Role/permission ops: 10%
- Administrative ops: 5%

**Timing Patterns**:
- Human-like delays (100ms - 2000ms)
- Burst patterns for search operations
- Longer delays for form submissions
- Session-based request clustering

### 5. Error Simulation

#### Realistic Error Scenarios
- Authentication failures (invalid credentials)
- Authorization failures (insufficient permissions)
- Validation errors (invalid data)
- Not found errors (non-existent resources)
- Rate limiting scenarios
- Network timeouts
- Server errors (5xx responses)

#### Error Distribution (realistic API behavior):
- Success: 85%
- Client Errors (4xx): 12%
- Server Errors (5xx): 3%

### 6. Data Management

#### Virtual User Generation
```php
class VirtualUserFactory
{
    public function generateUsers(int $count): array
    {
        return array_map(function($i) {
            return new VirtualUser([
                'username' => "sim_user_{$i}",
                'email' => "sim_user_{$i}@simulation.test",
                'password' => 'SimPass123!',
                'name' => "Simulation User {$i}",
                'profile' => $this->generateProfile(),
                'archetype' => $this->selectArchetype(),
                'behavior_config' => $this->generateBehaviorConfig()
            ]);
        }, range(1, $count));
    }
}
```

#### Data Cleanup Strategy
```php
class DataCleanupService
{
    public function cleanupSimulationData(): void
    {
        // Remove users created by simulation
        // Clear test tokens from cache
        // Reset metrics if needed
        // Cleanup temporary data
    }
}
```

### 7. Configuration Management

#### Simulation Configuration
```php
class SimulationConfig
{
    public int $userCount;
    public int $minRequestCount;
    public int $maxRequestCount;
    public int $durationSeconds;
    public string $baseUrl;
    public array $delayRange;
    public string $scenario;
    public bool $cleanupOnExit;
    public array $endpointWeights;
    public array $errorRates;
}
```

#### Environment-Specific Settings
- Development: Lower user counts, detailed logging
- Testing: Medium load, comprehensive metrics
- Staging: Production-like load, full monitoring

### 8. Monitoring and Reporting

#### Real-time Progress Display
```bash
Traffic Simulation Progress:
┌─────────────────────────────────────────┐
│ Users: 100/100 Active | Requests: 15,432│
│ Success Rate: 87.3% | Errors: 1,967     │
│ Avg Response: 245ms | Peak: 1,230ms     │
│ Duration: 05:23 | Remaining: ~12:15     │
└─────────────────────────────────────────┘

Recent Activity:
[12:34:15] User-045: POST /api/v1/users → 201 (123ms)
[12:34:15] User-021: GET /api/v1/users → 200 (89ms)
[12:34:16] User-087: POST /api/v1/auth/login → 401 (45ms)
```

#### Metrics Integration
The simulator will generate metrics that integrate with existing Prometheus collectors:
- Request volume per endpoint
- Response time distributions
- Error rates by endpoint and type
- Authentication success/failure rates
- User session durations
- Resource utilization during load

### 9. Implementation Phases

#### Phase 1: Core Infrastructure (Week 1)
- [ ] Command class implementation
- [ ] Basic coroutine architecture
- [ ] HTTP client setup
- [ ] Configuration management
- [ ] Simple user simulation

#### Phase 2: Behavior Engine (Week 2)
- [ ] User archetype definitions
- [ ] Realistic behavior patterns
- [ ] Request timing algorithms
- [ ] Session state management
- [ ] Error simulation logic

#### Phase 3: Advanced Features (Week 3)
- [ ] Complex user journeys
- [ ] Data cleanup mechanisms
- [ ] Comprehensive metrics integration
- [ ] Performance optimization
- [ ] Memory management

#### Phase 4: Production Readiness (Week 4)
- [ ] Extensive testing
- [ ] Documentation completion
- [ ] Performance benchmarking
- [ ] Error handling robustness
- [ ] Production deployment

## Implementation Guidelines

### 1. Memory Management
```php
// Prevent memory leaks in long-running simulations
class MemoryManager
{
    public function periodic_cleanup(): void
    {
        // Clear response caches
        // Reset accumulated metrics
        // Garbage collect unused objects
        // Monitor memory usage
    }
}
```

### 2. Graceful Shutdown
```php
// Handle SIGTERM/SIGINT gracefully
class ShutdownHandler
{
    public function handleShutdown(): void
    {
        $this->logger->info('Simulation shutdown initiated');
        $this->stopNewRequests();
        $this->waitForActiveRequests();
        $this->cleanupResources();
        $this->generateFinalReport();
    }
}
```

### 3. Error Recovery
```php
// Implement circuit breaker pattern
class CircuitBreaker
{
    public function executeWithBreaker(callable $operation): mixed
    {
        if ($this->isOpen()) {
            throw new CircuitBreakerOpenException();
        }
        
        try {
            $result = $operation();
            $this->recordSuccess();
            return $result;
        } catch (Exception $e) {
            $this->recordFailure();
            throw $e;
        }
    }
}
```

### 4. Performance Considerations

#### Coroutine Optimization
- Use connection pooling for HTTP clients
- Implement request batching where possible
- Monitor coroutine count to prevent resource exhaustion
- Use async/await patterns for I/O operations

#### Resource Management
- Limit concurrent connections per target
- Implement backoff strategies for failed requests
- Monitor system resources (CPU, memory, connections)
- Use lazy initialization for heavy objects

## Security Considerations

### 1. Test Data Isolation
- Use dedicated test database/environment
- Ensure simulation data is clearly marked
- Implement automatic cleanup mechanisms
- Prevent simulation data from affecting production

### 2. Authentication Safety
- Use dedicated test credentials
- Implement token lifecycle management
- Ensure proper session cleanup
- Monitor for authentication abuse

### 3. Rate Limiting Compliance
- Respect existing rate limits
- Implement backoff strategies
- Monitor for service degradation
- Provide emergency stop mechanisms

## Metrics and KPIs

### Generated Metrics
1. **Request Metrics**
   - Total requests per endpoint
   - Success/failure rates
   - Response time percentiles
   - Throughput (requests/second)

2. **User Metrics**
   - Active virtual users
   - Session durations
   - User journey completion rates
   - Behavior pattern distributions

3. **System Metrics**
   - Resource utilization
   - Error rates by category
   - Performance degradation indicators
   - Memory usage patterns

### Success Criteria
- Stable operation for extended periods (24+ hours)
- Realistic traffic patterns matching production
- Comprehensive metric generation for Grafana
- Minimal impact on system performance
- Reliable cleanup and resource management

## Configuration Examples

### Development Environment
```bash
php bin/hyperf.php user:populate \
    --users=10 \
    --min-request-count=5 \
    --max-request-count=50 \
    --base-url=http://localhost:9501 \
    --scenario=development
```

### Load Testing
```bash
php bin/hyperf.php user:populate \
    --users=500 \
    --min-request-count=100 \
    --max-request-count=5000 \
    --duration=3600 \
    --scenario=load_test
```

### Metrics Generation
```bash
php bin/hyperf.php user:populate \
    --users=100 \
    --min-request-count=50 \
    --max-request-count=1000 \
    --scenario=metrics_generation
```

## Conclusion

This specification provides a comprehensive roadmap for implementing a sophisticated traffic simulator that will generate meaningful metrics for Grafana visualization while maintaining system stability and realistic user behavior patterns. The implementation leverages Hyperf's coroutine capabilities for optimal performance and integrates seamlessly with the existing metrics infrastructure.

The simulator will serve multiple purposes:
- Generate realistic load for performance testing
- Create comprehensive metrics for dashboard development
- Provide insights into system behavior under various load conditions
- Support development and testing workflows

The phased implementation approach ensures steady progress while maintaining code quality and system stability throughout the development process.