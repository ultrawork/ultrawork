---
title: "Getting Started with Ultrawork"
category: "guides"
tags: ["getting-started", "setup", "installation", "quickstart"]
last_updated: "2026-02-07"
---

# Getting Started with Ultrawork

## Prerequisites
- Java 21 (Temurin recommended)
- Node.js 22
- PostgreSQL 16
- Redis 7
- Git

## Quick Start

### 1. Clone Repositories
```bash
git clone https://github.com/ultrawork/backend.git
git clone https://github.com/ultrawork/frontend.git
```

### 2. Backend Setup
```bash
cd backend
./gradlew bootRun
```

### 3. Frontend Setup
```bash
cd frontend
npm install
npm run dev
```

### 4. Access Application
- Frontend: http://localhost:3000
- Backend API: http://localhost:8080/api

## Next Steps
- Read [Architecture](../architecture/README.md)
- Review [Conventions](../conventions/README.md)
- Check [Deployment Guide](../deployment/README.md)
- Explore [API Documentation](../api/README.md)

## Related Documentation

- [Guides Overview](./README.md) - All guides and tutorials
- [Troubleshooting](./troubleshooting.md) - Common issues and solutions
- [Conventions](../conventions/README.md) - Coding standards
- [Architecture](../architecture/README.md) - System design

---

[Back to Guides](./README.md) | [Back to main docs](../README.md)
