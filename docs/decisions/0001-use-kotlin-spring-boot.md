# 0001. Use Kotlin and Spring Boot for Backend

**Status**: Accepted  
**Date**: 2026-02-07  
**Deciders**: Ultrawork Team

## Context

The Ultrawork project management platform requires a robust, scalable backend capable of handling:
- RESTful API endpoints for task management, user authentication, and system monitoring
- Database operations with PostgreSQL for persistent storage
- Redis integration for caching and session management
- Real-time system metrics collection and monitoring
- Secure authentication with JWT tokens and Argon2 password hashing
- Production-grade exception handling and validation

We needed a technology stack that provides:
- Strong type safety to prevent runtime errors
- Mature ecosystem with extensive libraries for enterprise features
- Excellent tooling for testing, linting, and static analysis
- High performance and scalability for production workloads
- Developer productivity with concise, expressive syntax

## Decision

We will use **Kotlin** as the primary programming language and **Spring Boot 3.4.0** as the application framework for the backend.

**Key Technologies:**
- **Language**: Kotlin 2.0.21 with Java 21 runtime
- **Framework**: Spring Boot 3.4.0
- **Build Tool**: Gradle with Kotlin DSL
- **Database**: PostgreSQL 16 with Flyway migrations
- **ORM**: Spring Data JPA with Hibernate
- **Caching**: Spring Data Redis
- **Security**: Spring Security with JWT (jjwt 0.12.5) and Argon2 password encoding
- **Validation**: Jakarta Validation API (Bean Validation)
- **Testing**: JUnit 5, MockK, Testcontainers
- **Static Analysis**: Detekt 1.23.8 with zero-tolerance policy (maxIssues: 0)

## Consequences

### Positive
- **Type Safety**: Kotlin's null safety eliminates entire classes of runtime errors (NullPointerException)
- **Concise Syntax**: Data classes, extension functions, and smart casts reduce boilerplate by ~40% compared to Java
- **Spring Boot Ecosystem**: Access to mature libraries for security, data access, validation, and monitoring
- **Excellent Tooling**: IntelliJ IDEA provides best-in-class Kotlin support with refactoring, debugging, and code analysis
- **Coroutines Support**: Built-in support for asynchronous programming (future-ready for Phase 2 WebSocket features)
- **Java Interoperability**: Seamless integration with Java libraries (PostgreSQL driver, JWT, Argon2)
- **Production-Ready**: Spring Boot Actuator provides health checks, metrics, and monitoring out-of-the-box
- **Testing Infrastructure**: Testcontainers enables realistic integration tests with PostgreSQL and Redis
- **Static Analysis**: Detekt enforces code quality standards automatically (max line length, cyclomatic complexity, naming conventions)

### Negative
- **Learning Curve**: Team members unfamiliar with Kotlin require onboarding time
- **Build Times**: Kotlin compilation slower than Java (mitigated by Gradle incremental compilation)
- **Ecosystem Maturity**: Some Spring Boot examples/tutorials use Java, requiring translation to Kotlin
- **JVM Dependency**: Requires Java 21 runtime, increasing deployment artifact size (~150MB base image)

### Neutral
- **Opinionated Framework**: Spring Boot's conventions reduce flexibility but increase consistency
- **Annotation-Heavy**: Spring relies heavily on annotations (@Service, @RestController, @Transactional)
- **Reflection Usage**: Spring's dependency injection uses reflection, impacting startup time (~5-10 seconds)

## Alternatives Considered

### Option 1: Java 21 with Spring Boot
- **Pros**: Larger talent pool, more tutorials/examples, slightly faster compilation
- **Cons**: Verbose syntax, no null safety, more boilerplate code, less modern language features
- **Why rejected**: Kotlin's null safety and concise syntax provide significant productivity gains without sacrificing Spring Boot ecosystem

### Option 2: Node.js with TypeScript and NestJS
- **Pros**: JavaScript ecosystem, faster startup time, smaller deployment artifacts
- **Cons**: Weaker type system, less mature ORM (TypeORM vs Hibernate), fewer enterprise libraries, single-threaded event loop limits CPU-bound operations
- **Why rejected**: Spring Boot's maturity and Kotlin's type safety better suited for enterprise task management platform

### Option 3: Go with Gin/Echo framework
- **Pros**: Fast compilation, small binaries, excellent concurrency primitives
- **Cons**: No ORM comparable to Hibernate, manual dependency injection, less mature ecosystem for enterprise features (validation, security)
- **Why rejected**: Lack of mature ORM and enterprise libraries would require significant custom development

### Option 4: Rust with Actix-web
- **Pros**: Memory safety, zero-cost abstractions, excellent performance
- **Cons**: Steep learning curve, immature ecosystem for enterprise features, longer development time
- **Why rejected**: Team expertise and time-to-market constraints favor established JVM ecosystem
