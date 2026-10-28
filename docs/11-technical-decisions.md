# RoShopSmart AI Retail Kiosk — Technical Decisions

**Document:** `11-technical-decisions.md`  
**Version:** `1.0`  
**Status:** Active  
**Project:** `roshopsmart-ai-retail-kiosk`

---

# 1. Purpose

This document records the major technical decisions made during the development of RoShopSmart AI Retail Kiosk.

The purpose is to answer:

> **Why was this technical approach selected instead of the alternatives?**

A technical decision should not be based only on:

- familiarity,
- popularity,
- convenience,
- or personal preference.

Instead, decisions should consider:

```text
Problem
↓
Requirements
↓
Constraints
↓
Alternatives
↓
Trade-offs
↓
Decision
↓
Validation
```

This document is also intended to demonstrate engineering reasoning for portfolio and technical interviews.

---

# 2. Decision-Making Principles

The project follows these principles:

### 2.1 Prefer simple architecture first

Do not introduce complexity before the problem requires it.

```text
Modular Monolith
      ↓
Scale when necessary
      ↓
Extract services if justified
```

---

### 2.2 Separate business logic from infrastructure

Business logic must not depend directly on:

```text
MQTT
ESP32
Database
LLM provider
Camera implementation
```

Instead, use interfaces and adapters.

---

### 2.3 Measure before optimizing

No optimization should be considered successful without benchmark evidence.

```text
Baseline
↓
Optimization
↓
Benchmark
↓
Comparison
```

---

### 2.4 Design for hardware migration

The first implementation uses simulation.

However:

```text
Simulator
      ↓
ESP32
```

must be possible without rewriting the business logic.

---

### 2.5 Prefer replaceable components

Major infrastructure components should be replaceable where practical.

Examples:

```text
LLM Provider
CV Model
MQTT Broker
Embedded Device
Payment Provider
Database
```

---

# 3. ADR Format

Each major decision follows this structure:

```text
Decision
Context
Problem
Options
Evaluation
Decision
Trade-offs
Consequences
Validation
Status
```

---

# 4. ADR-001 — Use FastAPI for Backend

## Context

The project requires a backend for:

- kiosk state management,
- product management,
- cart management,
- verification,
- checkout,
- transactions,
- AI integration,
- MQTT integration.

The project also has significant Python-based AI/CV components.

## Options

```text
A. FastAPI
B. Flask
C. Django
D. NestJS
```

## Evaluation

| Criteria | FastAPI | Flask | Django | NestJS |
|---|---:|---:|---:|---:|
| Python AI integration | Excellent | Excellent | Excellent | Medium |
| API development | Excellent | Good | Good | Excellent |
| Async support | Excellent | Good | Good | Excellent |
| Type hints | Excellent | Good | Good | Good |
| CV integration | Excellent | Excellent | Excellent | Medium |
| Project complexity | Low | Low | Higher | Medium |

## Decision

Use **FastAPI**.

## Reason

The project is heavily integrated with:

```text
Python
OpenCV
YOLO
InsightFace
MediaPipe
AI inference
```

Keeping the application backend and AI services in the same ecosystem reduces integration complexity for the prototype.

## Trade-offs

Advantages:

- excellent Python integration,
- strong typing,
- asynchronous capabilities,
- automatic OpenAPI documentation.

Disadvantages:

- less opinionated architecture than Django,
- architecture must be deliberately structured.

## Status

**Accepted**

---

# 5. ADR-002 — Use PostgreSQL

## Context

The kiosk requires persistent data:

```text
Products
Customers
Sessions
Cart
Transactions
Verification Events
AI Interactions
Device Events
```

## Options

```text
A. PostgreSQL
B. MySQL
C. MongoDB
D. SQLite
```

## Decision

Use **PostgreSQL**.

## Reason

The system contains strongly related transactional data.

For example:

```text
Session
 ↓
Cart
 ↓
CartItem
 ↓
Product
 ↓
Transaction
```

A relational database provides:

- foreign keys,
- transactions,
- constraints,
- indexing,
- consistent relational modeling.

## Trade-offs

PostgreSQL introduces more infrastructure than SQLite but provides a more realistic foundation for a production-oriented system.

## Status

**Accepted**

---

# 6. ADR-003 — Use Modular Monolith Initially

## Context

The architecture contains logical domains:

```text
Customer
Product
Cart
Verification
Checkout
Payment
AI
Device
Transaction
```

It is tempting to immediately create separate microservices.

## Options

```text
A. Microservices
B. Modular Monolith
C. Serverless
```

## Decision

Start with a **Modular Monolith**.

```text
FastAPI
│
├── customer
├── product
├── cart
├── verification
├── checkout
├── payment
├── transaction
├── ai
└── device
```

## Reason

The project is a portfolio prototype.

Premature microservices would introduce:

```text
Network complexity
Deployment complexity
Distributed debugging
Service discovery
More CI/CD complexity
```

without providing meaningful benefits at the current scale.

## Migration Strategy

If required:

```text
Modular Monolith
       ↓
Identify bottleneck
       ↓
Extract module
       ↓
Independent service
```

## Status

**Accepted**

---

# 7. ADR-004 — Use MQTT for Device Communication

## Context

The Smart Machine layer requires communication between:

```text
Backend
 ↓
Embedded Simulator
```

and eventually:

```text
Backend
 ↓
ESP32
```

## Options

```text
A. REST
B. WebSocket
C. MQTT
D. Raw TCP
```

## Decision

Use **MQTT**.

## Reason

MQTT is designed for lightweight device communication and supports:

- publish/subscribe,
- QoS,
- retained messages,
- device status,
- asynchronous communication,
- unreliable network environments.

Example:

```text
kiosk/KIOSK-001/device/servo-01/command
```

## Trade-offs

MQTT introduces a broker and additional infrastructure.

However, the benefits are significant for an IoT-oriented architecture.

## Status

**Accepted**

---

# 8. ADR-005 — Use Hardware Abstraction Layer

## Context

The first implementation does not use physical hardware.

The project uses:

```text
Web Embedded Simulator
```

Later:

```text
ESP32
```

should be supported.

## Problem

Without abstraction:

```text
Business Logic
      ↓
MQTT
      ↓
Simulator-specific code
```

would make hardware migration difficult.

## Decision

Introduce:

```text
SmartDevice Interface
```

Example:

```python
class SmartDevice:
    def display_message(self, message):
        ...

    def set_led(self, state):
        ...

    def open(self):
        ...

    def close(self):
        ...

    def get_status(self):
        ...
```

Implementations:

```text
SmartDevice
    │
    ├── SimulatorDevice
    ├── ESP32Device
    └── MockDevice
```

## Result

Business logic does not need to know whether the device is:

```text
Simulator
ESP32
Mock
```

## Status

**Accepted**

---

# 9. ADR-006 — Use Web-Based Embedded Simulator

## Context

Physical hardware is expensive and slows iteration.

The portfolio needs a demonstrable Smart Machine architecture.

## Options

```text
A. Build physical ESP32 immediately
B. Web simulator
C. No device simulation
```

## Decision

Use a **web-based embedded simulator** first.

Architecture:

```text
FastAPI
   ↓
MQTT
   ↓
Embedded Simulator
   ├── LCD
   ├── LED
   ├── Servo
   ├── Button
   └── Sensors
```

## Reason

The simulator provides:

- low cost,
- fast iteration,
- deterministic testing,
- fault injection,
- CI compatibility,
- easy portfolio demonstration.

## Important Constraint

The simulator must implement the same device contract expected from real hardware.

Therefore:

> The simulator is not a fake replacement for embedded engineering; it is the first implementation of the device abstraction.

## Migration

```text
Web Simulator
      ↓
Simulator + ESP32
      ↓
Physical Smart Kiosk
```

## Status

**Accepted**

---

# 10. ADR-007 — Use YOLO for Product Detection

## Context

The kiosk needs product detection.

Requirements:

```text
Real-time inference
Multiple products
Bounding boxes
Product classification
Python integration
```

## Options

```text
A. YOLO
B. Faster R-CNN
C. SSD
D. Traditional OpenCV
```

## Decision

Use **YOLO-based object detection**.

## Reason

YOLO provides a strong balance between:

```text
Accuracy
Speed
Deployment simplicity
Real-time inference
```

It is also suitable for benchmarking different model sizes.

## Trade-offs

A larger model may improve accuracy but increase:

```text
Latency
GPU usage
Memory
```

Therefore model selection will be benchmark-driven.

## Status

**Accepted**

---

# 11. ADR-008 — Use OpenCV as CV Pipeline Foundation

## Context

The system requires:

```text
Camera input
Frame processing
Image transformation
Visualization
Video processing
```

## Decision

Use **OpenCV** as the base computer vision pipeline.

Example:

```text
Camera
 ↓
OpenCV
 ↓
Frame preprocessing
 ↓
AI model
 ↓
Postprocessing
```

## Reason

OpenCV is mature and integrates well with Python-based CV models.

## Status

**Accepted**

---

# 12. ADR-009 — Use InsightFace for Face Recognition

## Context

The kiosk may identify returning users or authorized users.

Requirements include:

```text
Face detection
Face embeddings
Face recognition
Similarity comparison
```

## Options

```text
A. InsightFace
B. Custom CNN
C. Traditional OpenCV face recognition
D. Cloud face API
```

## Decision

Use **InsightFace** for the prototype.

## Reason

It provides modern face analysis capabilities while keeping the inference pipeline within Python.

## Security Consideration

Face recognition must not automatically be treated as authentication.

For high-risk actions:

```text
Face recognition
      +
Additional verification
```

may be required.

## Status

**Accepted**

---

# 13. ADR-010 — AI Assistant Uses Tool-Based Architecture

## Context

The AI assistant needs access to:

```text
Product information
Cart
Transaction state
Kiosk state
```

The LLM should not directly access the database.

## Bad Architecture

```text
LLM
 ↓
Database
```

## Decision

Use:

```text
User
 ↓
LLM
 ↓
Tool
 ↓
Application Service
 ↓
Database
```

Example:

```text
get_product()
get_cart()
search_products()
explain_error()
```

## Reason

This provides:

- authorization,
- validation,
- auditability,
- deterministic business rules,
- reduced hallucination risk.

## Critical Rule

The LLM cannot directly:

```text
Change product price
Modify transactions
Open physical hardware
Delete database records
```

without authorized application-level tools.

## Status

**Accepted**

---

# 14. ADR-011 — Product Service Is Source of Truth

## Context

AI may answer product questions.

However, product information must be authoritative.

## Decision

The Product Service is the source of truth for:

```text
Product name
Price
SKU
Availability
Inventory
Description
```

The LLM may explain this information but must not invent it.

Architecture:

```text
User
 ↓
AI Assistant
 ↓
Product Tool
 ↓
Product Service
 ↓
PostgreSQL
```

## Status

**Accepted**

---

# 15. ADR-012 — Use State Machine for Kiosk Control

## Context

The kiosk has sequential business states.

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
```

Without explicit state management, invalid operations could occur.

## Decision

Use an explicit state machine.

## Benefits

It makes:

```text
Valid transitions
Invalid transitions
Guards
Recovery
Testing
```

explicit.

## Example

Payment is only valid when:

```text
state == CHECKOUT
verification == PASSED
cart != empty
```

## Status

**Accepted**

---

# 16. ADR-013 — Separate Kiosk State from Device State

## Context

There are two different concepts:

```text
Kiosk State
```

and:

```text
Device State
```

Example:

```text
Kiosk:
PAYMENT

Servo:
CLOSED
```

These are not contradictory.

## Decision

Maintain separate state machines.

```text
Kiosk State Machine

IDLE
SHOPPING
VERIFY
CHECKOUT
PAYMENT
COMPLETED
ERROR
SUSPICIOUS
```

Device state:

```text
CLOSED
OPENING
OPEN
CLOSING
ERROR
```

## Reason

This prevents business state and physical state from becoming tightly coupled.

## Status

**Accepted**

---

# 17. ADR-014 — Use Event-Based Device Feedback

## Context

Sending a command is not equivalent to successful execution.

Example:

```text
Backend:
OPEN SERVO
```

does not guarantee:

```text
Servo:
OPEN
```

## Decision

Use command + reported-state architecture.

```text
Command
 ↓
Device
 ↓
Action
 ↓
Sensor / Device State
 ↓
Event
 ↓
Backend
```

## Reason

This enables:

- verification,
- retries,
- fault detection,
- synchronization.

## Status

**Accepted**

---

# 18. ADR-015 — Use Idempotent Device Commands

## Context

Network communication can produce:

```text
Duplicate messages
Retries
Timeouts
Reconnects
```

Sending:

```text
OPEN_SERVO
```

twice should not create an unsafe result.

## Decision

Every important command should have:

```text
command_id
```

Example:

```json
{
  "command_id": "cmd-123",
  "device_id": "servo-01",
  "command": "OPEN"
}
```

The device or Device Service must prevent unsafe duplicate execution.

## Status

**Accepted**

---

# 19. ADR-016 — Use Docker Compose for Local Infrastructure

## Context

The project requires:

```text
FastAPI
PostgreSQL
MQTT Broker
Embedded Simulator
```

## Decision

Use Docker Compose.

Example:

```text
docker-compose.yml
```

Services:

```text
backend
postgres
mqtt
simulator
```

## Benefits

```text
Reproducible environment
Easy onboarding
Easy local testing
Simple integration testing
```

## Status

**Accepted**

---

# 20. ADR-017 — Use Pytest for Testing

## Context

The project requires:

```text
Unit Tests
Integration Tests
E2E Tests
CV Tests
MQTT Tests
Security Tests
```

## Decision

Use **pytest**.

Additional tools may include:

```text
pytest-asyncio
pytest-cov
httpx
```

## Reason

Strong integration with Python and FastAPI.

## Status

**Accepted**

---

# 21. ADR-018 — Test Business Logic Before Hardware

## Context

Hardware debugging is expensive and slow.

## Decision

Implement and test business logic against:

```text
MockDevice
SimulatorDevice
```

before physical ESP32 integration.

Testing flow:

```text
Unit
 ↓
MockDevice
 ↓
Simulator
 ↓
ESP32
```

## Result

Hardware becomes an implementation detail rather than a prerequisite for validating business logic.

## Status

**Accepted**

---

# 22. ADR-019 — Security Boundaries Around Device Control

## Context

A compromised API or AI assistant must not freely control physical devices.

## Decision

Device commands must pass through:

```text
Authorization
 ↓
Validation
 ↓
Device Service
 ↓
Safety Rules
 ↓
MQTT
```

The AI assistant must not directly publish MQTT commands.

Bad:

```text
LLM → MQTT
```

Correct:

```text
LLM
 ↓
Authorized Tool
 ↓
Application Service
 ↓
Device Service
 ↓
MQTT
```

## Status

**Accepted**

---

# 23. ADR-020 — Simulation First, Hardware Second

## Context

The project needs to demonstrate embedded engineering while minimizing cost.

## Decision

Use a staged hardware strategy.

### Stage 1

```text
Web Simulator
```

### Stage 2

```text
Web Simulator + ESP32
```

### Stage 3

```text
Physical Smart Machine
```

## Reason

This allows architecture and business logic to be validated before hardware investment.

## Status

**Accepted**

---

# 24. ADR-021 — Avoid Premature Microservices

## Context

The project may eventually support:

```text
Multiple Kiosks
Multiple Devices
Cloud Deployment
```

However, these requirements do not justify microservices initially.

## Decision

Keep logical modules inside one backend.

Introduce independent services only when supported by evidence such as:

```text
CPU bottleneck
Independent scaling requirement
Deployment independence
Team ownership
Failure isolation
```

## Status

**Accepted**

---

# 25. ADR-022 — Prefer REST for Business APIs

## Context

The kiosk needs predictable business operations.

## Decision

Use REST for:

```text
Products
Sessions
Cart
Verification
Checkout
Transactions
```

Use WebSocket only where real-time updates are actually required.

Use MQTT for device communication.

Therefore:

```text
REST
→ Business operations

WebSocket
→ UI real-time updates

MQTT
→ IoT/device communication
```

## Status

**Accepted**

---

# 26. ADR-023 — Use PostgreSQL Transactions for Checkout

## Context

Checkout modifies multiple pieces of state.

Example:

```text
Create transaction
Create transaction items
Update inventory
Update session
```

Partial success would create inconsistent data.

## Decision

Use database transactions around critical operations.

Conceptually:

```text
BEGIN
 ↓
Validate
 ↓
Create Transaction
 ↓
Create Items
 ↓
Update Inventory
 ↓
Commit
```

If any critical operation fails:

```text
ROLLBACK
```

## Status

**Accepted**

---

# 27. ADR-024 — Payment and Physical Dispensing Must Be Decoupled

## Context

Payment success does not mean the physical device has successfully completed the action.

Bad assumption:

```text
Payment SUCCESS
      =
Servo OPEN
```

## Decision

Use separate stages:

```text
PAYMENT_SUCCESS
       ↓
DEVICE_COMMAND
       ↓
DEVICE_CONFIRMED
       ↓
TRANSACTION_COMPLETED
```

This creates a clear distinction between:

```text
Financial state
Physical state
Business state
```

## Status

**Accepted**

---

# 28. ADR-025 — Safe State on Device Failure

## Context

A physical smart machine must fail safely.

Example:

```text
MQTT disconnected
Servo failure
Sensor failure
Backend unavailable
```

## Decision

When a critical device failure occurs:

```text
Servo → SAFE / LOCKED
LED → ERROR
LCD → Error message
Kiosk → ERROR
```

The system must not dispense products merely because payment occurred if physical verification cannot confirm the safe operation.

## Future Physical Hardware

An emergency stop should eventually operate independently of network software.

## Status

**Accepted**

---

# 29. ADR-026 — Benchmark Before Model Optimization

## Context

There are many possible CV optimizations.

Examples:

```text
Smaller model
Lower resolution
Quantization
Frame skipping
GPU acceleration
```

Choosing without measurement is unreliable.

## Decision

Always benchmark:

```text
Baseline
 ↓
Optimization
 ↓
Benchmark
 ↓
Compare
```

Metrics:

```text
Accuracy
FPS
Latency
Memory
```

## Status

**Accepted**

---

# 30. ADR-027 — Keep AI Provider Replaceable

## Context

The AI assistant may use an external LLM provider.

However, the application should not become tightly coupled to one provider.

## Decision

Introduce an abstraction:

```text
AIProvider
```

Possible implementations:

```text
OpenAIProvider
FutureProvider
MockAIProvider
```

Business logic interacts with:

```text
AIService
```

rather than directly with provider-specific APIs.

## Benefits

```text
Testing
Provider migration
Cost optimization
Offline/local model experimentation
```

## Status

**Accepted**

---

# 31. ADR-028 — Use Synthetic Data During Development

## Context

The project handles potentially sensitive information:

```text
Face data
Customer data
Transaction data
```

## Decision

Development and testing should use:

```text
Synthetic customers
Synthetic transactions
Synthetic events
Controlled test images
```

Real personal data should not be committed to the repository.

## Status

**Accepted**

---

# 32. ADR-029 — Logging Must Be Structured

## Context

The system contains distributed-like components:

```text
Backend
AI
MQTT
Simulator
Database
```

Debugging plain text logs becomes difficult.

## Decision

Use structured logs containing fields such as:

```text
timestamp
level
service
kiosk_id
session_id
event_id
command_id
device_id
message
```

Example:

```json
{
  "level": "INFO",
  "service": "device-service",
  "kiosk_id": "KIOSK-001",
  "device_id": "servo-01",
  "command_id": "cmd-123",
  "message": "Servo opened"
}
```

## Status

**Accepted**

---

# 33. ADR-030 — Git History Is Part of the Engineering Evidence

## Context

A portfolio project should demonstrate engineering process, not only final code.

## Decision

Use meaningful commits.

Examples:

```text
feat: implement kiosk state machine
feat: add product verification service
feat: add MQTT device adapter
test: add checkout state transition tests
test: add suspicious transaction E2E test
perf: optimize product detection pipeline
docs: document embedded architecture
```

Avoid:

```text
update
fix
changes
final
final2
```

## Status

**Accepted**

---

# 34. Decision Reassessment

Technical decisions are not permanent.

A decision may be reconsidered when:

```text
Requirements change
Benchmark results invalidate assumptions
New hardware is introduced
Scale increases
Security requirements change
Cost becomes significant
```

When a decision changes:

```text
Old Decision
     ↓
Reason for Reconsideration
     ↓
New Options
     ↓
New Decision
     ↓
Migration Plan
```

The old decision should remain documented for historical context.

---

# 35. Decision Priority

Not all decisions have equal importance.

### Critical

```text
State Machine
Device Abstraction
Security Boundary
Transaction Integrity
Product Verification
```

### High

```text
Backend Architecture
Database
MQTT
CV Model
AI Tool Architecture
```

### Medium

```text
Frontend Framework
Logging Format
Benchmark Tooling
Visualization
```

### Low

```text
UI Styling
Minor Libraries
Development Utilities
```

Critical decisions require stronger validation.

---

# 36. Technical Decision Review Checklist

Before accepting a major decision:

```text
☐ Problem clearly defined
☐ Requirements identified
☐ Constraints identified
☐ Alternatives considered
☐ Trade-offs documented
☐ Security considered
☐ Performance considered
☐ Maintainability considered
☐ Testing impact considered
☐ Migration path considered
☐ Decision validated
```

---

# 37. Decision Matrix

| Decision | Primary Reason | Status |
|---|---|---|
| FastAPI | Python + AI integration | Accepted |
| PostgreSQL | Transactional relational data | Accepted |
| Modular Monolith | Avoid premature complexity | Accepted |
| MQTT | IoT communication | Accepted |
| SmartDevice HAL | Hardware portability | Accepted |
| Web Simulator | Low-cost hardware simulation | Accepted |
| YOLO | CV speed/accuracy trade-off | Accepted |
| OpenCV | CV pipeline | Accepted |
| InsightFace | Face recognition | Accepted |
| Tool-based AI | Security + grounding | Accepted |
| State Machine | Deterministic workflow | Accepted |
| Docker Compose | Reproducible environment | Accepted |
| Pytest | Python testing | Accepted |
| Synthetic Data | Privacy | Accepted |
| Structured Logging | Observability | Accepted |

---

# 38. Current Architecture Result

The combined technical decisions produce:

```text
                        ┌──────────────────┐
                        │    Customer      │
                        └────────┬─────────┘
                                 │
                                 ▼
                    ┌────────────────────────┐
                    │ Camera / Voice / UI    │
                    └───────────┬────────────┘
                                │
                                ▼
                    ┌────────────────────────┐
                    │ AI / Computer Vision   │
                    │ YOLO / InsightFace     │
                    └───────────┬────────────┘
                                │
                                ▼
                    ┌────────────────────────┐
                    │   Kiosk Controller     │
                    └───────────┬────────────┘
                                │
                                ▼
                    ┌────────────────────────┐
                    │    State Machine       │
                    └───────────┬────────────┘
                                │
                ┌───────────────┼────────────────┐
                ▼               ▼                ▼
        ┌────────────┐  ┌────────────┐   ┌──────────────┐
        │ Product    │  │ Checkout   │   │ AI Assistant │
        │ Service    │  │ Service    │   │              │
        └─────┬──────┘  └─────┬──────┘   └──────────────┘
              │               │
              └───────┬───────┘
                      ▼
               ┌─────────────┐
               │ PostgreSQL  │
               └─────────────┘

                      │
                      ▼
               ┌─────────────┐
               │ Device      │
               │ Service     │
               └──────┬──────┘
                      │
                      ▼
                  ┌───────┐
                  │ MQTT  │
                  └───┬───┘
                      │
             ┌────────┴────────┐
             ▼                 ▼
      ┌──────────────┐  ┌──────────────┐
      │ Web Simulator│  │    ESP32     │
      └──────────────┘  └──────────────┘
```

---

# 39. Engineering Principles Established

The technical decisions establish the following architecture principles:

```text
1. Business logic is hardware-independent.

2. AI cannot directly control physical devices.

3. Product data comes from authoritative services.

4. Kiosk state and device state are separate.

5. Device commands are validated and idempotent.

6. Critical transactions use database transactions.

7. Hardware starts with simulation and can migrate to ESP32.

8. Performance decisions require benchmark evidence.

9. Security boundaries exist between AI, backend and devices.

10. Architecture complexity must be justified by requirements.
```

---

# 40. Definition of Done

This document is complete when:

```text
☐ Major architecture decisions documented
☐ Alternatives considered
☐ Trade-offs documented
☐ Security implications documented
☐ Performance implications documented
☐ Hardware migration decisions documented
☐ AI architecture decisions documented
☐ Database decisions documented
☐ Testing decisions documented
☐ Benchmark decisions documented
☐ Decisions linked to implementation
☐ Decisions can be reassessed
```

---

# 41. Final Principle

The purpose of this document is not to prove:

> "I chose the best technology."

The purpose is to demonstrate:

> **I can identify an engineering problem, evaluate alternatives, understand trade-offs, make a defensible technical decision, validate that decision with evidence, and change it when the evidence says it should change.**

That is the core engineering mindset behind the RoShopSmart AI Retail Kiosk project.