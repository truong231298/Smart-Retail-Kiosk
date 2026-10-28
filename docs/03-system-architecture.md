# RoShopSmart AI Retail Kiosk — System Architecture

**Document:** `03-system-architecture.md`  
**Version:** `1.0`  
**Status:** Draft / Initial  
**Project:** `roshopsmart-ai-retail-kiosk`  
**Related Documents:**
- `01-project-vision.md`
- `02-system-requirements.md`

---

# 1. Purpose

This document defines the high-level architecture of the **RoShopSmart AI Retail Kiosk**.

The architecture describes:

- Major system components.
- Component responsibilities.
- Communication patterns.
- Data flow.
- AI integration.
- Computer Vision integration.
- Backend architecture.
- Database architecture.
- IoT architecture.
- Embedded abstraction.
- State-machine integration.
- Security boundaries.
- Deployment architecture.
- Scalability strategy.

The purpose is to establish a stable architectural foundation before implementation begins.

---

# 2. Architecture Goals

The architecture is designed around the following goals:

1. **Modularity**
2. **Separation of concerns**
3. **Testability**
4. **Replaceability**
5. **Hardware abstraction**
6. **AI model independence**
7. **Observable system behavior**
8. **Security**
9. **Scalability**
10. **Reproducibility**

The system should allow individual technologies to be replaced without requiring major changes to unrelated components.

For example:

```text
YOLO
  ↓
can be replaced by
  ↓
another object detector
```

without rewriting:

```text
Shopping Cart
Checkout
Database
MQTT
```

Similarly:

```text
Embedded Simulator
```

should eventually be replaceable by:

```text
ESP32
```

without rewriting the business logic.

---

# 3. Architectural Principle

The central architectural principle is:

> **Business logic should depend on interfaces and domain concepts, not concrete AI models, hardware devices, or external service providers.**

Conceptually:

```text
                 DOMAIN / BUSINESS LOGIC
                          │
             ┌────────────┼────────────┐
             │            │            │
             ▼            ▼            ▼
        AI Interface   Device API   Data Repository
             │            │            │
             ▼            ▼            ▼
           YOLO        MQTT Adapter  PostgreSQL
        InsightFace   Simulator
        OpenAI API    ESP32
```

This reduces coupling between system components.

---

# 4. High-Level Architecture

The overall system consists of six major layers:

```text
┌───────────────────────────────────────────────┐
│  1. EXPERIENCE LAYER                         │
│                                               │
│  Kiosk UI / Voice / Camera / Interaction      │
└───────────────────────┬───────────────────────┘
                        │
                        ▼
┌───────────────────────────────────────────────┐
│  2. INTELLIGENCE LAYER                       │
│                                               │
│  Computer Vision / AI Assistant / Tracking    │
└───────────────────────┬───────────────────────┘
                        │
                        ▼
┌───────────────────────────────────────────────┐
│  3. APPLICATION LAYER                        │
│                                               │
│  State Machine / Cart / Checkout / Services  │
└───────────────────────┬───────────────────────┘
                        │
                        ▼
┌───────────────────────────────────────────────┐
│  4. DATA & INTEGRATION LAYER                 │
│                                               │
│  PostgreSQL / REST / WebSocket / MQTT        │
└───────────────────────┬───────────────────────┘
                        │
                        ▼
┌───────────────────────────────────────────────┐
│  5. DEVICE LAYER                             │
│                                               │
│  Embedded Simulator / ESP32 / Sensors        │
└───────────────────────────────────────────────┘

                        +
                        
┌───────────────────────────────────────────────┐
│  6. INFRASTRUCTURE LAYER                     │
│                                               │
│  Docker / CI / Logging / Monitoring           │
└───────────────────────────────────────────────┘
```

---

# 5. Complete System Architecture

The target architecture is:

```text
                                      CUSTOMER
                                         │
                       ┌─────────────────┴─────────────────┐
                       │                                   │
                    Camera                              Voice
                       │                                   │
                       ▼                                   ▼
              ┌─────────────────┐                ┌─────────────────┐
              │ Camera Service  │                │ Voice Service   │
              └────────┬────────┘                └────────┬────────┘
                       │                                  │
                       ▼                                  ▼
              ┌─────────────────┐                ┌─────────────────┐
              │ Computer Vision │                │ AI Assistant    │
              │ Pipeline        │                │                 │
              └────────┬────────┘                └────────┬────────┘
                       │                                  │
                       └──────────────┬───────────────────┘
                                      │
                                      ▼
                         ┌────────────────────────┐
                         │     KIOSK CONTROLLER   │
                         └────────────┬───────────┘
                                      │
                                      ▼
                         ┌────────────────────────┐
                         │     STATE MACHINE      │
                         └────────────┬───────────┘
                                      │
                                      ▼
                         ┌────────────────────────┐
                         │      APPLICATION       │
                         │        SERVICES        │
                         ├────────────────────────┤
                         │ Customer Service       │
                         │ Product Service        │
                         │ Cart Service           │
                         │ Verification Service   │
                         │ Checkout Service       │
                         │ Payment Service        │
                         │ Device Service         │
                         └────────────┬───────────┘
                                      │
                    ┌─────────────────┼──────────────────┐
                    │                 │                  │
                    ▼                 ▼                  ▼
             ┌────────────┐    ┌────────────┐    ┌────────────┐
             │ PostgreSQL │    │ AI Gateway │    │ MQTT       │
             └────────────┘    └────────────┘    │ Broker     │
                                                 └─────┬──────┘
                                                       │
                                                       ▼
                                            ┌─────────────────────┐
                                            │ Embedded Simulator   │
                                            ├─────────────────────┤
                                            │ LCD                 │
                                            │ LED                 │
                                            │ Servo               │
                                            │ Sensors             │
                                            └─────────────────────┘
```

---

# 6. Architectural Layers

## 6.1 Experience Layer

The Experience Layer is responsible for interaction with the customer.

Components:

```text
Kiosk UI
Camera
Microphone
Speaker
Touch Interface
```

Responsibilities:

- Display kiosk state.
- Display products.
- Display shopping cart.
- Display errors.
- Capture camera frames.
- Capture voice input.
- Play TTS output.

The Experience Layer should not contain core business logic.

---

# 7. Intelligence Layer

The Intelligence Layer contains AI-related capabilities.

Components:

```text
Computer Vision
Object Detection
Object Tracking
Face Recognition
Gesture Recognition
Speech Recognition
AI Assistant
TTS
```

The Intelligence Layer transforms raw inputs into structured information.

Example:

```text
Camera Frame
     ↓
Object Detection
     ↓
ProductDetectedEvent
```

The application should consume:

```text
ProductDetectedEvent
```

rather than directly depending on:

```text
YOLO model output
```

This is an important abstraction boundary.

---

# 8. Application Layer

The Application Layer contains the core kiosk business logic.

Components:

```text
Kiosk Controller
State Machine
Customer Service
Product Service
Cart Service
Verification Service
Checkout Service
Payment Service
Transaction Service
Device Service
```

This layer determines what the kiosk should do.

Example:

```text
Product Verification Failed
             ↓
Application Logic
             ↓
SUSPICIOUS_TRANSACTION
             ↓
Display Warning
             ↓
Log Event
```

---

# 9. Data & Integration Layer

This layer connects the application to external systems.

Components:

```text
PostgreSQL
REST API
WebSocket
MQTT
AI Gateway
```

Responsibilities:

- Persist application state.
- Communicate with AI providers.
- Communicate with embedded devices.
- Expose APIs.
- Publish events.

---

# 10. Device Layer

The Device Layer represents the physical machine.

Initial implementation:

```text
Web Embedded Simulator
```

Future implementation:

```text
ESP32
Sensors
Servo
LCD
LED
```

The application should not directly depend on the simulator.

Instead:

```text
Application
     ↓
SmartDevice Interface
     ↓
Device Adapter
     ↓
MQTT
     ↓
Simulator / ESP32
```

---

# 11. Infrastructure Layer

The Infrastructure Layer provides runtime and development infrastructure.

Components:

```text
Docker
Docker Compose
GitHub Actions
Logging
Monitoring
Configuration
Secrets
```

The infrastructure layer should make the project reproducible.

---

# 12. Component Architecture

## 12.1 Kiosk Controller

The Kiosk Controller coordinates interaction between the UI and application services.

Responsibilities:

- Receive customer events.
- Receive CV events.
- Trigger state transitions.
- Invoke application services.
- Update UI.

Conceptually:

```text
Camera
  ↓
CV Event
  ↓
Kiosk Controller
  ↓
State Machine
  ↓
Application Service
```

The controller should coordinate rather than implement detailed business rules.

---

# 13. State Machine

The kiosk uses an explicit state machine.

Initial state set:

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

Main flow:

```text
             ┌──────────────┐
             │     IDLE     │
             └──────┬───────┘
                    │
                    ▼
       ┌─────────────────────────┐
       │   CUSTOMER_DETECTED     │
       └────────────┬────────────┘
                    │
                    ▼
             ┌──────────────┐
             │   SHOPPING   │
             └──────┬───────┘
                    │
                    ▼
          ┌─────────────────────┐
          │   PRODUCT_VERIFY   │
          └──────────┬──────────┘
                     │
              ┌──────┴──────┐
              │             │
           MATCH         MISMATCH
              │             │
              ▼             ▼
         ┌─────────┐  ┌──────────────────────┐
         │ CHECKOUT│  │ SUSPICIOUS_TRANSACTION│
         └────┬────┘  └──────────────────────┘
              │
              ▼
         ┌──────────┐
         │ PAYMENT  │
         └────┬─────┘
              │
              ▼
         ┌───────────┐
         │ COMPLETED │
         └─────┬─────┘
               │
               ▼
              IDLE
```

Error handling:

```text
Any State
    │
    ▼
  ERROR
    │
    ▼
Recovery / Reset
    │
    ▼
  IDLE
```

Detailed transitions will be defined in:

```text
docs/07-state-machine.md
```

---

# 14. Computer Vision Architecture

The Computer Vision pipeline is separated from the application.

```text
                 Camera
                   │
                   ▼
            Frame Capture
                   │
                   ▼
             Preprocessing
                   │
                   ▼
             AI Inference
                   │
         ┌─────────┼──────────┐
         ▼         ▼          ▼
      Person    Product      Face
      Detect    Detect      Detect
         │         │          │
         └─────────┼──────────┘
                   ▼
                Tracking
                   │
                   ▼
             Post Processing
                   │
                   ▼
             Event Builder
                   │
                   ▼
          Structured CV Events
```

The CV system communicates with the application through structured events.

---

# 15. AI Assistant Architecture

The AI Assistant is isolated behind an AI Gateway.

```text
Customer
   │
   ▼
Voice / Text
   │
   ▼
AI Assistant Service
   │
   ▼
AI Gateway
   │
   ├───────────────┐
   │               │
   ▼               ▼
OpenAI API     Product Service
                   │
                   ▼
               PostgreSQL
```

The LLM should not directly access the database.

Instead:

```text
LLM
 ↓
Tool / Service
 ↓
Product Service
 ↓
Database
```

This provides better control over:

- Data access.
- Validation.
- Security.
- Business logic.
- Deterministic product information.

---

# 16. AI Gateway

The AI Gateway provides an abstraction over external AI providers.

Example interface:

```text
AIProvider
    │
    ├── OpenAIProvider
    └── FutureProvider
```

Potential responsibilities:

- Prompt management.
- Model selection.
- Tool calling.
- Token tracking.
- Error handling.
- Retry policy.
- Response normalization.

This prevents the rest of the application from directly depending on an external AI provider.

---

# 17. Product Service

The Product Service owns product-related business logic.

Responsibilities:

```text
Get Product
Search Product
Check Availability
Get Price
Get Description
```

The Product Service is the authoritative source for:

```text
Price
SKU
Inventory
Product Metadata
```

The AI Assistant should query this service instead of inventing product information.

---

# 18. Cart Service

The Cart Service manages:

```text
Create Cart
Add Product
Remove Product
Update Quantity
Calculate Total
Validate Cart
```

Conceptual flow:

```text
Customer
   ↓
Cart Service
   ↓
Cart
   ↓
Product Service
   ↓
Price / Availability
```

---

# 19. Verification Service

The Verification Service is responsible for comparing:

```text
Expected Products
        vs
Detected Products
```

Architecture:

```text
Cart Service
     │
     ▼
Expected Product Set
     │
     │
     ▼
Verification Service
     ▲
     │
Detected Product Set
     │
     ▲
Computer Vision
```

Result:

```text
MATCH
MISMATCH
UNKNOWN
INCOMPLETE
```

The Verification Service should not depend directly on YOLO.

---

# 20. Checkout Service

The Checkout Service coordinates final transaction validation.

Flow:

```text
Cart
 ↓
Validate Cart
 ↓
Verify Products
 ↓
Validate Inventory
 ↓
Calculate Total
 ↓
Create Checkout
 ↓
Payment
```

The service should prevent checkout if product verification fails.

---

# 21. Payment Service

The initial implementation uses a simulated payment provider.

Architecture:

```text
PaymentService
      │
      ▼
PaymentProvider
      │
      └── MockPaymentProvider
```

Future:

```text
PaymentProvider
      │
      ├── MockPaymentProvider
      ├── RealProviderA
      └── RealProviderB
```

This maintains provider independence.

---

# 22. Transaction Service

The Transaction Service manages completed transactions.

Responsibilities:

```text
Create Transaction
Store Transaction Items
Store Payment Result
Update Transaction Status
```

Example:

```text
PAYMENT_SUCCESS
      ↓
TransactionService
      ↓
Transaction Created
      ↓
Inventory Update
      ↓
Device Command
```

---

# 23. Database Architecture

PostgreSQL is the primary persistence layer.

Conceptual data model:

```text
Customer
    │
    └── KioskSession
            │
            └── Cart
                  │
                  └── CartItem
                        │
                        └── Product

KioskSession
      │
      ├── VerificationEvent
      ├── AIInteraction
      └── Transaction
              │
              └── TransactionItem

Kiosk
  │
  └── DeviceEvent
```

Detailed schema design will be defined separately.

---

# 24. Repository Pattern

The application should use repository abstractions where appropriate.

Conceptual design:

```text
Application Service
       ↓
Repository Interface
       ↓
Repository Implementation
       ↓
PostgreSQL
```

Example:

```text
ProductRepository
      │
      └── PostgreSQLProductRepository
```

This improves testability.

Unit tests can use:

```text
InMemoryProductRepository
```

without requiring PostgreSQL.

---

# 25. API Architecture

The backend exposes REST APIs.

Conceptual API structure:

```text
/api
│
├── /customers
├── /sessions
├── /products
├── /cart
├── /verification
├── /checkout
├── /payment
├── /transactions
├── /ai
├── /kiosk
└── /devices
```

Example:

```text
GET    /api/products
GET    /api/products/{id}

POST   /api/sessions
GET    /api/sessions/{id}

POST   /api/cart/items
DELETE /api/cart/items/{id}

POST   /api/verification

POST   /api/checkout

POST   /api/payment

GET    /api/transactions/{id}
```

Exact API contracts will be designed later.

---

# 26. WebSocket Architecture

WebSocket may be used for real-time kiosk updates.

Potential use cases:

```text
CV Detection
Kiosk State
Transaction Status
Device Status
```

Architecture:

```text
Computer Vision
       │
       ▼
    Backend
       │
       ▼
   WebSocket
       │
       ▼
    Kiosk UI
```

REST remains the primary interface for request/response operations.

WebSocket is used only where real-time updates provide value.

---

# 27. MQTT Architecture

MQTT is used for device communication.

Architecture:

```text
                 Backend
                    │
                    ▼
              MQTT Broker
                    │
          ┌─────────┼─────────┐
          ▼         ▼         ▼
       LCD Cmd   LED Cmd   Servo Cmd
          │         │         │
          └─────────┼─────────┘
                    ▼
             Embedded Simulator
```

The MQTT broker is not part of business logic.

The Device Service communicates through an adapter.

---

# 28. MQTT Topic Design

Initial topic structure:

```text
kiosk/{kiosk_id}/device/{device_id}/command
kiosk/{kiosk_id}/device/{device_id}/status
kiosk/{kiosk_id}/sensor/{sensor_id}/event
```

Example:

```text
kiosk/kiosk-001/device/lcd-001/command
```

Payload example:

```json
{
  "command": "DISPLAY",
  "message": "PAYMENT SUCCESSFUL"
}
```

---

# 29. Hardware Abstraction Layer

The project will use a hardware abstraction concept.

Conceptual interface:

```text
SmartDevice
│
├── display_message()
├── set_led()
├── move_servo()
├── read_sensor()
└── get_status()
```

Implementations:

```text
SmartDevice
   │
   ├── SimulatorDevice
   │
   └── ESP32Device
```

The application only depends on:

```text
SmartDevice
```

and not:

```text
SimulatorDevice
ESP32Device
```

---

# 30. Embedded Simulator Architecture

The simulator represents the embedded system.

```text
                MQTT Broker
                    │
                    ▼
          ┌────────────────────┐
          │ Embedded Simulator │
          └─────────┬──────────┘
                    │
       ┌────────────┼─────────────┐
       ▼            ▼             ▼
      LCD          LED          Servo
       │            │             │
       └────────────┼─────────────┘
                    │
                    ▼
                 Sensors
```

The simulator should expose a visual representation of the device state.

Example:

```text
┌──────────────────────────────┐
│       SMART KIOSK            │
│                              │
│ LCD: PAYMENT SUCCESSFUL      │
│ LED: ● ON                    │
│ SERVO: OPEN                  │
│ SENSOR: READY                │
└──────────────────────────────┘
```

---

# 31. Event-Driven Architecture

Some system interactions are naturally event-driven.

Examples:

```text
CustomerDetected
ProductDetected
ProductVerificationFailed
PaymentCompleted
DeviceStatusChanged
```

Conceptual event flow:

```text
Event Producer
      │
      ▼
 Event
      │
      ▼
 Event Handler
      │
      ▼
 Application Action
```

Example:

```text
PaymentCompleted
      ↓
TransactionService
      ↓
DeviceService
      ↓
MQTT
      ↓
Servo OPEN
```

---

# 32. Event Model

Events should contain standardized metadata.

Example:

```json
{
  "event_id": "evt_001",
  "event_type": "PRODUCT_DETECTED",
  "timestamp": "2026-10-28T10:00:00Z",
  "kiosk_id": "kiosk-001",
  "session_id": "session-001",
  "payload": {}
}
```

Common metadata:

```text
event_id
event_type
timestamp
kiosk_id
session_id
payload
```

---

# 33. Data Flow — Customer Detection

```text
Camera
  ↓
Frame Capture
  ↓
Person Detection
  ↓
Confidence Check
  ↓
CustomerDetectedEvent
  ↓
Kiosk Controller
  ↓
State Machine
  ↓
CUSTOMER_DETECTED
  ↓
Create Session
  ↓
UI Update
```

---

# 34. Data Flow — Product Detection

```text
Camera
  ↓
Frame
  ↓
Object Detector
  ↓
Product Detection
  ↓
Tracking
  ↓
Post Processing
  ↓
ProductDetectedEvent
  ↓
Backend / Verification Service
```

---

# 35. Data Flow — Product Verification

```text
Shopping Cart
     │
     ▼
Expected Products
     │
     ├──────────────────┐
     │                  │
     ▼                  ▼
Verification       CV Pipeline
Service                │
     ▲                 ▼
     │           Detected Products
     │                  │
     └──────────────────┘
              │
              ▼
          Comparison
              │
       ┌──────┴──────┐
       ▼             ▼
     MATCH        MISMATCH
       │             │
       ▼             ▼
   Checkout       Suspicious
                    Event
```

---

# 36. Data Flow — Checkout

```text
Customer
   ↓
Checkout Request
   ↓
Checkout Service
   ↓
Validate Cart
   ↓
Validate Product Verification
   ↓
Validate Inventory
   ↓
Calculate Total
   ↓
Payment Service
   ↓
Payment Provider
   │
   ├── SUCCESS
   │
   └── FAILED
```

---

# 37. Data Flow — Successful Transaction

```text
Payment Success
      ↓
Transaction Service
      ↓
Persist Transaction
      ↓
Update Inventory
      ↓
Publish TransactionCompleted
      ↓
Device Service
      ↓
MQTT
      ↓
Embedded Simulator
      │
      ├── LCD → SUCCESS
      ├── LED → ON
      └── Servo → OPEN
```

---

# 38. Data Flow — Suspicious Transaction

```text
Cart
 ↓
Expected Products
 ↓
Verification
 ↓
Detected Products
 ↓
Mismatch
 ↓
SUSPICIOUS_TRANSACTION
 ↓
Log Event
 ↓
Display Warning
 ↓
Block Checkout
```

Optional future workflow:

```text
Suspicious Transaction
        ↓
Admin Review
        ↓
Approve / Reject
```

---

# 39. Error Architecture

Errors are divided into categories.

## Client Errors

Examples:

```text
Invalid Request
Product Not Found
Invalid Cart
```

---

## Application Errors

Examples:

```text
Invalid State Transition
Verification Failed
Checkout Failed
```

---

## Infrastructure Errors

Examples:

```text
Database Unavailable
MQTT Unavailable
AI Provider Unavailable
```

---

## Device Errors

Examples:

```text
Servo Failure
Sensor Failure
Device Offline
```

Errors should be normalized at service boundaries.

---

# 40. Resilience Strategy

The system should isolate failures.

Example:

```text
AI Service Down
      │
      ▼
AI Assistant unavailable
      │
      ├───────────────┐
      ▼               ▼
Shopping         Checkout
still works      still works
```

Another example:

```text
TTS Down
  ↓
Text Response Available
```

Another:

```text
MQTT Down
  ↓
Transaction can be recorded
  ↓
Device command can be retried
```

Exact retry and recovery policies will be defined during implementation.

---

# 41. Security Architecture

Security boundaries:

```text
┌───────────────────────────────┐
│         KIOSK CLIENT          │
└───────────────┬───────────────┘
                │
                ▼
         API Authentication
                │
                ▼
┌───────────────────────────────┐
│           BACKEND             │
├───────────────────────────────┤
│ Validation                    │
│ Authorization                │
│ Business Logic               │
│ Audit Logging                │
└───────────────┬───────────────┘
                │
        ┌───────┼────────┐
        ▼       ▼        ▼
     Database   AI      MQTT
```

Sensitive information must not be trusted from the client.

---

# 42. Configuration Architecture

Configuration should be externalized.

Example:

```text
Environment Variables
        │
        ▼
Configuration Layer
        │
        ├── Database
        ├── MQTT
        ├── AI
        ├── Camera
        ├── CV Threshold
        └── Session Timeout
```

Example configuration:

```text
DATABASE_URL
MQTT_BROKER_URL
OPENAI_API_KEY
CV_CONFIDENCE_THRESHOLD
SESSION_TIMEOUT_SECONDS
```

Real credentials must never be committed.

---

# 43. Deployment Architecture

Initial deployment should use Docker Compose.

Conceptual architecture:

```text
┌────────────────────────────────────────────┐
│              Docker Compose                │
│                                            │
│  ┌───────────┐    ┌────────────┐           │
│  │ Backend   │    │ AI Service │           │
│  └─────┬─────┘    └────────────┘           │
│        │                                   │
│        ├──────────────┐                    │
│        ▼              ▼                    │
│  ┌──────────┐   ┌─────────────┐            │
│  │PostgreSQL│   │ MQTT Broker │            │
│  └──────────┘   └──────┬──────┘            │
│                        │                   │
│                        ▼                   │
│                ┌────────────────┐          │
│                │ Embedded       │          │
│                │ Simulator      │          │
│                └────────────────┘          │
└────────────────────────────────────────────┘
```

The exact service decomposition may be simplified during MVP implementation.

---

# 44. Initial Deployment Topology

The first implementation should avoid unnecessary microservices.

Recommended initial topology:

```text
                    ┌─────────────────┐
                    │   Kiosk UI      │
                    └────────┬────────┘
                             │
                             ▼
                    ┌─────────────────┐
                    │    FastAPI      │
                    │    Backend      │
                    └───────┬─────────┘
                            │
               ┌────────────┼─────────────┐
               ▼            ▼             ▼
        ┌────────────┐ ┌─────────┐ ┌────────────┐
        │ PostgreSQL │ │ AI/CV   │ │ MQTT       │
        │            │ │ Services│ │ Broker     │
        └────────────┘ └─────────┘ └─────┬──────┘
                                         │
                                         ▼
                                Embedded Simulator
```

The architecture remains logically modular even if some components run inside the same process initially.

---

# 45. Logical vs Physical Architecture

The project distinguishes between:

## Logical Architecture

How components are organized conceptually.

```text
Kiosk
AI
Application
Database
IoT
```

## Physical Architecture

Where components actually run.

Initial version:

```text
Laptop
 ├── Kiosk UI
 ├── FastAPI
 ├── AI/CV
 ├── PostgreSQL
 ├── MQTT
 └── Embedded Simulator
```

Future:

```text
Kiosk Computer
 ├── UI
 ├── CV
 └── Backend

Network
 │
 ├── Cloud Backend
 └── MQTT Broker

Physical Device
 └── ESP32
```

This separation allows the architecture to evolve without changing the domain model.

---

# 46. Local Development Architecture

A developer should be able to run the system locally.

Expected environment:

```text
Developer Machine
│
├── Docker
│
├── FastAPI
├── PostgreSQL
├── MQTT Broker
├── Embedded Simulator
└── AI/CV Runtime
```

Development workflow:

```text
git clone
      ↓
configure .env
      ↓
docker compose up
      ↓
database starts
      ↓
MQTT starts
      ↓
backend starts
      ↓
simulator starts
      ↓
kiosk starts
```

---

# 47. Scalability Architecture

The system should support future multi-kiosk deployment.

```text
                         Cloud / Backend
                               │
                ┌──────────────┼──────────────┐
                │              │              │
                ▼              ▼              ▼
             Kiosk 01       Kiosk 02       Kiosk 03
                │              │              │
              MQTT           MQTT           MQTT
                │              │              │
              Device        Device         Device
```

Each kiosk should have a unique:

```text
kiosk_id
```

This allows centralized monitoring later.

---

# 48. Observability Architecture

Observability consists of:

```text
Logs
Metrics
Events
```

Conceptually:

```text
Application
    │
    ├── Logs
    ├── Metrics
    └── Events
           │
           ▼
     Observability
```

Important metrics:

```text
CV inference latency
API latency
AI latency
MQTT latency
Transaction duration
Verification accuracy
Error rate
Device availability
```

---

# 49. Testing Architecture

Testing occurs at multiple levels.

```text
                  Tests
                    │
        ┌───────────┼────────────┐
        ▼           ▼            ▼
      Unit      Integration      E2E
        │           │            │
        ▼           ▼            ▼
    Services      API/DB       Full Kiosk
    State         MQTT         Workflow
    Cart          AI
    Verification  CV
```

Example:

```text
Unit Test
    ↓
VerificationService

Integration Test
    ↓
Verification + Database

E2E Test
    ↓
Shopping → Verification → Checkout → Payment
```

---

# 50. AI Model Boundary

AI models must be isolated from business logic.

Bad architecture:

```text
CheckoutService
      ↓
YOLO.predict()
```

Preferred:

```text
CheckoutService
      ↓
VerificationService
      ↓
DetectionProvider
      ↓
YOLODetector
```

This allows:

```text
YOLO
YOLOvNext
Alternative Detector
Edge Model
```

to be replaced without changing checkout logic.

---

# 51. External Provider Boundary

External services should be wrapped behind interfaces.

Example:

```text
AIProvider
PaymentProvider
SpeechProvider
TTSProvider
```

Architecture:

```text
Application
    │
    ▼
Provider Interface
    │
    ├── External Implementation
    └── Mock Implementation
```

This is important for testing.

---

# 52. Dependency Direction

Dependencies should generally point inward toward stable application/domain concepts.

Preferred:

```text
Infrastructure
      ↓
Application
      ↓
Domain
```

AI and hardware adapters should depend on application-defined interfaces.

Conceptually:

```text
             Domain
            ▲     ▲
            │     │
     Application  │
        ▲         │
        │         │
 Infrastructure / AI / IoT
```

The exact implementation may evolve as the codebase grows.

---

# 53. Domain Events

The system should use domain-oriented events where useful.

Examples:

```text
CustomerDetected
ShoppingStarted
ProductDetected
ProductAdded
VerificationStarted
VerificationPassed
VerificationFailed
CheckoutStarted
PaymentSucceeded
PaymentFailed
TransactionCompleted
DeviceCommandSent
DeviceError
```

Events provide a clean way to connect independent modules.

---

# 54. End-to-End Architecture Example

A complete successful transaction:

```text
                    CUSTOMER
                       │
                       ▼
                    Camera
                       │
                       ▼
               Person Detection
                       │
                       ▼
              CUSTOMER_DETECTED
                       │
                       ▼
                Kiosk Controller
                       │
                       ▼
                 State Machine
                       │
                       ▼
                   SHOPPING
                       │
                       ▼
                Product Selection
                       │
                       ▼
                Product Detection
                       │
                       ▼
              ProductDetectedEvent
                       │
                       ▼
                  Cart Service
                       │
                       ▼
               Product Verification
                       │
                       ▼
                    MATCH
                       │
                       ▼
                   CHECKOUT
                       │
                       ▼
                    PAYMENT
                       │
                       ▼
               Payment Successful
                       │
                       ▼
               Transaction Service
                       │
                       ├──────────────┐
                       │              │
                       ▼              ▼
                  PostgreSQL        Event
                                      │
                                      ▼
                                Device Service
                                      │
                                      ▼
                                     MQTT
                                      │
                                      ▼
                              Embedded Simulator
                               │       │       │
                               ▼       ▼       ▼
                              LCD     LED     Servo
```

---

# 55. End-to-End Failure Architecture

Example suspicious transaction:

```text
Customer
   ↓
Shopping
   ↓
Expected Cart
   │
   ▼
Computer Vision
   │
   ▼
Detected Products
   │
   ▼
Verification
   │
   ▼
MISMATCH
   │
   ▼
SUSPICIOUS_TRANSACTION
   │
   ├── Log Event
   ├── Block Checkout
   └── Display Explanation
```

The system must not proceed directly to payment.

---

# 56. Architecture Decision Principles

When choosing technologies or changing architecture, decisions should be evaluated against:

```text
1. Simplicity
2. Maintainability
3. Testability
4. Performance
5. Replaceability
6. Security
7. Cost
8. Portfolio Value
```

The simplest architecture capable of satisfying the requirements should be preferred.

---

# 57. Avoiding Premature Microservices

Although the system contains multiple logical components, the initial implementation should not automatically become a large distributed microservice system.

For MVP:

```text
FastAPI Backend
   ├── Customer
   ├── Product
   ├── Cart
   ├── Verification
   ├── Checkout
   ├── Payment
   └── Device
```

AI/CV may run as separate processes or services when required by resource constraints.

The architecture should preserve logical boundaries without introducing unnecessary operational complexity.

---

# 58. Architecture Evolution

The architecture is expected to evolve.

### Version 1

```text
Laptop
 ├── Kiosk
 ├── FastAPI
 ├── AI/CV
 ├── PostgreSQL
 ├── MQTT
 └── Simulator
```

### Version 2

```text
Kiosk Computer
      │
      ├── UI
      └── CV

Cloud
      │
      ├── Backend
      ├── Database
      └── AI Gateway

IoT
      │
      └── ESP32
```

### Future

```text
Edge AI
   +
Cloud AI
   +
Physical Smart Machine
   +
Multi-Kiosk Fleet
```

---

# 59. Architecture Constraints

The following constraints must be respected:

1. The project is independent.
2. No proprietary technology or source code is used.
3. Initial development must not require physical hardware.
4. The embedded layer must be replaceable.
5. AI providers should be replaceable where practical.
6. Sensitive credentials must not be committed.
7. Business logic should remain testable without AI hardware.
8. The system must support benchmark collection.
9. Major architectural decisions must be documented.

---

# 60. Architecture Quality Attributes

The architecture is evaluated using the following quality attributes.

| Attribute | Goal |
|---|---|
| Modularity | High |
| Testability | High |
| Maintainability | High |
| Replaceability | High |
| Observability | High |
| Security | High |
| Performance | Measurable |
| Scalability | Future-ready |
| Cost Efficiency | High |
| Hardware Independence | High |

---

# 61. Architecture Validation Checklist

Before implementation begins, verify:

### Architecture

- [ ] Major components identified.
- [ ] Responsibilities defined.
- [ ] Dependencies defined.
- [ ] Communication paths defined.

### AI

- [ ] AI boundary defined.
- [ ] CV boundary defined.
- [ ] AI provider abstraction defined.

### Backend

- [ ] Application services identified.
- [ ] API boundary defined.
- [ ] Database boundary defined.

### IoT

- [ ] MQTT architecture defined.
- [ ] Device abstraction defined.
- [ ] Simulator architecture defined.
- [ ] Future ESP32 path defined.

### State Machine

- [ ] States identified.
- [ ] Main transitions identified.
- [ ] Error path identified.

### Infrastructure

- [ ] Docker strategy defined.
- [ ] Configuration strategy defined.
- [ ] CI strategy identified.

### Testing

- [ ] Unit testing strategy identified.
- [ ] Integration testing strategy identified.
- [ ] E2E strategy identified.

---

# 62. Architecture Definition of Done

This architecture document is considered complete for the current phase when:

```text
☐ System boundaries defined
☐ Major components identified
☐ Component responsibilities documented
☐ Data flows documented
☐ State-machine integration documented
☐ AI integration documented
☐ CV integration documented
☐ Backend architecture documented
☐ Database architecture documented
☐ MQTT architecture documented
☐ Embedded abstraction documented
☐ Deployment architecture documented
☐ Security boundaries documented
☐ Testing architecture documented
☐ Scalability direction documented
☐ Architecture validation completed
```

---

# 63. Final Architecture Principle

The RoShopSmart AI Retail Kiosk architecture is built around one fundamental idea:

> **The kiosk is an integrated intelligent system in which AI perceives and reasons, software manages business state, and embedded devices execute physical actions.**

The architectural relationship is:

```text
              PERCEPTION
                  │
                  ▼
          Computer Vision / AI
                  │
                  ▼
              DECISION
                  │
                  ▼
          Application Logic
                  │
                  ▼
             STATE MACHINE
                  │
                  ▼
              EXECUTION
                  │
                  ▼
           IoT / Smart Device
                  │
                  ▼
             PHYSICAL ACTION
```

The complete system therefore follows:

```text
Sense
  ↓
Understand
  ↓
Decide
  ↓
Act
  ↓
Observe
  ↓
Verify
  ↓
Learn / Improve
```

This architecture provides the foundation for implementing the RoShopSmart AI Retail Kiosk as an independent **AI Product Engineering & Smart Machines** portfolio project.