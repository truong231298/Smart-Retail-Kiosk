# RoShopSmart AI Retail Kiosk — State Machine Specification

**Document:** `07-state-machine.md`  
**Version:** `1.0`  
**Status:** Draft / Initial  
**Project:** `roshopsmart-ai-retail-kiosk`

---

# 1. Purpose

This document defines the state machine governing the lifecycle of a **RoShopSmart AI Retail Kiosk session**.

The state machine is responsible for:

- Controlling kiosk workflow.
- Defining valid state transitions.
- Processing system events.
- Coordinating Computer Vision, AI, backend services and devices.
- Preventing invalid business operations.
- Handling errors.
- Handling suspicious transactions.
- Recovering from temporary failures.
- Providing a deterministic system lifecycle.

The state machine is the central coordination mechanism between:

```text
Computer Vision
       ↓
Kiosk Controller
       ↓
State Machine
       ↓
Business Services
       ↓
Device / IoT
```

---

# 2. Design Principle

The state machine must answer three questions for every system event:

```text
1. What state are we currently in?
2. Is this event valid in this state?
3. What state should we transition to?
```

Conceptually:

```text
Current State
      +
     Event
      +
   Guard/Rule
      ↓
Next State
      +
    Actions
```

---

# 3. State Machine Scope

The state machine controls the **kiosk session lifecycle**.

It does not directly implement:

```text
Computer Vision algorithms
Database queries
Payment provider implementation
MQTT implementation
ESP32 GPIO control
LLM inference
```

Instead, it invokes application services.

Example:

```text
State Machine
      ↓
VerificationService
      ↓
Computer Vision Result
```

and:

```text
State Machine
      ↓
DeviceService
      ↓
MQTT
      ↓
Simulator / ESP32
```

---

# 4. High-Level State Diagram

```text
                         ┌──────────────┐
                         │     IDLE     │
                         └──────┬───────┘
                                │
                       CUSTOMER_DETECTED
                                │
                                ▼
                   ┌──────────────────────┐
                   │ CUSTOMER_DETECTED    │
                   └──────────┬───────────┘
                              │
                       SESSION_STARTED
                              │
                              ▼
                     ┌────────────────┐
                     │    SHOPPING    │
                     └───────┬────────┘
                             │
                    CHECKOUT_REQUESTED
                             │
                             ▼
                   ┌───────────────────┐
                   │  PRODUCT_VERIFY   │
                   └───────┬───────────┘
                           │
                ┌──────────┴───────────┐
                │                      │
              MATCH                  MISMATCH
                │                      │
                ▼                      ▼
        ┌──────────────┐      ┌──────────────────────┐
        │   CHECKOUT   │      │ SUSPICIOUS_TRANSACTION│
        └──────┬───────┘      └──────────┬───────────┘
               │                         │
      PAYMENT_REQUESTED             RESOLUTION
               │                         │
               ▼                         │
        ┌──────────────┐                  │
        │   PAYMENT    │                  │
        └──────┬───────┘                  │
               │                          │
          PAYMENT_SUCCESS                 │
               │                          │
               ▼                          │
        ┌──────────────┐                  │
        │  COMPLETED   │◄─────────────────┘
        └──────┬───────┘
               │
        SESSION_CLOSED
               │
               ▼
            IDLE


Any recoverable failure:
        ───────────────► ERROR
                              │
                         RECOVER / RESET
                              │
                              ▼
                            IDLE
```

---

# 5. States

The initial state set is:

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

# 6. State: IDLE

## Purpose

The kiosk is waiting for a customer.

```text
IDLE
```

Expected behavior:

```text
Camera → monitoring
UI → welcome screen
LCD → welcome message
LED → standby
Servo → safe/closed position
Cart → empty
```

Example LCD:

```text
┌────────────────────────┐
│   WELCOME              │
│                        │
│   Start shopping       │
└────────────────────────┘
```

---

# 7. IDLE Events

Valid events:

```text
CUSTOMER_DETECTED
DEVICE_ERROR
SYSTEM_RESET
```

Main transition:

```text
IDLE
  +
CUSTOMER_DETECTED
  ↓
CUSTOMER_DETECTED
```

---

# 8. State: CUSTOMER_DETECTED

## Purpose

The system has detected a person approaching the kiosk.

At this stage the system may perform:

```text
Person Detection
Face Detection
Face Recognition
Customer Identification
Session Initialization
```

The system should not assume that every detected person is automatically a registered customer.

---

# 9. CUSTOMER_DETECTED Events

Possible events:

```text
CUSTOMER_CONFIRMED
CUSTOMER_LEFT
IDENTIFICATION_FAILED
TIMEOUT
DEVICE_ERROR
```

Successful flow:

```text
CUSTOMER_DETECTED
       ↓
CUSTOMER_CONFIRMED
       ↓
SHOPPING
```

If the person leaves:

```text
CUSTOMER_DETECTED
       ↓
CUSTOMER_LEFT
       ↓
IDLE
```

---

# 10. State: SHOPPING

## Purpose

The customer is actively interacting with the kiosk.

Possible interactions:

```text
Product Selection
Product Information
AI Assistant
Voice Input
Gesture Input
Cart Modification
Product Scanning
Product Detection
```

The AI assistant may answer questions such as:

```text
"What drinks do you have?"

"How much is this?"

"Do you have a larger bottle?"

"Which product is cheaper?"
```

---

# 11. SHOPPING Events

Examples:

```text
PRODUCT_SELECTED
PRODUCT_ADDED
PRODUCT_REMOVED
CART_UPDATED
AI_QUERY
VOICE_COMMAND
GESTURE_COMMAND
CHECKOUT_REQUESTED
CUSTOMER_LEFT
DEVICE_ERROR
```

Normal transition:

```text
SHOPPING
    ↓
CHECKOUT_REQUESTED
    ↓
PRODUCT_VERIFY
```

---

# 12. Customer Leaving During Shopping

If the customer leaves:

```text
SHOPPING
    ↓
CUSTOMER_LEFT
    ↓
SESSION_CANCELLED
    ↓
IDLE
```

Before returning to `IDLE`, the system should:

```text
Clear cart
Terminate session
Release temporary resources
Reset UI
Reset device state
Log session termination
```

---

# 13. State: PRODUCT_VERIFY

## Purpose

This state verifies that the physical products associated with the transaction match the expected cart.

The core comparison is:

```text
Expected Products
        vs
Detected Products
```

Example:

```text
Expected:
Coca Cola × 1
Water × 1

Detected:
Coca Cola × 1
Water × 1

Result:
MATCH
```

---

# 14. Product Verification Pipeline

```text
Camera
  ↓
Frame Capture
  ↓
Object Detection
  ↓
Object Tracking
  ↓
Product Classification
  ↓
Product Counting
  ↓
Expected vs Detected
  ↓
Verification Result
```

---

# 15. Verification Results

The verification service should return a structured result.

Example:

```json
{
  "status": "MATCH",
  "expected": [
    {
      "sku": "COKE-330",
      "quantity": 1
    }
  ],
  "detected": [
    {
      "sku": "COKE-330",
      "quantity": 1
    }
  ],
  "confidence": 0.96
}
```

Possible statuses:

```text
MATCH
MISMATCH
UNCERTAIN
TIMEOUT
SYSTEM_ERROR
```

---

# 16. MATCH Transition

```text
PRODUCT_VERIFY
       │
       │ MATCH
       ▼
   CHECKOUT
```

Before transition:

```text
Verification result logged
Cart frozen
Transaction preparation started
```

---

# 17. MISMATCH Transition

Example:

```text
Expected:
Coca Cola × 1

Detected:
Coca Cola × 2
```

Result:

```text
MISMATCH
```

Transition:

```text
PRODUCT_VERIFY
       │
       │ MISMATCH
       ▼
SUSPICIOUS_TRANSACTION
```

---

# 18. UNCERTAIN Verification

The system should distinguish:

```text
MISMATCH
```

from:

```text
UNCERTAIN
```

Example:

```text
Product partially occluded
Low detection confidence
Poor lighting
Multiple overlapping products
```

Instead of immediately treating this as fraud:

```text
PRODUCT_VERIFY
       ↓
UNCERTAIN
       ↓
RETRY_VERIFICATION
```

Possible policy:

```text
Retry 1
Retry 2
Manual confirmation
```

If still uncertain:

```text
UNCERTAIN
       ↓
ERROR
```

or:

```text
UNCERTAIN
       ↓
SUSPICIOUS_TRANSACTION
```

depending on business policy.

---

# 19. Verification Guard

Transition to `CHECKOUT` requires:

```text
verification.status == MATCH
```

Additional guards may include:

```text
verification.confidence >= threshold
cart is not empty
customer session is active
products are valid
```

Conceptually:

```python
if (
    verification.status == MATCH
    and verification.confidence >= MIN_CONFIDENCE
    and cart.is_valid()
):
    transition(CHECKOUT)
```

---

# 20. State: SUSPICIOUS_TRANSACTION

## Purpose

This state represents a transaction that requires additional verification or intervention.

It does not automatically mean fraud occurred.

Possible causes:

```text
Product mismatch
Unexpected product
Missing product
Duplicate product
Low-confidence detection
Impossible quantity
Tampering signal
Device anomaly
```

---

# 21. Suspicious Transaction Actions

When entering this state:

```text
LCD → "Verification Required"
LED → RED
Servo → LOCKED
Transaction → HOLD
Event → Logged
```

Example:

```text
┌────────────────────────┐
│ VERIFICATION REQUIRED  │
│                        │
│ Please wait...         │
└────────────────────────┘
```

---

# 22. Suspicious Transaction Events

Possible events:

```text
MANUAL_APPROVAL
MANUAL_REJECTION
CUSTOMER_CANCELLED
VERIFICATION_RESOLVED
TIMEOUT
DEVICE_ERROR
```

Approval:

```text
SUSPICIOUS_TRANSACTION
       ↓
MANUAL_APPROVAL
       ↓
CHECKOUT
```

Rejection:

```text
SUSPICIOUS_TRANSACTION
       ↓
MANUAL_REJECTION
       ↓
ERROR
```

---

# 23. State: CHECKOUT

## Purpose

The cart has passed product verification and the system prepares the transaction.

Actions:

```text
Freeze cart
Calculate total
Validate prices
Validate inventory
Create transaction
Prepare payment
```

Example:

```text
Cart:
Coca Cola × 1
Water × 1

Total:
25,000 VND
```

---

# 24. CHECKOUT Events

```text
CHECKOUT_VALID
PAYMENT_REQUESTED
CART_CHANGED
VALIDATION_FAILED
INVENTORY_ERROR
DEVICE_ERROR
TIMEOUT
```

Normal:

```text
CHECKOUT
    ↓
PAYMENT_REQUESTED
    ↓
PAYMENT
```

---

# 25. Checkout Guard

Before entering `PAYMENT`:

```text
Cart is valid
Products are available
Total is valid
Transaction is created
Verification is successful
```

No payment should be initiated if:

```text
Product verification failed
Cart is empty
Price validation failed
Transaction creation failed
```

---

# 26. State: PAYMENT

## Purpose

The system is waiting for payment processing.

Possible payment states:

```text
WAITING
PROCESSING
SUCCESS
FAILED
CANCELLED
TIMEOUT
```

---

# 27. PAYMENT Events

```text
PAYMENT_SUCCESS
PAYMENT_FAILED
PAYMENT_CANCELLED
PAYMENT_TIMEOUT
PAYMENT_RETRY
DEVICE_ERROR
```

Successful:

```text
PAYMENT
   ↓
PAYMENT_SUCCESS
   ↓
COMPLETED
```

Failed:

```text
PAYMENT
   ↓
PAYMENT_FAILED
   ↓
CHECKOUT
```

or:

```text
PAYMENT
   ↓
PAYMENT_FAILED
   ↓
ERROR
```

depending on failure type.

---

# 28. Payment Idempotency

Payment operations must be protected from duplicate execution.

Example:

```text
PAYMENT_SUCCESS
```

must not cause:

```text
Two transactions
Two charges
Two dispensing actions
```

A unique:

```text
payment_id
transaction_id
idempotency_key
```

should be used.

---

# 29. State: COMPLETED

## Purpose

The transaction has successfully completed.

Actions:

```text
Mark transaction completed
Display success
Turn LED green
Open product mechanism
Record analytics
Publish transaction event
```

Example:

```text
Transaction
     ↓
COMPLETED
     ↓
LCD = Payment Successful
LED = GREEN
Servo = OPEN
```

---

# 30. Completion Sequence

```text
Payment Success
      ↓
Transaction Completed
      ↓
LCD Success
      ↓
LED Green
      ↓
Servo Open
      ↓
Wait
      ↓
Servo Close
      ↓
Session Cleanup
      ↓
IDLE
```

---

# 31. State: ERROR

## Purpose

`ERROR` represents an abnormal condition that prevents normal workflow continuation.

Error categories:

```text
APPLICATION_ERROR
AI_ERROR
CV_ERROR
PAYMENT_ERROR
DEVICE_ERROR
MQTT_ERROR
DATABASE_ERROR
TIMEOUT
```

---

# 32. Error Entry

Any state may transition to `ERROR` when a non-recoverable failure occurs.

Example:

```text
SHOPPING
   ↓
DATABASE_ERROR
   ↓
ERROR
```

or:

```text
PAYMENT
   ↓
PAYMENT_SERVICE_UNAVAILABLE
   ↓
ERROR
```

---

# 33. Error Actions

When entering `ERROR`:

```text
Log error
Stop unsafe actions
Set devices to safe state
Notify user
Record diagnostic information
```

Example:

```text
LCD → "System Error"
LED → RED
Servo → CLOSED
```

---

# 34. Error Recovery

Not every error requires a permanent reset.

Possible recovery:

```text
ERROR
  ↓
RECOVERY_ATTEMPT
  ↓
SUCCESS
  ↓
Previous State
```

or:

```text
ERROR
  ↓
RESET
  ↓
IDLE
```

---

# 35. Recovery Policy

Errors should be classified.

### Recoverable

Examples:

```text
Temporary MQTT disconnect
Temporary AI timeout
Temporary CV frame failure
Temporary database connection failure
```

Possible action:

```text
Retry
Backoff
Recover
Continue
```

### Non-Recoverable

Examples:

```text
Corrupted transaction
Critical device failure
Payment inconsistency
Security violation
```

Action:

```text
ERROR
 ↓
Session termination
 ↓
Manual investigation
```

---

# 36. Global Events

Some events may occur from any state.

Examples:

```text
EMERGENCY_STOP
SYSTEM_FAILURE
DEVICE_CRITICAL_FAILURE
SESSION_TIMEOUT
```

Example:

```text
ANY STATE
   ↓
EMERGENCY_STOP
   ↓
ERROR
```

The device subsystem should immediately move actuators into a safe state.

---

# 37. Emergency Stop

Emergency stop has higher priority than normal business events.

Example:

```text
PAYMENT
   +
EMERGENCY_STOP
   ↓
ERROR
```

or:

```text
SHOPPING
   +
EMERGENCY_STOP
   ↓
ERROR
```

Physical implementations should eventually support a local hardware emergency stop that does not depend exclusively on network connectivity.

---

# 38. State Transition Table

| Current State | Event | Guard | Next State |
|---|---|---|---|
| IDLE | CUSTOMER_DETECTED | Person detected | CUSTOMER_DETECTED |
| CUSTOMER_DETECTED | CUSTOMER_CONFIRMED | Session valid | SHOPPING |
| CUSTOMER_DETECTED | CUSTOMER_LEFT | — | IDLE |
| CUSTOMER_DETECTED | TIMEOUT | — | IDLE |
| SHOPPING | CHECKOUT_REQUESTED | Cart not empty | PRODUCT_VERIFY |
| SHOPPING | CUSTOMER_LEFT | — | IDLE |
| PRODUCT_VERIFY | MATCH | Confidence valid | CHECKOUT |
| PRODUCT_VERIFY | MISMATCH | — | SUSPICIOUS_TRANSACTION |
| PRODUCT_VERIFY | UNCERTAIN | Retry available | PRODUCT_VERIFY |
| PRODUCT_VERIFY | TIMEOUT | Retry exhausted | ERROR |
| SUSPICIOUS_TRANSACTION | MANUAL_APPROVAL | Authorized | CHECKOUT |
| SUSPICIOUS_TRANSACTION | MANUAL_REJECTION | Authorized | ERROR |
| CHECKOUT | PAYMENT_REQUESTED | Checkout valid | PAYMENT |
| CHECKOUT | VALIDATION_FAILED | — | ERROR |
| PAYMENT | PAYMENT_SUCCESS | Payment valid | COMPLETED |
| PAYMENT | PAYMENT_FAILED | Retry available | PAYMENT |
| PAYMENT | PAYMENT_TIMEOUT | Retry exhausted | ERROR |
| COMPLETED | SESSION_CLOSED | — | IDLE |
| ERROR | RESET | Safe | IDLE |
| ERROR | RECOVERED | Recovery successful | Previous State |
| ANY | EMERGENCY_STOP | — | ERROR |

---

# 39. State Transition Rules

Every transition must have:

```text
Source State
Event
Guard
Action
Target State
```

Example:

```text
Source:
PRODUCT_VERIFY

Event:
VERIFICATION_COMPLETED

Guard:
status == MATCH
AND confidence >= threshold

Action:
freeze_cart()

Target:
CHECKOUT
```

---

# 40. Event Model

Events should be structured.

Example:

```json
{
  "event_id": "evt-001",
  "event_type": "CUSTOMER_DETECTED",
  "session_id": "session-001",
  "kiosk_id": "KIOSK-001",
  "timestamp": "2026-10-28T10:00:00Z",
  "payload": {}
}
```

---

# 41. Event Categories

### Customer Events

```text
CUSTOMER_DETECTED
CUSTOMER_CONFIRMED
CUSTOMER_LEFT
```

### Product Events

```text
PRODUCT_DETECTED
PRODUCT_ADDED
PRODUCT_REMOVED
PRODUCT_MISMATCH
```

### Cart Events

```text
CART_UPDATED
CHECKOUT_REQUESTED
```

### Verification Events

```text
VERIFICATION_STARTED
VERIFICATION_COMPLETED
VERIFICATION_FAILED
VERIFICATION_UNCERTAIN
```

### Payment Events

```text
PAYMENT_REQUESTED
PAYMENT_SUCCESS
PAYMENT_FAILED
PAYMENT_CANCELLED
PAYMENT_TIMEOUT
```

### Device Events

```text
DEVICE_ONLINE
DEVICE_OFFLINE
DEVICE_ERROR
SENSOR_EVENT
```

### System Events

```text
SYSTEM_ERROR
SYSTEM_RESET
TIMEOUT
EMERGENCY_STOP
```

---

# 42. State Transition Actions

Actions should be separated from transition logic.

Example:

```text
Transition:
PAYMENT → COMPLETED

Actions:
complete_transaction()
show_success()
set_led_green()
open_gate()
publish_event()
```

This avoids putting large amounts of business logic inside the state machine.

---

# 43. State Machine Responsibility

The state machine should decide:

```text
What is allowed?
What happens next?
Which action should execute?
```

It should not directly implement:

```text
How YOLO detects an object
How PostgreSQL queries execute
How MQTT packets are transmitted
How OpenAI generates a response
How ESP32 controls GPIO
```

These belong to dedicated services.

---

# 44. State Machine Architecture

```text
                    Event
                      │
                      ▼
             ┌────────────────┐
             │ Kiosk Controller│
             └───────┬────────┘
                     │
                     ▼
             ┌────────────────┐
             │ State Machine  │
             └───────┬────────┘
                     │
             ┌───────┴────────┐
             │                │
             ▼                ▼
          Guard            Transition
                              │
                              ▼
                           Actions
                              │
              ┌───────────────┼───────────────┐
              ▼               ▼               ▼
          CartService    DeviceService   TransactionService
```

---

# 45. State Machine Implementation

A basic implementation may use an enum:

```python
from enum import Enum


class KioskState(str, Enum):
    IDLE = "IDLE"
    CUSTOMER_DETECTED = "CUSTOMER_DETECTED"
    SHOPPING = "SHOPPING"
    PRODUCT_VERIFY = "PRODUCT_VERIFY"
    CHECKOUT = "CHECKOUT"
    PAYMENT = "PAYMENT"
    COMPLETED = "COMPLETED"
    ERROR = "ERROR"
    SUSPICIOUS_TRANSACTION = "SUSPICIOUS_TRANSACTION"
```

---

# 46. Transition Representation

Conceptually:

```python
Transition(
    source=KioskState.SHOPPING,
    event=Event.CHECKOUT_REQUESTED,
    target=KioskState.PRODUCT_VERIFY,
    guard=cart_is_valid,
)
```

This makes transitions explicit and testable.

---

# 47. Transition Validation

Invalid transitions should be rejected.

Example:

```text
IDLE
  +
PAYMENT_SUCCESS
```

is invalid.

The system should not silently transition.

Expected behavior:

```text
InvalidTransitionError
```

and the event should be logged.

---

# 48. State Machine Invariants

Important invariants:

### Invariant 1

Payment cannot start before verification.

```text
PRODUCT_VERIFY
    ↓
CHECKOUT
    ↓
PAYMENT
```

### Invariant 2

A transaction cannot complete without successful payment.

```text
PAYMENT_SUCCESS
    ↓
COMPLETED
```

### Invariant 3

Suspicious transactions cannot dispense products.

```text
SUSPICIOUS_TRANSACTION
    ↓
Servo = LOCKED
```

### Invariant 4

Error state cannot leave the system in an unsafe device state.

```text
ERROR
    ↓
Safe Device State
```

---

# 49. State Persistence

The current state should be persisted for important sessions.

Example:

```text
KioskSession
├── session_id
├── kiosk_id
├── customer_id
├── current_state
├── created_at
├── updated_at
└── status
```

This allows recovery after a backend restart.

---

# 50. State Recovery

Example:

```text
Before crash:
PAYMENT

Backend restarts
      ↓
Load session
      ↓
current_state = PAYMENT
      ↓
Check transaction/payment status
      ↓
Recover safely
```

The system must not blindly repeat payment or dispensing actions.

---

# 51. State Transition Logging

Every transition should be logged.

Example:

```json
{
  "session_id": "session-001",
  "from": "PRODUCT_VERIFY",
  "event": "VERIFICATION_COMPLETED",
  "to": "CHECKOUT",
  "timestamp": "2026-10-28T10:00:00Z"
}
```

This creates an auditable session timeline.

---

# 52. Session Timeline

Example:

```text
10:00:01 IDLE
10:00:05 CUSTOMER_DETECTED
10:00:07 SHOPPING
10:01:30 PRODUCT_VERIFY
10:01:35 CHECKOUT
10:01:40 PAYMENT
10:01:48 COMPLETED
10:01:55 IDLE
```

This is useful for:

```text
Debugging
Analytics
Benchmarking
Fraud analysis
Performance analysis
```

---

# 53. Timeout Handling

Every waiting state should have a timeout policy.

Examples:

```text
CUSTOMER_DETECTED → 30s
SHOPPING → configurable
PRODUCT_VERIFY → 10s
PAYMENT → 60s
SUSPICIOUS_TRANSACTION → configurable
```

Timeout values should be configuration, not hardcoded business logic.

---

# 54. Retry Policy

Retryable operations should define:

```text
Maximum retries
Backoff
Timeout
Fallback state
```

Example:

```text
CV Verification
   ↓
Attempt 1
   ↓
Failure
   ↓
Attempt 2
   ↓
Failure
   ↓
Attempt 3
   ↓
ERROR
```

---

# 55. State Machine + Computer Vision

CV produces events.

Example:

```text
Camera
 ↓
Person Detection
 ↓
CUSTOMER_DETECTED
```

Later:

```text
Object Detection
 ↓
Product Events
 ↓
VerificationService
 ↓
VERIFICATION_COMPLETED
```

The state machine consumes structured events rather than raw model outputs.

---

# 56. State Machine + AI Assistant

The AI assistant should operate within the current state.

Example:

```text
SHOPPING
   +
AI_QUERY
   ↓
AI Assistant
```

But:

```text
PAYMENT
   +
AI_QUERY
```

may be restricted or handled differently.

The state machine therefore provides contextual constraints.

---

# 57. State Machine + Embedded

Example:

```text
PAYMENT_SUCCESS
      ↓
COMPLETED
      ↓
DeviceService
      ├── LCD SUCCESS
      ├── LED GREEN
      └── SERVO OPEN
```

Example suspicious flow:

```text
MISMATCH
      ↓
SUSPICIOUS_TRANSACTION
      ↓
DeviceService
      ├── LCD VERIFY
      ├── LED RED
      └── SERVO LOCK
```

---

# 58. State Machine + MQTT

MQTT events should be translated into application events.

Example:

```text
MQTT:
DEVICE_OFFLINE

        ↓

DeviceEventHandler

        ↓

Application Event:
DEVICE_FAILURE

        ↓

State Machine
```

The state machine should not parse MQTT topics directly.

---

# 59. Concurrency

A kiosk may receive multiple events.

Example:

```text
CUSTOMER_LEFT
PAYMENT_SUCCESS
DEVICE_ERROR
```

at approximately the same time.

The system must define event ordering.

Recommended approach:

```text
Event Queue
     ↓
Single Session Processor
     ↓
State Transition
```

For each kiosk session, state transitions should be serialized.

---

# 60. Race Condition Example

Bad:

```text
Payment Success
        +
Customer Left
        ↓
Both execute simultaneously
        ↓
Servo opens incorrectly
```

Correct:

```text
Event Queue
   ↓
Process event 1
   ↓
Update state
   ↓
Process event 2
   ↓
Evaluate new state
```

---

# 61. Event Priority

Some events may have higher priority.

Suggested priority:

```text
1. EMERGENCY_STOP
2. DEVICE_CRITICAL_FAILURE
3. PAYMENT_EVENTS
4. VERIFICATION_EVENTS
5. CUSTOMER_EVENTS
6. UI / informational events
```

The exact priority model should be validated during implementation.

---

# 62. State Machine Testing

Testing should cover:

```text
Valid transitions
Invalid transitions
Guards
Timeouts
Retries
Error recovery
Emergency stop
Concurrent events
Persistence
Recovery after restart
```

---

# 63. Unit Test Example

Conceptually:

```python
def test_checkout_requires_verification():
    machine = create_machine()

    machine.start_session()

    assert machine.state == KioskState.SHOPPING

    machine.handle(Event.PAYMENT_REQUESTED)

    assert machine.state != KioskState.PAYMENT
```

The exact implementation may differ.

---

# 64. Product Verification Test

```text
Initial:
PRODUCT_VERIFY

Event:
VERIFICATION_COMPLETED

Payload:
status = MATCH

Expected:
CHECKOUT
```

---

# 65. Suspicious Transaction Test

```text
Initial:
PRODUCT_VERIFY

Event:
VERIFICATION_COMPLETED

Payload:
status = MISMATCH

Expected:
SUSPICIOUS_TRANSACTION

Device:
Servo = LOCKED
LED = RED
```

---

# 66. Payment Test

```text
Initial:
PAYMENT

Event:
PAYMENT_SUCCESS

Expected:
COMPLETED

Actions:
Transaction completed
LED green
LCD success
Servo open
```

---

# 67. Emergency Test

```text
Initial:
PAYMENT

Event:
EMERGENCY_STOP

Expected:
ERROR

Device:
Servo closed
LED red
LCD error
```

---

# 68. Invalid Transition Test

```text
Initial:
IDLE

Event:
PAYMENT_SUCCESS

Expected:
InvalidTransitionError
```

The system must not silently accept invalid workflow transitions.

---

# 69. State Machine Metrics

Useful metrics:

```text
Session duration
Time spent per state
Verification duration
Checkout duration
Payment duration
Error rate
Suspicious transaction rate
Recovery rate
Timeout rate
```

Example:

```text
Average Session:
82 seconds

Average Verification:
1.4 seconds

Average Checkout:
0.8 seconds

Average Payment:
4.2 seconds
```

These metrics can later support performance optimization.

---

# 70. State Machine Observability

Each transition should emit an event:

```text
STATE_CHANGED
```

Example:

```json
{
  "event_type": "STATE_CHANGED",
  "from": "PAYMENT",
  "to": "COMPLETED",
  "session_id": "session-001"
}
```

This can be consumed by:

```text
Logging
Analytics
Monitoring
Dashboard
Debugging
```

---

# 71. Security Considerations

Sensitive transitions should require authorization.

Examples:

```text
MANUAL_APPROVAL
MANUAL_REJECTION
EMERGENCY_RESET
DEVICE_OVERRIDE
```

A normal customer should not be able to trigger:

```text
MANUAL_APPROVAL
```

through a public UI action.

---

# 72. Device Safety Invariants

The state machine should enforce:

```text
IDLE:
Servo closed

SHOPPING:
Servo locked

PRODUCT_VERIFY:
Servo locked

SUSPICIOUS_TRANSACTION:
Servo locked

PAYMENT:
Servo locked

COMPLETED:
Servo may open

ERROR:
Servo safe/closed
```

This is an important bridge between software state and physical state.

---

# 73. State Machine and Transaction Boundary

A transaction should be considered completed only after:

```text
Payment confirmed
AND
Transaction persisted
AND
Required device action initiated
```

The exact business definition should be finalized during implementation.

This avoids inconsistent states such as:

```text
Payment successful
but transaction not recorded
```

or:

```text
Transaction marked completed
but payment failed
```

---

# 74. State Machine Recovery Example

Suppose:

```text
PAYMENT
```

and the backend crashes.

After restart:

```text
Load session
      ↓
Load transaction
      ↓
Check payment provider
      ↓
Determine actual payment status
```

Then:

```text
Payment success
      ↓
COMPLETED
```

or:

```text
Payment failed
      ↓
CHECKOUT / ERROR
```

The system must use the authoritative payment status rather than assuming the previous state was correct.

---

# 75. Complete Lifecycle

The ideal lifecycle is:

```text
IDLE
 │
 │ Customer detected
 ▼
CUSTOMER_DETECTED
 │
 │ Customer confirmed
 ▼
SHOPPING
 │
 │ Checkout requested
 ▼
PRODUCT_VERIFY
 │
 ├── MATCH ────────────────┐
 │                         ▼
 │                      CHECKOUT
 │                         │
 │                         ▼
 │                      PAYMENT
 │                         │
 │                    PAYMENT_SUCCESS
 │                         │
 │                         ▼
 │                     COMPLETED
 │                         │
 │                    Session closed
 │                         │
 └─────────────────────────┴──────► IDLE


PRODUCT_VERIFY
 │
 ├── MISMATCH
 ▼
SUSPICIOUS_TRANSACTION
 │
 ├── APPROVED ───────────► CHECKOUT
 │
 └── REJECTED ──────────► ERROR


ANY STATE
 │
 ├── EMERGENCY_STOP ────► ERROR
 │
 ├── CRITICAL_FAILURE ──► ERROR
 │
 └── SESSION_TIMEOUT ────► IDLE / ERROR
```

---

# 76. Architecture Principle

The state machine provides the deterministic control layer:

```text
AI
 ↓
Provides intelligence

Computer Vision
 ↓
Provides perception

Backend Services
 ↓
Provides business logic

State Machine
 ↓
Provides workflow control

Device Service
 ↓
Provides machine control

MQTT
 ↓
Provides communication

ESP32 / Simulator
 ↓
Provides physical execution
```

Together:

```text
PERCEIVE
   ↓
UNDERSTAND
   ↓
DECIDE
   ↓
ACT
   ↓
OBSERVE
   ↓
VERIFY
```

---

# 77. Definition of Done

The State Machine phase is complete when:

```text
☐ All states defined
☐ All valid transitions defined
☐ Invalid transitions rejected
☐ Guards documented
☐ Events documented
☐ Timeout policies defined
☐ Retry policies defined
☐ Error states defined
☐ Suspicious transaction flow defined
☐ Emergency stop defined
☐ Device safety rules defined
☐ State persistence defined
☐ Recovery strategy defined
☐ State transition logging implemented
☐ Unit tests implemented
☐ Integration tests implemented
☐ CV integration verified
☐ Device integration verified
☐ Payment flow verified
☐ Documentation updated
☐ Git commit created
☐ Demo evidence captured
```

---

# 78. Final Principle

The RoShopSmart state machine must make the kiosk's behavior:

```text
Deterministic
Observable
Testable
Recoverable
Safe
Extensible
```

The central engineering model is:

```text
                    ┌───────────────┐
                    │   PERCEIVE    │
                    │ Camera / Voice│
                    └───────┬───────┘
                            ↓
                    ┌───────────────┐
                    │   UNDERSTAND  │
                    │ AI / CV       │
                    └───────┬───────┘
                            ↓
                    ┌───────────────┐
                    │    DECIDE     │
                    │ State Machine │
                    └───────┬───────┘
                            ↓
                    ┌───────────────┐
                    │     ACT       │
                    │ Device / IoT  │
                    └───────┬───────┘
                            ↓
                    ┌───────────────┐
                    │    OBSERVE    │
                    │ Sensors / CV  │
                    └───────┬───────┘
                            ↓
                    ┌───────────────┐
                    │    VERIFY     │
                    │ Business Rule │
                    └───────┬───────┘
                            │
                            └──────► Next State
```

The state machine is therefore not merely a UI navigation mechanism. It is the **workflow and safety control layer of the Smart Machine**, connecting perception, business decisions and physical actions into one deterministic system.