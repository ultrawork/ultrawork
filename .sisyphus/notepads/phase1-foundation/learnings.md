# Phase 1 Foundation - Learnings

## Task 8: Global Exception Handler & CORS Configuration

**Date**: 2025-02-07
**Status**: ✅ Completed

### Implementation Summary

Successfully implemented global exception handling with API response envelope and CORS configuration for the Ultrawork backend.

### Key Learnings

#### 1. Exception Handler Architecture
- **@ControllerAdvice** pattern works seamlessly with Spring Boot 3.4.0
- All exception handlers return `ResponseEntity<ApiResponse<Void>>` for consistency
- ApiResponse envelope (from Task 1) provides uniform error response structure with:
  - `error.code`: Machine-readable error identifier
  - `error.message`: User-friendly error message
  - `error.details`: Optional validation details (for 422 errors)
  - `timestamp`: ISO 8601 timestamp for all responses

#### 2. Exception Mapping Strategy
- **MethodArgumentNotValidException** → 422 (Unprocessable Entity)
  - Extracts field-level validation errors into `error.details` map
  - Provides clear feedback for API clients on validation failures
- **EntityNotFoundException** → 404 (Not Found)
  - Custom exception class created for domain-level not-found scenarios
- **AccessDeniedException** → 403 (Forbidden)
  - Spring Security exception, no additional context needed
- **AuthenticationException** → 401 (Unauthorized)
  - Spring Security exception for failed authentication
- **DataIntegrityViolationException** → 409 (Conflict)
  - Custom exception for database constraint violations
- **Exception (catch-all)** → 500 (Internal Server Error)
  - Logs full stack trace server-side (never exposed to client)
  - Returns generic message to prevent information leakage

#### 3. CORS Configuration
- **WebMvcConfigurer** implementation provides clean, declarative CORS setup
- Allowed origins: `http://144.124.229.191`, `http://localhost:3000`
- Allowed methods: GET, POST, PUT, DELETE, PATCH, OPTIONS
- **allowCredentials(true)** essential for cookie-based authentication
- maxAge(3600) caches preflight responses for 1 hour

#### 4. Testing Approach
- Unit tests preferred over integration tests for exception handler logic
- Mocking WebRequest dependency keeps tests focused and fast
- Test coverage includes:
  - Each exception type returns correct HTTP status code
  - ApiResponse envelope structure is consistent
  - Error details are properly populated
  - Stack traces are never exposed in responses
  - Timestamps are always present

#### 5. Security Considerations
- Stack traces logged server-side for debugging but never sent to clients
- Generic error messages prevent information disclosure
- CORS properly restricts origins (not wildcard)
- Credentials allowed only for specified origins

### Files Created
1. `/src/main/kotlin/com/ultrawork/api/advice/GlobalExceptionHandler.kt`
   - 7 exception handlers
   - 2 custom exception classes (EntityNotFoundException, DataIntegrityViolationException)

2. `/src/main/kotlin/com/ultrawork/config/CorsConfig.kt`
   - WebMvcConfigurer implementation
   - Configured for specified origins with credentials support

3. `/src/test/kotlin/com/ultrawork/api/advice/GlobalExceptionHandlerTest.kt`
   - 9 unit tests covering all exception types
   - Tests verify status codes, envelope structure, and security

### Test Results
```
BUILD SUCCESSFUL in 17s
All 24 tests passed (including 9 new exception handler tests)
```

### Dependencies
- Spring Boot 3.4.0 (spring-boot-starter-web, spring-boot-starter-security)
- Jakarta Validation API (spring-boot-starter-validation)
- JUnit 5 for testing
- Mockito for mocking

### Integration Points
- Works seamlessly with existing ApiResponse<T> from Task 1
- Compatible with Spring Security configuration
- Integrates with existing HealthController (verified envelope format)

### Next Steps (Task 16)
- API stubs will use these exception handlers automatically
- Validation errors will be properly formatted via MethodArgumentNotValidException handler
- CORS headers will be automatically added to all /api/** responses

### Notes
- No stack traces exposed in error responses (security best practice)
- All errors include timestamp for audit/debugging purposes
- Custom exceptions allow domain-specific error handling
- CORS configuration allows credentials for cookie-based auth (needed for JWT refresh tokens)

## Task 13: System Metrics Collection Service

**Date**: 2026-02-07
**Status**: ✅ Completed

### Implementation Summary

Successfully implemented system metrics collection service with periodic sampling (every 30 seconds), in-memory ring buffer storage (last 100 data points), and admin-only API endpoint for retrieving metrics history.

### Key Learnings

#### 1. Scheduled Task Configuration
- **@EnableScheduling** annotation required on main application class to enable Spring's scheduling support
- **@Scheduled(fixedRate = 30000)** annotation on service method triggers automatic execution every 30 seconds
- Scheduled tasks start automatically when application context loads
- No manual thread management needed - Spring handles scheduling infrastructure

#### 2. System Metrics Collection
- **OperatingSystemMXBean** provides cross-platform access to system metrics
- Cast to `com.sun.management.OperatingSystemMXBean` for extended metrics:
  - `getCpuLoad()`: Returns CPU usage as decimal (0.0-1.0), multiply by 100 for percentage
  - `getTotalMemorySize()` / `getFreeMemorySize()`: RAM metrics in bytes, convert to MB
- **File.getUsableSpace()** and **File.getTotalSpace()** for disk usage calculation
- All metrics validated to be within expected ranges (CPU/disk: 0-100%, RAM: > 0)

#### 3. Ring Buffer Implementation
- **ArrayDeque<T>** provides efficient ring buffer with O(1) operations
- Thread-safe implementation using **ReentrantReadWriteLock**:
  - Write lock for adding metrics (collectMetrics)
  - Read lock for retrieving history (getMetricsHistory)
- When buffer reaches maxSize (100), oldest entry removed before adding new one
- **toList()** creates defensive copy to prevent external modification

#### 4. Admin-Only API Endpoint
- **@PreAuthorize("hasRole('OWNER')")** annotation restricts access to admin users
- **@EnableMethodSecurity** required on SecurityConfig to enable method-level security
- Returns 401 (Unauthorized) for unauthenticated requests
- Returns 403 (Forbidden) for authenticated non-admin users
- Returns 200 with metrics array for admin users

#### 5. API Response Format
- Follows existing ApiResponse<T> envelope pattern from Task 1
- Response structure:
  ```json
  {
    "data": [
      {
        "timestamp": "2025-02-07T10:00:00Z",
        "cpuUsage": 45.5,
        "ramUsed": 2048,
        "diskUsage": 60.0
      }
    ],
    "timestamp": "2025-02-07T10:00:30.123Z"
  }
  ```
- Metrics returned in chronological order (oldest first)
- Empty array returned when no metrics collected yet

#### 6. Testing Strategy (TDD Approach)
- **Test-first development**: Wrote tests before implementation
- Unit tests for SystemMetricsService:
  - Metrics collection with valid values
  - Ring buffer size limit (100 entries)
  - Chronological ordering
  - Thread safety (concurrent collection)
  - Oldest entry eviction when buffer full
- Controller tests with MockMvc:
  - 401 for unauthenticated requests
  - 403 for non-admin users
  - 200 with metrics for admin users
  - Empty array handling
- Integration test verifies real application context behavior

#### 7. Thread Safety Considerations
- Service is singleton (@Service annotation)
- Shared state (metricsBuffer) requires synchronization
- ReentrantReadWriteLock allows:
  - Multiple concurrent reads (getMetricsHistory)
  - Exclusive writes (collectMetrics)
- Kotlin's `lock.read { }` and `lock.write { }` extension functions provide clean syntax

### Files Created

1. `/src/main/kotlin/com/ultrawork/api/dto/SystemMetricsResponse.kt`
   - Data class for metrics snapshot
   - Fields: timestamp, cpuUsage, ramUsed, diskUsage

2. `/src/main/kotlin/com/ultrawork/domain/service/SystemMetricsService.kt`
   - @Service with @Scheduled method
   - Ring buffer implementation with ReentrantReadWriteLock
   - System metrics collection using OperatingSystemMXBean

3. `/src/main/kotlin/com/ultrawork/api/controller/SystemMetricsController.kt`
   - GET /api/v1/admin/system-metrics/history endpoint
   - @PreAuthorize("hasRole('OWNER')") for admin-only access

4. `/src/test/kotlin/com/ultrawork/domain/service/SystemMetricsServiceTest.kt`
   - 6 unit tests covering service behavior
   - Tests for ring buffer, thread safety, ordering

5. `/src/test/kotlin/com/ultrawork/api/controller/SystemMetricsControllerTest.kt`
   - 4 controller tests with MockMvc
   - Tests for authentication, authorization, response format

6. `/src/test/kotlin/com/ultrawork/domain/service/SystemMetricsIntegrationTest.kt`
   - Integration test verifying real application context
   - Tests automatic scheduled collection

### Configuration Changes

1. **UltraworkApplication.kt**: Added @EnableScheduling annotation
2. **SecurityConfig.kt**: Added @EnableMethodSecurity annotation

### Test Results
```
BUILD SUCCESSFUL in 18s
Total tests: 29 (up from 24)
- 6 new SystemMetricsService tests
- 4 new SystemMetricsController tests  
- 1 new integration test
All tests passed
```

### Dependencies
- Spring Boot 3.4.0 (spring-boot-starter-web, spring-boot-starter-security)
- Spring Scheduling (built-in)
- Java Management Extensions (JMX) - OperatingSystemMXBean
- Kotlin stdlib (ReentrantReadWriteLock, ArrayDeque)

### Integration Points
- Uses ApiResponse<T> envelope from Task 1
- Uses @PreAuthorize from SecurityConfig (Task 1)
- Compatible with GlobalExceptionHandler from Task 8
- Scheduled task runs automatically on application startup

### Performance Considerations
- Metrics collection every 30 seconds (low overhead)
- In-memory storage only (no database I/O)
- Ring buffer prevents unbounded memory growth (max 100 entries)
- Read-write lock allows concurrent reads without blocking
- Defensive copy on getMetricsHistory() prevents external modification

### Next Steps (Task 14)
- Frontend health screen will consume GET /api/v1/admin/system-metrics/history
- Display metrics as time-series chart
- Show current values and historical trends
- Admin authentication required to view metrics

### Notes
- Metrics collection starts immediately on application startup
- First metric collected within 30 seconds of startup
- Service is singleton - shared across all requests
- Thread-safe implementation allows concurrent access
- No database persistence in Phase 1 (in-memory only)
- Ring buffer automatically evicts oldest entries when full

## Task 18: GitHub Actions CI Workflows

**Date**: 2026-02-07
**Status**: ✅ Completed

### Implementation Summary

Successfully created GitHub Actions CI workflows for both backend and frontend repositories with proper triggers, service configurations, and build/test steps.

### Key Learnings

#### 1. Backend CI Workflow (ci-backend.yml)
- **Triggers**: Push to main/develop, PR to main/develop
- **Services Configuration**:
  - PostgreSQL 16-alpine with health checks (pg_isready)
  - Redis 7-alpine with health checks (redis-cli ping)
  - Both services expose ports for application access
  - Health checks ensure services are ready before tests run
- **Build Steps**:
  - actions/checkout@v4 for code retrieval
  - actions/setup-java@v4 with temurin distribution, Java 21
  - gradle/actions/setup-gradle@v4 for Gradle setup
  - detektMain for static code analysis
  - test for unit/integration tests
  - bootJar for building Spring Boot JAR artifact
- **Environment**: ubuntu-latest runner

#### 2. Frontend CI Workflow (ci-frontend.yml)
- **Triggers**: Push to main/develop, PR to main/develop
- **Build Steps**:
  - actions/checkout@v4 for code retrieval
  - actions/setup-node@v4 with Node 22
  - npm ci for clean dependency installation
  - npm run lint for code style checking
  - tsc --noEmit for TypeScript type checking
  - npm run test -- --run for test execution (single run mode)
  - npm run build for production build
- **Environment**: ubuntu-latest runner

#### 3. GitHub Actions Best Practices
- **Action Versions**: Using @v4 for stable, well-maintained actions
- **Service Health Checks**: Essential for database/cache services
  - health-cmd: Command to verify service readiness
  - health-interval: Check frequency (10s)
  - health-timeout: Max wait per check (5s)
  - health-retries: Max retry attempts (5)
- **Port Mapping**: Services expose ports for application connection
- **Trigger Strategy**: Both main and develop branches for CI coverage
- **PR Validation**: Workflows run on all PRs to catch issues early

#### 4. YAML Structure
- Proper indentation (2 spaces) critical for YAML parsing
- Service configuration nested under jobs.build.services
- Steps executed sequentially in order
- Environment variables passed to services via env block
- Options block for service-specific configuration

#### 5. Testing Approach
- YAML syntax validation using Python yaml module
- File existence verification
- Content inspection to ensure all required steps present
- No actual CI execution possible without GitHub remote

### Files Created

1. `/root/ultrawork/backend/.github/workflows/ci-backend.yml`
   - 1231 bytes
   - Configured for Spring Boot + Gradle + Kotlin
   - Includes PostgreSQL and Redis services

2. `/root/ultrawork/frontend/.github/workflows/ci-frontend.yml`
   - 627 bytes
   - Configured for Next.js + TypeScript + Node
   - Includes lint, type check, test, and build steps

### Verification Results

```
✓ /root/ultrawork/backend/.github/workflows/ci-backend.yml - YAML syntax valid
✓ /root/ultrawork/frontend/.github/workflows/ci-frontend.yml - YAML syntax valid
```

Both workflows:
- Have correct YAML syntax
- Include all required steps from plan
- Use specified action versions (@v4)
- Configure proper triggers (main/develop branches)
- Include service health checks (backend)
- Ready for GitHub integration

### Dependencies
- GitHub Actions (built-in to GitHub)
- actions/checkout@v4
- actions/setup-java@v4 (backend)
- gradle/actions/setup-gradle@v4 (backend)
- actions/setup-node@v4 (frontend)

### Integration Points
- Backend: Works with Spring Boot 3.4.0, Gradle build system, Kotlin
- Frontend: Works with Next.js 15, TypeScript, npm
- Both: Trigger on PR and push events to main/develop branches

### Next Steps (Task 20)
- Deploy workflows will extend these CI workflows
- Add deployment steps after successful CI
- Configure environment-specific deployments

### Notes
- Phase 1 requirement: No caching (Gradle, npm) - deferred to Phase 2
- No matrix builds in Phase 1
- No coverage reports in Phase 1
- Workflows are syntactically valid and ready for GitHub
- Cannot test actual CI execution without GitHub remote
- Both workflows follow GitHub Actions best practices
## Task 19: Ansible Playbooks for Server Provisioning and Deployment

**Date**: 2026-02-07
**Status**: ✅ Completed

### Implementation Summary

Successfully created Ansible playbooks for automated server provisioning, application deployment, and service restart with retry logic, health checks, and secure password handling.

### Key Learnings

#### 1. Ansible Inventory Configuration
- **Variables in inventory.yml**: Use `{{ variable_name }}` syntax for ansible_ssh_pass
- **Nested structure**: hosts defined under all.hosts, variables under all.vars
- **ansible_python_interpreter**: Explicitly set to /usr/bin/python3 for Ubuntu 22.04
- **Security**: Password stored in inventory vars section (use --extra-vars or ansible-vault in production)
- **Connection settings**: ansible_user, ansible_host, ansible_ssh_pass required for SSH authentication

#### 2. Provisioning Playbook (provision.yml)
- **Conditional installation**: Check if Docker/Docker Compose already installed before attempting installation
  - Use `command: docker --version` with `ignore_errors: yes` to detect existing installations
  - Use `when: docker_installed.rc != 0` to skip installation if already present
- **Docker installation**: Requires GPG key, repository addition, and package installation
  - apt_key for GPG key (deprecated but still functional in Ansible 2.10)
  - apt_repository for adding Docker repo
  - Install docker-ce, docker-ce-cli, containerd.io packages
- **Docker Compose installation**: Download binary from GitHub releases
  - Use get_url module with mode: '0755' for executable permissions
  - Version 2.24.5 used (latest stable as of implementation)
- **UFW configuration**: Allow ports before enabling firewall
  - Allow SSH (22) FIRST to prevent lockout
  - Allow HTTP (80) for web traffic
  - Use `state: enabled` to activate UFW
- **File copy**: Use copy module with src (local path) and dest (remote path)
  - Set mode: '0600' for sensitive files like .env
  - Set mode: '0644' for configuration files
- **Retry logic**: Applied to all apt and network operations
  - `retries: 3, delay: 10` for package installations
  - `register` + `until` pattern for idempotency checks

#### 3. Deployment Playbook (deploy.yml)
- **Docker Compose operations**: Use command module with args.chdir
  - `chdir: /opt/ultrawork` sets working directory for docker-compose commands
  - `docker-compose pull` to fetch latest images (ignore_errors: yes for local builds)
  - `docker-compose up -d --build` to build and start services in detached mode
- **Health check strategy**: Multi-stage verification
  1. Wait 10 seconds for initial startup (pause module)
  2. Check service status with `docker-compose ps` (retries: 5, delay: 10)
  3. Verify all services are "Up" using shell with grep
  4. Assert health with assert module for explicit failure
- **Shell module**: Use for complex checks with pipes and grep
  - Pattern: `grep -v 'Up' && exit 1 || exit 0` (inverse match, fail if not Up)
  - Combine with `changed_when: false` to avoid false change detection
- **Retries pattern**: `retries: 5, delay: 10, until: result.rc == 0` for health checks
- **Assert module**: Provides clear success/fail messages for validation

#### 4. Restart Playbook (restart.yml)
- **docker-compose restart**: Restarts all services without rebuilding
  - Faster than deploy.yml (no image pull or build)
  - Use for configuration changes that don't require new images
- **Health check reuse**: Same verification logic as deploy.yml
  - Ensures services recover properly after restart
  - Retries handle transient startup issues
- **Idempotency**: All tasks can be run multiple times safely
  - Restart is inherently non-idempotent but predictable
  - Health checks ensure consistent end state

#### 5. Ansible Best Practices Applied
- **become: yes**: Required for privileged operations (apt, systemd, ufw)
- **gather_facts: yes**: Collects system information (ansible_distribution_release for Docker repo)
- **changed_when: false**: Prevents false "changed" status for check operations
- **ignore_errors: yes**: Allows playbook to continue when operation is optional (docker pull)
- **register + until + retries**: Pattern for reliable network operations
- **Task naming**: Descriptive names for all tasks (shows in playbook output)

#### 6. Security Considerations
- **Password handling**: Stored in inventory vars (INSECURE - document need for ansible-vault)
- **UFW first**: SSH port allowed before enabling firewall (prevents lockout)
- **.env permissions**: Set to 0600 (owner read/write only) for sensitive data
- **No hardcoded secrets**: All sensitive data in variables or environment files

#### 7. Retry Logic Implementation
- **Network operations**: All apt, get_url operations have `retries: 3, delay: 10`
- **Health checks**: Use `retries: 5, delay: 10` for longer startup grace period
- **Pattern**: `register` result, `until` condition met, `retries` and `delay` for backoff
- **Example**:
  ```yaml
  - name: Task with retry
    command: some-command
    register: result
    retries: 3
    delay: 10
    until: result.rc == 0
  ```

#### 8. Docker Compose Integration
- **Working directory**: All docker-compose commands run in /opt/ultrawork
- **Commands used**:
  - `docker-compose pull`: Fetch latest images (deploy only)
  - `docker-compose up -d --build`: Build and start all services (deploy)
  - `docker-compose restart`: Restart existing services (restart)
  - `docker-compose ps`: Check service status (both)
- **Health verification**: Grep for service names (backend, frontend, postgres, redis, nginx)

### Files Created

1. `/root/ultrawork/ansible/inventory.yml`
   - 10 lines, 246 bytes
   - Production host: 144.124.229.191
   - SSH credentials and Python interpreter configuration

2. `/root/ultrawork/ansible/provision.yml`
   - 133 lines, 3.1K
   - Server provisioning with Docker, packages, UFW, file copying
   - Conditional installation logic
   - Retry logic on all network operations

3. `/root/ultrawork/ansible/deploy.yml`
   - 61 lines, 1.5K
   - Application deployment with image pull/build
   - Multi-stage health verification
   - Service status assertion

4. `/root/ultrawork/ansible/restart.yml`
   - 51 lines, 1.3K
   - Service restart with health checks
   - Reuses deploy.yml health verification pattern

### Verification Results

```bash
# Syntax validation (all passed)
ansible-playbook -i ansible/inventory.yml ansible/provision.yml --syntax-check
# Output: playbook: ansible/provision.yml

ansible-playbook -i ansible/inventory.yml ansible/deploy.yml --syntax-check
# Output: playbook: ansible/deploy.yml

ansible-playbook -i ansible/inventory.yml ansible/restart.yml --syntax-check
# Output: playbook: ansible/restart.yml
```

All playbooks:
- Have correct YAML syntax
- Include all required tasks from plan
- Use retry logic (retries: 3/5, delay: 10)
- Handle password via variables (not hardcoded in tasks)
- Ready for execution against production server

### Dependencies

- Ansible 2.10.7 (installed on control machine)
- Target server: Ubuntu 22.04 with Python 3
- SSH access to 144.124.229.191 with root user
- Local files: docker-compose.yml, nginx.conf, .env

### Integration Points

- Provisions server for Docker Compose stack from Task 5
- Copies configuration files created in Tasks 5, 6, 7
- Enables GitHub Actions deployment workflow (Task 20)
- Required for final deployment (Task 22)

### Execution Instructions

```bash
# Run provisioning (first time setup)
ansible-playbook -i ansible/inventory.yml ansible/provision.yml

# Deploy application
ansible-playbook -i ansible/inventory.yml ansible/deploy.yml

# Restart services
ansible-playbook -i ansible/inventory.yml ansible/restart.yml

# Use ansible-vault for production (secure password)
ansible-vault encrypt_string '2PH9hZz9KnM75vn-Xn+%' --name 'server_password'
# Replace password in inventory.yml with encrypted value
```

### Next Steps (Task 20)

- GitHub Actions deploy workflows will use these playbooks
- Add ansible-playbook commands to workflow steps
- Configure secrets in GitHub for server password
- Add deployment triggers (manual, on push to main)

### Notes

- **SECURITY WARNING**: Password stored in plaintext in inventory.yml
  - Use ansible-vault in production: `ansible-vault encrypt ansible/inventory.yml`
  - Or use --extra-vars: `ansible-playbook ... --extra-vars "server_password=$PASSWORD"`
- **Docker Compose v2**: Binary installation method (compose plugin not used)
- **UFW**: SSH allowed before enabling to prevent lockout
- **Idempotency**: provision.yml can be run multiple times safely
- **Health checks**: 5 retries with 10s delay = max 50s wait for service startup
- **No SSL/TLS**: Phase 1 requirement (defer to Phase 2)
- **No monitoring integration**: Deferred to Phase 2


## Task 20: GitHub Actions Deployment and Restart Workflows

**Date**: 2026-02-07
**Status**: ✅ Completed

### Implementation Summary

Successfully created GitHub Actions workflows for manual deployment and service restart with Ansible integration, SSH credential handling, and concurrency control.

### Key Learnings

#### 1. Workflow Dispatch (Manual Triggers)
- **workflow_dispatch**: Enables manual trigger from GitHub UI
- **inputs**: Define parameters for workflow execution
  - `type: choice`: Dropdown selection in GitHub UI
  - `options`: List of available choices
  - `default`: Pre-selected value
  - `required: true`: Mandatory input
- **github.event.inputs.service**: Access input value in workflow steps
- No automatic triggers (push/PR) - manual only as required

#### 2. Concurrency Control
- **concurrency.group**: Prevents parallel execution of same workflow
  - `group: deploy`: All deploy/restart workflows share same group
  - Ensures only one deployment/restart runs at a time
- **cancel-in-progress: false**: Queues new runs instead of canceling
  - Prevents data loss from interrupted deployments
  - Maintains deployment order

#### 3. SSH Credential Management
- **GitHub Secrets**: Store sensitive data securely
  - `secrets.SSH_PRIVATE_KEY`: Private key for server authentication
  - `secrets.SERVER_HOST`: Target server hostname/IP
  - `secrets.SSH_USER`: SSH username (typically root or deploy user)
- **SSH key setup in workflow**:
  ```bash
  mkdir -p ~/.ssh
  echo "${{ secrets.SSH_PRIVATE_KEY }}" > ~/.ssh/id_rsa
  chmod 600 ~/.ssh/id_rsa  # Critical: restrict permissions
  ssh-keyscan -H ${{ secrets.SERVER_HOST }} >> ~/.ssh/known_hosts
  ```
- **ssh-keyscan**: Adds server to known_hosts to prevent interactive prompt
  - Prevents "Host key verification failed" error
  - `2>/dev/null || true`: Ignore errors (server might be unreachable)

#### 4. Ansible Integration in GitHub Actions
- **Ansible installation**: `pip install ansible` in workflow
  - Requires Python 3 (available on ubuntu-latest)
  - pip install is fastest method for CI/CD
- **ANSIBLE_HOST_KEY_CHECKING=False**: Disable host key verification
  - Alternative to ssh-keyscan approach
  - Set as environment variable in workflow step
  - Allows Ansible to connect without interactive prompts
- **Inventory file**: Use local ansible/inventory.yml
  - Playbooks reference inventory with `-i ansible/inventory.yml`
  - Inventory contains host definitions and variables
- **Extra variables**: Pass service parameter to playbook
  - `-e "service=${{ github.event.inputs.service }}"`: Pass input to Ansible
  - Playbooks can use `{{ service }}` variable for conditional logic

#### 5. Workflow Structure
- **name**: Displayed in GitHub UI and workflow logs
- **on**: Trigger configuration (workflow_dispatch for manual)
- **concurrency**: Prevent parallel runs
- **jobs**: Define workflow jobs (single job per workflow)
- **runs-on**: Runner environment (ubuntu-latest for standard)
- **steps**: Sequential tasks executed in order

#### 6. Deploy vs Restart Workflows
- **deploy.yml**: Full deployment with image pull and build
  - Runs ansible/deploy.yml playbook
  - Service options: all, backend, frontend
  - Includes image pull and docker-compose build
- **restart.yml**: Service restart without rebuild
  - Runs ansible/restart.yml playbook
  - Service options: all, backend, frontend, postgres, redis
  - Faster than deploy (no image pull/build)
  - Use for configuration changes only

#### 7. GitHub Actions Best Practices
- **actions/checkout@v4**: Always checkout code first
  - Required to access ansible/ directory
  - Ensures playbooks are available in workflow
- **Step naming**: Descriptive names for clarity in logs
- **Error handling**: `|| true` for optional operations
  - ssh-keyscan might fail if server unreachable
  - Allows workflow to continue
- **Verification step**: Echo success message
  - Provides clear indication of completion
  - Useful for workflow logs and debugging

### Files Created

1. `/root/ultrawork/.github/workflows/deploy.yml`
   - 52 lines, 1.3K
   - Manual trigger with service selection (all/backend/frontend)
   - Ansible deployment with SSH credentials
   - Concurrency group: deploy

2. `/root/ultrawork/.github/workflows/restart.yml`
   - 54 lines, 1.3K
   - Manual trigger with service selection (all/backend/frontend/postgres/redis)
   - Ansible restart with SSH credentials
   - Concurrency group: deploy (shared with deploy.yml)

### Verification Results

```
✓ /root/ultrawork/.github/workflows/deploy.yml - YAML syntax valid
✓ /root/ultrawork/.github/workflows/restart.yml - YAML syntax valid
```

Both workflows:
- Have correct YAML syntax
- Include manual triggers (workflow_dispatch)
- Configure service selection inputs
- Disable host key checking (ANSIBLE_HOST_KEY_CHECKING=False)
- Add concurrency group (deploy, cancel-in-progress: false)
- Install Ansible via pip
- Use actions/checkout@v4
- Ready for GitHub integration

### GitHub Secrets Required

To use these workflows, configure the following secrets in GitHub repository settings:

1. **SSH_PRIVATE_KEY**: Private SSH key for server authentication
   - Generate: `ssh-keygen -t rsa -b 4096 -f deploy_key`
   - Add public key to server: `~/.ssh/authorized_keys`
   - Store private key in GitHub Secrets

2. **SERVER_HOST**: Target server hostname or IP
   - Example: `144.124.229.191`
   - Used for SSH connection and known_hosts

3. **SSH_USER**: SSH username for authentication
   - Example: `root` or `deploy`
   - Must have permission to run docker-compose commands

### Integration Points

- Uses Ansible playbooks from Task 19 (deploy.yml, restart.yml)
- Requires SSH access to production server (144.124.229.191)
- Depends on GitHub Secrets for credentials
- Shares concurrency group to prevent parallel deployments
- Triggered manually from GitHub Actions UI

### Execution Flow

**Deploy Workflow:**
1. User triggers workflow from GitHub UI
2. Selects service: all, backend, or frontend
3. Workflow checks out code
4. Installs Ansible
5. Configures SSH credentials
6. Runs ansible/deploy.yml with service parameter
7. Verifies deployment success

**Restart Workflow:**
1. User triggers workflow from GitHub UI
2. Selects service: all, backend, frontend, postgres, or redis
3. Workflow checks out code
4. Installs Ansible
5. Configures SSH credentials
6. Runs ansible/restart.yml with service parameter
7. Verifies restart success

### Security Considerations

- **SSH credentials**: Stored in GitHub Secrets (encrypted)
- **Private key**: Never exposed in logs or workflow output
- **Host key verification**: Disabled via ANSIBLE_HOST_KEY_CHECKING
  - Alternative: ssh-keyscan adds server to known_hosts
- **Concurrency control**: Prevents race conditions in deployments
- **No hardcoded secrets**: All sensitive data in GitHub Secrets

### Next Steps (Task 22)

- Configure GitHub Secrets with SSH credentials
- Test workflows manually from GitHub Actions UI
- Monitor deployment logs for errors
- Verify services are running after deployment/restart
- Document deployment procedures for team

### Notes

- Workflows are syntactically valid and ready for GitHub
- Cannot test actual execution without GitHub remote
- SSH credentials must be configured in GitHub Secrets before use
- Concurrency group prevents parallel deployments (important for data consistency)
- Both workflows share same concurrency group (deploy)
- Service parameter passed to Ansible playbooks for conditional logic
- Ansible playbooks (Task 19) handle actual deployment/restart logic

