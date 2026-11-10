# API Contract — Laundry Management System

**Version:** 1.2  
**Status:** V1 implementation baseline  
**Base URL:** `/api/v1`  
**Backend:** NestJS + Prisma  
**Database:** PostgreSQL  
**Client:** Expo React Native + TypeScript

## 1. API standards

- JSON request and response.
- UUID resource identifiers.
- ISO 8601 UTC timestamps.
- Money and decimal quantities serialized as strings.
- `camelCase` JSON properties.
- Protected endpoints require `Authorization: Bearer <accessToken>`.
- Backend is the source of truth for authorization, membership, pricing and financial calculations.
- Never accept trusted actor IDs, role IDs, calculated totals or membership flags from the client.

### Standard success

```json
{
  "data": {},
  "meta": {
    "requestId": "request-uuid"
  }
}
```

### Standard error

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "The request contains invalid data.",
    "details": [
      {
        "field": "customer.phone",
        "message": "Phone number is required."
      }
    ]
  },
  "meta": {
    "requestId": "request-uuid"
  }
}
```

### HTTP status

| Code | Meaning                         |
| ---- | ------------------------------- |
| 200  | Successful request              |
| 201  | Resource created                |
| 204  | Successful request without body |
| 400  | Invalid request                 |
| 401  | Unauthenticated                 |
| 403  | Permission denied               |
| 404  | Resource not found              |
| 409  | Conflict or duplicate operation |
| 422  | Business validation failed      |
| 429  | Rate limit exceeded             |
| 500  | Unexpected server error         |

---

## 2. Authentication

### POST `/auth/login`

Request:

```json
{
  "username": "admin",
  "password": "password"
}
```

Response:

```json
{
  "data": {
    "accessToken": "access-token",
    "refreshToken": "refresh-token",
    "tokenType": "Bearer",
    "expiresIn": 900,
    "user": {
      "id": "user-uuid",
      "username": "admin",
      "fullName": "System Administrator",
      "roles": ["ADMIN"],
      "permissions": ["customer.read", "order.create", "order.read.all"]
    }
  }
}
```

### POST `/auth/refresh`

Request:

```json
{
  "refreshToken": "refresh-token"
}
```

Returns a new access token and rotated refresh token.

### POST `/auth/logout`

Request:

```json
{
  "refreshToken": "refresh-token"
}
```

Revokes the refresh session. Does not require an unexpired access token.

### GET `/auth/me`

Returns the authenticated user's profile, roles and current permissions.

Authentication rules:

- Username must be unique.
- Passwords must be securely hashed.
- Inactive users cannot authenticate.
- Access tokens should be short-lived.
- Refresh tokens must be rotated and stored hashed on the server.
- Reused refresh tokens must be rejected.
- Rate-limit login attempts.

---

## 3. Users and authorization

| Method | Endpoint                 | Permission          |
| ------ | ------------------------ | ------------------- |
| GET    | `/users`                 | `user.read`         |
| GET    | `/users/:id`             | `user.read`         |
| POST   | `/users`                 | `user.create`       |
| PATCH  | `/users/:id`             | `user.update`       |
| POST   | `/users/:id/deactivate`  | `user.manage`       |
| GET    | `/roles`                 | `role.read`         |
| GET    | `/permissions`           | `role.read`         |
| PUT    | `/users/:id/roles`       | `role.assign`       |
| PUT    | `/roles/:id/permissions` | `permission.assign` |

ADMIN can manage employee accounts within their authority but cannot create or assign SUPERADMIN.

---

## 4. Customers and membership

### 4.1 Domain model

- Every person placing an order is a customer.
- A member is a customer with a currently active membership.
- A non-member is a customer without a currently active membership.
- Membership is not a user role.
- Customers do not need application login accounts in V1.
- A customer can have multiple historical memberships, but only one active membership at a time.

### 4.2 Search customers

`GET /customers/search?query=budi&page=1&limit=20&membershipStatus=ACTIVE`

Permission: `customer.read`

Supported `membershipStatus` filters:

- `ACTIVE`
- `NON_MEMBER`
- `EXPIRED`
- `ALL`

The backend calculates membership status using membership status and validity dates.

Response:

```json
{
  "data": [
    {
      "id": "customer-uuid",
      "name": "Budi Santoso",
      "phone": "+6281234567890",
      "whatsapp": "+6281234567890",
      "membership": {
        "status": "ACTIVE",
        "membershipId": "membership-uuid",
        "planName": "Monthly Member",
        "expiresAt": "2026-11-30T16:59:59.000Z"
      }
    },
    {
      "id": "customer-uuid-2",
      "name": "Siti Aminah",
      "phone": "+6281234567891",
      "whatsapp": null,
      "membership": {
        "status": "NONE",
        "membershipId": null,
        "planName": null,
        "expiresAt": null
      }
    }
  ],
  "meta": {
    "pagination": {
      "page": 1,
      "limit": 20,
      "totalItems": 2,
      "totalPages": 1
    }
  }
}
```

The UI can filter this result into Members and Non-Members. The search endpoint must not expose address or private notes.

### 4.3 Get customer details

`GET /customers/:id`

Permission: `customer.read`

Returns contact details, membership summary and customer profile.

### 4.4 Create customer independently

`POST /customers`

Permission: `customer.create`

Request:

```json
{
  "name": "Siti Aminah",
  "phone": "081234567890",
  "whatsapp": "081234567890",
  "address": "Pekanbaru",
  "notes": null
}
```

Response: `201 Created`.

### 4.5 Update customer

`PATCH /customers/:id`

Permission: `customer.update.basic` or `customer.update.admin`, depending on the fields.

### 4.6 Customer orders

`GET /customers/:id/orders?page=1&limit=20`

Permission: `order.read.all`.

### 4.7 Customer rules

- Normalize phone numbers before storing them.
- Use a phone-number parsing library suitable for international formats.
- Store a normalized international representation where possible.
- Phone numbers are not unique.
- Do not automatically merge records with matching phone numbers.
- Search matches must be selected by staff.
- Deactivate customers instead of hard-deleting them.

---

## 5. Membership

### 5.1 Membership plans

| Method | Endpoint                           | Permission               |
| ------ | ---------------------------------- | ------------------------ |
| GET    | `/membership-plans`                | `membership.read`        |
| POST   | `/membership-plans`                | `membership.plan.manage` |
| PATCH  | `/membership-plans/:id`            | `membership.plan.manage` |
| POST   | `/membership-plans/:id/deactivate` | `membership.plan.manage` |

V1 supports one-time membership plans. Automated recurring billing is future scope.

### 5.2 Create membership

`POST /memberships`

Permission: `membership.create`

Request:

```json
{
  "customerId": "customer-uuid",
  "membershipPlanId": "plan-uuid"
}
```

Response:

```json
{
  "data": {
    "id": "membership-uuid",
    "customerId": "customer-uuid",
    "membershipPlan": {
      "id": "plan-uuid",
      "name": "Monthly Member"
    },
    "status": "PENDING",
    "startedAt": null,
    "expiresAt": null,
    "invoice": {
      "id": "invoice-uuid",
      "amount": "100000.00",
      "status": "ISSUED"
    }
  }
}
```

The backend creates the membership and its invoice transactionally.

Membership becomes active only after the invoice is fully paid. The backend sets activation and expiry timestamps.

### 5.3 Membership invoices and payments

| Method | Endpoint                            | Permission                  |
| ------ | ----------------------------------- | --------------------------- |
| GET    | `/memberships/:id/invoices`         | `membership.billing.read`   |
| GET    | `/membership-invoices/:id`          | `membership.billing.read`   |
| POST   | `/membership-invoices/:id/payments` | `membership.billing.pay`    |
| POST   | `/membership-payments/:id/refunds`  | `membership.billing.refund` |

Membership payments are separate from order payments.

V1 requires full payment to activate a membership. Do not activate membership after a partial payment.

---

## 6. Master data

| Method | Endpoint                  | Permission              |
| ------ | ------------------------- | ----------------------- |
| GET    | `/service-categories`     | `service.read`          |
| POST   | `/service-categories`     | `service.manage`        |
| PATCH  | `/service-categories/:id` | `service.manage`        |
| GET    | `/services`               | `service.read`          |
| GET    | `/services/:id`           | `service.read`          |
| POST   | `/services`               | `service.manage`        |
| PATCH  | `/services/:id`           | `service.manage`        |
| GET    | `/item-types`             | `service.read`          |
| POST   | `/item-types`             | `service.manage`        |
| GET    | `/scents`                 | `service.read`          |
| POST   | `/scents`                 | `service.manage`        |
| GET    | `/service-prices`         | `service.read`          |
| POST   | `/service-prices`         | `service.price.manage`  |
| PATCH  | `/service-prices/:id`     | `service.price.manage`  |
| GET    | `/payment-methods`        | `payment-method.read`   |
| POST   | `/payment-methods`        | `payment-method.manage` |
| PATCH  | `/payment-methods/:id`    | `payment-method.manage` |

### Service price request

```json
{
  "serviceId": "service-uuid",
  "itemTypeId": "item-type-uuid",
  "unit": "KG",
  "price": "8000.00",
  "effectiveFrom": "2026-11-01T00:00:00.000Z",
  "effectiveTo": null
}
```

Price periods for the same service, item type and unit must not overlap.

---

## 7. Orders

### 7.1 Order creation design

One endpoint supports both existing customers and new guest customers.

The request uses a discriminated customer selection:

- `EXISTING`: references a customer already stored in the database.
- `NEW_GUEST`: creates a customer in the same transaction as the order.

Do not use separate order endpoints for members and non-members.

### 7.2 POST `/orders`

Permission: `order.create`

Header:

```http
Idempotency-Key: <unique-operation-key>
```

#### Existing member or existing non-member

```json
{
  "customer": {
    "type": "EXISTING",
    "customerId": "customer-uuid"
  },
  "expectedCompletionAt": "2026-11-12T03:00:00.000Z",
  "notes": "Separate white and colored clothes",
  "items": [
    {
      "serviceId": "service-uuid",
      "itemTypeId": "item-type-uuid",
      "scentId": "scent-uuid",
      "quantity": "3.50",
      "unit": "KG",
      "notes": "White clothes"
    }
  ]
}
```

#### New non-member

```json
{
  "customer": {
    "type": "NEW_GUEST",
    "name": "Siti Aminah",
    "phone": "081234567890",
    "whatsapp": "081234567890"
  },
  "expectedCompletionAt": "2026-11-12T03:00:00.000Z",
  "notes": "Separate white and colored clothes",
  "items": [
    {
      "serviceId": "service-uuid",
      "itemTypeId": "item-type-uuid",
      "scentId": "scent-uuid",
      "quantity": "3.50",
      "unit": "KG",
      "notes": "White clothes"
    }
  ]
}
```

### 7.3 Customer creation rules inside order creation

For `EXISTING`:

- Verify customer exists and is not deactivated.
- Recalculate current membership eligibility.
- Do not trust membership information previously displayed by the app.

For `NEW_GUEST`:

- Name and phone are required.
- WhatsApp is optional.
- Create a customer with no membership.
- Use the authenticated user as `createdBy`.
- Do not automatically attach the new guest to an existing customer with the same phone number.
- The app should warn staff if a possible duplicate is found.
- The transaction must roll back if order creation fails.

`order.create` grants permission for this narrowly scoped customer creation. It does not grant general customer administration or unrestricted customer updates.

### 7.4 Order response

```json
{
  "data": {
    "id": "order-uuid",
    "orderNumber": "LND-2026-000001",
    "customer": {
      "id": "customer-uuid",
      "name": "Siti Aminah",
      "phone": "+6281234567890",
      "membership": {
        "status": "NONE",
        "planName": null
      }
    },
    "status": "RECEIVED",
    "receivedAt": "2026-11-10T03:00:00.000Z",
    "expectedCompletionAt": "2026-11-12T03:00:00.000Z",
    "currency": "IDR",
    "subtotal": "28000.00",
    "discountAmount": "0.00",
    "taxAmount": "0.00",
    "totalAmount": "28000.00",
    "paidAmount": "0.00",
    "refundedAmount": "0.00",
    "balanceDue": "28000.00",
    "paymentStatus": "UNPAID",
    "items": [
      {
        "id": "order-item-uuid",
        "serviceName": "Wash & Fold",
        "itemTypeName": "Regular Clothes",
        "scentName": "Lavender",
        "quantity": "3.50",
        "unit": "KG",
        "unitPrice": "8000.00",
        "lineTotal": "28000.00",
        "notes": "White clothes"
      }
    ]
  },
  "meta": {
    "requestId": "request-uuid"
  }
}
```

The backend sets `receivedAt`, `createdBy`, order number, status, price and totals. These values are not trusted from the request.

### 7.5 Order endpoints

| Method | Endpoint                     | Permission            |
| ------ | ---------------------------- | --------------------- |
| POST   | `/orders`                    | `order.create`        |
| GET    | `/orders`                    | `order.read.all`      |
| GET    | `/orders/:id`                | `order.read.all`      |
| PATCH  | `/orders/:id`                | `order.update`        |
| POST   | `/orders/:id/status`         | `order.status.update` |
| POST   | `/orders/:id/cancel`         | `order.cancel`        |
| GET    | `/orders/:id/status-history` | `order.read.all`      |

V1 is single-branch, so authorized operational employees can view all operational orders. Financial reports and administrative actions remain separately protected.

### 7.6 Status update

`POST /orders/:id/status`

```json
{
  "status": "WASHING",
  "notes": "Washing started"
}
```

Backend validates the transition and creates a history record.

### 7.7 Cancellation

`POST /orders/:id/cancel`

```json
{
  "reason": "Customer cancelled the order"
}
```

Cancellation does not delete the order. Paid amounts must be handled by the refund process.

---

## 8. Order payments and refunds

### POST `/orders/:id/payments`

Permission: `payment.create`

```json
{
  "paymentMethodId": "payment-method-uuid",
  "amount": "20000.00",
  "paidAt": "2026-11-10T04:00:00.000Z",
  "referenceNumber": "QRIS-001",
  "notes": "Partial payment"
}
```

### GET `/orders/:id/payments`

Permission: `order-payment.read`

### POST `/order-payments/:id/refunds`

Permission: `payment.refund`

```json
{
  "amount": "5000.00",
  "reason": "Overpayment"
}
```

The backend must prevent duplicate payments and over-refunds, including under concurrent requests.

---

## 9. Payroll

| Method | Endpoint                        | Permission         |
| ------ | ------------------------------- | ------------------ |
| GET    | `/compensation-profiles`        | `payroll.read`     |
| POST   | `/compensation-profiles`        | `payroll.manage`   |
| PATCH  | `/compensation-profiles/:id`    | `payroll.manage`   |
| GET    | `/payroll-periods`              | `payroll.read`     |
| POST   | `/payroll-periods`              | `payroll.manage`   |
| POST   | `/payroll-periods/:id/generate` | `payroll.manage`   |
| POST   | `/payroll-records/:id/approve`  | `payroll.approve`  |
| POST   | `/payroll-records/:id/payments` | `payroll.pay`      |
| GET    | `/payroll-records/me`           | `payroll.read.own` |

An employee can only retrieve their own payroll records. An administrator cannot approve payroll that they generated or adjusted.

---

## 10. Reports and audit

| Method | Endpoint                        | Permission                |
| ------ | ------------------------------- | ------------------------- |
| GET    | `/reports/dashboard`            | `reports.read`            |
| GET    | `/reports/orders`               | `reports.read`            |
| GET    | `/reports/cash-flow`            | `reports.financial.read`  |
| GET    | `/reports/order-payments`       | `reports.financial.read`  |
| GET    | `/reports/membership-billing`   | `membership.billing.read` |
| GET    | `/reports/payroll`              | `payroll.read`            |
| GET    | `/reports/employee-performance` | `reports.employee.read`   |
| GET    | `/audit-logs`                   | `audit.read`              |

Cash flow must be based on actual payments and refunds, not order totals or unpaid invoices.

---

## 11. Critical backend transaction rules

### Create order

One database transaction must:

1. Authenticate the user and authorize `order.create`.
2. Resolve or create the customer.
3. Validate customer active status.
4. Validate current membership eligibility.
5. Validate service and item-type compatibility.
6. Select the applicable price.
7. Calculate totals.
8. Create order.
9. Create order items with historical snapshots.
10. Create initial order status history.
11. Write audit record.
12. Commit.

Any failure must roll back all changes, including a newly created guest customer.

### Financial calculations

- PostgreSQL `NUMERIC`.
- Prisma `Decimal`.
- No JavaScript floating-point calculations for money.
- All monetary JSON values are strings.
- Prices and totals are calculated by the backend.
- Payment and refund updates must be concurrency-safe.

### Idempotency

Require idempotency keys for order creation, payment creation and refund creation.

Scope keys by authenticated actor and operation. Reusing a key with a different payload must return a conflict.

---

## 12. Enumerations

Membership: `PENDING`, `ACTIVE`, `EXPIRED`, `CANCELLED`, `SUSPENDED`

Membership lookup: `ACTIVE`, `EXPIRED`, `SUSPENDED`, `CANCELLED`, `NONE`

Order: `RECEIVED`, `CONFIRMED`, `WASHING`, `DRYING`, `IRONING`, `READY_FOR_PICKUP`, `COMPLETED`, `CANCELLED`

Order payment: `UNPAID`, `PARTIALLY_PAID`, `PAID`, `PARTIALLY_REFUNDED`, `REFUNDED`

Units: `KG`, `PCS`, `PAIR`, `SET`, `METER`

**Implementation status:** This document defines the expected contract. It does not mean the endpoints have already been implemented.
