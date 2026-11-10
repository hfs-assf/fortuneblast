# ADR-001: Adopt Monorepo Architecture

## Status

Accepted

## Date

2026-11-10

## Context

The Laundry Management System consists of multiple
applications:

- Mobile application using Expo and React Native.
- Backend API using NestJS.
- Shared TypeScript package.

These applications have different deployment targets
but belong to the same business product.

Maintaining multiple repositories would introduce
additional repository management and coordination.

## Decision

We will use a single Git repository with npm Workspaces.

Repository structure:

- apps/api
- apps/mobile
- packages/shared

## Consequences

### Positive

- Centralized source code.
- Shared TypeScript definitions.
- Easier cross-application changes.
- Unified dependency management.
- Consistent development standards.
- Easier local development.

### Negative

- Repository contains multiple applications.
- CI/CD requires path-based workflow filtering.
- Dependency management requires workspace awareness.
- Teams must maintain clear application boundaries.

## Deployment Strategy

Applications will be deployed independently.

Mobile:

- Expo EAS Build.
- Google Play Store.

Backend:

- Docker image.
- DigitalOcean.

Database:

- PostgreSQL.
- Private cloud infrastructure.

## Alternatives Considered

1. Separate repositories for mobile and backend.
2. Single repository with npm Workspaces.

The second option was selected because it provides
simpler coordination and shared code management.
