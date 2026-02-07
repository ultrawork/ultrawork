---
title: "API Documentation"
category: "api"
tags: ["api", "rest", "endpoints", "authentication", "backend"]
last_updated: "2026-02-07"
---

# API Documentation

## Base URL
- Development: `http://localhost:8080/api`
- Production: `http://144.124.229.191/api`

## Authentication
- JWT-based authentication
- Cookie: `access_token` (HttpOnly, 24h expiry)
- Secure flag: `false` (development), should be `true` in production

## Response Format

All API responses follow this structure:

```json
{
  "data": { ... },
  "error": null,
  "timestamp": "2026-02-07T12:34:56.789Z"
}
```

### Success Response
```json
{
  "data": {
    "id": "123e4567-e89b-12d3-a456-426614174000",
    "username": "user@example.com"
  },
  "error": null,
  "timestamp": "2026-02-07T12:34:56.789Z"
}
```

### Error Response
```json
{
  "data": null,
  "error": {
    "code": "INVALID_CREDENTIALS",
    "message": "Invalid username or password"
  },
  "timestamp": "2026-02-07T12:34:56.789Z"
}
```

## API Endpoints

### Authentication (`/api/v1/auth`)

#### `POST /api/v1/auth/register`
Register a new user account.

**Request Body:**
```json
{
  "username": "user@example.com",
  "password": "SecurePassword123!",
  "fullName": "John Doe"
}
```

**Response:** `201 Created`
```json
{
  "data": {
    "id": "123e4567-e89b-12d3-a456-426614174000",
    "username": "user@example.com",
    "fullName": "John Doe",
    "role": "USER"
  },
  "error": null,
  "timestamp": "2026-02-07T12:34:56.789Z"
}
```

**Errors:**
- `409 Conflict` - `DUPLICATE`: User already exists

---

#### `POST /api/v1/auth/login`
Authenticate user and receive access token.

**Request Body:**
```json
{
  "username": "user@example.com",
  "password": "SecurePassword123!"
}
```

**Response:** `200 OK`
```json
{
  "data": {
    "id": "123e4567-e89b-12d3-a456-426614174000",
    "username": "user@example.com",
    "fullName": "John Doe",
    "role": "USER"
  },
  "error": null,
  "timestamp": "2026-02-07T12:34:56.789Z"
}
```

**Errors:**
- `401 Unauthorized` - `INVALID_CREDENTIALS`: Invalid username or password
- `429 Too Many Requests` - `ACCOUNT_LOCKED`: Account locked due to too many failed attempts (15 minutes lockout)
  - Header: `Retry-After: 900` (seconds)

---

#### `POST /api/v1/auth/logout`
Logout user and invalidate token.

**Response:** `200 OK`
```json
{
  "data": null,
  "error": null,
  "timestamp": "2026-02-07T12:34:56.789Z"
}
```

---

#### `GET /api/v1/auth/me`
Get current authenticated user information.

**Authentication:** Required

**Response:** `200 OK`
```json
{
  "data": {
    "id": "123e4567-e89b-12d3-a456-426614174000",
    "username": "user@example.com",
    "fullName": "John Doe",
    "role": "USER"
  },
  "error": null,
  "timestamp": "2026-02-07T12:34:56.789Z"
}
```

**Errors:**
- `401 Unauthorized` - `UNAUTHORIZED`: Not authenticated

---

### Health Check (`/api/v1/health`)

#### `GET /api/v1/health`
Check system health status.

**Response:** `200 OK`
```json
{
  "data": {
    "status": "UP",
    "version": "0.1.0",
    "components": {
      "db": "UP",
      "redis": "UP",
      "disk": "UP",
      "jvm": "UP"
    }
  },
  "error": null,
  "timestamp": "2026-02-07T12:34:56.789Z"
}
```

**Status Values:**
- `UP` - All components operational
- `DEGRADED` - Some components down but system functional

---

### Admin - System Info (`/api/v1/admin`)

#### `GET /api/v1/admin/system-info`
Get detailed system information.

**Authentication:** Required (OWNER role)

**Response:** `200 OK`
```json
{
  "data": {
    "hostname": "ultrawork-server",
    "osName": "Linux",
    "osVersion": "5.15.0",
    "javaVersion": "21.0.1",
    "uptime": 3600000,
    "cpuCores": 4,
    "totalMemory": 8589934592,
    "freeMemory": 4294967296
  },
  "error": null,
  "timestamp": "2026-02-07T12:34:56.789Z"
}
```

---

#### `GET /api/v1/admin/system-metrics/history`
Get historical system metrics.

**Authentication:** Required (OWNER role)

**Response:** `200 OK`
```json
{
  "data": [
    {
      "timestamp": "2026-02-07T12:00:00Z",
      "cpuUsage": 45.2,
      "memoryUsage": 67.8,
      "diskUsage": 34.5,
      "activeConnections": 12
    }
  ],
  "error": null,
  "timestamp": "2026-02-07T12:34:56.789Z"
}
```

---

### Projects (`/api/v1/projects`)

**Authentication:** Required for all endpoints

All project endpoints return `501 Not Implemented` with:
```json
{
  "data": null,
  "error": {
    "code": "NOT_IMPLEMENTED",
    "message": "Будет доступно в Фазе 2"
  },
  "timestamp": "2026-02-07T12:34:56.789Z"
}
```

**Endpoints:**
- `GET /api/v1/projects` - List all projects
- `POST /api/v1/projects` - Create new project
- `GET /api/v1/projects/{id}` - Get project details
- `PUT /api/v1/projects/{id}` - Update project
- `DELETE /api/v1/projects/{id}` - Delete project
- `GET /api/v1/projects/{id}/interview` - Get project interview
- `PUT /api/v1/projects/{id}/interview` - Update project interview
- `GET /api/v1/projects/{id}/documents` - Get project documents (Phase 3)
- `GET /api/v1/projects/{id}/tasks` - List project tasks
- `POST /api/v1/projects/{id}/tasks` - Create task in project

---

### Tasks (`/api/v1/tasks`)

**Authentication:** Required for all endpoints

All task endpoints return `501 Not Implemented` (Phase 2).

**Endpoints:**
- `GET /api/v1/tasks/{id}` - Get task details
- `PUT /api/v1/tasks/{id}` - Update task
- `DELETE /api/v1/tasks/{id}` - Delete task
- `POST /api/v1/tasks/{id}/transition` - Transition task status
- `GET /api/v1/tasks/{id}/comments` - Get task comments
- `POST /api/v1/tasks/{id}/comments` - Add comment to task
- `GET /api/v1/tasks/{id}/status-log` - Get task status history
- `POST /api/v1/tasks/{id}/execute` - Execute task

---

### Executions (`/api/v1/executions`)

**Authentication:** Required for all endpoints

All execution endpoints return `501 Not Implemented` (Phase 2).

**Endpoints:**
- `GET /api/v1/executions/{id}` - Get execution details
- `GET /api/v1/executions/{id}/logs` - Get execution logs

---

### Search (`/api/v1/search`)

**Authentication:** Required

Returns `501 Not Implemented` (Phase 3).

**Endpoints:**
- `GET /api/v1/search` - Search across projects and tasks

---

### Analytics (`/api/v1/usage`)

**Authentication:** Required for all endpoints

All analytics endpoints return `501 Not Implemented` (Phase 3).

**Endpoints:**
- `GET /api/v1/usage/summary` - Get usage summary
- `GET /api/v1/usage/projects/{id}` - Get project-specific usage

---

### Notifications (`/api/v1/notifications`)

**Authentication:** Required for all endpoints

All notification endpoints return `501 Not Implemented` (Phase 2).

**Endpoints:**
- `GET /api/v1/notifications` - List notifications
- `POST /api/v1/notifications/{id}/read` - Mark notification as read

---

### Settings (`/api/v1/settings`)

**Authentication:** Required for all endpoints

All settings endpoints return `501 Not Implemented` (Phase 2).

**Endpoints:**
- `GET /api/v1/settings/providers` - Get AI providers
- `POST /api/v1/settings/providers` - Add AI provider
- `GET /api/v1/settings/runtimes` - Get runtime configurations

---

### Instance Management (`/api/v1/instance`)

**Authentication:** Required (OWNER role)

All instance endpoints return `501 Not Implemented` (Phase 3).

**Endpoints:**
- `POST /api/v1/instance/backup` - Create system backup
- `GET /api/v1/instance/config` - Get instance configuration

---

## Error Codes

| Code | HTTP Status | Description |
|------|-------------|-------------|
| `INVALID_CREDENTIALS` | 401 | Invalid username or password |
| `ACCOUNT_LOCKED` | 429 | Account locked due to too many failed login attempts |
| `DUPLICATE` | 409 | Resource already exists (e.g., username taken) |
| `UNAUTHORIZED` | 401 | Not authenticated or invalid token |
| `NOT_IMPLEMENTED` | 501 | Feature not yet implemented |

---

## Security Notes

1. **Cookie Security:**
   - `HttpOnly` flag prevents JavaScript access
   - `Secure` flag should be enabled in production
   - 24-hour expiration (86400 seconds)
   - Path: `/` (site-wide)

2. **Account Lockout:**
   - Triggered after multiple failed login attempts
   - 15-minute lockout period (900 seconds)
   - `Retry-After` header indicates wait time

3. **Role-Based Access:**
    - `USER` - Standard user access
    - `OWNER` - Administrative access to system info, metrics, and instance management

## Related Documentation

- [Architecture](../architecture/README.md) - System design and components
- [Conventions](../conventions/README.md) - Coding standards and practices
- [Deployment](../deployment/README.md) - Deployment procedures
- [ADR-0001: Kotlin and Spring Boot](../decisions/0001-use-kotlin-spring-boot.md) - Backend technology decision

---

[Back to main docs](../README.md)
