# ADR-003: Independent Deployment Strategy

**Status:** Accepted  
**Date:** 2026-11-10  
**Decision Owners:** Project Team

## 1. Context

The Laundry Management System will use a monorepo containing multiple applications and shared packages.

The project has two primary applications:

- Mobile application: Expo, React Native and TypeScript.
- Backend API: NestJS and TypeScript.

Both applications share some code through a shared package, such as validation schemas, TypeScript types and API contract definitions.

Although the applications live in one Git repository, they have different build processes, release schedules and deployment targets.

The mobile application will be distributed through the Google Play Store, while the backend API will be hosted on DigitalOcean.

## 2. Decision

We will use a **monorepo with independent CI/CD pipelines**.

Each application will be built, tested and deployed independently.

The repository structure will be:

```text
laundry/
├── apps/
│   ├── api/
│   └── mobile/
│
├── packages/
│   └── shared/
│
├── infra/
│   ├── docker/
│   └── deployment/
│
├── docs/
├── scripts/
├── .github/
│   └── workflows/
│
├── package.json
└── README.md
```

## 3. Deployment Architecture

### 3.1 Mobile application

Technology:

- Expo
- React Native
- TypeScript
- Expo Application Services (EAS)

Deployment flow:

```text
Developer
   |
   v
GitHub Repository
   |
   v
Mobile CI Pipeline
   |
   v
Install Dependencies
   |
   v
Lint and Type Check
   |
   v
Run Tests
   |
   v
EAS Build
   |
   v
Google Play Store
   |
   v
Android Users
```

The mobile application will communicate with the backend through HTTPS REST APIs.

The mobile application must never connect directly to PostgreSQL.

### 3.2 Backend API

Technology:

- NestJS
- TypeScript
- Docker
- PostgreSQL
- Prisma

Deployment flow:

```text
Developer
   |
   v
GitHub Repository
   |
   v
API CI Pipeline
   |
   v
Install Dependencies
   |
   v
Lint and Type Check
   |
   v
Run Tests
   |
   v
Build NestJS Application
   |
   v
Build Docker Image
   |
   v
Deploy to DigitalOcean
   |
   v
HTTPS API
```

The backend will connect to PostgreSQL through a private database connection.

The initial deployment may use a DigitalOcean Droplet with Docker. Managed PostgreSQL can be adopted for production to reduce database administration responsibilities.

## 4. Independent CI/CD Pipelines

The repository will have separate workflows.

```text
.github/workflows/
├── api-ci.yml
├── api-deploy.yml
├── mobile-ci.yml
└── mobile-release.yml
```

### Pipeline responsibilities

| Pipeline       | Trigger                                   | Responsibility                          |
| -------------- | ----------------------------------------- | --------------------------------------- |
| API CI         | API or relevant shared package changes    | Validate and test backend               |
| API Deploy     | Approved API release                      | Build and deploy backend                |
| Mobile CI      | Mobile or relevant shared package changes | Validate and test mobile                |
| Mobile Release | Approved mobile release                   | Build and distribute mobile application |

Path filters should be used to avoid running unnecessary pipelines.

For example:

- Changes under `apps/api/**` trigger API workflows.
- Changes under `apps/mobile/**` trigger mobile workflows.
- Changes under `packages/shared/**` trigger the workflows for applications that consume the changed package.

A shared package change may therefore trigger both pipelines.

## 5. Environment Management

We will separate environments where appropriate:

| Environment | Purpose                             |
| ----------- | ----------------------------------- |
| Local       | Developer workstation               |
| Development | Integration and development testing |
| Production  | Live application                    |

Sensitive values must be stored in environment variables or the relevant CI/CD secret store.

Examples include:

- Database connection strings.
- JWT secrets.
- API credentials.
- Deployment credentials.

Secrets must never be committed to Git or embedded in the mobile application.

## 6. Database Migration Strategy

Database schema changes will be managed by Prisma Migrate.

Migrations must be reviewed and applied as a controlled backend release step.

The mobile application and backend may be released at different times. Therefore, API changes should maintain backward compatibility whenever possible.

For breaking API changes, use a planned migration strategy rather than assuming both applications update simultaneously.

## 7. Alternatives Considered

| Strategy                              | Advantages                                     | Disadvantages                                               |
| ------------------------------------- | ---------------------------------------------- | ----------------------------------------------------------- |
| Monorepo with independent deployment  | Shared code, one repository, separate releases | Requires pipeline path filters and dependency management    |
| Separate repositories                 | Strong repository isolation                    | More repository administration and shared-code coordination |
| Monorepo with one deployment pipeline | Simple initial setup                           | Unnecessary builds and releases when only one app changes   |

## 8. Consequences

**Positive consequences**

- One source of truth for the project.
- Shared types and validation schemas.
- Independent mobile and backend release schedules.
- Less duplicated configuration.
- Easier coordination of API contract changes.

**Trade-offs**

- CI/CD configuration requires path filtering.
- Shared package changes must be tested against affected applications.
- Dependency management must remain consistent across workspaces.

## 9. Future Considerations

The deployment strategy may evolve as the application grows.

Potential future improvements include:

- Automated staging deployments.
- Automated database backups.
- Rollback procedures.
- Monitoring and error tracking.
- Infrastructure as Code.

The monorepo structure will remain independent from the deployment architecture.
