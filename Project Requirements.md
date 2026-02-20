# Technical Requirements: PayMaya Integration Microservice

## 1. High-Level Architecture
This microservice acts as the bridge between the **UISP (Ubiquiti ISP)** management system and the **PayMaya** payment gateway.

### 1.1 Logical Flow


1. **Customer** clicks the payment link on their invoice.
2. **Payment Microservice** processes the request and initiates a session with **PayMaya**.
3. **PayMaya** handles the secure transaction.
4. **Payment Microservice** receives updates and posts the payment back to the **UISP API**.

### 1.2 Responsibilities Separation
| Component | Responsibility |
| :--- | :--- |
| **UISP** | Invoice generation, CRM, service management, and final payment posting. |
| **Payment Microservice** | Session creation, webhook handling, reconciliation, and transaction logging. |
| **PayMaya** | Secure payment processing (Cards, E-wallets, etc.). |

---

## 2. Integration Flow

### 2.1 Payment Initiation
- **Trigger:** Customer clicks "Pay Invoice" in UISP.
- **Request:** UISP sends a `POST` request to `/payments/create`.
- **Payload:**
  ```json
  {
    "invoice_id": "INV-123",
    "customer_id": "CUST-456",
    "amount": 1500.00,
    "currency": "PHP"
  }
