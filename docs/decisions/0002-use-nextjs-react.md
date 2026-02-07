---
title: "ADR-0002: Use Next.js and React for Frontend"
category: "decisions"
tags: ["frontend", "nextjs", "react", "typescript", "adr", "accepted"]
status: "Accepted"
date: "2026-02-07"
last_updated: "2026-02-07"
---

# 0002. Use Next.js and React for Frontend

**Status**: Accepted  
**Date**: 2026-02-07  
**Deciders**: Ultrawork Team

## Context

The Ultrawork project management platform requires a modern, responsive frontend capable of:
- Server-side rendering (SSR) for improved SEO and initial page load performance
- Client-side routing for seamless navigation without full page reloads
- Component-based architecture for reusable UI elements (task cards, modals, forms)
- Type-safe development to prevent runtime errors
- Drag-and-drop functionality for task management (Kanban boards)
- Form validation with schema-based validation
- State management for user authentication, task data, and UI state
- Responsive design with utility-first CSS framework
- Production-grade testing infrastructure (unit, integration, E2E)

We needed a technology stack that provides:
- Strong TypeScript support for type safety
- Mature ecosystem with extensive UI component libraries
- Excellent developer experience with hot module replacement and fast refresh
- Built-in routing and data fetching patterns
- SEO-friendly rendering strategies (SSR, SSG, ISR)
- Production-ready performance optimizations (code splitting, image optimization)

## Decision

We will use **Next.js 15** as the React framework and **TypeScript** as the primary language for the frontend.

**Key Technologies:**
- **Framework**: Next.js 15.5.12 with App Router
- **Language**: TypeScript 5 with strict mode
- **UI Library**: React 19.1.0
- **Styling**: Tailwind CSS 4 with utility-first approach
- **Component Library**: Radix UI (headless components) + shadcn/ui patterns
- **Form Handling**: React Hook Form 7.71.1 with Zod 4.3.6 schema validation
- **State Management**: Zustand 5.0.11 for global state
- **Drag-and-Drop**: @dnd-kit/core 6.3.1 and @dnd-kit/sortable 10.0.0
- **Icons**: Lucide React 0.563.0
- **Testing**: Vitest 4.0.18 (unit), Testing Library (integration), Playwright 1.58.2 (E2E)
- **Linting**: ESLint 9 with Airbnb TypeScript config, zero-tolerance policy (max-warnings: 0)
- **Formatting**: Prettier 3.8.1

## Consequences

### Positive
- **Type Safety**: TypeScript with strict mode eliminates entire classes of runtime errors
- **Server-Side Rendering**: Next.js App Router provides SSR out-of-the-box, improving SEO and initial load performance
- **Developer Experience**: Fast Refresh enables instant feedback during development (sub-second updates)
- **Built-in Routing**: File-system based routing eliminates need for separate routing library
- **Image Optimization**: Next.js Image component automatically optimizes images (WebP conversion, lazy loading, responsive sizes)
- **Code Splitting**: Automatic code splitting reduces initial bundle size by ~60%
- **Tailwind CSS**: Utility-first approach enables rapid UI development with consistent design system
- **Radix UI**: Headless components provide accessibility (ARIA attributes, keyboard navigation) out-of-the-box
- **React Hook Form + Zod**: Type-safe form validation with minimal re-renders (performance optimization)
- **Zustand**: Lightweight state management (~1KB) with TypeScript inference and minimal boilerplate
- **@dnd-kit**: Modern drag-and-drop library with accessibility support and touch device compatibility
- **Testing Infrastructure**: Vitest provides fast unit tests (~10x faster than Jest), Playwright enables reliable E2E tests
- **Strict Linting**: ESLint with Airbnb config enforces best practices (explicit return types, no `any`, max line length 120)

### Negative
- **Learning Curve**: Next.js App Router (new in v13+) requires learning server/client component patterns
- **Build Complexity**: Next.js build process more complex than Create React App (server/client bundle separation)
- **Bundle Size**: React 19 + Next.js base bundle ~85KB gzipped (larger than Vue/Svelte alternatives)
- **Hydration Overhead**: SSR requires client-side hydration, adding ~200ms to time-to-interactive
- **Tailwind Verbosity**: Utility classes can become verbose for complex components (mitigated by component extraction)

### Neutral
- **Opinionated Framework**: Next.js conventions reduce flexibility but increase consistency across team
- **React Ecosystem**: Large ecosystem means more choices (state management, form libraries, styling solutions)
- **Node.js Dependency**: Requires Node.js 22 runtime for development and build process

## Alternatives Considered

### Option 1: Create React App (CRA) with React Router
- **Pros**: Simpler setup, no SSR complexity, smaller learning curve
- **Cons**: No SSR (poor SEO), manual routing setup, no built-in image optimization, slower build times
- **Why rejected**: Lack of SSR and built-in optimizations would require significant custom development for production-grade application

### Option 2: Vue 3 with Nuxt 3
- **Pros**: Smaller bundle size (~50KB), simpler reactivity model, less verbose templates
- **Cons**: Smaller ecosystem, fewer UI component libraries, less TypeScript maturity, smaller talent pool
- **Why rejected**: React's larger ecosystem and team expertise favor React/Next.js

### Option 3: Svelte with SvelteKit
- **Pros**: No virtual DOM (faster runtime), smallest bundle size (~30KB), simpler syntax
- **Cons**: Immature ecosystem, fewer component libraries, less TypeScript support, smaller talent pool
- **Why rejected**: Ecosystem immaturity and lack of enterprise-grade component libraries (Radix UI equivalent)

### Option 4: Angular 17 with SSR
- **Pros**: Full-featured framework (routing, forms, HTTP client built-in), strong TypeScript support
- **Cons**: Steeper learning curve, more boilerplate, larger bundle size (~120KB), opinionated architecture
- **Why rejected**: React's component-based approach and Next.js optimizations provide better developer experience and performance

### Option 5: Remix
- **Pros**: Excellent data loading patterns, nested routing, progressive enhancement
- **Cons**: Smaller ecosystem, fewer examples/tutorials, less mature than Next.js
- **Why rejected**: Next.js App Router provides similar benefits with larger ecosystem and more mature tooling

## Related Documentation

- [ADR-0001: Use Kotlin and Spring Boot for Backend](./0001-use-kotlin-spring-boot.md)
- [ADR-0003: Strict Linting with Zero Tolerance](./0003-strict-linting-zero-tolerance.md)
- [Coding Style Guide](../conventions/coding-style.md) - Frontend implementation details
- [Testing Standards](../conventions/testing-standards.md) - Frontend testing practices

---

[Back to Architecture Decisions](./README.md) | [Back to main docs](../README.md)
