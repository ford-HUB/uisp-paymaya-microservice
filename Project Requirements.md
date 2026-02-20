# Software Requirements Specification (SRS)

---

## 1. Project Overview
This microservice serves as a secure middleware to integrate **PayMaya** payment processing with the **UISP** management system. It automates the workflow from invoice generation to payment reconciliation.

## 2. System Architecture
### 2.1 Logical Flow
1. **Customer** → Clicks Payment Link from Invoice.
2. **Payment Microservice** → Validates request and creates PayMaya Session.
3. **PayMaya** → Processes payment (E-wallet/Card).
4. **Webhook** → PayMaya notifies Microservice of success/failure.
5. **UISP API** → Microservice updates the invoice status to "Paid".

### 2.2 Component Responsibilities
| Component | Responsibility |
| :--- | :--- |
| **UISP** | CRM, Invoice Generation, Service Management. |
| **Microservice** | Session creation, Webhook handling, Reconciliation, Retry logic. |
| **PayMaya** | Payment processing and PCI-compliant data handling. |

---

## 3. Functional Requirements

### 3.1 Payment Initiation
* **FR-1:** Receive payment requests via `POST /payments/create` containing `invoice_id`, `amount`, and `customer_id`.
* **FR-2:** Validate the invoice status via UISP API before contacting PayMaya.
* **FR-3:** Generate a PayMaya Checkout URL and redirect the user.
* **FR-4:** Store a local record of the transaction with a `PENDING` status.

### 3.2 Webhook Processing
* **FR-5:** Expose a public endpoint `POST /webhook/paymaya` to receive status updates.
* **FR-6:** **Server-to-Server Verification:** Verify transaction integrity via PayMaya’s GET API (never trust the webhook payload alone).
* **FR-7:** Update UISP via `POST /invoices/{id}/payments` upon successful verification.

---

## 4. Technical Requirements

### 4.1 Tech Stack
* **Framework:** Laravel (API-only mode)
* **Database:** MySQL
* **Queue Management:** Laravel Horizon (for handling UISP API retries)
* **Deployment:** Dockerized (Nginx + PHP-FPM)

### 4.2 Data Model (Core Fields)
**Table: `payments`**
* `id`: UUID (Primary Key)
* `invoice_id`: String (UISP Reference)
* `paymaya_checkout_id`: String (Provider Reference)
* `status`: Enum (PENDING, SUCCESS, FAILED, REFUNDED)
* `amount`: Decimal(10,2)
* `idempotency_key`: String (Prevents duplicate processing)

---

## 5. Security & Compliance
* **HMAC Validation:** All PayMaya webhooks must be validated using the Secret Key.
* **PCI Scope:** No credit card data shall be stored on local servers; PayMaya handles all sensitive financial data.
* **Idempotency:** Implement keys to ensure that a payment is never posted twice to UISP, even if the webhook is sent multiple times.
* **Audit Trail:** Maintain `webhook_logs` to track all incoming communication from the payment provider.

---

## 6. Error Handling Strategy
| Scenario | Handling Procedure |
| :--- | :--- |
| **UISP API Downtime** | Store job in Redis/Horizon; retry with exponential backoff. |
| **Invalid Signature** | Log the attempt as a security event and return `401 Unauthorized`. |
| **Duplicate Webhook** | Detect via `paymaya_checkout_id` and ignore subsequent "Success" signals. |

---

## 7. Development Roadmap
1. **Phase 1:** API Contract & DB Schema Design.
2. **Phase 2:** Core PayMaya & Webhook Integration.
3. **Phase 3:** Resilience (Retry queues & Observability).
4. **Phase 4:** UAT with real UISP invoices.
