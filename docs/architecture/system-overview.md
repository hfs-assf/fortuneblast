# Laundry Management System

## System Overview

### 1. Purpose

The Laundry Management System is an internal business
application designed to manage laundry operations,
customers, memberships, orders, payments, and reports.

The system will initially support a single laundry branch
and internal staff operations.

### 2. System Objectives

- Centralize customer information.
- Manage laundry services and pricing.
- Record customer orders.
- Track order processing and completion.
- Manage partial and full payments.
- Maintain membership information.
- Generate operational and financial reports.
- Control access based on user roles and permissions.

### 3. Target Users

The initial system supports three internal roles:

| Role       | Responsibility                  |
| ---------- | ------------------------------- |
| Superadmin | Full system administration      |
| Admin      | Operational and user management |
| Employee   | Daily laundry operations        |

Customers do not have application accounts in version 1.

Employees and administrators can register customers,
including customers who are not members.

### 4. Supported Platforms

| Platform | Technology          | Distribution                 |
| -------- | ------------------- | ---------------------------- |
| Android  | Expo / React Native | Google Play Store            |
| iOS      | Expo / React Native | Apple App Store              |
| Backend  | NestJS              | Cloud server                 |
| Database | PostgreSQL          | Private cloud infrastructure |

### 5. High-Level System Components

- Mobile application
- Backend REST API
- Authentication and authorization
- Customer management
- Membership management
- Service and pricing management
- Order management
- Payment management
- Reporting
- Database

### 6. System Constraints

- One repository using npm Workspaces.
- Backend uses a modular monolith architecture.
- Mobile and backend are deployed independently.
- PostgreSQL is accessible only by the backend.
- All authorization is enforced by the backend.
- Financial records must preserve historical transaction data.
- Order prices must be stored as snapshots.
- Payment status is independent of order status.

### 7. Future Considerations

The architecture should allow future expansion:

- Multiple laundry branches.
- Customer mobile accounts.
- Online customer ordering.
- Inventory management.
- Delivery management.
- Promotions and discounts.
- Customer notifications.
- Advanced financial reporting.
