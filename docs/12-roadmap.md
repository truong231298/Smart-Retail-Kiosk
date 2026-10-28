# RoShopSmart AI Retail Kiosk — Development Roadmap

**Document:** `12-roadmap.md`  
**Version:** `1.0`  
**Status:** Active  
**Project:** `roshopsmart-ai-retail-kiosk`

---

# 1. Purpose

This document defines the development roadmap for the RoShopSmart AI Retail Kiosk.

The roadmap transforms the project's architecture and requirements into an executable engineering plan.

The development process follows:

```text
Problem
   ↓
Requirements
   ↓
Architecture
   ↓
Implementation
   ↓
Testing
   ↓
Benchmark
   ↓
Optimization
   ↓
Documentation
   ↓
Demo
```

The project is developed incrementally.

Each phase must produce measurable evidence before the next phase begins.

---

# 2. Development Philosophy

The project is not developed with the goal of:

> "Build as many AI features as possible."

Instead, the objective is:

> **Build a complete Smart Machine system incrementally and prove that each layer works.**

The roadmap prioritizes:

```text
Working System
      +
Engineering Quality
      +
Measurable Performance
      +
Technical Understanding
      +
Demonstrable Evidence
```

---

# 3. Phase Overview

```text
Phase 0
Project Foundation
        ↓
Phase 1
Software Simulation
        ↓
Phase 2
AI Integration
        ↓
Phase 3
Smart Machine Simulation
        ↓
Phase 4
Productization
        ↓
Phase 5
Optimization & Portfolio Hardening
```

---

# 4. Phase 0 — Project Foundation

## Objective

Create the engineering foundation before implementing business functionality.

---

## 4.1 Repository

Create:

```text
roshopsmart-ai-retail-kiosk/
```

Repository structure:

```text
roshopsmart-ai-retail-kiosk/
│
├── README.md
├── docs/
├── ai/
├── backend/
├── embedded/
├── kiosk/
├── tests/
├── benchmarks/
├── scripts/
├── docker-compose.yml
└── .github/
    └── workflows/
```

---

## 4.2 Documentation

Complete:

```text
docs/
├── 01-project-vision.md
├── 02-system-requirements.md
├── 03-system-architecture.md
├── 04-ai-architecture.md
├── 05-computer-vision.md
├── 06-embedded-iot.md
├── 07-state-machine.md
├── 08-security.md
├── 09-testing-strategy.md
├── 10-benchmark-strategy.md
├── 11-technical-decisions.md
└── 12-roadmap.md
```

---

## 4.3 Infrastructure

Set up:

```text
Python
FastAPI
PostgreSQL
Docker
Docker Compose
Pytest
Git
GitHub Actions
```

---

## 4.4 Git Strategy

Initial commits should establish the architecture clearly.

Example:

```text
chore: initialize repository
docs: add project vision
docs: add system requirements
docs: add system architecture
docs: add AI architecture
docs: add embedded architecture
docs: add state machine design
docs: add security strategy
docs: add testing strategy
docs: add benchmark strategy
docs: add technical decisions
docs: add development roadmap
```

---

## Phase 0 Exit Criteria

```text
☐ Repository created
☐ Project structure created
☐ Docker Compose starts successfully
☐ FastAPI starts
☐ PostgreSQL starts
☐ Tests execute
☐ CI pipeline executes
☐ Documentation baseline complete
☐ README explains project
☐ Initial Git history established
```

Evidence:

```text
GitHub repository
README
CI result
Docker screenshot
Initial architecture diagrams
```

---

# 5. Phase 1 — Software Simulation

## Objective

Validate the complete retail business workflow without AI or physical hardware.

The first goal is:

> **Make the kiosk business logic work deterministically.**

---

# 5.1 Backend Modules

Implement:

```text
Customer
Product
Cart
Verification
Checkout
Payment
Transaction
Kiosk Session
```

---

# 5.2 Database

Implement initial entities:

```text
Customer
Kiosk
KioskSession
Product
Cart
CartItem
VerificationEvent
Transaction
TransactionItem
```

Add:

```text
Primary Keys
Foreign Keys
Indexes
Constraints
Transactions
```

---

# 5.3 REST API

Implement:

```text
POST   /api/sessions
GET    /api/products
GET    /api/products/{id}

POST   /api/cart/items
GET    /api/cart

POST   /api/verification
POST   /api/checkout
POST   /api/payment

GET    /api/transactions/{id}
```

---

# 5.4 Kiosk State Machine

Implement:

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

Error states:

```text
ERROR
SUSPICIOUS_TRANSACTION
```

---

# 5.5 Product Verification

Initially use simulated detection.

Example:

```json
{
  "expected": [
    {"sku": "COKE", "quantity": 1}
  ],
  "detected": [
    {"sku": "COKE", "quantity": 1}
  ]
}
```

Verification result:

```text
PASSED
```

Mismatch:

```text
SUSPICIOUS_TRANSACTION
```

---

# 5.6 Checkout

Implement:

```text
Cart validation
Price calculation
Inventory validation
Transaction creation
Payment simulation
```

---

# 5.7 Testing

Minimum:

```text
Unit tests
State machine tests
API tests
Database integration tests
E2E happy path
E2E suspicious transaction
Payment failure test
```

---

# 5.8 Phase 1 Benchmark

Measure:

```text
API latency
Database latency
State transition latency
Checkout latency
E2E transaction latency
```

---

## Phase 1 Exit Criteria

The following flow must work:

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

Additionally:

```text
☐ Product mismatch detected
☐ Suspicious transaction blocked
☐ Payment failure handled
☐ Duplicate payment prevented
☐ Database transaction integrity verified
☐ Unit tests pass
☐ Integration tests pass
☐ E2E tests pass
☐ Benchmark baseline recorded
☐ Documentation updated
☐ Git commit created
```

Evidence:

```text
API screenshots
Test report
E2E test
Database records
Benchmark report
Demo video
```

---

# 6. Phase 2 — AI Integration

## Objective

Replace simulated perception with actual AI/CV components.

The system evolves from:

```text
Simulated Detection
```

to:

```text
Real Computer Vision
```

---

# 6.1 Camera Pipeline

Implement:

```text
Camera
 ↓
OpenCV
 ↓
Frame Processing
 ↓
AI Model
 ↓
Detection
 ↓
Business Logic
```

---

# 6.2 Product Detection

Implement YOLO-based product detection.

Pipeline:

```text
Frame
 ↓
YOLO
 ↓
Bounding Boxes
 ↓
Class
 ↓
Confidence
 ↓
Product Mapping
 ↓
Cart Verification
```

---

# 6.3 Product Dataset

Create a controlled dataset.

Example:

```text
datasets/
└── products/
    ├── train/
    ├── val/
    └── test/
```

Classes may include:

```text
COKE
PEPSI
WATER
SNACK
```

The initial number of classes should remain small.

---

# 6.4 Face Recognition

Implement:

```text
Face Detection
 ↓
Face Embedding
 ↓
Similarity
 ↓
Identity Decision
```

Face recognition should remain separate from critical transaction authorization.

---

# 6.5 Gesture Recognition

Implement a small gesture vocabulary.

Example:

```text
OPEN_HAND
POINT
THUMBS_UP
CANCEL
HELP
```

Avoid implementing too many gestures initially.

---

# 6.6 Object Tracking

Add tracking where temporal consistency is required.

Pipeline:

```text
Detection
 ↓
Tracking
 ↓
Stable Object Identity
```

---

# 6.7 AI Assistant

Implement:

```text
User
 ↓
Voice/Text
 ↓
AI Assistant
 ↓
Tool Selection
 ↓
Application Service
 ↓
Response
```

Initial tools:

```text
search_product
get_product
get_cart
get_session
explain_verification
```

---

# 6.8 Voice

Implement:

```text
Speech
 ↓
Speech Recognition
 ↓
AI Assistant
 ↓
TTS
 ↓
Voice Response
```

Voice should initially remain optional.

Text interaction should always be available for testing.

---

# 6.9 AI Safety

Test:

```text
Prompt injection
Tool abuse
Unauthorized device control
Price manipulation
Database manipulation
Credential extraction
```

---

# 6.10 Phase 2 Benchmark

Measure:

```text
mAP50
mAP50-95
Precision
Recall
F1
FPS
Inference latency
Face FAR
Face FRR
Gesture accuracy
AI latency
AI groundedness
```

---

## Phase 2 Exit Criteria

```text
☐ Product detection works
☐ Product verification uses real CV
☐ Face recognition prototype works
☐ Gesture recognition prototype works
☐ Tracking works where required
☐ AI assistant works
☐ Product information is grounded in Product Service
☐ AI cannot directly control devices
☐ Voice pipeline works or has documented limitation
☐ CV tests pass
☐ AI evaluation suite passes
☐ Prompt injection tests pass
☐ Benchmark report created
☐ Documentation updated
☐ Git commit created
☐ Demo evidence created
```

---

# 7. Phase 3 — Smart Machine Simulation

## Objective

Connect the software system to a simulated physical machine.

Architecture:

```text
FastAPI
 ↓
Device Service
 ↓
SmartDevice Interface
 ↓
MQTT
 ↓
Embedded Simulator
```

---

# 7.1 MQTT Broker

Deploy:

```text
Mosquitto
```

through Docker Compose.

---

# 7.2 Device Service

Implement:

```text
Device Registry
Device State
Device Commands
Device Events
Device Health
```

---

# 7.3 SmartDevice Interface

Implement:

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

---

# 7.4 Simulator

Build web-based embedded simulator.

Visualize:

```text
LCD
LED
Servo
Buttons
Sensors
```

---

# 7.5 MQTT Topics

Implement:

```text
kiosk/{kiosk_id}/device/{device_id}/command
kiosk/{kiosk_id}/device/{device_id}/status
kiosk/{kiosk_id}/sensor/{sensor_id}/event
```

---

# 7.6 Device State Machine

Example:

```text
CLOSED
 ↓
OPENING
 ↓
OPEN
 ↓
CLOSING
 ↓
CLOSED
```

Failure:

```text
ERROR
```

---

# 7.7 Closed-Loop Device Control

Implement:

```text
Application
 ↓
Command
 ↓
MQTT
 ↓
Device
 ↓
Action
 ↓
Sensor
 ↓
Status Event
 ↓
MQTT
 ↓
Application
```

---

# 7.8 Payment → Device Workflow

Successful payment:

```text
PAYMENT_SUCCESS
      ↓
DeviceService
      ↓
LCD = Payment Successful
LED = GREEN
Servo = OPEN
      ↓
Device confirms OPEN
      ↓
Timer
      ↓
Servo = CLOSE
      ↓
Transaction = COMPLETED
```

---

# 7.9 Suspicious Transaction

```text
Product mismatch
      ↓
SUSPICIOUS_TRANSACTION
      ↓
LCD = Verification Required
LED = RED
Servo = LOCKED
```

---

# 7.10 Fault Injection

Simulator must support:

```text
MQTT disconnect
Servo failure
LCD failure
Sensor failure
Delayed response
Device restart
Invalid status
```

---

# 7.11 Phase 3 Benchmark

Measure:

```text
MQTT latency
MQTT round-trip time
Device command latency
Device response latency
State synchronization time
Recovery time
Command success rate
```

---

## Phase 3 Exit Criteria

```text
☐ MQTT broker works
☐ Device Service works
☐ SmartDevice abstraction works
☐ Simulator connects through MQTT
☐ LCD control works
☐ LED control works
☐ Servo simulation works
☐ Sensor events work
☐ Device state machine works
☐ Payment controls device
☐ Suspicious transaction locks device
☐ Fault injection works
☐ Device recovery works
☐ MQTT tests pass
☐ E2E Smart Machine test passes
☐ Benchmark report created
☐ Documentation updated
☐ Git commit created
☐ Demo evidence created
```

---

# 8. Phase 4 — Productization

## Objective

Transform the prototype into a portfolio-level engineering project.

---

# 8.1 Security

Implement:

```text
Authentication
Authorization
API validation
Rate limiting
Secret management
MQTT authentication
MQTT authorization
TLS where applicable
Audit logging
```

---

# 8.2 Observability

Implement:

```text
Structured logs
Request IDs
Session IDs
Command IDs
Device IDs
Error tracking
Performance metrics
```

---

# 8.3 Testing

Complete:

```text
Unit
Integration
E2E
Security
CV
AI
MQTT
Device
Performance
Reliability
Regression
```

---

# 8.4 CI/CD

Pipeline:

```text
Git Push
 ↓
Lint
 ↓
Type Check
 ↓
Unit Tests
 ↓
Integration Tests
 ↓
Security Tests
 ↓
Benchmark Checks
 ↓
Docker Build
```

---

# 8.5 Docker

Containerize:

```text
Backend
PostgreSQL
MQTT
Simulator
```

The complete project should start with:

```bash
docker compose up
```

where practical.

---

# 8.6 Documentation

Complete:

```text
README
Architecture
API
AI Pipeline
CV
IoT
State Machine
Security
Testing
Benchmark
Technical Decisions
Roadmap
```

---

# 8.7 API Documentation

Document:

```text
Endpoints
Request schemas
Response schemas
Authentication
Errors
Examples
```

---

# 8.8 Demo

Create a 2–3 minute technical demo.

Target flow:

```text
00:00 Customer approaches

00:15 Customer detected

00:25 AI greeting

00:40 Product selected

01:00 Product verification

01:20 Customer asks AI

01:30 AI responds

01:45 Checkout

02:00 MQTT command

02:10 Simulator:
      LCD → Payment Successful
      LED → GREEN
      Servo → OPEN

02:20 Transaction completed
```

---

## Phase 4 Exit Criteria

```text
☐ Security baseline implemented
☐ CI pipeline green
☐ Docker deployment works
☐ Unit tests pass
☐ Integration tests pass
☐ E2E tests pass
☐ Security tests pass
☐ CV benchmark complete
☐ AI benchmark complete
☐ MQTT benchmark complete
☐ E2E benchmark complete
☐ Regression strategy implemented
☐ Logging implemented
☐ Documentation complete
☐ Demo video complete
☐ Git history clean
```

---

# 9. Phase 5 — Optimization & Portfolio Hardening

## Objective

Improve the project based on measured evidence.

This phase begins only after the complete system works.

---

# 9.1 Identify Bottlenecks

Use benchmark results to identify:

```text
CV bottleneck
API bottleneck
Database bottleneck
MQTT bottleneck
AI latency
Memory usage
CPU usage
```

---

# 9.2 Optimization Experiments

Possible experiments:

```text
YOLO model size
Input resolution
FP16
INT8
Frame skipping
Tracking optimization
Database indexes
Caching
Async processing
MQTT tuning
AI prompt optimization
```

Each experiment must follow:

```text
Baseline
 ↓
Hypothesis
 ↓
Change
 ↓
Benchmark
 ↓
Result
 ↓
Decision
```

---

# 9.3 Portfolio Benchmark

Produce final report:

```text
Model Performance
Backend Performance
MQTT Performance
Device Performance
E2E Performance
Resource Usage
Reliability
```

---

# 9.4 Architecture Review

Review:

```text
Coupling
Cohesion
Security
Scalability
Testability
Maintainability
Hardware portability
```

---

# 9.5 Technical Debt

Create:

```text
docs/
└── technical-debt.md
```

Track:

```text
Issue
Impact
Priority
Workaround
Future Solution
```

---

## Phase 5 Exit Criteria

```text
☐ Major bottlenecks identified
☐ At least one optimization experiment completed
☐ Before/after benchmark recorded
☐ Architecture reviewed
☐ Technical debt documented
☐ README updated
☐ Final demo updated
☐ Final benchmark report completed
☐ Portfolio explanation prepared
```

---

# 10. Cross-Phase Engineering Gates

Every phase must pass the following gate.

```text
                    PHASE
                      │
                      ▼
               Implementation
                      │
                      ▼
                   Tests
                      │
                      ▼
                Benchmark
                      │
                      ▼
              Documentation
                      │
                      ▼
               Technical Review
                      │
                      ▼
                Git Commit
                      │
                      ▼
                Demo / Evidence
                      │
                      ▼
              ┌───────────────┐
              │   PASS ?      │
              └───────┬───────┘
                      │
             ┌────────┴────────┐
             │                 │
            YES               NO
             │                 │
             ▼                 ▼
        Next Phase          Fix Issues
```

---

# 11. Definition of Done for Every Phase

A phase is **NOT complete** merely because the code runs.

It must have:

```text
1. Working Implementation
2. Automated Tests
3. Documentation
4. Technical Explanation
5. Git Commit
6. Demo / Evidence
7. Benchmark where applicable
```

Therefore:

```text
Code works
      ≠
Phase complete
```

The correct definition is:

```text
Working Code
+
Tests
+
Documentation
+
Evidence
+
Engineering Understanding
=
Phase Complete
```

---

# 12. Git Commit Strategy

Each milestone should produce meaningful commits.

Examples:

```text
feat: implement product service
feat: implement cart service
feat: implement kiosk state machine
test: add state transition tests
test: add checkout integration tests
feat: add product verification
feat: integrate YOLO product detector
feat: add AI product assistant
feat: add MQTT device service
feat: add embedded simulator
test: add MQTT integration tests
perf: optimize product detection
docs: update benchmark results
docs: update architecture
```

Avoid meaningless commits:

```text
update
fix
test
final
final-final
```

---

# 13. Milestone Structure

## M0 — Foundation

```text
Repository
Architecture
Infrastructure
CI
```

---

## M1 — Business MVP

```text
Product
Cart
Verification
Checkout
Payment
Transaction
State Machine
```

---

## M2 — AI MVP

```text
YOLO
Face Recognition
Gesture
Tracking
AI Assistant
```

---

## M3 — Smart Machine MVP

```text
MQTT
Device Service
Simulator
LCD
LED
Servo
Sensors
```

---

## M4 — Productized MVP

```text
Security
Testing
CI/CD
Logging
Benchmark
Documentation
Demo
```

---

## M5 — Portfolio Release

```text
Optimization
Benchmark Report
Architecture Review
Technical Debt
Final Demo
GitHub Release
```

---

# 14. Suggested Repository Release Tags

Use Git tags for important milestones.

```text
v0.1.0-foundation
v0.2.0-business-mvp
v0.3.0-ai-mvp
v0.4.0-smart-machine
v0.5.0-productized
v1.0.0-portfolio
```

---

# 15. Evidence Strategy

Every milestone should produce evidence.

## Code Evidence

```text
GitHub commit
Pull Request
Tests
```

## Architecture Evidence

```text
Architecture diagram
State machine
AI pipeline
IoT diagram
```

## Performance Evidence

```text
Benchmark table
Charts
Raw benchmark results
```

## Product Evidence

```text
Screenshots
Demo
E2E workflow
```

## Engineering Evidence

```text
Technical decisions
Trade-offs
Known limitations
Optimization experiments
```

---

# 16. Final Demo Requirements

The final demo should prove the entire system rather than isolated features.

Minimum flow:

```text
Customer
 ↓
Camera
 ↓
Detection
 ↓
Product Selection
 ↓
Product Verification
 ↓
AI Assistant
 ↓
Checkout
 ↓
Payment
 ↓
MQTT
 ↓
Embedded Simulator
 ↓
LCD / LED / Servo
 ↓
Transaction Completed
```

---

# 17. Final Portfolio Package

The final repository should contain:

```text
README
Architecture Documentation
Source Code
Tests
Benchmark Reports
Demo Video
Technical Decisions
Known Limitations
Roadmap
CI
Docker
```

Recommended README sections:

```text
Project Overview
Why This Project
Features
Architecture
AI Pipeline
Smart Machine Architecture
Demo
Benchmark
Testing
Security
Technical Decisions
Limitations
Roadmap
How to Run
```

---

# 18. Interview Preparation Gate

Before considering the project portfolio-ready, the developer should be able to explain:

### Architecture

```text
Why FastAPI?
Why PostgreSQL?
Why modular monolith?
Why MQTT?
Why simulator?
```

### AI

```text
Why YOLO?
How is product verification performed?
How do you evaluate the model?
What causes false positives?
How do you optimize inference?
```

### Smart Machine

```text
How does MQTT work?
How does the simulator communicate with the backend?
How would you replace the simulator with ESP32?
What happens when MQTT fails?
How does the device recover?
```

### Security

```text
Can the LLM directly control the servo?
How do you prevent prompt injection?
How do you authenticate devices?
What happens when the backend is compromised?
```

### Engineering

```text
How do you test the state machine?
How do you benchmark the system?
What was your biggest bottleneck?
What optimization did you perform?
Why did you choose this architecture?
```

---

# 19. Known Limitations

The project must explicitly document limitations.

Possible limitations:

```text
No physical kiosk
No production payment gateway
Limited product dataset
Limited lighting conditions
Simulation rather than real ESP32
Limited gesture vocabulary
AI latency depends on external API
Face recognition may require additional liveness protection
No production-scale load validation
```

Limitations are not failures.

The important point is:

> The project clearly understands what has and has not been validated.

---

# 20. Future Hardware Roadmap

After the portfolio MVP:

```text
Web Simulator
      ↓
ESP32
      ↓
LCD
      ↓
LED
      ↓
Servo
      ↓
Sensors
      ↓
Physical Kiosk
```

Potential future components:

```text
Weight Sensor
Barcode Scanner
NFC
Payment Terminal
Door Lock
Buzzer
Thermal Printer
```

These should only be introduced when justified by the product requirements.

---

# 21. Future AI Roadmap

Potential improvements:

```text
Better product detection
Multi-camera perception
Improved tracking
Liveness detection
Better gesture recognition
Local speech recognition
Local TTS
Vision-language model
Anomaly detection
Predictive maintenance
```

These are future extensions, not Phase 1 requirements.

---

# 22. Future Scalability Roadmap

Initial:

```text
One Kiosk
```

Then:

```text
Multiple Kiosks
      ↓
Central Backend
      ↓
Device Registry
      ↓
Kiosk-specific MQTT namespaces
```

Example:

```text
kiosk/KIOSK-001/...
kiosk/KIOSK-002/...
kiosk/KIOSK-003/...
```

Only introduce distributed architecture when scaling requirements justify it.

---

# 23. Final Architecture Evolution

The expected evolution is:

```text
LEVEL 1
Software Simulation

Camera/Input
     ↓
Backend
     ↓
Database


LEVEL 2
AI Kiosk

Camera
 ↓
CV / AI
 ↓
Backend
 ↓
Database


LEVEL 3
Smart Machine

Camera
 ↓
AI
 ↓
Kiosk
 ↓
MQTT
 ↓
Simulator


LEVEL 4
Hardware Prototype

Camera
 ↓
AI
 ↓
Kiosk Computer
 ↓
MQTT
 ↓
ESP32
 ↓
Physical Devices


LEVEL 5
Scalable System

Multiple Kiosks
       ↓
Central Backend
       ↓
MQTT Infrastructure
       ↓
Multiple Smart Machines
```

---

# 24. Final Project Success Criteria

The project is considered successful when it demonstrates:

```text
AI
+
Computer Vision
+
Backend Engineering
+
State Machine
+
IoT Communication
+
Embedded Abstraction
+
Testing
+
Benchmarking
+
Security
+
Product Thinking
```

and the complete workflow can be demonstrated:

```text
SENSE
 ↓
UNDERSTAND
 ↓
DECIDE
 ↓
COMMAND
 ↓
ACT
 ↓
OBSERVE
 ↓
VERIFY
 ↓
RECOVER / IMPROVE
```

---

# 25. Final Definition of Done

The project reaches `v1.0-portfolio` when:

```text
☐ Software MVP works
☐ AI integration works
☐ Smart Machine simulator works
☐ MQTT communication works
☐ Device abstraction works
☐ State machine is implemented
☐ Product verification works
☐ AI assistant works
☐ Security baseline implemented
☐ Unit tests implemented
☐ Integration tests implemented
☐ E2E tests implemented
☐ CV benchmarks completed
☐ AI benchmarks completed
☐ MQTT benchmarks completed
☐ E2E benchmarks completed
☐ CI pipeline works
☐ Docker environment works
☐ Structured logging implemented
☐ Technical decisions documented
☐ Known limitations documented
☐ Final architecture documented
☐ Demo video completed
☐ Git history cleaned
☐ Portfolio README completed
☐ Interview explanation prepared
```

---

# 26. Final Engineering Principle

The roadmap follows one central rule:

> **Do not move forward simply because the feature works. Move forward when the engineering evidence says it is ready.**

Therefore:

```text
BUILD
  ↓
TEST
  ↓
MEASURE
  ↓
REVIEW
  ↓
DOCUMENT
  ↓
DEMONSTRATE
  ↓
COMMIT
  ↓
NEXT PHASE
```

The final objective is not merely to build an AI kiosk.

The objective is to demonstrate the complete engineering capability:

> **Design → Build → Integrate → Test → Measure → Optimize → Explain**

This is the foundation of the RoShopSmart AI Retail Kiosk as an independent **AI Product Engineering & Smart Machines portfolio project**.