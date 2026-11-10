# ADR-002: Modular Monolith Architecture

**Status:** Accepted  
**Date:** 2026-11-10  
**Decision Owners:** Project Team

## 1. Context

The Laundry Management System requires a backend to support daily laundry operations, including:

- User authentication and authorization.
- Customer and membership management.
- Service and pricing management.
- Order creation and processing.
- Payment management.
- Employee payroll.
- Membership billing.
- Operational and financial reporting.
- Audit logging.

The system will initially support one laundry branch, with the possibility of supporting multiple branches in the future.

The backend needs to be maintainable, easy to develop, and suitable for a small development team without introducing unnecessary infrastructure complexity.

## 2. Decision

We will use a **Modular Monolith architecture** with NestJS.

The backend will run as one deployable application, but its source code will be separated into independent business modules.

Each module is responsible for its own business logic and exposes clearly defined interfaces to other modules.

### Proposed backend modules

```text
apps/api/src/
├── main.ts
├── app.module.ts
│
├── common/
│   ├── decorators/
│   ├── filters/
│   ├── guards/
│   ├── interceptors/
│   └── utils/
│
├── config/
│
├── database/
│   ├── prisma/
│   └── database.module.ts
│
└── modules/
    ├── auth/
    ├── users/
    ├── roles/
    ├── permissions/
    ├── customers/
    ├── memberships/
    ├── membership-billing/
    ├── services/
    ├── orders/
    ├── payments/
    ├── payroll/
    ├── reports/
    └── audit-logs/
```

### Module responsibilities

| Module             | Responsibility                                   |
| ------------------ | ------------------------------------------------ |
| Auth               | Authentication, JWT and session handling         |
| Users              | Internal staff account management                |
| Roles              | Role management                                  |
| Permissions        | Permission definitions and authorization         |
| Customers          | Customer information and customer records        |
| Memberships        | Membership registration, status and history      |
| Membership Billing | Membership fees, billing periods and payments    |
| Services           | Laundry services, item types, scents and pricing |
| Orders             | Order creation, order items and order status     |
| Payments           | Customer order payments and refunds              |
| Payroll            | Employee salary calculation and salary payments  |
| Reports            | Operational and financial report aggregation     |
| Audit Logs         | Tracking important business activities           |

## 3. Architectural Principles

### 3.1 Module boundaries

Each module must own its business logic and responsibilities.

For example:

- Orders manages order creation and order status.
- Payments manages customer payments.
- Payroll manages employee salary records and disbursements.
- Reports reads and aggregates information from relevant modules.

Modules should communicate through exported services or clearly defined interfaces rather than accessing another module's internal implementation.

### 3.2 Database access

Prisma will provide database access.

Modules should follow clear data ownership boundaries. A module must not casually modify another module's data or bypass its business rules.

Cross-module operations should use explicit service interfaces and database transactions where necessary.

### 3.3 Authentication and authorization

The backend will enforce authentication and role-based access control (RBAC).

The frontend must not be treated as the security boundary.

For example:

- Employees cannot manage roles.
- Employees cannot delete customers.
- Only authorized users can manage payroll.
- Payroll and financial operations must be audited.

### 3.4 API design

The backend will expose REST APIs using versioned endpoints.

Example:

```text
/api/v1/auth
/api/v1/customers
/api/v1/orders
/api/v1/payments
/api/v1/payroll
/api/v1/membership-billing
/api/v1/reports
```

## 4. Alternatives Considered

| Architecture         | Advantages                                                     | Disadvantages                                             |
| -------------------- | -------------------------------------------------------------- | --------------------------------------------------------- |
| Modular Monolith     | Simple deployment, clear module boundaries, easier maintenance | Requires discipline to maintain module boundaries         |
| Microservices        | Independent services and scaling                               | More infrastructure, networking and monitoring complexity |
| Traditional Monolith | Simple initial implementation                                  | Can become tightly coupled as features grow               |

## 5. Consequences

**Positive consequences**

- One backend deployment to manage.
- Easier local development.
- Clear separation of business responsibilities.
- Lower infrastructure and operational costs.
- Easier to understand and test the system.
- Modules can evolve independently within the application.

**Trade-offs**

- Modules share the same deployment lifecycle.
- A failure in the application may affect multiple modules.
- Developers must prevent circular dependencies and tightly coupled modules.

## 6. Future Considerations

Microservices may be considered only when there is a demonstrated need for independent scaling, deployment, or team ownership.

The initial implementation will remain a modular monolith.
