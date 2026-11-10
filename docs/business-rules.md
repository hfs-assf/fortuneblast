# Business Rules — Laundry Management System

**Version:** 1.2  
**Status:** V1 implementation baseline  
**Initial deployment:** One branch

## 1. Domain definitions

### User

An employee or administrator who authenticates into the application.

### Customer

A person who places laundry orders. A customer does not need an application account.

### Member

A customer who has a currently active membership.

### Non-member

A customer who does not currently have an active membership.

### Order

A laundry transaction belonging to exactly one customer.

**Important:** Members and non-members are both customers. Never create separate `members` and `non_members` customer tables.

## 2. Customer rules

1. Every order must reference a customer.
2. Customer records can be created from the customer management screen or during order creation.
3. Name and phone number are required for new customers.
4. WhatsApp, address and notes are optional.
5. Normalize phone numbers to an international format where possible.
6. Phone numbers are not unique.
7. Duplicate phone numbers are allowed.
8. The application must not automatically merge customers using a phone-number match.
9. Staff must select the correct existing customer when multiple search results match.
10. Customer records should be deactivated instead of hard-deleted.
11. Historical orders must remain associated with their original customer.

### New customer during order creation

When staff selects the new guest option:

- The backend creates the customer.
- The backend creates the order.
- Both operations occur in one database transaction.
- The authenticated user is recorded as the creator.
- If order creation fails, the new customer creation rolls back.

A new guest customer is initially a non-member.

### Existing customer during order creation

When staff selects an existing customer:

- The backend validates the customer ID.
- The customer must not be deactivated.
- Membership is recalculated by the backend.
- The app's previously displayed membership status is not trusted.

## 3. Membership rules

1. Membership belongs to a customer.
2. A customer may have multiple historical membership records.
3. A customer may have at most one active membership at a time.
4. V1 supports one-time membership plans.
5. Automated recurring billing is future scope.
6. A new membership starts in `PENDING`.
7. The backend creates a membership invoice.
8. Membership activates only after the invoice is fully paid.
9. The backend calculates activation and expiry timestamps.
10. Expired, suspended and cancelled memberships do not qualify as active.
11. Membership activation, cancellation and suspension must be audited.
12. Membership benefits are not assumed by the application.

### Membership benefits

V1 must not automatically apply a discount merely because a customer is a member.

When the business confirms a membership benefit, implement it as an explicit rule:

- Eligibility
- Benefit type
- Value
- Applicable services
- Effective period
- Maximum discount or usage
- Combination with other promotions

The backend calculates and snapshots any benefit applied to an order.

## 4. Customer search and selection

The mobile application uses one search endpoint for all customers.

The search screen should support:

- All customers
- Active members
- Non-members
- Name search
- Phone-number search

Display:

- Customer name
- Phone number
- Membership status
- Membership expiry, if applicable

Do not expose unnecessary address and private notes in search results.

If no suitable existing customer is selected, staff may create a new guest customer during order creation.

A phone match is a possible match, not proof of identity.

## 5. User and permission rules

V1 roles:

- `SUPERADMIN`
- `ADMIN`
- `EMPLOYEE`

### SUPERADMIN

Can manage all system resources, roles and permissions.

### ADMIN

Can manage day-to-day business operations and employee accounts within authority.

Cannot:

- Create or assign SUPERADMIN.
- Change system-level permission definitions.
- Approve payroll they generated or adjusted.

### EMPLOYEE

Can:

- Search customers and active memberships.
- Create customers as part of order creation.
- Create orders.
- View operational orders in the single branch.
- Update permitted order statuses.
- Record order payments.
- Create memberships.
- Record membership payments if the business enables that workflow.
- View their own payslip.

Cannot:

- Manage users or permissions.
- Manage services or prices.
- Issue refunds.
- Access financial reports.
- Manage or approve payroll.

Backend authorization is mandatory, including record-level checks.

## 6. Master data rules

### Services

Examples:

- Wash & Fold
- Wash & Iron
- Wash Only
- Iron Only
- Dry Cleaning
- Shoe Cleaning
- Bag Cleaning
- Doll Cleaning

### Item types

Examples:

- Regular Clothes
- Bedcover
- Blanket
- Curtain
- Shoes
- Bag
- Doll

### Units

- `KG`
- `PCS`
- `PAIR`
- `SET`
- `METER`

### Scents

- `NONE`
- `FLORAL`
- `LAVENDER`
- `ROSE`
- `OCEAN`
- `BABY_FRESH`
- `LEMON`
- `PREMIUM`

### Payment methods

- `CASH`
- `BANK_TRANSFER`
- `QRIS`
- `DEBIT_CARD`
- `E_WALLET`
- `OTHER`

### Price rules

- Prices are effective-dated.
- The backend chooses the applicable price.
- Overlapping effective periods for the same service, item type and unit are prohibited.
- Existing order prices must not change when master prices are updated.
- Price changes are audited.

## 7. Order rules

1. Every order belongs to exactly one customer.
2. An order must contain at least one item.
3. The backend generates a unique order number.
4. The backend sets the received timestamp.
5. The backend sets the initial status to `RECEIVED`.
6. The backend records the authenticated creator.
7. The backend calculates all prices and totals.
8. The order preserves historical service, item, scent, unit and price snapshots.
9. A customer may have multiple orders.
10. Orders must not be hard-deleted.

### Order item calculation

`lineTotal = quantity × unitPrice`

`subtotal = sum(lineTotal)`

`totalAmount = subtotal - discountAmount + taxAmount`

The backend performs calculations using decimal arithmetic.

### Order status

Initial status: `RECEIVED`.

Supported statuses:

- `RECEIVED`
- `CONFIRMED`
- `WASHING`
- `DRYING`
- `IRONING`
- `READY_FOR_PICKUP`
- `COMPLETED`
- `CANCELLED`

The backend validates allowed transitions. Every transition creates a history record.

### Cancellation

- A reason is required.
- Cancellation must not delete the order.
- Completed orders cannot use the ordinary cancellation flow.
- Any captured payment must be handled under the refund policy.
- Cancellation is audited.

## 8. Payment and refund rules

### Order payments

- An order can have multiple payments.
- Partial payments are allowed.
- Payments must reference an active payment method at the time of recording.
- Payments are separate immutable financial events.
- Payment requests must be idempotent.
- Total completed payments must be checked against the amount due.

### Refunds

- A refund references an original payment.
- A refund requires a reason.
- Total refunds cannot exceed the captured payment amount.
- Refunds are separate financial records.
- Refunds must be audited.
- Refunds must be concurrency-safe.

### Financial domains

Keep three distinct payment domains:

1. Customer order payments.
2. Membership payments.
3. Employee payroll payments.

## 9. Payroll rules

Compensation types:

- `MONTHLY_FIXED`
- `DAILY_WAGE`
- `PER_ORDER`

Payroll process:

1. Configure employee compensation.
2. Create payroll period.
3. Generate payroll.
4. Review payroll.
5. Approve payroll.
6. Record salary payment.
7. Reconcile payroll balance.

Rules:

- One payroll record per employee per payroll period.
- Payroll generation must not duplicate records.
- Adjustments require reasons.
- Approved payroll must not be silently edited.
- Partial salary payments are permitted.
- Payments cannot exceed the approved payable amount without an approved adjustment.
- An administrator cannot approve payroll they generated or adjusted.
- Employees can view only their own payslips.

Attendance integration and per-order compensation calculation rules are future scope until the business defines the required inputs.

## 10. Reporting rules

Separate:

- Operational order reports.
- Order payment reports.
- Cash flow.
- Membership billing.
- Payroll.
- Employee performance.

Cash flow is based on actual money received and refunded, not unpaid order values or invoices.

Reports must apply authorization rules.

## 11. Audit and historical data

Audit:

- User and role changes.
- Customer updates.
- Membership changes.
- Service and pricing changes.
- Order creation and updates.
- Order status changes and cancellation.
- Payments and refunds.
- Payroll generation, adjustments, approval and payments.

Never hard-delete historical financial or order records through normal application workflows.

## 12. V1 versus future scope

### V1

- Single branch.
- Admin-managed employee accounts.
- Customers without login accounts.
- Member and non-member customer selection.
- One-time memberships.
- Manual order processing.
- Manual payment recording.
- Basic payroll.
- Operational and financial reporting.

### Future

- Multi-branch support.
- Customer login and self-service.
- Automated recurring membership billing.
- Online payment integration.
- Inventory.
- Expenses.
- Delivery.
- Promotions.
- Attendance integration.
- Advanced payroll.

These future capabilities must be added through explicit schema and authorization changes, not assumptions.
