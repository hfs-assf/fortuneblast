# Application Architecture

## 1. Architecture Pattern

The system uses a Modular Monolith architecture.

The backend is deployed as one application but internally
organized into independent business modules.

This allows the project to maintain clear boundaries
without introducing distributed system complexity.

## 2. Backend Technology

- NestJS
- TypeScript
- Prisma ORM
- PostgreSQL
- REST API
- JWT authentication
- Role-Based Access Control (RBAC)

## 3. Backend Modules

### Authentication

Responsible for:

- Login
- Password hashing
- JWT access tokens
- Authentication validation

### Users

Responsible for:

- Internal user management
- User status
- User profiles
- Role assignments

### Roles and Permissions

Responsible for:

- Roles
- Permissions
- Role-permission assignments
- Access control

### Customers

Responsible for:

- Customer registration
- Customer information
- Customer search
- Duplicate phone warnings
- Customer updates

### Memberships

Responsible for:

- Membership registration
- Membership status
- Membership history
- Member numbers

### Services

Responsible for:

- Service categories
- Laundry services
- Item types
- Service-item pricing
- Fragrance options

### Orders

Responsible for:

- Order creation
- Order items
- Order status
- Order history
- Price snapshots

### Payments

Responsible for:

- Payment transactions
- Partial payments
- Payment methods
- Refunds
- Outstanding balances

### Reports

Responsible for:

- Revenue reports
- Payment reports
- Operational reports
- Employee payment reports

### Audit

Responsible for:

- Important user activities
- Financial changes
- Administrative changes

## 4. Backend Module Rules

- Each module owns its business logic.
- Controllers handle HTTP requests and responses.
- Services handle business operations.
- DTOs validate incoming requests.
- Database access is handled through Prisma.
- Modules communicate through clearly defined interfaces.
- Authorization is enforced on the backend.

## 5. Mobile Architecture

The mobile application uses a feature-oriented structure.

Main areas:

- Authentication
- Dashboard
- Customers
- Memberships
- Orders
- Services
- Payments
- Reports
- User management
- Settings

Shared mobile components include:

- Buttons
- Inputs
- Form controls
- Tables and lists
- Dialogs
- Notifications
- Loading states
- Empty states

## 6. Shared Package

The shared package contains reusable definitions:

- TypeScript types
- Zod validation schemas
- Business constants
- API contracts

Database-specific Prisma models must not be
exposed directly to the mobile application.
