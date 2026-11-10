# Permissions Matrix — Laundry Management System

**Version:** 1.2  
**Status:** V1 implementation baseline  
**Authorization:** RBAC + backend record-level and field-level validation

## 1. Roles

### SUPERADMIN

Full system administration and business access.

### ADMIN

Daily operational and financial administration, including employee account management within authority.

Cannot manage system-level permissions or approve their own generated or adjusted payroll.

### EMPLOYEE

Daily laundry operations, customer search, order processing, membership operations and permitted payment collection.

Cannot manage users, roles, prices, refunds, payroll or financial reports.

## 2. Permission catalogue

### Authentication and users

| Permission          | Description                    |
| ------------------- | ------------------------------ |
| `user.read`         | View users                     |
| `user.create`       | Create user accounts           |
| `user.update`       | Update permitted user fields   |
| `user.manage`       | Deactivate and manage accounts |
| `role.read`         | View roles                     |
| `role.manage`       | Manage role definitions        |
| `role.assign`       | Assign roles                   |
| `permission.assign` | Assign permissions             |
| `session.revoke`    | Revoke sessions                |

### Customers

| Permission              | Description                                 |
| ----------------------- | ------------------------------------------- |
| `customer.read`         | Search and view customers                   |
| `customer.create`       | Create a customer independently of an order |
| `customer.update.basic` | Update basic customer information           |
| `customer.update.admin` | Update administrative customer information  |
| `customer.deactivate`   | Deactivate customer records                 |

**Important:** Creating a guest customer as part of `POST /orders` is authorized by `order.create`. It is a narrowly scoped operation that only accepts the required customer details. It does not grant permission for unrestricted customer administration.

### Membership

| Permission                  | Description                 |
| --------------------------- | --------------------------- |
| `membership.read`           | Search and view memberships |
| `membership.create`         | Create memberships          |
| `membership.update`         | Update membership records   |
| `membership.plan.manage`    | Manage membership plans     |
| `membership.billing.read`   | View membership invoices    |
| `membership.billing.manage` | Manage membership invoices  |
| `membership.billing.pay`    | Record membership payments  |
| `membership.billing.refund` | Refund membership payments  |

### Master data

| Permission              | Description                                      |
| ----------------------- | ------------------------------------------------ |
| `service.read`          | View services, categories, item types and scents |
| `service.manage`        | Manage service master data                       |
| `service.price.manage`  | Manage service prices                            |
| `payment-method.read`   | View payment methods                             |
| `payment-method.manage` | Manage payment methods                           |

### Orders

| Permission            | Description                                             |
| --------------------- | ------------------------------------------------------- |
| `order.create`        | Create orders, including narrowly scoped guest creation |
| `order.read.all`      | View all operational orders                             |
| `order.update`        | Update permitted order details                          |
| `order.status.update` | Update permitted order statuses                         |
| `order.cancel`        | Cancel orders                                           |

V1 has one branch and no assignment workflow. Therefore, there is no `order.read.assigned` permission in this version.

### Order payments and refunds

| Permission           | Description                |
| -------------------- | -------------------------- |
| `order-payment.read` | View order payment records |
| `payment.create`     | Record order payments      |
| `payment.refund`     | Refund order payments      |

### Payroll

| Permission         | Description                                         |
| ------------------ | --------------------------------------------------- |
| `payroll.read`     | View all payroll records                            |
| `payroll.read.own` | View authenticated employee's own payslip           |
| `payroll.manage`   | Manage compensation, periods and payroll generation |
| `payroll.approve`  | Approve payroll                                     |
| `payroll.pay`      | Record salary payments                              |
| `payroll.adjust`   | Create payroll adjustments                          |

### Reports and audit

| Permission               | Description                       |
| ------------------------ | --------------------------------- |
| `reports.read`           | View operational reports          |
| `reports.financial.read` | View financial reports            |
| `reports.employee.read`  | View employee performance reports |
| `audit.read`             | View audit logs                   |

---

## 3. Role permission matrix

Legend:

- `✓` Granted
- `—` Not granted

| Permission                  | SUPERADMIN | ADMIN | EMPLOYEE |
| --------------------------- | :--------: | :---: | :------: |
| **Users and authorization** |            |       |          |
| `user.read`                 |     ✓      |   ✓   |    —     |
| `user.create`               |     ✓      |  ✓\*  |    —     |
| `user.update`               |     ✓      |  ✓\*  |    —     |
| `user.manage`               |     ✓      |  ✓\*  |    —     |
| `role.read`                 |     ✓      |   —   |    —     |
| `role.manage`               |     ✓      |   —   |    —     |
| `role.assign`               |     ✓      |   —   |    —     |
| `permission.assign`         |     ✓      |   —   |    —     |
| `session.revoke`            |     ✓      |  ✓\*  |    —     |
| **Customers**               |            |       |          |
| `customer.read`             |     ✓      |   ✓   |    ✓     |
| `customer.create`           |     ✓      |   ✓   |    ✓     |
| `customer.update.basic`     |     ✓      |   ✓   |    ✓     |
| `customer.update.admin`     |     ✓      |   ✓   |    —     |
| `customer.deactivate`       |     ✓      |   ✓   |    —     |
| **Membership**              |            |       |          |
| `membership.read`           |     ✓      |   ✓   |    ✓     |
| `membership.create`         |     ✓      |   ✓   |    ✓     |
| `membership.update`         |     ✓      |   ✓   |    —     |
| `membership.plan.manage`    |     ✓      |   ✓   |    —     |
| `membership.billing.read`   |     ✓      |   ✓   |    —     |
| `membership.billing.manage` |     ✓      |   ✓   |    —     |
| `membership.billing.pay`    |     ✓      |   ✓   |    ✓     |
| `membership.billing.refund` |     ✓      |   ✓   |    —     |
| **Master data**             |            |       |          |
| `service.read`              |     ✓      |   ✓   |    ✓     |
| `service.manage`            |     ✓      |   ✓   |    —     |
| `service.price.manage`      |     ✓      |   ✓   |    —     |
| `payment-method.read`       |     ✓      |   ✓   |    ✓     |
| `payment-method.manage`     |     ✓      |   ✓   |    —     |
| **Orders**                  |            |       |          |
| `order.create`              |     ✓      |   ✓   |    ✓     |
| `order.read.all`            |     ✓      |   ✓   |    ✓     |
| `order.update`              |     ✓      |   ✓   |    —     |
| `order.status.update`       |     ✓      |   ✓   |    ✓     |
| `order.cancel`              |     ✓      |   ✓   |    —     |
| **Order payments**          |            |       |          |
| `order-payment.read`        |     ✓      |   ✓   |    —     |
| `payment.create`            |     ✓      |   ✓   |    ✓     |
| `payment.refund`            |     ✓      |   ✓   |    —     |
| **Payroll**                 |            |       |          |
| `payroll.read`              |     ✓      |   ✓   |    —     |
| `payroll.read.own`          |     ✓      |   ✓   |    ✓     |
| `payroll.manage`            |     ✓      |   ✓   |    —     |
| `payroll.approve`           |     ✓      |  ✓\*  |    —     |
| `payroll.pay`               |     ✓      |   ✓   |    —     |
| `payroll.adjust`            |     ✓      |   ✓   |    —     |
| **Reports and audit**       |            |       |          |
| `reports.read`              |     ✓      |   ✓   |    —     |
| `reports.financial.read`    |     ✓      |   ✓   |    —     |
| `reports.employee.read`     |     ✓      |   ✓   |    —     |
| `audit.read`                |     ✓      |   —   |    —     |

### Role restrictions

`user.create`, `user.update`, `user.manage` and `session.revoke` for ADMIN are restricted to employee accounts within their authority. ADMIN cannot create or promote SUPERADMIN.

`payroll.approve` for ADMIN is conditional: an ADMIN cannot approve payroll they generated or adjusted. SUPERADMIN can act as an independent approver.

`payroll.read.own` for EMPLOYEE is strictly restricted to the authenticated user's own payroll records.

---

## 4. Customer and member authorization

### Customer search

Endpoint:

`GET /customers/search`

Permission: `customer.read`.

Allowed roles:

- SUPERADMIN
- ADMIN
- EMPLOYEE

Returned search fields:

- Customer ID
- Name
- Phone
- WhatsApp, if needed
- Membership status
- Membership plan name
- Membership expiry

Do not return address or private notes in search results.

### Create new customer during order creation

Endpoint:

`POST /orders`

Permission: `order.create`.

This permission includes the ability to create a minimal guest customer as part of the order transaction.

It does not grant unrestricted access to update or deactivate customer records.

### Create customer independently

Endpoint:

`POST /customers`

Permission: `customer.create`.

### Membership creation

Endpoint:

`POST /memberships`

Permission: `membership.create`.

Employee may create memberships but cannot manage membership plans or change billing rules.

### Membership payment

Endpoint:

`POST /membership-invoices/:id/payments`

Permission: `membership.billing.pay`.

Employee may record a membership payment. Only authorized administrators can issue a refund.

---

## 5. Order authorization

### Create order

Requires:

- `order.create`
- Existing active customer or valid new guest details
- Valid service and item type
- Positive quantity
- Valid pricing configuration

The backend derives:

- Customer membership status
- Order number
- Received timestamp
- Creator
- Price
- Discount
- Tax
- Total

The client cannot submit trusted values for these fields.

### View orders

V1 is single-branch. Employees can view operational orders across the branch.

The response should contain the operational fields needed to process orders. Payment information may be limited to the payment summary required for collection and order completion.

Employees do not receive access to financial reports or unrestricted payment history.

### Update status

Employee can only perform allowed order status transitions.

The backend validates:

- Current order status
- Target order status
- Authenticated actor
- Business transition rules

### Cancel order

Only ADMIN and SUPERADMIN can cancel orders in V1.

Cancellation requires a reason and must be audited.

---

## 6. Payroll authorization

### Administrator operations

- Generate payroll: `payroll.manage`
- Adjust payroll: `payroll.adjust`
- Approve payroll: `payroll.approve`
- Record salary payment: `payroll.pay`

### Separation of duties

An ADMIN cannot approve a payroll record if the same user generated or adjusted it.

The backend must check:

```ts
if (
  payrollRecord.generatedBy === user.id ||
  payrollRecord.adjustments.some(
    (adjustment) => adjustment.createdBy === user.id,
  )
) {
  throw new ForbiddenException(
    'You cannot approve payroll that you generated or adjusted.',
  );
}
```

SUPERADMIN is the independent approver when no other authorized approver is available.

### Employee payslip

Endpoint:

`GET /payroll-records/me`

Permission: `payroll.read.own`.

The backend derives employee ID from the authenticated user. It must not accept a user-supplied employee ID to select another person's payroll.

---

## 7. NestJS implementation

Suggested structure:

```text
apps/api/src/
├── common/
│   ├── decorators/
│   │   ├── permissions.decorator.ts
│   │   └── current-user.decorator.ts
│   ├── guards/
│   │   ├── jwt-auth.guard.ts
│   │   └── permissions.guard.ts
│   └── interfaces/
│       └── authenticated-user.interface.ts
│
├── modules/
│   ├── auth/
│   ├── users/
│   ├── roles/
│   ├── customers/
│   ├── memberships/
│   ├── services/
│   ├── orders/
│   ├── payments/
│   ├── payroll/
│   └── reports/
```

### Permission decorator

```ts
@Permissions("order.create")
@Post()
createOrder(
  @CurrentUser() user: AuthenticatedUser,
  @Body() dto: CreateOrderDto,
) {
  return this.ordersService.create(dto, user);
}
```

### Authorization responsibilities

JWT guard:

- Validates authentication.
- Resolves the authenticated user.
- Rejects inactive or invalid sessions.

Permission guard:

- Checks required permission.
- Rejects unauthorized requests.

Service layer:

- Enforces record-level access.
- Enforces field-level authorization.
- Validates business rules.
- Performs transactional operations.

Never rely on frontend role checks as security.

---

## 8. Required authorization tests

| Scenario                                      | Expected                                           |
| --------------------------------------------- | -------------------------------------------------- |
| Unauthenticated user creates order            | 401                                                |
| Employee searches customers                   | 200                                                |
| Employee creates order with existing customer | 201                                                |
| Employee creates order with new guest         | 201                                                |
| Employee submits `isMember: true`             | Rejected or ignored; membership derived by backend |
| Employee creates standalone customer          | Allowed by `customer.create`                       |
| Employee views operational orders             | 200                                                |
| Employee creates an order payment             | 201                                                |
| Employee refunds a payment                    | 403                                                |
| Employee manages service prices               | 403                                                |
| Employee views financial reports              | 403                                                |
| Employee views own payslip                    | 200                                                |
| Employee views another employee's payslip     | 403 or 404                                         |
| ADMIN assigns SUPERADMIN                      | 403                                                |
| ADMIN edits system permissions                | 403                                                |
| ADMIN approves own generated payroll          | 403                                                |
| Inactive user uses an old session             | 401                                                |
| Employee uses an inactive payment method      | 422                                                |

---

## 9. Permission governance

1. Keep permission codes centralized.
2. Seed permissions through controlled database seed scripts.
3. Audit role and permission changes.
4. Test every protected endpoint.
5. Never add a new endpoint without defining its permission.
6. Keep record-level checks in the service layer.
7. Keep role restrictions in backend policy logic.
8. Do not let the client assign its own role or permission.
9. Revoke sessions when users are deactivated.
10. Review permission changes as part of every API feature implementation.
