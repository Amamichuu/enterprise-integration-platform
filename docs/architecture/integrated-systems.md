# Integrated Systems — Pacific Logistics

## 1. Overview of the System Landscape

Pacific Logistics operates through three independent enterprise systems: an Order Management System (OMS), a Transportation Management System (TMS), and a Billing & Financial Management System.

Each system is responsible for a specific business domain and maintains ownership of its own data. The systems operate independently and expose interfaces that allow other applications to exchange information.

The Enterprise Integration Platform acts as the central integration layer, coordinating communication, transforming data, validating messages, and managing the exchange of information between the three systems.

### 1.1 System Landscape

| System | Business Domain | Primary Responsibility |
|---|---|---|
| Order Management System (OMS) | Order Management | Manages customers, delivery requests, shipments, and order lifecycle. |
| Transportation Management System (TMS) | Transportation | Manages drivers, carriers, dispatch operations, tracking, and delivery confirmations. |
| Billing & Financial Management System | Finance | Manages delivery charges, invoices, payments, and financial reconciliation. |

### 1.2 Integration Principles

The system landscape follows these principles:

- Each system is an independent application.
- Each system owns and manages its own database.
- Each system is the authoritative source for its business domain.
- Systems communicate through defined interfaces.
- The Enterprise Integration Platform coordinates cross-system communication.
- Data exchanged between systems must be validated and transformed when necessary.
- Integration failures must not result in silent data loss.
- Operations that may be repeated must support idempotent processing.

## 2. Order Management System (OMS)

### 2.1 Overview

The Order Management System is responsible for managing delivery requests submitted by Pacific Logistics' business customers.

It acts as the entry point for delivery operations, storing customer information, shipment details, delivery addresses, and order lifecycle information.

The OMS is the authoritative source for customer and delivery order information.

### 2.2 Main Responsibilities

- Register and manage business customers.
- Receive and register delivery requests.
- Store shipment and delivery address information.
- Calculate estimated delivery prices.
- Manage delivery order status.
- Provide order information to authorized systems.
- Receive transportation and delivery status updates.
- Maintain the operational history of delivery orders.

### 2.3 Main Data Entities

| Entity | Description |
|---|---|
| Customer | Business customer requesting delivery services. |
| Delivery Order | Main record representing a delivery request. |
| Shipment | Information about the items or goods being transported. |
| Delivery Address | Pickup and destination addresses. |
| Order Status | Current operational state of a delivery order. |
| Delivery Quote | Estimated delivery price and service information. |

### 2.4 Example Order Lifecycle

A delivery order may follow these statuses:

- `PENDING`: Order registered and awaiting processing.
- `VALIDATED`: Order information successfully validated.
- `DISPATCH_REQUESTED`: Transportation assignment requested.
- `IN_TRANSIT`: Delivery is being transported.
- `DELIVERED`: Delivery successfully completed.
- `CANCELLED`: Delivery order cancelled.
- `FAILED`: Delivery could not be completed.

The OMS owns the delivery order lifecycle, while transportation-specific statuses and events originate from the TMS.

## 3. Transportation Management System (TMS)

### 3.1 Overview

The Transportation Management System manages the operational execution of delivery services.

After receiving a dispatch request, the TMS is responsible for assigning available transportation resources, managing delivery tasks, tracking shipment progress, and recording delivery confirmations.

The TMS is the authoritative source for transportation and delivery execution information.

### 3.2 Main Responsibilities

- Manage drivers and transportation carriers.
- Maintain driver and carrier availability.
- Receive delivery dispatch requests.
- Assign deliveries to available transportation resources.
- Manage transportation tasks.
- Generate tracking events and status updates.
- Record delivery confirmations.
- Report transportation failures and unsuccessful delivery attempts.

### 3.3 Main Data Entities

| Entity | Description |
|---|---|
| Driver | Individual responsible for executing a delivery. |
| Carrier | Transportation provider responsible for delivery services. |
| Transport Assignment | Relationship between a delivery task and its assigned transportation resource. |
| Delivery Task | Operational task representing a delivery execution. |
| Tracking Event | Record of a shipment's transportation status and location. |
| Delivery Confirmation | Evidence that a delivery was completed or unsuccessfully attempted. |

### 3.4 Example Delivery Lifecycle

A delivery task may follow these statuses:

- `RECEIVED`: Dispatch request received.
- `ASSIGNED`: Driver or carrier assigned.
- `PICKED_UP`: Shipment collected.
- `IN_TRANSIT`: Shipment is being transported.
- `DELIVERED`: Shipment successfully delivered.
- `DELIVERY_FAILED`: Delivery attempt unsuccessful.
- `CANCELLED`: Transportation task cancelled.

The TMS communicates relevant transportation events to the integration platform, which coordinates the corresponding updates in the OMS and Billing System.

## 4. Billing & Financial Management System

### 4.1 Overview

The Billing & Financial Management System manages the financial records associated with Pacific Logistics' delivery operations.

It receives relevant delivery information, registers delivery charges, generates invoices, records payments, and supports financial reconciliation.

The Billing System is the authoritative source for financial transactions, invoices, and payment information.

### 4.2 Main Responsibilities

- Manage business billing accounts.
- Register delivery charges.
- Maintain applicable billing amounts and adjustments.
- Generate and manage invoices.
- Record payment information.
- Track outstanding balances.
- Perform financial reconciliation.
- Identify inconsistencies between operational and financial records.

### 4.3 Main Data Entities

| Entity | Description |
|---|---|
| Billing Account | Financial account associated with a business customer. |
| Delivery Charge | Financial charge associated with a delivery order. |
| Invoice | Financial document containing one or more charges. |
| Payment | Record of a payment received or processed. |
| Financial Transaction | Record of a financial operation. |
| Reconciliation Record | Record used to compare and validate financial information. |

### 4.4 Example Billing Lifecycle

A delivery charge may follow these statuses:

- `PENDING`: Charge received and awaiting processing.
- `CONFIRMED`: Charge validated and registered.
- `INVOICED`: Charge included in an invoice.
- `PAID`: Associated amount successfully paid.
- `DISPUTED`: Charge is under review.
- `CANCELLED`: Charge cancelled.

The Billing System must validate financial information before registering charges or generating invoices.

## 5. Data Ownership and Responsibilities

To maintain clear separation of responsibilities, each system owns its business data.

| Data Domain | Authoritative System | Other Systems |
|---|---|---|
| Customer information | OMS | May consume customer references and required details. |
| Delivery orders | OMS | TMS consumes dispatch information; Billing consumes order references. |
| Shipment information | OMS | TMS consumes transportation requirements. |
| Estimated delivery price | OMS | Billing receives relevant pricing information. |
| Drivers and carriers | TMS | OMS may receive assignment references and status updates. |
| Transportation assignments | TMS | OMS receives relevant assignment information. |
| Tracking events | TMS | OMS consumes updates for order status synchronization. |
| Delivery confirmations | TMS | OMS and Billing consume completion information. |
| Delivery charges | Billing | Other systems may consume charge references and status. |
| Invoices and payments | Billing | OMS may consume relevant billing status when required. |

### 5.1 Data Ownership Rules

- The OMS is responsible for creating and maintaining delivery orders.
- The TMS is responsible for transportation assignments and execution records.
- The Billing System is responsible for financial records and payment information.
- The integration platform must not become the authoritative owner of business data.
- Systems must exchange identifiers to maintain traceability across business operations.
- Updates to business records must be performed through the system that owns the corresponding data.

## 6. Integration Requirements and Data Exchange

The three systems must exchange information to support the complete delivery lifecycle.

The Enterprise Integration Platform coordinates these interactions.

### 6.1 Main Integration Flows

#### Flow 1 — Delivery Order Dispatch

**Source:** OMS

**Destination:** TMS

**Trigger:** A delivery order is validated and ready for transportation.

**Information exchanged:**
- Order identifier.
- Customer reference.
- Pickup address.
- Destination address.
- Shipment details.
- Requested delivery service.
- Estimated delivery price, when relevant.

**Expected outcome:** The TMS receives the dispatch request and creates a delivery task.

#### Flow 2 — Transportation Assignment

**Source:** TMS

**Destination:** OMS

**Trigger:** A driver or carrier is successfully assigned.

**Information exchanged:**
- Order identifier.
- Delivery task identifier.
- Driver or carrier reference.
- Assignment timestamp.
- Transportation status.

**Expected outcome:** The OMS receives the assignment information and updates the delivery order accordingly.

#### Flow 3 — Tracking and Delivery Status Synchronization

**Source:** TMS

**Destination:** OMS

**Trigger:** A transportation status changes or a tracking event is generated.

**Information exchanged:**
- Order identifier.
- Delivery task identifier.
- Event identifier.
- Event type.
- Event timestamp.
- Location information, when available.

**Expected outcome:** The OMS maintains an updated view of the delivery lifecycle.

#### Flow 4 — Delivery Charge Registration

**Source:** TMS / OMS

**Destination:** Billing System

**Trigger:** A delivery is successfully completed and the required order and execution information is available.

**Information exchanged:**
- Order identifier.
- Delivery task identifier.
- Customer or billing account reference.
- Confirmed delivery status.
- Agreed pricing information.
- Applicable delivery charges.

**Expected outcome:** The Billing System validates and registers the delivery charge.

The OMS provides the order and agreed pricing information, while the TMS provides the delivery execution confirmation.

#### Flow 5 — Billing Status Synchronization

**Source:** Billing System

**Destination:** OMS

**Trigger:** A relevant financial status changes.

**Information exchanged:**
- Order identifier.
- Billing account reference.
- Invoice identifier, when available.
- Charge identifier.
- Financial status.

**Expected outcome:** The OMS can display relevant billing information without becoming the owner of financial records.

### 6.2 Integration Requirements

The integration platform must support:

- REST API communication between systems.
- Event-based communication for asynchronous operations.
- Data validation and transformation.
- Consistent identifiers across systems.
- Idempotent message processing.
- Retry mechanisms for transient failures.
- Error handling and dead-letter message management.
- Event traceability and correlation.
- Financial and operational data consistency.

The specific communication protocols, infrastructure components, and implementation details will be defined in the architecture design.

## 7. Example JSON Payloads

The following payloads illustrate the conceptual data exchanged between systems. They are examples of integration contracts, not final API specifications.

### 7.1 Delivery Dispatch Request (OMS → TMS)

```json
{
  "event_id": "evt-10001",
  "event_type": "delivery.dispatch_requested",
  "occurred_at": "2026-11-10T10:30:00Z",
  "correlation_id": "corr-50001",
  "data": {
    "order_id": "ord-10001",
    "customer_id": "cus-20001",
    "service_type": "SAME_DAY",
    "pickup_address": {
      "city": "Lisbon",
      "country": "Portugal"
    },
    "delivery_address": {
      "city": "Cascais",
      "country": "Portugal"
    },
    "shipment": {
      "description": "Package",
      "weight_kg": 3.5
    },
    "estimated_price": {
      "amount": 12.50,
      "currency": "EUR"
    }
  }
}
```

### 7.2 Transportation Status Update (TMS → OMS)

```json
{
  "event_id": "evt-10002",
  "event_type": "delivery.status_updated",
  "occurred_at": "2026-11-10T11:15:00Z",
  "correlation_id": "corr-50001",
  "data": {
    "order_id": "ord-10001",
    "delivery_task_id": "task-30001",
    "status": "IN_TRANSIT",
    "driver_id": "drv-40001",
    "location": {
      "city": "Lisbon",
      "country": "Portugal"
    }
  }
}
```

### 7.3 Delivery Confirmation (TMS → Billing)

```json
{
  "event_id": "evt-10003",
  "event_type": "delivery.completed",
  "occurred_at": "2026-11-10T12:00:00Z",
  "correlation_id": "corr-50001",
  "data": {
    "order_id": "ord-10001",
    "delivery_task_id": "task-30001",
    "customer_id": "cus-20001",
    "delivery_status": "DELIVERED",
    "completed_at": "2026-11-10T12:00:00Z",
    "agreed_price": {
      "amount": 12.50,
      "currency": "EUR"
    }
  }
}
```

### 7.4 Delivery Charge Registration (OMS / TMS → Billing)

```json
{
  "event_id": "evt-10004",
  "event_type": "billing.delivery_charge_requested",
  "occurred_at": "2026-11-10T12:05:00Z",
  "correlation_id": "corr-50001",
  "data": {
    "order_id": "ord-10001",
    "delivery_task_id": "task-30001",
    "billing_account_id": "acc-60001",
    "charge": {
      "amount": 12.50,
      "currency": "EUR"
    },
    "description": "Same-day delivery service"
  }
}
```

### 7.5 Billing Status Update (Billing → OMS)

```json
{
  "event_id": "evt-10005",
  "event_type": "billing.charge_confirmed",
  "occurred_at": "2026-11-10T12:10:00Z",
  "correlation_id": "corr-50001",
  "data": {
    "order_id": "ord-10001",
    "charge_id": "chg-70001",
    "billing_account_id": "acc-60001",
    "status": "CONFIRMED",
    "amount": {
      "value": 12.50,
      "currency": "EUR"
    }
  }
}
```

## 8. System Independence and Assumptions

### 8.1 System Independence

For this project, OMS, TMS, and Billing will be implemented as independent simulated applications.

Each application will have:

- Its own codebase or clearly isolated application boundary.
- Its own database.
- Its own data models.
- Its own REST API interfaces.
- Independent business logic.
- Independent configuration and execution.

The systems will not directly share database tables or access each other's databases.

All cross-system communication will be coordinated through the Enterprise Integration Platform.

### 8.2 Project Assumptions

- All three systems are fictional applications developed specifically for this portfolio project.
- No real third-party logistics provider will be integrated initially.
- Driver and carrier information will be simulated.
- No physical fleet tracking or real GPS integration is required.
- Payments will be simulated; no real payment gateway is required.
- Financial operations will focus on delivery charges, invoices, payment records, and reconciliation.
- The integration platform will handle communication and data synchronization rather than owning the business transactions themselves.
- Initial interfaces and payloads may evolve as implementation requirements become clearer.

## 9. Expected Outcomes

The defined system landscape establishes clear business responsibilities and integration boundaries for Pacific Logistics.

The resulting solution will demonstrate how independent enterprise applications can exchange operational and financial information while maintaining data ownership and consistency.

The three systems provide the foundation for implementing and demonstrating:

- Enterprise application integration.
- REST API development.
- Event-driven communication.
- Data transformation and validation.
- Asynchronous processing.
- Distributed transaction handling.
- Idempotency and duplicate prevention.
- Retry and failure recovery mechanisms.
- Operational and financial data synchronization.

This document establishes the conceptual system boundaries and integration requirements that will guide the initial architecture and subsequent implementation.

---

**Document Status:** Initial system definition

**Related Document:** `docs/requirements/business-scenario.md`

**Related Project:** Enterprise Integration Platform — Pacific Logistics