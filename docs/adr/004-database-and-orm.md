# ADR-004: PostgreSQL and Prisma

**Status:** Accepted  
**Date:** 2026-11-10  
**Decision Owners:** Project Team

## 1. Context

The Laundry Management System requires a relational database to manage interconnected business information.

Examples include:

- Internal users, roles and permissions.
- Customers and memberships.
- Laundry services and pricing.
- Orders and order items.
- Customer payments.
- Employee payroll.
- Membership billing.
- Financial and operational reports.
- Audit logs.

The system must maintain data consistency, preserve financial history and support future features such as multiple branches.

We need a database and ORM that are reliable, maintainable and suitable for a TypeScript backend.

## 2. Decision

We will use:

- **PostgreSQL** as the relational database.
- **Prisma ORM** as the database access and schema management tool.
- **Prisma Migrate** for database schema migrations.

PostgreSQL will be the source of truth for application data.

Prisma will be used by the NestJS backend. The mobile application will access data only through backend APIs.

## 3. Why PostgreSQL?

PostgreSQL was selected because it provides:

- Relational data modeling.
- Foreign key constraints.
- Transactions.
- Indexes and unique constraints.
- Reliable handling of structured business data.
- Decimal numeric types for financial values.
- JSON support when needed.
- Strong ecosystem and open-source licensing.

These capabilities are important for order processing, payment records and financial reporting.

## 4. Why Prisma?

Prisma was selected because it provides:

- Type-safe database queries.
- A schema-based data model.
- Migration management.
- Generated TypeScript types.
- Integration with NestJS.
- A developer-friendly workflow.

Prisma will help maintain consistency between the database schema and backend TypeScript code.

## 5. Database Architecture

The database will initially use one PostgreSQL database.

Business data will be organized into related tables.

### Main domain areas

| Domain                    | Example tables                                                           |
| ------------------------- | ------------------------------------------------------------------------ |
| Authentication and access | users, roles, permissions, user_roles, role_permissions                  |
| Customers                 | customers                                                                |
| Memberships               | memberships                                                              |
| Membership billing        | membership_plans, membership_invoices, membership_payments               |
| Services                  | service_categories, services, item_types, scents                         |
| Orders                    | orders, order_items, order_status_history                                |
| Customer payments         | payments, payment_methods                                                |
| Employee payroll          | payroll_periods, employee_payroll, payroll_adjustments, payroll_payments |
| Audit                     | audit_logs                                                               |

The final schema and relationships will be documented in `docs/erd.md`.

## 6. Data Integrity Rules

The following principles must be followed.

### 6.1 Foreign keys

Related records must use foreign key constraints where appropriate.

For example:

- An order belongs to a customer.
- An order item belongs to an order.
- A payment belongs to an order.
- A membership belongs to a customer.

### 6.2 Transactions

Related financial and operational operations must use database transactions when partial completion would leave inconsistent data.

For example, creating an order and its order items should be handled as one transaction.

### 6.3 Financial precision

Financial values must use PostgreSQL `DECIMAL` / `NUMERIC` and Prisma `Decimal`.

Do not use JavaScript floating-point arithmetic as the source of truth for financial calculations.

### 6.4 Historical price snapshots

Order items must preserve the price applied when the order was created.

Changes to service prices must not change the value of historical orders.

### 6.5 Membership history

Membership records must preserve historical information.

A customer may have at most one active membership at a time.

The database should enforce this rule with a partial unique index:

```sql
CREATE UNIQUE INDEX "memberships_one_active_per_customer"
ON "memberships" ("customer_id")
WHERE "status" = 'ACTIVE';
```

This constraint will be managed through a Prisma migration because it is a PostgreSQL-specific database feature.

### 6.6 Customer phone numbers

Customer phone numbers will be normalized and indexed for searching and duplicate warnings.

They will not be globally unique because different customers may share a phone number.

Internal user phone numbers may have a separate uniqueness rule.

### 6.7 Auditability

Important business changes must be recorded, particularly:

- Order status changes.
- Customer updates.
- Payment creation and refunds.
- Payroll changes and salary payments.
- Membership billing changes.

Financial records should not be silently overwritten or deleted.

## 7. Prisma Project Structure

The database-related code will be maintained inside the API application.

```text
apps/api/
├── prisma/
│   ├── schema.prisma
│   ├── migrations/
│   └── seed.ts
│
└── src/
    ├── database/
    │   ├── database.module.ts
    │   └── prisma.service.ts
    │
    └── modules/
```

### File responsibilities

| File               | Responsibility                             |
| ------------------ | ------------------------------------------ |
| schema.prisma      | Database models, relationships and enums   |
| migrations/        | Version-controlled database changes        |
| seed.ts            | Initial roles, permissions and master data |
| prisma.service.ts  | Shared Prisma client integration           |
| database.module.ts | NestJS database module                     |

Seed scripts should be idempotent so they can safely be run more than once without creating duplicate master records.

Business-specific pricing should not be invented or inserted as generic seed data.

## 8. Database Access and Security

- Only the backend API can access PostgreSQL.
- Mobile applications must never contain database credentials.
- Database credentials must be stored in environment variables.
- Production database access should be private and restricted.
- Database migrations must be reviewed before production deployment.
- Backups and restore procedures must be established before production launch.

Prisma models must not be exposed directly as public API responses.

The backend should use DTOs and explicit response structures.

The shared package may contain API types and Zod schemas, but it must not expose Prisma database models to the mobile application.

## 9. Alternatives Considered

| Technology             | Advantages                                              | Disadvantages                                                               |
| ---------------------- | ------------------------------------------------------- | --------------------------------------------------------------------------- |
| PostgreSQL + Prisma    | Relational integrity, type safety and migration tooling | Requires understanding of relational modeling and migrations                |
| MySQL + Prisma         | Mature relational database and Prisma support           | Not selected for this project's initial database choice                     |
| MongoDB                | Flexible document structure                             | Less natural for the system's relational and transactional business records |
| PostgreSQL without ORM | Direct SQL control                                      | More manual query and type management                                       |

## 10. Consequences

**Positive consequences**

- Strong relational data integrity.
- Type-safe database access.
- Version-controlled schema changes.
- Reliable transaction support.
- Appropriate financial precision.
- Easier integration with NestJS and TypeScript.

**Trade-offs**

- Developers must understand relational database design.
- Prisma migrations require review and discipline.
- Some PostgreSQL-specific features may require raw SQL migrations.
- ORM abstractions do not replace database optimization knowledge.

## 11. Future Considerations

Potential future improvements include:

- Managed PostgreSQL hosting.
- Automated backups and restore testing.
- Database monitoring.
- Query optimization and indexing.
- Read replicas if reporting workloads eventually require them.
- Multi-branch data isolation and reporting.

PostgreSQL and Prisma will remain the initial database foundation.
