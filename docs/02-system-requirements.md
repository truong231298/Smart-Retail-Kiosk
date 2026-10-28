# RoShopSmart AI Retail Kiosk — System Requirements

**Document:** `02-system-requirements.md`  
**Version:** `1.0`  
**Status:** Draft / Initial  
**Project:** `roshopsmart-ai-retail-kiosk`  
**Related Document:** `01-project-vision.md`

---

# 1. Purpose

This document defines the functional and non-functional requirements for the **RoShopSmart AI Retail Kiosk**.

The purpose is to transform the project vision into a concrete engineering specification that can be used to:

- Design the system architecture.
- Define application modules.
- Design APIs.
- Design database entities.
- Define kiosk states.
- Design AI pipelines.
- Design IoT communication.
- Create test cases.
- Define acceptance criteria.
- Evaluate implementation completeness.

The requirements defined here establish the baseline for future implementation.

---

# 2. System Scope

The system is an AI-powered Smart Retail Kiosk prototype that supports:

```text
Customer Detection
        ↓
Customer Interaction
        ↓
Product Selection
        ↓
Product Detection
        ↓
Product Verification
        ↓
Shopping Cart
        ↓
Checkout
        ↓
Payment Simulation
        ↓
IoT Device Control
        ↓
Transaction Completion
```

The system also includes:

```text
AI Assistant
Computer Vision
State Machine
Backend
Database
MQTT
Embedded Simulator
Logging
Testing
Benchmarking
```

---

# 3. System Actors

The system has the following primary actors.

## 3.1 Customer

The customer interacts with the kiosk.

Possible actions:

- Approach kiosk.
- Start shopping.
- View products.
- Select products.
- Ask AI assistant questions.
- Review cart.
- Checkout.
- Simulate payment.
- Receive transaction feedback.

---

## 3.2 Kiosk

The kiosk is the primary system interface.

Responsibilities:

- Detect customer.
- Display UI.
- Capture camera input.
- Capture voice input.
- Display product information.
- Display cart.
- Display checkout status.
- Communicate with backend.
- Control kiosk state.

---

## 3.3 Computer Vision System

Responsibilities:

- Detect people.
- Detect products.
- Track objects.
- Optionally recognize customers.
- Verify products.
- Produce structured detection results.

---

## 3.4 AI Assistant

Responsibilities:

- Answer customer questions.
- Provide product information.
- Explain kiosk operations.
- Explain errors.
- Assist shopping.
- Generate natural-language responses.
- Provide voice responses when TTS is enabled.

---

## 3.5 Backend

Responsibilities:

- Manage business logic.
- Manage customers.
- Manage products.
- Manage carts.
- Manage transactions.
- Manage verification.
- Manage kiosk state.
- Communicate with IoT layer.
- Persist data.

---

## 3.6 Database

The database stores persistent system information.

Potential entities:

```text
Customer
Product
Cart
CartItem
Transaction
TransactionItem
KioskSession
VerificationEvent
SuspiciousEvent
AIInteraction
DeviceEvent
AuditLog
```

The final schema will be defined separately.

---

## 3.7 Embedded Simulator

The Embedded Simulator represents the physical smart-machine layer.

Simulated devices include:

```text
LCD
LED
Servo
Button
Sensors
```

It communicates with the backend through MQTT.

---

## 3.8 Administrator

The administrator is an optional future actor.

Potential capabilities:

- View transactions.
- Monitor kiosk status.
- Review suspicious transactions.
- View logs.
- Manage products.
- Monitor device health.

Administrator functionality is not required for the initial MVP.

---

# 4. Functional Requirements

Functional requirements describe what the system must do.

Requirement identifiers follow this format:

```text
FR-XXX
```

---

# 5. Customer Detection Requirements

## FR-001 — Detect Customer

The system shall detect when a person approaches the kiosk.

### Input

```text
Camera Frame
```

### Processing

```text
Frame
 ↓
Person Detection
 ↓
Detection Confidence
 ↓
Customer Detection Event
```

### Output

```text
CUSTOMER_DETECTED
```

### Acceptance Criteria

- The system detects a person within the configured camera area.
- Detection confidence must exceed a configurable threshold.
- False detections should be logged for evaluation.
- Detection latency must be measurable.

---

## FR-002 — Customer Session Creation

When a customer is detected, the system shall create a kiosk session.

Example:

```text
Customer Detected
        ↓
Create Session
        ↓
CUSTOMER_DETECTED
```

The session must have a unique identifier.

Example:

```text
session_id = kiosk_session_001
```

---

## FR-003 — Customer Session Timeout

The system shall terminate inactive customer sessions after a configurable timeout.

Example:

```text
SHOPPING
    ↓
No interaction
    ↓
Timeout
    ↓
IDLE
```

The timeout value must be configurable.

---

# 6. Customer Recognition Requirements

Customer recognition is an optional capability.

## FR-004 — Face Detection

The system may detect a human face within the camera frame.

Output:

```text
Face Bounding Box
Confidence
```

---

## FR-005 — Face Recognition

If enabled, the system may identify a known customer.

Pipeline:

```text
Face
 ↓
Face Detection
 ↓
Face Alignment
 ↓
Embedding
 ↓
Similarity Search
 ↓
Customer Identity
```

The system must support an unknown identity.

Example:

```text
Known Customer
Unknown Customer
```

---

## FR-006 — Biometric Data Protection

The system shall not expose raw biometric data through public APIs.

Face embeddings must be treated as sensitive data.

The prototype shall avoid storing unnecessary raw camera footage.

---

# 7. Product Management Requirements

## FR-007 — Product Catalog

The system shall maintain a product catalog.

Each product should contain at least:

```text
Product ID
Name
SKU
Category
Price
Description
Image
Stock Status
```

---

## FR-008 — Product Information

The kiosk shall allow the customer to view product information.

Example:

```text
Product:
Coca Cola

Price:
15,000 VND

Description:
Carbonated soft drink.
```

---

## FR-009 — Product Availability

The system shall determine whether a product is available.

Possible states:

```text
AVAILABLE
OUT_OF_STOCK
DISABLED
```

---

# 8. Computer Vision Requirements

## FR-010 — Product Detection

The Computer Vision system shall detect supported products in camera frames.

Output:

```text
Product ID
Bounding Box
Confidence
Timestamp
```

Example:

```json
{
  "product_id": "coca_cola",
  "confidence": 0.94,
  "bbox": [120, 80, 250, 320]
}
```

---

## FR-011 — Multiple Product Detection

The system shall support detecting multiple products in the same frame.

Example:

```text
Frame
 ├── Coca Cola
 ├── Water
 └── Snack
```

---

## FR-012 — Object Tracking

The system shall optionally track detected products across frames.

Example:

```text
Frame 1 → Object #1
Frame 2 → Object #1
Frame 3 → Object #1
Frame 4 → Object #1
```

Tracking IDs must remain stable within the tracking lifecycle whenever possible.

---

## FR-013 — Detection Confidence

Each detection shall contain a confidence score.

Example:

```text
Product: Water
Confidence: 0.91
```

The minimum accepted confidence shall be configurable.

---

## FR-014 — Detection Event

The CV system shall convert raw model output into a standardized detection event.

Example:

```json
{
  "event_type": "PRODUCT_DETECTED",
  "product_id": "water",
  "confidence": 0.91,
  "timestamp": "2026-10-28T10:00:00"
}
```

This prevents the backend from depending directly on a specific AI framework.

---

# 9. Product Verification Requirements

Product verification is one of the core business functions.

## FR-015 — Expected Product Set

The system shall maintain an expected product set based on the customer's cart.

Example:

```text
Expected:

Water × 1
Coca Cola × 1
```

---

## FR-016 — Detected Product Set

The Computer Vision system shall produce an observed product set.

Example:

```text
Detected:

Water × 1
Coca Cola × 2
```

---

## FR-017 — Product Set Comparison

The system shall compare expected and detected products.

Conceptually:

```text
Expected Products
        vs
Detected Products
```

Possible results:

```text
MATCH
MISMATCH
UNKNOWN
INCOMPLETE
```

---

## FR-018 — Quantity Verification

The system shall verify product quantities.

Example:

```text
Expected:
Coca Cola × 1

Detected:
Coca Cola × 2

Result:
MISMATCH
```

---

## FR-019 — Verification Event

Each verification attempt shall produce a structured event.

Example:

```json
{
  "expected": {
    "water": 1,
    "coca_cola": 1
  },
  "detected": {
    "water": 1,
    "coca_cola": 2
  },
  "result": "MISMATCH"
}
```

---

## FR-020 — Suspicious Transaction Detection

If verification fails during checkout, the system shall be capable of transitioning to:

```text
SUSPICIOUS_TRANSACTION
```

The event must be logged.

---

# 10. Shopping Cart Requirements

## FR-021 — Create Cart

The system shall create a shopping cart when a customer starts shopping.

---

## FR-022 — Add Product

The system shall allow a product to be added to the cart.

Example:

```text
ADD_PRODUCT
    ↓
Cart
```

---

## FR-023 — Remove Product

The system shall allow a product to be removed from the cart.

---

## FR-024 — Update Quantity

The system shall allow product quantity to be changed.

---

## FR-025 — Calculate Cart Total

The system shall calculate:

```text
Subtotal
+
Applicable Discounts
+
Taxes / Fees if configured
=
Total
```

For the initial prototype, tax and discount logic may be simplified.

---

## FR-026 — Cart Validation

Before checkout, the system shall validate:

```text
Product Availability
Quantity
Price
Cart State
Verification State
```

---

# 11. Checkout Requirements

## FR-027 — Start Checkout

The customer shall be able to initiate checkout.

State:

```text
SHOPPING
    ↓
CHECKOUT
```

---

## FR-028 — Checkout Summary

The kiosk shall display:

```text
Products
Quantities
Prices
Total
```

---

## FR-029 — Checkout Validation

The backend shall validate the transaction before payment.

Validation includes:

```text
Cart Exists
Products Valid
Prices Valid
Stock Valid
Verification Valid
```

---

## FR-030 — Checkout Failure

If validation fails, checkout shall not proceed.

Example:

```text
Verification Failed
        ↓
Checkout Blocked
```

---

# 12. Payment Requirements

The initial system will use payment simulation.

## FR-031 — Payment Simulation

The system shall support a simulated payment process.

Possible states:

```text
PAYMENT_PENDING
PAYMENT_SUCCESS
PAYMENT_FAILED
```

---

## FR-032 — Payment Success

When payment succeeds:

```text
PAYMENT_SUCCESS
      ↓
Transaction Completed
```

---

## FR-033 — Payment Failure

When payment fails:

```text
PAYMENT_FAILED
      ↓
Display Error
      ↓
Retry / Cancel
```

---

## FR-034 — Transaction Record

A successful payment shall create a transaction record.

The record shall contain:

```text
Transaction ID
Session ID
Products
Total
Payment Status
Timestamp
```

---

# 13. AI Assistant Requirements

## FR-035 — AI Chat

The kiosk shall support natural-language customer interaction.

Example:

```text
Customer:
"What is this product?"

AI:
"This is Coca Cola..."
```

---

## FR-036 — Product Knowledge

The AI assistant shall be able to retrieve product information from a trusted product data source.

The LLM should not be treated as the authoritative source for product price or inventory.

Architecture:

```text
Customer
   ↓
LLM
   ↓
Tool / Product Service
   ↓
Database
   ↓
Verified Information
```

---

## FR-037 — Error Explanation

The AI assistant shall explain system errors in customer-friendly language.

Example:

```text
System:
Verification Failed

AI:
"I detected a difference between the selected products
and the products currently visible to the kiosk."
```

---

## FR-038 — Shopping Assistance

The AI assistant may provide:

- Product recommendations.
- Product comparisons.
- Product descriptions.
- Shopping guidance.

Recommendations must be based on available product information.

---

## FR-039 — AI Interaction Logging

The system shall optionally log AI interactions for evaluation.

Logged information may include:

```text
Session ID
User Query
Response
Latency
Token Usage
Model
Timestamp
```

Sensitive information must not be logged unnecessarily.

---

# 14. Voice Requirements

## FR-040 — Speech Input

The system shall optionally support speech input.

Pipeline:

```text
Microphone
 ↓
Speech Recognition
 ↓
Text
```

---

## FR-041 — Text-to-Speech

The system shall optionally convert AI responses to speech.

Pipeline:

```text
AI Response
 ↓
TTS
 ↓
Audio
```

---

# 15. Kiosk State Machine Requirements

## FR-042 — State Management

The kiosk shall use an explicit state machine.

Initial states:

```text
IDLE
CUSTOMER_DETECTED
SHOPPING
PRODUCT_VERIFY
CHECKOUT
PAYMENT
COMPLETED
ERROR
SUSPICIOUS_TRANSACTION
```

---

## FR-043 — Valid State Transitions

The system shall only allow valid state transitions.

Example:

```text
IDLE
 ↓
CUSTOMER_DETECTED
 ↓
SHOPPING
 ↓
PRODUCT_VERIFY
 ↓
CHECKOUT
 ↓
PAYMENT
 ↓
COMPLETED
 ↓
IDLE
```

---

## FR-044 — Error State

Unexpected failures shall be represented by an explicit error state.

Example:

```text
Any State
    ↓
ERROR
```

The exact transition policy will be defined in the state-machine specification.

---

## FR-045 — State Persistence

Important state transitions should be logged.

Example:

```text
SHOPPING → PRODUCT_VERIFY
PRODUCT_VERIFY → CHECKOUT
```

---

# 16. Backend Requirements

## FR-046 — REST API

The backend shall expose REST APIs for core business operations.

Potential API groups:

```text
/api/customers
/api/products
/api/cart
/api/verification
/api/checkout
/api/payment
/api/kiosk
/api/ai
```

Exact endpoints will be defined later.

---

## FR-047 — API Validation

All external API inputs shall be validated.

Invalid requests must return appropriate error responses.

---

## FR-048 — API Error Handling

The API shall return structured error responses.

Example:

```json
{
  "error": {
    "code": "PRODUCT_NOT_FOUND",
    "message": "The requested product does not exist."
  }
}
```

---

# 17. Database Requirements

## FR-049 — Persistent Storage

The system shall persist important business information.

Minimum expected entities:

```text
Products
Customers
Sessions
Carts
Cart Items
Transactions
Verification Events
```

---

## FR-050 — Transaction Consistency

Checkout operations must preserve transaction consistency.

For example:

```text
Payment Success
     ↓
Transaction Created
     ↓
Inventory Updated
```

These operations should be handled using appropriate database transaction boundaries.

---

# 18. MQTT / IoT Requirements

## FR-051 — MQTT Communication

The system shall use MQTT for communication between backend services and the embedded simulator.

---

## FR-052 — Device Command

The backend shall be capable of publishing device commands.

Example:

```text
device/kiosk/lcd
device/kiosk/led
device/kiosk/servo
```

---

## FR-053 — Device Status

The embedded simulator shall be capable of publishing device status.

Example:

```text
device/kiosk/status
```

---

## FR-054 — Device Abstraction

The backend business logic shall not directly depend on the simulator implementation.

Required architecture:

```text
Business Logic
      ↓
SmartDevice Interface
      ↓
MQTT Adapter
      ↓
Simulator
```

---

## FR-055 — Device Failure

The system shall detect device communication failures when possible.

Example:

```text
MQTT Connection Lost
        ↓
DEVICE_UNAVAILABLE
        ↓
Log Event
```

---

# 19. Embedded Simulator Requirements

## FR-056 — LCD Simulation

The simulator shall display kiosk messages.

Example:

```text
+----------------------+
| PAYMENT SUCCESSFUL   |
| THANK YOU!           |
+----------------------+
```

---

## FR-057 — LED Simulation

The simulator shall support LED states.

Example:

```text
OFF
ON
BLINKING
ERROR
```

---

## FR-058 — Servo Simulation

The simulator shall simulate servo movement.

Example:

```text
LOCKED
 ↓
OPEN
 ↓
CLOSED
```

---

## FR-059 — Sensor Simulation

The simulator may provide virtual sensor events.

Examples:

```text
Button Press
Door Open
Door Closed
Object Present
```

---

# 20. Logging Requirements

## FR-060 — Application Logging

The system shall produce structured logs for important operations.

Examples:

```text
Customer detected
Product detected
Verification started
Verification failed
Checkout started
Payment completed
MQTT command published
Device error
```

---

## FR-061 — Audit Events

Security-sensitive and business-critical operations should generate audit events.

---

# 21. Error Handling Requirements

## FR-062 — Graceful Failure

The system shall fail gracefully when a component becomes unavailable.

Examples:

```text
AI Service unavailable
Database unavailable
MQTT unavailable
Camera unavailable
```

---

## FR-063 — User-Friendly Errors

Technical errors should not be exposed directly to customers.

Instead:

```text
Internal Error:
DATABASE_CONNECTION_TIMEOUT

Customer Message:
"Sorry, the kiosk is temporarily unavailable."
```

---

## FR-064 — Error Logging

Internal error details must be logged for debugging.

---

# 22. Security Requirements

## FR-065 — Secret Management

API keys and credentials shall not be committed to Git.

Sensitive configuration must use environment variables or a secure secret-management mechanism.

---

## FR-066 — API Authentication

Administrative or sensitive endpoints shall require authentication.

---

## FR-067 — Input Validation

All external inputs must be validated and sanitized.

---

## FR-068 — Sensitive Data Protection

The system shall minimize collection and storage of:

```text
Biometric Data
Voice Data
Camera Footage
Personal Information
```

---

## FR-069 — MQTT Security

Production-oriented MQTT architecture should support:

```text
Authentication
Authorization
TLS
```

The simulator environment may use a simplified configuration.

---

# 23. Non-Functional Requirements

Non-functional requirements define how the system should behave.

Requirement identifiers:

```text
NFR-XXX
```

---

# 24. Performance Requirements

## NFR-001 — Computer Vision Latency

The system should measure Computer Vision inference latency.

Target for initial prototype:

```text
< 100 ms per frame
```

This is an engineering target rather than a guaranteed requirement and may depend on hardware/model configuration.

---

## NFR-002 — Interactive Response

Normal kiosk interactions should feel responsive.

Target:

```text
API response:
< 500 ms
```

excluding external AI inference and other long-running operations.

---

## NFR-003 — AI Response

AI response latency shall be measured separately.

Metrics:

```text
Time to First Token
Total Response Time
TTS Latency
```

No fixed latency target is required initially because external LLM and TTS services may introduce variable latency.

---

# 25. Availability Requirements

## NFR-004 — Component Isolation

Failure of one optional component should not necessarily crash the entire system.

Example:

```text
TTS unavailable
      ↓
AI text response remains available
```

---

## NFR-005 — Recovery

The system should recover from temporary failures where possible.

Example:

```text
MQTT Connection Lost
        ↓
Reconnect
        ↓
Resume Communication
```

---

# 26. Scalability Requirements

## NFR-006 — Modular Services

Major components should be independently replaceable.

```text
CV Service
AI Service
Backend
IoT Service
Database
```

---

## NFR-007 — Multi-Kiosk Support

The architecture should allow future support for multiple kiosks.

Example:

```text
             Backend
                │
      ┌─────────┼─────────┐
      ▼         ▼         ▼
   Kiosk 01  Kiosk 02  Kiosk 03
```

This is a future scalability requirement rather than an MVP implementation requirement.

---

# 27. Maintainability Requirements

## NFR-008 — Modular Code

Code shall be organized by responsibility.

Example:

```text
AI
Backend
Kiosk
Embedded
Infrastructure
Tests
```

---

## NFR-009 — Documentation

Major components must have technical documentation.

---

## NFR-010 — Configuration

Important parameters must be configurable without modifying source code.

Examples:

```text
Detection Threshold
Session Timeout
MQTT Broker
Database URL
AI Model
Camera Device
```

---

# 28. Testability Requirements

## NFR-011 — Unit Testing

Core business logic shall have unit tests.

Minimum targets:

```text
State Machine
Cart
Pricing
Verification
Validation
```

---

## NFR-012 — Integration Testing

The system shall support integration testing for:

```text
API
Database
MQTT
AI Service
CV Service
```

---

## NFR-013 — End-to-End Testing

At least one complete automated workflow should be testable:

```text
Customer
 ↓
Shopping
 ↓
Verification
 ↓
Checkout
 ↓
Payment
 ↓
Completion
```

---

# 29. Observability Requirements

## NFR-014 — Structured Logging

Logs should contain structured information.

Example:

```json
{
  "event": "PRODUCT_VERIFICATION",
  "session_id": "session_001",
  "result": "MATCH",
  "latency_ms": 82
}
```

---

## NFR-015 — Metrics

The system should measure:

```text
CV latency
API latency
AI latency
MQTT latency
Transaction duration
Verification accuracy
Error rate
```

---

# 30. Reliability Requirements

## NFR-016 — Idempotency

Critical operations should avoid accidental duplication.

Example:

```text
Payment request
      ↓
Same request repeated
      ↓
Should not create duplicate transaction
```

---

## NFR-017 — Consistent Transaction State

The system should prevent inconsistent states such as:

```text
Payment Failed
+
Transaction Completed
```

---

# 31. Security Non-Functional Requirements

## NFR-018 — No Secrets in Repository

The GitHub repository must not contain:

```text
API Keys
Passwords
Private Tokens
Database Credentials
```

---

## NFR-019 — Secure Configuration

The repository shall provide:

```text
.env.example
```

without real secrets.

---

## NFR-020 — Data Minimization

Only necessary data should be collected and persisted.

---

# 32. Deployment Requirements

## NFR-021 — Reproducible Environment

The system should be runnable using Docker where practical.

---

## NFR-022 — Local Development

A new developer should be able to initialize the project with documented commands.

Expected workflow:

```text
git clone
    ↓
configure .env
    ↓
docker compose up
    ↓
run services
```

---

## NFR-023 — CI

GitHub Actions should eventually execute:

```text
Lint
Unit Tests
Integration Tests
Build
```

---

# 33. Documentation Requirements

## NFR-024 — Architecture Documentation

System architecture must be documented.

---

## NFR-025 — Technical Decisions

Important architecture decisions must be documented.

---

## NFR-026 — API Documentation

Backend APIs should have machine-readable or interactive documentation.

---

## NFR-027 — Benchmark Documentation

AI and system performance results must be documented.

---

# 34. Usability Requirements

## NFR-028 — Clear Feedback

The kiosk shall clearly communicate system status.

Example:

```text
Detecting customer...
```

```text
Scanning product...
```

```text
Verifying cart...
```

```text
Payment successful.
```

---

## NFR-029 — Error Explanation

Errors should provide actionable information.

Bad:

```text
ERROR 500
```

Better:

```text
We could not verify the selected products.
Please check the products and try again.
```

---

# 35. Privacy Requirements

## NFR-030 — Camera Privacy

Camera data should be processed only for the intended functionality.

---

## NFR-031 — Biometric Privacy

If face recognition is enabled:

- Store only necessary information.
- Avoid storing raw images unless required.
- Protect embeddings.
- Provide clear documentation about the experimental nature of the feature.

---

## NFR-032 — AI Privacy

Customer queries should not unnecessarily contain sensitive personal information.

---

# 36. System Constraints

The initial prototype has the following constraints.

### Hardware

The project should initially work without physical hardware.

```text
Physical ESP32
      ↓
Optional Future Hardware
```

The first implementation will use:

```text
Web Embedded Simulator
```

---

### Compute

Computer Vision performance depends on available hardware.

The benchmark must therefore record:

```text
CPU
GPU
RAM
Model
Resolution
FPS
Latency
```

---

### External AI Services

LLM, Speech Recognition, and TTS may depend on external APIs.

The system should isolate external AI providers behind service interfaces where practical.

---

# 37. MVP Definition

The minimum viable prototype should support:

```text
1. Customer detection
2. Product catalog
3. Product detection
4. Shopping cart
5. Product verification
6. Checkout
7. Payment simulation
8. Kiosk state machine
9. Backend API
10. PostgreSQL
11. MQTT
12. Embedded simulator
```

AI Assistant, face recognition, voice, gesture recognition, and advanced tracking may initially be implemented as incremental capabilities.

---

# 38. MVP User Flow

The MVP must support this flow:

```text
┌──────────────┐
│     IDLE     │
└──────┬───────┘
       │
       ▼
Customer Detected
       │
       ▼
    Shopping
       │
       ▼
Add Products
       │
       ▼
Product Verification
       │
       ├───────────────┐
       │               │
      MATCH         MISMATCH
       │               │
       ▼               ▼
   Checkout       Suspicious
       │           Transaction
       ▼
Payment
       │
       ├──────────────┐
       │              │
    SUCCESS         FAILED
       │              │
       ▼              ▼
 Completed          Error
       │
       ▼
      IDLE
```

---

# 39. Acceptance Criteria

The system shall be considered MVP-complete when:

### Customer

- [ ] Customer can start a kiosk session.
- [ ] Session has a unique identifier.
- [ ] Session timeout works.

### Product

- [ ] Product catalog exists.
- [ ] Products can be added to cart.
- [ ] Products can be removed.
- [ ] Cart total is calculated correctly.

### Computer Vision

- [ ] Supported products can be detected.
- [ ] Detection confidence is available.
- [ ] Detection events have timestamps.
- [ ] Detection results can be consumed by backend.

### Verification

- [ ] Expected products are generated from cart.
- [ ] Detected products can be compared.
- [ ] Matching cart is accepted.
- [ ] Mismatched cart is rejected.
- [ ] Suspicious transaction is logged.

### Checkout

- [ ] Checkout validates the cart.
- [ ] Invalid checkout is rejected.
- [ ] Payment simulation works.
- [ ] Successful payment creates a transaction.

### State Machine

- [ ] Valid transitions work.
- [ ] Invalid transitions are rejected.
- [ ] Error state exists.
- [ ] Completed transaction returns to IDLE.

### IoT

- [ ] MQTT communication works.
- [ ] Backend can publish commands.
- [ ] Simulator receives commands.
- [ ] LCD responds.
- [ ] LED responds.
- [ ] Servo responds.

### Engineering

- [ ] Unit tests exist.
- [ ] Integration tests exist.
- [ ] Docker environment works.
- [ ] Secrets are not committed.
- [ ] Documentation exists.

---

# 40. Traceability

Requirements will later be mapped to:

```text
Requirement
    ↓
Architecture Component
    ↓
Implementation
    ↓
Test Case
    ↓
Benchmark
    ↓
Evidence
```

Example:

```text
FR-010 Product Detection
        ↓
CV Detection Service
        ↓
YOLO Detector
        ↓
test_product_detection.py
        ↓
mAP / Precision / Recall / Latency
        ↓
Benchmark Report
```

Another example:

```text
FR-052 Device Command
        ↓
IoT Service
        ↓
MQTT Adapter
        ↓
test_mqtt_commands.py
        ↓
Command Latency
        ↓
IoT Benchmark
```

---

# 41. Requirement Priority

Requirements will use the following priority levels.

## P0 — Critical

Required for MVP.

Examples:

```text
Product Catalog
Shopping Cart
Checkout
State Machine
Backend
Database
```

---

## P1 — Important

Required for the complete portfolio prototype.

Examples:

```text
Product Detection
Product Verification
MQTT
Embedded Simulator
Testing
Benchmarking
```

---

## P2 — Enhancement

Can be implemented after the core system works.

Examples:

```text
Face Recognition
Gesture Recognition
Voice
TTS
Advanced AI Assistant
```

---

# 42. Initial Priority Matrix

| Requirement | Priority | Phase |
|---|---|---|
| Customer Session | P0 | Phase 1 |
| Product Catalog | P0 | Phase 1 |
| Shopping Cart | P0 | Phase 1 |
| Checkout | P0 | Phase 1 |
| Payment Simulation | P0 | Phase 1 |
| State Machine | P0 | Phase 1 |
| Product Detection | P1 | Phase 2 |
| Product Verification | P1 | Phase 2 |
| Object Tracking | P1 | Phase 2 |
| AI Assistant | P1 | Phase 2 |
| MQTT | P1 | Phase 3 |
| Embedded Simulator | P1 | Phase 3 |
| Face Recognition | P2 | Phase 2 |
| Gesture Recognition | P2 | Phase 2 |
| Voice Input | P2 | Phase 2 |
| TTS | P2 | Phase 2 |
| Real ESP32 | P2 | Future |
| Real Payment | P2 | Future |

---

# 43. Requirement Change Policy

Requirements may change during development.

However, changes must be documented.

A requirement change should answer:

```text
What changed?
Why did it change?
What components are affected?
What tests are affected?
Does the architecture need to change?
```

Major changes should be recorded in:

```text
docs/11-technical-decisions.md
```

---

# 44. Requirements and Development Gate

A development phase should not be considered complete merely because the code runs.

Each requirement must eventually have evidence:

```text
Requirement
    ↓
Implementation
    ↓
Test
    ↓
Evidence
```

For example:

```text
FR-017 Product Set Comparison
        ↓
VerificationService
        ↓
Unit Test
        ↓
Integration Test
        ↓
Demo Scenario
```

---

# 45. Definition of Done for Requirements

A requirement is considered implemented only when:

```text
☐ Requirement understood
☐ Architecture component identified
☐ Implementation completed
☐ Unit test added where applicable
☐ Integration test added where applicable
☐ Error case considered
☐ Logging considered
☐ Security considered
☐ Documentation updated
☐ Demo evidence available where appropriate
```

---

# 46. Summary

The RoShopSmart AI Retail Kiosk requirements define a modular AI-powered retail system consisting of:

```text
                 CUSTOMER
                    │
                    ▼
              KIOSK INTERFACE
                    │
          ┌─────────┴─────────┐
          ▼                   ▼
   COMPUTER VISION       AI ASSISTANT
          │                   │
          └─────────┬─────────┘
                    ▼
              STATE MACHINE
                    │
                    ▼
                 BACKEND
                    │
        ┌───────────┼───────────┐
        ▼           ▼           ▼
    PostgreSQL     AI          MQTT
                                │
                                ▼
                       EMBEDDED SIMULATOR
                                │
                     ┌──────────┼──────────┐
                     ▼          ▼          ▼
                    LCD        LED       SERVO
```

The system will be developed incrementally, beginning with a software-only retail simulation and gradually integrating AI, Computer Vision, MQTT, and embedded-device simulation.

The key engineering principle remains:

> **Every major feature must have a requirement, an implementation, a test, and measurable evidence.**

This requirements document serves as the baseline for the next architectural stage.