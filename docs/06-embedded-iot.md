# RoShopSmart AI Retail Kiosk — Embedded & IoT Architecture

**Document:** `06-embedded-iot.md`  
**Version:** `1.0`  
**Status:** Draft / Initial  
**Project:** `roshopsmart-ai-retail-kiosk`

---

# 1. Purpose

This document defines the Embedded and IoT architecture of the **RoShopSmart AI Retail Kiosk**.

The system initially uses a **web-based embedded simulator** instead of physical hardware.

The simulator represents:

- ESP32
- LCD
- LED
- Servo
- Buttons
- Sensors
- Device state
- MQTT communication

The architecture is deliberately designed around a Hardware Abstraction Layer (HAL) so that the future transition is:

```text
Current:

FastAPI
   ↓
MQTT
   ↓
Embedded Simulator


Future:

FastAPI
   ↓
MQTT
   ↓
ESP32
   ↓
Real Hardware
```

without changing the kiosk's business logic.

---

# 2. Embedded Engineering Goals

The Embedded/IoT subsystem must demonstrate:

1. Hardware abstraction.
2. Device control.
3. MQTT communication.
4. Device state management.
5. Sensor event handling.
6. Actuator control.
7. Fault handling.
8. Real-time event propagation.
9. Simulator-to-hardware portability.
10. Testability.

The goal is not simply to simulate LEDs and servos.

The goal is to demonstrate how a software system can control a **Smart Machine** through a clean device architecture.

---

# 3. Core Design Principle

The most important architectural principle is:

> **Business logic must not know whether the device is a simulator or real hardware.**

Incorrect:

```text
CheckoutService
      ↓
ESP32-specific code
      ↓
GPIO
```

Correct:

```text
CheckoutService
      ↓
DeviceService
      ↓
SmartDevice Interface
      ↓
MQTT / Device Adapter
      ↓
Simulator OR ESP32
```

Therefore:

```text
Simulator → Real ESP32
```

becomes an infrastructure change rather than a business-logic rewrite.

---

# 4. High-Level Architecture

```text
                    KIOSK APPLICATION
                           │
                           ▼
                    Device Service
                           │
                           ▼
                  SmartDevice Interface
                           │
                           ▼
                    MQTT Adapter
                           │
                    MQTT Broker
                           │
              ┌────────────┴────────────┐
              │                         │
              ▼                         ▼
     Embedded Simulator             ESP32 Device
              │                         │
       ┌──────┼──────┐           ┌──────┼──────┐
       ▼      ▼      ▼           ▼      ▼      ▼
      LCD    LED   Servo        LCD    LED   Servo
                              Sensors / Buttons
```

---

# 5. Device Layer

The device layer represents physical or simulated hardware.

Main components:

```text
Device Layer
│
├── Device Service
├── SmartDevice Interface
├── Device Registry
├── Device State
├── Device Events
└── Device Adapters
```

---

# 6. SmartDevice Interface

The core abstraction is:

```python
class SmartDevice:

    def display_message(self, message: str):
        ...

    def set_led(self, state: bool):
        ...

    def move_servo(self, angle: int):
        ...

    def read_sensor(self, sensor_id: str):
        ...

    def get_status(self):
        ...
```

The application depends on this interface.

It does not depend on:

```text
ESP32 GPIO
Arduino API
Browser DOM
MQTT implementation
```

---

# 7. Device Implementations

The interface can have multiple implementations.

```text
SmartDevice
│
├── SimulatorDevice
│
├── ESP32Device
│
└── MockDevice
```

### SimulatorDevice

Used during development.

### ESP32Device

Used when physical hardware is available.

### MockDevice

Used for automated testing.

---

# 8. Hardware Abstraction Layer

The Hardware Abstraction Layer separates:

```text
Application Logic
```

from:

```text
Hardware Implementation
```

Architecture:

```text
┌───────────────────────────┐
│      Application          │
└─────────────┬─────────────┘
              │
              ▼
┌───────────────────────────┐
│ SmartDevice Interface     │
└─────────────┬─────────────┘
              │
       ┌──────┴──────┐
       ▼             ▼
SimulatorDevice   ESP32Device
       │             │
       ▼             ▼
   Web Runtime     GPIO / MCU
```

---

# 9. Device Registry

The kiosk may contain multiple devices.

Example:

```text
KIOSK-001
│
├── display-01
├── led-01
├── servo-01
├── button-01
└── sensor-01
```

A Device Registry maintains metadata.

Example:

```json
{
  "device_id": "servo-01",
  "device_type": "servo",
  "status": "ONLINE",
  "kiosk_id": "KIOSK-001"
}
```

---

# 10. Device Types

Initial device types:

```text
LCD
LED
SERVO
BUTTON
SENSOR
```

Future devices:

```text
BUZZER
RELAY
MOTOR
WEIGHT_SENSOR
BARCODE_SCANNER
DOOR_LOCK
NFC_READER
```

The architecture should allow new devices without changing business logic.

---

# 11. LCD Architecture

The LCD represents the kiosk's physical display.

Example states:

```text
IDLE
Welcome!

SHOPPING
Select a product

PAYMENT
Processing...

COMPLETED
Payment Successful

ERROR
Please contact staff
```

Interface:

```python
class DisplayDevice:

    def display(self, text: str):
        ...

    def clear(self):
        ...
```

---

# 12. LCD Simulator

The web simulator should render an LCD-like component.

Example:

```text
┌──────────────────────────────┐
│                              │
│     PAYMENT SUCCESSFUL       │
│                              │
│        Thank You!            │
│                              │
└──────────────────────────────┘
```

The simulator should visually reflect MQTT commands sent by the backend.

---

# 13. LED Architecture

The LED provides machine status feedback.

Possible states:

```text
OFF
GREEN
RED
YELLOW
BLUE
```

Semantic meanings:

```text
GREEN  → Success
RED    → Error / Suspicious
YELLOW → Processing
BLUE   → Active / Waiting
```

The application should send semantic commands where possible.

Example:

```text
SET_STATUS_LED(SUCCESS)
```

rather than scattering:

```text
set_led("green")
```

throughout business logic.

---

# 14. Servo Architecture

The servo simulates a physical mechanism such as:

```text
Product door
Dispensing mechanism
Lock
Gate
```

Initial behavior:

```text
LOCKED
   ↓
Payment successful
   ↓
OPEN
   ↓
Wait
   ↓
CLOSE
```

Example:

```text
0°   → Closed
90°  → Open
```

Exact angles are configurable.

---

# 15. Servo Interface

```python
class ServoDevice:

    def move(self, angle: int):
        ...

    def open(self):
        ...

    def close(self):
        ...
```

Business logic should prefer:

```text
servo.open()
```

rather than:

```text
servo.move(90)
```

when the semantic operation is "open."

---

# 16. Button Architecture

Buttons represent physical user input.

Examples:

```text
CONFIRM
CANCEL
HELP
EMERGENCY_STOP
```

Simulator:

```text
Web Button
   ↓
MQTT Event
   ↓
Backend
```

Physical implementation:

```text
Physical Button
   ↓
ESP32 GPIO
   ↓
MQTT Event
   ↓
Backend
```

The application receives the same logical event.

---

# 17. Sensor Architecture

Sensors provide environmental or machine state information.

Initial simulator sensors:

```text
Door Sensor
Presence Sensor
Payment Sensor
Temperature Sensor
```

The actual sensor set may evolve.

Example:

```json
{
  "sensor_id": "door-01",
  "value": "OPEN",
  "timestamp": "2026-10-28T10:00:00Z"
}
```

---

# 18. Sensor Abstraction

```python
class Sensor:

    def read(self):
        ...

    def subscribe(self, callback):
        ...
```

Possible implementations:

```text
Sensor
│
├── SimulatorSensor
└── ESP32Sensor
```

---

# 19. MQTT Architecture

MQTT is the communication protocol between backend and devices.

Architecture:

```text
Backend
   │
   ▼
MQTT Client
   │
   ▼
MQTT Broker
   │
   ├───────────────┐
   ▼               ▼
Simulator        ESP32
```

MQTT is chosen because it is lightweight and suitable for IoT-style messaging.

---

# 20. MQTT Broker

The initial development environment should run an MQTT broker through Docker.

Example:

```text
Docker Compose
│
├── backend
├── postgres
├── mqtt-broker
└── embedded-simulator
```

A broker such as Mosquitto may be used.

The broker is infrastructure and should remain replaceable.

---

# 21. MQTT Topic Architecture

Topics should be hierarchical.

Base:

```text
kiosk/{kiosk_id}/
```

Device commands:

```text
kiosk/{kiosk_id}/device/{device_id}/command
```

Device status:

```text
kiosk/{kiosk_id}/device/{device_id}/status
```

Sensor events:

```text
kiosk/{kiosk_id}/sensor/{sensor_id}/event
```

Kiosk events:

```text
kiosk/{kiosk_id}/event
```

---

# 22. MQTT Command Example

Backend:

```text
Topic:
kiosk/KIOSK-001/device/servo-01/command
```

Payload:

```json
{
  "command": "OPEN"
}
```

Simulator:

```text
MQTT
 ↓
Receive OPEN
 ↓
Servo state = OPEN
 ↓
Update UI
 ↓
Publish status
```

---

# 23. MQTT Status Example

Simulator publishes:

```text
Topic:
kiosk/KIOSK-001/device/servo-01/status
```

Payload:

```json
{
  "device_id": "servo-01",
  "state": "OPEN",
  "timestamp": "2026-10-28T10:00:00Z"
}
```

The backend can update device state.

---

# 24. MQTT Sensor Event

Example:

```text
Topic:
kiosk/KIOSK-001/sensor/door-01/event
```

Payload:

```json
{
  "event": "DOOR_OPENED",
  "timestamp": "2026-10-28T10:00:00Z"
}
```

Backend:

```text
MQTT Event
   ↓
Device Event Handler
   ↓
Application Event
```

---

# 25. MQTT QoS

Different messages may use different QoS levels.

Conceptually:

```text
Telemetry:
QoS 0

Important state:
QoS 1

Critical messages:
Evaluate QoS 1/2 based on requirements
```

The project should avoid using the highest QoS everywhere without justification.

---

# 26. MQTT Retained Messages

Device status may optionally use retained messages.

Example:

```text
device/servo-01/status
```

A newly connected client can immediately learn:

```text
Servo = OPEN
```

instead of waiting for the next status update.

This behavior should be documented and tested.

---

# 27. Device Heartbeat

Each device should periodically publish health information.

Example:

```json
{
  "device_id": "servo-01",
  "status": "ONLINE",
  "uptime": 1200,
  "timestamp": "2026-10-28T10:00:00Z"
}
```

The backend can detect:

```text
ONLINE
OFFLINE
DEGRADED
```

---

# 28. Device State Machine

Each smart device may have its own state.

Example for a servo:

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
ANY STATE
   ↓
ERROR
```

The kiosk-level state machine and device-level state machines are separate.

---

# 29. Kiosk State vs Device State

Important distinction:

```text
Kiosk State:
PAYMENT

Device State:
Servo = CLOSED
LCD = "Processing..."
LED = YELLOW
```

The kiosk state represents business workflow.

Device states represent physical machine state.

They should not be mixed.

---

# 30. Device Command Lifecycle

Example:

```text
Payment Successful
       ↓
DeviceService
       ↓
Servo.open()
       ↓
MQTT Command
       ↓
Simulator / ESP32
       ↓
Servo Opens
       ↓
Device Status
       ↓
Backend
```

This provides a closed control loop.

---

# 31. Closed-Loop Smart Machine Architecture

The system should follow:

```text
                 ┌──────────────┐
                 │ Application  │
                 └──────┬───────┘
                        │
                     COMMAND
                        │
                        ▼
                 ┌──────────────┐
                 │ MQTT Broker  │
                 └──────┬───────┘
                        │
                        ▼
                 ┌──────────────┐
                 │ Device       │
                 └──────┬───────┘
                        │
                      ACTION
                        │
                        ▼
                    SENSOR
                        │
                       EVENT
                        │
                        ▼
                 ┌──────────────┐
                 │ MQTT Broker  │
                 └──────┬───────┘
                        │
                        ▼
                 ┌──────────────┐
                 │ Application  │
                 └──────────────┘
```

This is the core Smart Machine control loop.

---

# 32. Embedded Simulator

The simulator is a web application that visually represents the machine.

Example:

```text
┌────────────────────────────────────┐
│        SMART RETAIL KIOSK          │
├────────────────────────────────────┤
│                                    │
│   ┌────────────────────────────┐   │
│   │       LCD DISPLAY          │   │
│   │                            │   │
│   │    PAYMENT SUCCESSFUL      │   │
│   └────────────────────────────┘   │
│                                    │
│       ● LED                        │
│                                    │
│       ┌───────────┐                │
│       │   SERVO   │                │
│       │    OPEN   │                │
│       └───────────┘                │
│                                    │
│   [CONFIRM] [CANCEL] [HELP]        │
│                                    │
│   Door: OPEN                       │
│   Sensor: ACTIVE                   │
│   MQTT: CONNECTED                  │
└────────────────────────────────────┘
```

---

# 33. Simulator Responsibilities

The simulator should:

- Connect to MQTT.
- Subscribe to device commands.
- Update virtual hardware state.
- Render device state.
- Publish device status.
- Generate simulated sensor events.
- Simulate failures.
- Display connection status.

It should not contain business logic.

---

# 34. Simulator Architecture

```text
Web Simulator
│
├── MQTT Client
├── Device Manager
│
├── LCD
├── LED
├── Servo
├── Buttons
├── Sensors
│
└── State Renderer
```

---

# 35. Simulator Device Manager

The Device Manager coordinates virtual devices.

Example:

```text
DeviceManager
│
├── LCDDevice
├── LEDDevice
├── ServoDevice
├── ButtonDevice
└── SensorDevice
```

Each component maintains its own state.

---

# 36. Simulator State

Example:

```json
{
  "kiosk_id": "KIOSK-001",
  "devices": {
    "lcd-01": {
      "state": "DISPLAYING",
      "message": "Payment Successful"
    },
    "led-01": {
      "state": "GREEN"
    },
    "servo-01": {
      "state": "OPEN"
    }
  }
}
```

This state is rendered by the simulator UI.

---

# 37. Simulator Controls

The simulator should allow manual testing.

Example controls:

```text
[Open Door]
[Close Door]
[Press Confirm]
[Press Cancel]
[Trigger Sensor]
[Disconnect MQTT]
[Simulate Device Failure]
```

This makes the simulator useful as an engineering tool rather than merely a visual mockup.

---

# 38. Simulated Faults

The simulator should support fault injection.

Examples:

```text
MQTT disconnected
Servo failure
LCD failure
Sensor failure
Delayed response
Device offline
```

Example:

```text
Servo command
      ↓
Simulated delay: 2 seconds
      ↓
Servo opens
```

This allows resilience testing.

---

# 39. Device Service

The backend should expose a Device Service.

Responsibilities:

```text
Send device commands
Track device state
Handle device events
Check device health
Manage device registry
```

Example:

```python
class DeviceService:

    def open_door(self):
        ...

    def display_message(self, message):
        ...

    def set_status(self, status):
        ...
```

---

# 40. Semantic Device Commands

Prefer semantic operations.

Example:

```text
show_payment_success()
open_product_gate()
show_error()
set_processing_indicator()
```

instead of exposing MQTT details throughout the application.

Architecture:

```text
CheckoutService
      ↓
DeviceService
      ↓
Semantic Command
      ↓
MQTT
```

---

# 41. Payment Success Workflow

Example:

```text
Payment Successful
       ↓
Transaction Completed
       ↓
DeviceService
       │
       ├── LCD → "Payment Successful"
       ├── LED → GREEN
       └── Servo → OPEN
```

Then:

```text
Timer
 ↓
Servo → CLOSE
```

---

# 42. Suspicious Transaction Workflow

Example:

```text
Mismatch Detected
       ↓
SUSPICIOUS_TRANSACTION
       ↓
DeviceService
       │
       ├── LCD → "Verification Required"
       ├── LED → RED
       └── Servo → LOCKED
```

This connects CV, application logic and embedded control.

---

# 43. Error Workflow

Example:

```text
System Error
       ↓
DeviceService
       │
       ├── LCD → "System Error"
       ├── LED → RED
       └── Servo → SAFE POSITION
```

The physical system should move to a safe state.

---

# 44. Device Safety

Device actions should be validated.

Example:

```text
Servo.open()
```

should not execute when:

```text
Payment not completed
```

unless explicitly permitted by the state machine.

The Device Service should not bypass business authorization.

---

# 45. Emergency Stop

A future hardware design should support:

```text
EMERGENCY_STOP
```

Flow:

```text
Emergency Button
      ↓
ESP32
      ↓
MQTT
      ↓
Backend
      ↓
Kiosk ERROR
      ↓
All actuators → SAFE STATE
```

The physical emergency stop may eventually need to operate independently of network connectivity for safety-critical hardware.

The portfolio prototype can simulate this behavior.

---

# 46. Device Authentication

Future ESP32 devices should authenticate with the MQTT broker.

Possible mechanisms:

```text
Username / Password
TLS
Client Certificates
Device Credentials
```

The initial simulator may use development credentials.

Production credentials must never be committed to Git.

---

# 47. MQTT Security

Production architecture should use:

```text
TLS
Authentication
Authorization
Restricted Topics
```

Example:

```text
ESP32-001
can publish:
kiosk/KIOSK-001/sensor/#

can subscribe:
kiosk/KIOSK-001/device/+/command
```

Topic permissions should be scoped by device.

---

# 48. Device Configuration

Device configuration should be externalized.

Example:

```yaml
device:
  kiosk_id: KIOSK-001

mqtt:
  host: mqtt
  port: 1883

devices:
  servo:
    id: servo-01

  lcd:
    id: lcd-01

  led:
    id: led-01
```

Production credentials should come from environment variables or a secret-management mechanism.

---

# 49. Docker Architecture

Development environment:

```text
┌─────────────────────────────────────────┐
│              Docker Compose             │
│                                         │
│  ┌───────────┐                          │
│  │ FastAPI   │                          │
│  └─────┬─────┘                          │
│        │                                 │
│        ▼                                 │
│  ┌───────────┐      ┌───────────────┐   │
│  │ MQTT      │◄────►│ Simulator     │   │
│  │ Broker    │      │               │   │
│  └───────────┘      └───────────────┘   │
│                                         │
│  ┌───────────┐                          │
│  │ PostgreSQL│                          │
│  └───────────┘                          │
└─────────────────────────────────────────┘
```

---

# 50. Real ESP32 Evolution

When physical hardware is introduced:

```text
Current:

FastAPI
 ↓
MQTT
 ↓
Simulator


Future:

FastAPI
 ↓
MQTT
 ↓
ESP32
 ↓
GPIO
 ↓
LCD / LED / Servo / Sensors
```

The following should remain unchanged:

```text
CheckoutService
PaymentService
TransactionService
StateMachine
VerificationService
DeviceService Interface
```

Only the device implementation/infrastructure changes.

---

# 51. ESP32 Responsibilities

The ESP32 should have a limited responsibility.

It should:

```text
Receive commands
Control GPIO
Read sensors
Publish device state
Handle local hardware timing
Handle connectivity
```

It should not contain:

```text
Payment business logic
Product pricing
Cart logic
AI logic
Checkout rules
Transaction database logic
```

These remain in the backend.

---

# 52. Edge vs Backend Responsibilities

| Responsibility | Backend | ESP32 |
|---|---:|---:|
| Product Catalog | ✓ | |
| Cart | ✓ | |
| AI | ✓ | |
| Verification | ✓ | |
| Payment Workflow | ✓ | |
| Transaction | ✓ | |
| Device Commands | ✓ | |
| GPIO | | ✓ |
| Sensor Reading | | ✓ |
| Servo Control | | ✓ |
| LCD Hardware | | ✓ |
| Device Health | ✓ | ✓ |

---

# 53. Local Device Autonomy

The ESP32 may perform simple local safety logic.

Example:

```text
Servo moving
 ↓
Limit sensor triggered
 ↓
Stop servo
```

This is preferable to requiring a cloud round trip for immediate physical safety.

However, the business decision remains on the backend.

---

# 54. Offline Behavior

A real smart machine must consider network loss.

Example:

```text
MQTT connection lost
      ↓
ESP32 detects disconnect
      ↓
Enter SAFE MODE
```

Possible safe behavior:

```text
Servo → CLOSED
LED → RED/YELLOW
LCD → "Connection Lost"
```

The exact policy depends on hardware requirements.

---

# 55. Device Reconnection

When connectivity returns:

```text
ESP32
 ↓
Reconnect MQTT
 ↓
Publish ONLINE
 ↓
Publish Current State
 ↓
Backend Updates Device Registry
```

The system should not assume that device state remained unchanged during the outage.

---

# 56. Device State Synchronization

The backend should distinguish:

```text
Desired State
vs
Reported State
```

Example:

```text
Desired:
Servo = OPEN

Reported:
Servo = CLOSED
```

This indicates a possible device failure.

Architecture:

```text
Backend Desired State
        ↓
MQTT Command
        ↓
Device
        ↓
Reported State
        ↓
Backend
```

---

# 57. Command IDs

Commands should have unique identifiers.

Example:

```json
{
  "command_id": "CMD-001",
  "command": "OPEN",
  "timestamp": "2026-10-28T10:00:00Z"
}
```

The device can report:

```json
{
  "command_id": "CMD-001",
  "status": "EXECUTED"
}
```

This helps with observability and debugging.

---

# 58. Idempotency

Some commands should be safe to repeat.

For example:

```text
SET_LED(GREEN)
```

can safely be repeated.

More dangerous operations such as:

```text
DISPENSE_PRODUCT
```

require stronger safeguards.

The device protocol should distinguish:

```text
State-setting commands
```

from:

```text
One-time action commands
```

---

# 59. Device Event Model

Base device event:

```json
{
  "event_id": "uuid",
  "event_type": "DEVICE_STATUS_CHANGED",
  "kiosk_id": "KIOSK-001",
  "device_id": "servo-01",
  "timestamp": "2026-10-28T10:00:00Z",
  "payload": {
    "state": "OPEN"
  }
}
```

Possible event types:

```text
DEVICE_ONLINE
DEVICE_OFFLINE
DEVICE_ERROR
DEVICE_STATUS_CHANGED
SENSOR_CHANGED
BUTTON_PRESSED
COMMAND_EXECUTED
COMMAND_FAILED
```

---

# 60. Device Monitoring

The backend should monitor:

```text
Connection
Heartbeat
Command latency
Command failures
Device errors
State mismatch
```

Example:

```text
Device:
servo-01

Status:
ONLINE

Command latency:
120 ms

Last heartbeat:
2 seconds ago
```

---

# 61. Device Metrics

Important metrics:

```text
MQTT latency
Command success rate
Command failure rate
Device uptime
Reconnect count
Sensor event latency
Simulator response time
```

For real hardware:

```text
Power consumption
Temperature
CPU usage
Memory
```

may be added later.

---

# 62. IoT Testing

Testing layers:

```text
Unit Tests
   ↓
MQTT Integration Tests
   ↓
Simulator Tests
   ↓
Backend + Simulator E2E
   ↓
Real ESP32 Hardware Tests
```

The simulator allows most integration tests to run without physical hardware.

---

# 63. Mock Device

For backend unit tests:

```python
class MockDevice(SmartDevice):
    ...
```

Example:

```text
CheckoutService
      ↓
MockDevice
      ↓
Assert:
LCD called
LED called
Servo opened
```

This makes tests deterministic.

---

# 64. MQTT Integration Tests

Test:

```text
Backend publishes command
        ↓
MQTT broker
        ↓
Simulator receives
        ↓
Simulator changes state
        ↓
Simulator publishes status
        ↓
Backend receives
```

This validates the complete communication loop.

---

# 65. Simulator E2E Test

Example:

```text
Transaction Completed
        ↓
Backend
        ↓
MQTT
        ↓
Simulator
```

Expected:

```text
LCD = Payment Successful
LED = GREEN
Servo = OPEN
```

Then:

```text
Timeout
 ↓
Servo = CLOSED
```

---

# 66. Fault Injection Tests

The simulator should support tests such as:

```text
MQTT disconnect
Servo failure
LCD failure
Sensor failure
Delayed response
Device offline
```

Expected behavior should be documented.

Example:

```text
Servo failure
    ↓
DEVICE_ERROR
    ↓
Kiosk ERROR
    ↓
LCD error
    ↓
Transaction marked for review
```

---

# 67. Embedded Simulator Technology

The simulator can be implemented as a web application.

Possible stack:

```text
Frontend:
HTML / CSS / JavaScript
```

or:

```text
React
```

Backend communication:

```text
MQTT over WebSocket
```

The simulator does not need to reproduce actual ESP32 hardware internals.

It needs to reproduce:

```text
Commands
State
Events
Timing
Failures
```

---

# 68. Simulator Architecture Recommendation

For MVP:

```text
embedded/
└── simulator/
    ├── frontend/
    ├── mqtt/
    ├── devices/
    ├── state/
    └── tests/
```

Possible components:

```text
Simulator UI
     ↓
Simulator Device Manager
     ↓
MQTT Client
```

---

# 69. Hardware Abstraction Repository Structure

Recommended:

```text
embedded/
│
├── simulator/
│   ├── frontend/
│   ├── devices/
│   └── mqtt/
│
├── esp32/
│   ├── src/
│   ├── devices/
│   ├── mqtt/
│   └── config/
│
├── sensors/
│   ├── base.py
│   └── simulator.py
│
├── servo/
│   ├── base.py
│   └── simulator.py
│
└── lcd/
    ├── base.py
    └── simulator.py
```

The exact structure may change during implementation.

---

# 70. Device Protocol

A common device protocol should be defined before implementing the ESP32.

Example command:

```json
{
  "command_id": "CMD-001",
  "device_id": "servo-01",
  "command": "OPEN",
  "timestamp": "2026-10-28T10:00:00Z"
}
```

Example response:

```json
{
  "command_id": "CMD-001",
  "device_id": "servo-01",
  "status": "EXECUTED",
  "timestamp": "2026-10-28T10:00:00Z"
}
```

---

# 71. Protocol Versioning

Messages should include a protocol version.

Example:

```json
{
  "protocol_version": "1.0",
  "command": "OPEN"
}
```

This allows future protocol evolution.

---

# 72. Device API vs MQTT

The application should not expose MQTT directly to every service.

Correct:

```text
Application Service
       ↓
Device Service
       ↓
MQTT Adapter
```

Incorrect:

```text
CheckoutService → MQTT
PaymentService → MQTT
TransactionService → MQTT
```

Centralizing device communication reduces coupling.

---

# 73. Device Service Example

Conceptually:

```python
class DeviceService:

    def show_message(self, message):
        self.lcd.display(message)

    def show_success(self):
        self.lcd.display("Payment Successful")
        self.led.green()

    def open_gate(self):
        self.servo.open()

    def show_error(self):
        self.lcd.display("System Error")
        self.led.red()
```

The implementation internally communicates through MQTT.

---

# 74. Smart Machine Workflow

The complete smart-machine workflow:

```text
Customer
   ↓
Computer Vision
   ↓
Application Decision
   ↓
Device Command
   ↓
MQTT
   ↓
Smart Device
   ↓
Physical Action
   ↓
Sensor Feedback
   ↓
MQTT
   ↓
Application
```

This creates the complete:

```text
Sense → Decide → Act → Observe
```

loop.

---

# 75. Example: Successful Checkout

```text
Customer
   ↓
Product Detection
   ↓
Verification
   ↓
MATCH
   ↓
Payment
   ↓
SUCCESS
   ↓
DeviceService
   ├── LCD = SUCCESS
   ├── LED = GREEN
   └── Servo = OPEN
          ↓
      Product Access
          ↓
      Servo = CLOSED
```

---

# 76. Example: Suspicious Transaction

```text
Camera
   ↓
Product Detection
   ↓
Tracking
   ↓
Expected ≠ Detected
   ↓
MISMATCH
   ↓
SUSPICIOUS_TRANSACTION
   ↓
DeviceService
   ├── LCD = VERIFY
   ├── LED = RED
   └── Servo = LOCKED
```

This demonstrates the integration of:

```text
Computer Vision
+
Backend
+
IoT
+
State Machine
```

---

# 77. Scalability

The architecture supports multiple kiosks.

Example:

```text
MQTT Broker
│
├── KIOSK-001
│   ├── servo-01
│   ├── lcd-01
│   └── led-01
│
├── KIOSK-002
│   ├── servo-01
│   ├── lcd-01
│   └── led-01
│
└── KIOSK-003
```

Every command is scoped by:

```text
kiosk_id
device_id
```

---

# 78. Cloud Deployment

Future architecture:

```text
                    CLOUD
┌────────────────────────────────────┐
│ FastAPI                            │
│ PostgreSQL                         │
│ MQTT Broker                        │
│ Monitoring                         │
└───────────────┬────────────────────┘
                │
             Internet
                │
       ┌────────┴─────────┐
       ▼                  ▼
   Kiosk 001           Kiosk 002
       │                  │
     ESP32              ESP32
       │                  │
   Sensors             Sensors
```

The initial project can run everything locally.

---

# 79. Cost Strategy

The project intentionally starts with simulation.

Advantages:

```text
No hardware cost
Fast iteration
Easy CI testing
Easy demo
Easy debugging
```

Only after validating the architecture should physical hardware be introduced.

Possible future hardware:

```text
ESP32
Servo
LCD
LED
Buttons
Sensors
```

---

# 80. Hardware Migration Plan

### Stage 1

```text
Web Simulator
```

### Stage 2

```text
ESP32 + Simulator
```

The real ESP32 controls one or more components while the simulator represents the remaining devices.

### Stage 3

```text
Full Physical Prototype
```

The architecture remains:

```text
Backend
 ↓
MQTT
 ↓
Device Adapter
 ↓
ESP32
 ↓
Hardware
```

---

# 81. Hardware Migration Acceptance Criteria

Before replacing the simulator with real hardware:

```text
☐ MQTT protocol documented
☐ Device command schema stable
☐ Device status schema stable
☐ Device state machine tested
☐ Simulator E2E tests pass
☐ Failure handling implemented
☐ Device authentication designed
☐ ESP32 firmware architecture documented
```

---

# 82. Definition of Done

The Embedded/IoT phase is complete when:

```text
☐ SmartDevice abstraction defined
☐ Device Service implemented
☐ Device Registry implemented
☐ MQTT broker integrated
☐ MQTT topics documented
☐ Command protocol documented
☐ Device status protocol documented
☐ LCD simulator implemented
☐ LED simulator implemented
☐ Servo simulator implemented
☐ Button simulator implemented
☐ Sensor simulator implemented
☐ Device heartbeat implemented
☐ Device failure simulation implemented
☐ MQTT integration tests implemented
☐ Backend + Simulator E2E test implemented
☐ Transaction success workflow implemented
☐ Suspicious transaction workflow implemented
☐ Simulator demo recorded
☐ Documentation updated
☐ Git commit created
```

---

# 83. Final Embedded/IoT Architecture

The complete architecture is:

```text
                         CUSTOMER
                            │
                            ▼
                     COMPUTER VISION
                            │
                            ▼
                    KIOSK APPLICATION
                            │
                            ▼
                      STATE MACHINE
                            │
                            ▼
                      DEVICE SERVICE
                            │
                            ▼
                   SMARTDEVICE INTERFACE
                            │
                            ▼
                      MQTT ADAPTER
                            │
                            ▼
                       MQTT BROKER
                            │
                  ┌─────────┴─────────┐
                  │                   │
                  ▼                   ▼
             WEB SIMULATOR          ESP32
                  │                   │
        ┌─────────┼─────────┐         │
        ▼         ▼         ▼         ▼
       LCD       LED       SERVO    GPIO/Sensors
        │         │         │         │
        └─────────┴─────────┴─────────┘
                            │
                         PHYSICAL
                          ACTION
                            │
                         SENSORS
                            │
                            ▼
                       MQTT EVENTS
                            │
                            ▼
                    KIOSK APPLICATION
```

The fundamental Smart Machine loop is:

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

The key engineering decision of this project is:

> **The simulator is not a shortcut that replaces embedded engineering. It is an implementation of the same device contract that the future ESP32 implementation will satisfy.**

Therefore:

```text
                  SmartDevice
                       │
             ┌─────────┴─────────┐
             ▼                   ▼
       SimulatorDevice       ESP32Device
             │                   │
             ▼                   ▼
        Web Hardware         Real Hardware
```

This makes the project capable of demonstrating **Embedded/IoT architecture, MQTT communication, device abstraction, Smart Machine control and hardware migration** even before physical hardware is purchased.