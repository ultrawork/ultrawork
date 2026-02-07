# Architecture Decision Records

This directory contains Architecture Decision Records (ADRs) documenting significant technical decisions made during the Ultrawork project development.

## What are ADRs?

Architecture Decision Records capture important architectural decisions along with their context and consequences. Each ADR describes:
- **Context**: The issue motivating the decision
- **Decision**: The change being proposed or implemented
- **Consequences**: The positive, negative, and neutral impacts
- **Alternatives**: Other options considered and why they were rejected

## Index

- [ADR-0001: Use Kotlin and Spring Boot for Backend](./0001-use-kotlin-spring-boot.md)
- [ADR-0002: Use Next.js and React for Frontend](./0002-use-nextjs-react.md)
- [ADR-0003: Strict Linting with Zero Tolerance](./0003-strict-linting-zero-tolerance.md)

## Process

### Creating a New ADR

1. **Copy the template**: Use `template.md` as the starting point
   ```bash
   cp docs/decisions/template.md docs/decisions/NNNN-descriptive-title.md
   ```

2. **Number sequentially**: Use the next available number (0001, 0002, 0003, ...)

3. **Fill in the sections**:
   - **Status**: Start with "Proposed", change to "Accepted" when approved
   - **Date**: Use YYYY-MM-DD format
   - **Context**: Explain the problem and constraints
   - **Decision**: Be specific and concrete
   - **Consequences**: List positive, negative, and neutral impacts
   - **Alternatives**: Document options considered and why they were rejected

4. **Update this index**: Add a link to the new ADR in the Index section above

5. **Commit with the ADR**: Include the ADR in the same commit as the implementation
   ```bash
   git add docs/decisions/NNNN-descriptive-title.md docs/decisions/README.md
   git commit -m "Add ADR-NNNN: Descriptive title"
   ```

### ADR Statuses

- **Proposed**: Decision under discussion, not yet approved
- **Accepted**: Decision approved and implemented
- **Rejected**: Decision considered but not adopted
- **Deprecated**: Decision no longer relevant (superseded by newer decision)
- **Superseded**: Decision replaced by a newer ADR (link to replacement)

### When to Create an ADR

Create an ADR for decisions that:
- Impact system architecture or design patterns
- Affect multiple components or teams
- Involve significant trade-offs
- Are difficult or expensive to reverse
- Require explanation for future team members

**Examples:**
- Technology stack choices (language, framework, database)
- Architectural patterns (microservices, monolith, event-driven)
- Security approaches (authentication, authorization, encryption)
- Development practices (testing strategy, linting policy, deployment process)

### When NOT to Create an ADR

Skip ADRs for:
- Minor implementation details (variable naming, file organization)
- Temporary decisions (workarounds, experiments)
- Obvious choices with no alternatives (using Git for version control)

## Template

See [template.md](./template.md) for the ADR template based on Michael Nygard's format.

---

[Link back to main docs](../README.md)
