# Ultrawork

Enterprise-grade AI-powered task management and execution platform.

## Overview

Ultrawork is a comprehensive monorepo containing documentation, infrastructure configurations, and orchestration workflows for the Ultrawork platform.

## Repository Structure

```
ultrawork/
├── docs/                    # 📚 Comprehensive knowledge base
│   ├── architecture/        # System design and architecture
│   ├── decisions/           # Architecture Decision Records (ADRs)
│   ├── conventions/         # Coding standards and workflows
│   ├── deployment/          # Deployment procedures
│   ├── guides/              # Getting started and troubleshooting
│   └── api/                 # API documentation
├── ansible/                 # 🔧 Infrastructure automation
│   ├── inventory.yml        # Server inventory
│   ├── provision.yml        # Server provisioning
│   ├── deploy.yml           # Application deployment
│   └── restart.yml          # Service restart
├── .github/workflows/       # 🚀 CI/CD workflows
│   ├── deploy.yml           # Manual deployment trigger
│   └── restart.yml          # Service restart trigger
└── .sisyphus/               # 📋 Task planning and learnings
    ├── plans/               # Task execution plans
    └── notepads/            # Accumulated knowledge
```

## Related Repositories

- **[ultrawork/backend](https://github.com/ultrawork/backend)** - Kotlin/Spring Boot backend
- **[ultrawork/frontend](https://github.com/ultrawork/frontend)** - Next.js/React frontend
- **[ultrawork/infrastructure](https://github.com/ultrawork/infrastructure)** - Deployment automation

## Documentation

📖 **Start here:** [docs/README.md](docs/README.md)

### Quick Links

- [Architecture Overview](docs/architecture/README.md)
- [Getting Started Guide](docs/guides/getting-started.md)
- [API Documentation](docs/api/README.md)
- [Coding Conventions](docs/conventions/README.md)
- [Architecture Decisions](docs/decisions/README.md)

## Tech Stack

### Backend
- **Language:** Kotlin 2.0.21
- **Framework:** Spring Boot 3.4.0
- **Database:** PostgreSQL 16
- **Cache:** Redis 7
- **Runtime:** Java 21

### Frontend
- **Framework:** Next.js 15
- **UI Library:** React 19
- **Language:** TypeScript 5
- **Styling:** Tailwind CSS + Radix UI

### Infrastructure
- **Containerization:** Docker + Docker Compose
- **Orchestration:** Ansible
- **CI/CD:** GitHub Actions
- **Server:** Ubuntu 22.04

## Deployment

### Prerequisites

- Ansible 2.10.7+
- SSH access to production server
- GitHub CLI (for workflow triggers)

### Deploy Application

```bash
# Option 1: Manual Ansible
ansible-playbook -i ansible/inventory.yml ansible/deploy.yml

# Option 2: GitHub Actions Workflow
gh workflow run "Deploy Application" --repo ultrawork/ultrawork
```

### Restart Services

```bash
# Option 1: Manual Ansible
ansible-playbook -i ansible/inventory.yml ansible/restart.yml

# Option 2: GitHub Actions Workflow
gh workflow run "Restart Services" --repo ultrawork/ultrawork
```

## Development Workflow

1. **Backend Development:** See [backend repository](https://github.com/ultrawork/backend)
2. **Frontend Development:** See [frontend repository](https://github.com/ultrawork/frontend)
3. **Documentation Updates:** Edit files in `docs/` directory
4. **Infrastructure Changes:** Update Ansible playbooks in `ansible/` directory

## Contributing

### Code Quality Standards

- **Backend:** Detekt with `maxIssues: 0` (zero-tolerance)
- **Frontend:** ESLint with `--max-warnings 0` (zero-tolerance)
- **All Commits:** Follow [Conventional Commits](docs/conventions/git-workflow.md)

### Documentation

All major architectural decisions are documented in [Architecture Decision Records](docs/decisions/README.md).

## Architecture Decision Records

Key technical decisions documented:

- [ADR-0001: Use Kotlin and Spring Boot](docs/decisions/0001-use-kotlin-spring-boot.md)
- [ADR-0002: Use Next.js and React](docs/decisions/0002-use-nextjs-react.md)
- [ADR-0003: Strict Linting with Zero Tolerance](docs/decisions/0003-strict-linting-zero-tolerance.md)

## License

Proprietary - All rights reserved

## Contact

For questions or support, please contact the development team.

---

**Built with ❤️ by the Ultrawork team**
