---
title: "Troubleshooting Guide"
category: "guides"
tags: ["troubleshooting", "debugging", "common-issues", "solutions"]
last_updated: "2026-02-07"
---

# Troubleshooting Guide

## Common Issues and Solutions

### Build Errors

#### Backend Build Failures

**Issue**: Gradle build fails with "Could not resolve dependencies"
```
Solution:
1. Check internet connection
2. Clear Gradle cache: ./gradlew clean --refresh-dependencies
3. Verify build.gradle.kts dependencies are correct
4. Check if Maven Central is accessible
```

**Issue**: Detekt static analysis failures
```
Solution:
1. Run detekt locally: ./gradlew detektMain
2. Review detekt.yml configuration in project root
3. Fix code style violations reported
4. Common violations:
   - Line length exceeds 120 characters
   - Missing KDoc comments on public APIs
   - Unused imports
   - Magic numbers without constants
```

**Issue**: Test failures after code changes
```
Solution:
1. Run tests locally: ./gradlew test
2. Check test logs in build/reports/tests/test/index.html
3. Verify database/Redis services are running
4. Check application-test.yml configuration
5. Ensure test data is properly set up
```

#### Frontend Build Failures

**Issue**: npm install fails with dependency conflicts
```
Solution:
1. Delete node_modules and package-lock.json
2. Run: npm install --legacy-peer-deps
3. Check Node.js version: node --version (should be 22.x)
4. Verify package.json dependencies are compatible
```

**Issue**: ESLint errors during build
```
Solution:
1. Run lint locally: npm run lint
2. Auto-fix issues: npm run lint -- --fix
3. Review .eslintrc.json configuration
4. Common violations:
   - Unused variables
   - Missing semicolons
   - Incorrect import order
   - Console.log statements in production code
```

**Issue**: TypeScript compilation errors
```
Solution:
1. Run type check: tsc --noEmit
2. Check tsconfig.json configuration
3. Verify all imports have correct types
4. Install missing @types packages: npm install --save-dev @types/package-name
5. Clear TypeScript cache: rm -rf .next
```

### Database Connection Issues

**Issue**: Backend cannot connect to PostgreSQL
```
Error: org.postgresql.util.PSQLException: Connection refused
Solution:
1. Verify PostgreSQL is running: docker ps | grep postgres
2. Check connection string in application.yml:
   - Host: localhost (dev) or postgres (Docker)
   - Port: 5432
   - Database: ultrawork
   - Username/password match .env file
3. Test connection: psql -h localhost -U ultrawork -d ultrawork
4. Check PostgreSQL logs: docker logs ultrawork-postgres
```

**Issue**: Database migrations fail
```
Solution:
1. Check Flyway migration files in src/main/resources/db/migration
2. Verify migration naming: V{version}__{description}.sql
3. Check migration history: SELECT * FROM flyway_schema_history;
4. Rollback if needed: ./gradlew flywayClean (WARNING: deletes all data)
5. Rerun migrations: ./gradlew flywayMigrate
```

**Issue**: Connection pool exhausted
```
Error: HikariPool - Connection is not available
Solution:
1. Check application.yml connection pool settings:
   - maximum-pool-size: 10 (increase if needed)
   - connection-timeout: 30000
2. Verify connections are properly closed in code
3. Check for long-running transactions
4. Monitor active connections: SELECT count(*) FROM pg_stat_activity;
```

### Port Conflicts

**Issue**: Backend fails to start - port 8080 already in use
```
Error: Web server failed to start. Port 8080 was already in use.
Solution:
1. Find process using port: lsof -i :8080 (macOS/Linux) or netstat -ano | findstr :8080 (Windows)
2. Kill process: kill -9 <PID>
3. Or change backend port in application.yml:
   server:
     port: 8081
4. Update frontend API_URL in .env.local
```

**Issue**: Frontend fails to start - port 3000 already in use
```
Error: Port 3000 is already in use
Solution:
1. Find process using port: lsof -i :3000 (macOS/Linux)
2. Kill process: kill -9 <PID>
3. Or change frontend port: PORT=3001 npm run dev
4. Update NEXT_PUBLIC_API_URL if needed
```

**Issue**: PostgreSQL port conflict
```
Error: Bind for 0.0.0.0:5432 failed: port is already allocated
Solution:
1. Check if PostgreSQL is already running locally
2. Stop local PostgreSQL: sudo systemctl stop postgresql (Linux)
3. Or change Docker port mapping in docker-compose.yml:
   ports:
     - "5433:5432"
4. Update backend application.yml with new port
```

### Docker Issues

**Issue**: Docker Compose services fail to start
```
Solution:
1. Check Docker daemon is running: docker info
2. Verify docker-compose.yml syntax: docker-compose config
3. Check service logs: docker-compose logs <service-name>
4. Rebuild images: docker-compose up -d --build
5. Remove old containers: docker-compose down -v
```

**Issue**: Out of disk space
```
Error: no space left on device
Solution:
1. Check disk usage: df -h
2. Remove unused images: docker image prune -a
3. Remove unused volumes: docker volume prune
4. Remove unused containers: docker container prune
5. Clean everything: docker system prune -a --volumes (WARNING: removes all unused data)
```

**Issue**: Container health check failing
```
Solution:
1. Check container status: docker ps -a
2. View health check logs: docker inspect <container-id> | grep Health
3. Verify health check command in docker-compose.yml
4. Check service is actually responding:
   - Backend: curl http://localhost:8080/api/health
   - Frontend: curl http://localhost:3000
5. Increase health check interval/timeout if service is slow to start
```

### Redis Connection Issues

**Issue**: Backend cannot connect to Redis
```
Error: Unable to connect to Redis
Solution:
1. Verify Redis is running: docker ps | grep redis
2. Check connection string in application.yml:
   - Host: localhost (dev) or redis (Docker)
   - Port: 6379
3. Test connection: redis-cli -h localhost ping (should return PONG)
4. Check Redis logs: docker logs ultrawork-redis
5. Verify Redis password matches .env file
```

**Issue**: Redis out of memory
```
Error: OOM command not allowed when used memory > 'maxmemory'
Solution:
1. Check Redis memory usage: redis-cli info memory
2. Increase maxmemory in docker-compose.yml:
   command: redis-server --maxmemory 512mb --maxmemory-policy allkeys-lru
3. Clear Redis cache: redis-cli FLUSHALL (WARNING: deletes all data)
4. Review cache eviction policy (allkeys-lru recommended)
```

### CI/CD Issues

**Issue**: GitHub Actions workflow fails
```
Solution:
1. Check workflow logs in GitHub Actions tab
2. Verify workflow syntax: yamllint .github/workflows/*.yml
3. Check secrets are configured in GitHub repository settings:
   - SSH_PRIVATE_KEY
   - SERVER_HOST
   - SSH_USER
4. Test playbooks locally: ansible-playbook -i ansible/inventory.yml ansible/deploy.yml --syntax-check
```

**Issue**: Ansible playbook fails
```
Solution:
1. Run playbook with verbose output: ansible-playbook -vvv ...
2. Check SSH connectivity: ssh user@host
3. Verify inventory.yml has correct host/credentials
4. Check target server has required packages:
   - Docker
   - Docker Compose
   - Python 3
5. Review playbook task that failed and check server logs
```

**Issue**: Deployment succeeds but services not accessible
```
Solution:
1. Check service status on server: docker-compose ps
2. Verify all services are "Up" (not "Restarting")
3. Check UFW firewall rules: sudo ufw status
4. Ensure ports 80, 443, 8080 are allowed
5. Check nginx logs: docker logs ultrawork-nginx
6. Verify DNS/IP address is correct
7. Test locally on server: curl http://localhost
```

### Authentication Issues

**Issue**: JWT token validation fails
```
Error: 401 Unauthorized
Solution:
1. Check JWT_SECRET in backend .env matches frontend
2. Verify token expiration time (default: 1 hour)
3. Check token format in Authorization header: Bearer <token>
4. Inspect token at jwt.io to verify claims
5. Clear browser cookies/localStorage and re-login
```

**Issue**: CORS errors in browser console
```
Error: Access to fetch at 'http://...' from origin 'http://...' has been blocked by CORS policy
Solution:
1. Check CorsConfig.kt allowed origins include frontend URL
2. Verify allowCredentials(true) is set
3. Check frontend API_URL matches backend CORS configuration
4. Clear browser cache and hard reload (Ctrl+Shift+R)
5. Check nginx proxy_pass configuration if using reverse proxy
```

## Getting Help

If you encounter issues not covered here:

1. Check application logs:
   - Backend: `./gradlew bootRun` output or `docker logs ultrawork-backend`
   - Frontend: `npm run dev` output or `docker logs ultrawork-frontend`

2. Review recent changes:
   - `git log --oneline -10`
   - `git diff HEAD~1`

3. Search existing issues:
   - GitHub Issues: https://github.com/ultrawork/backend/issues
   - GitHub Issues: https://github.com/ultrawork/frontend/issues

4. Ask the team:
   - Slack: #ultrawork-dev channel
   - Email: dev@ultrawork.com

5. Create a new issue:
    - Include error messages, logs, and steps to reproduce
    - Specify environment (dev/staging/production)
    - Attach relevant configuration files (redact secrets!)

## Related Documentation

- [Guides Overview](./README.md) - All guides and tutorials
- [Getting Started](./getting-started.md) - Initial setup guide
- [Conventions](../conventions/README.md) - Coding standards
- [Deployment](../deployment/README.md) - Deployment procedures

---

[Back to Guides](./README.md) | [Back to main docs](../README.md)
