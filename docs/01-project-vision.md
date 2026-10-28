# RoShopSmart AI Retail Kiosk — Project Vision

**Document:** `01-project-vision.md`  
**Version:** `1.0  
**Status:** Draft / Initial  
**Project:** `roshopsmart-ai-retail-kiosk`  
**Type:** Independent AI Product Engineering Portfolio Project

---

## 1. Project Overview

**RoShopSmart AI Retail Kiosk** is an independent AI-powered Smart Retail Kiosk prototype designed to demonstrate the integration of:

- Computer Vision
- Artificial Intelligence
- Software Engineering
- Embedded Systems
- IoT Communication
- Human-Machine Interaction
- Automated Retail Workflows

The system simulates a self-service retail kiosk capable of detecting customers, understanding product interactions, verifying purchased products, assisting customers through an AI assistant, processing checkout workflows, and controlling simulated embedded devices.

The project is designed as a **portfolio-level AI Product Engineering project**, with an emphasis on the complete engineering lifecycle:

```text
Problem
   ↓
Requirements
   ↓
Architecture
   ↓
Implementation
   ↓
Integration
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

The objective is not simply to build an AI model that works in isolation, but to demonstrate the ability to transform AI technologies into an integrated product and smart-machine workflow.

---

# 2. Project Motivation

Modern retail systems increasingly combine software, artificial intelligence, computer vision, and physical devices to automate repetitive tasks and improve customer experience.

A Smart Retail Kiosk provides a practical engineering problem because it requires multiple systems to work together:

```text
Customer
   ↓
Sensors / Camera
   ↓
Computer Vision
   ↓
AI Decision
   ↓
Application Logic
   ↓
Backend
   ↓
IoT Communication
   ↓
Embedded Device
```

This creates an opportunity to study and demonstrate several important engineering challenges:

- Real-time computer vision
- Object detection and tracking
- Identity recognition
- Product verification
- State-machine design
- AI-assisted interaction
- Backend architecture
- IoT communication
- Embedded-device abstraction
- Error handling
- Security
- Performance optimization
- System testing

The project therefore focuses on **system integration and product engineering**, rather than treating AI as an isolated component.

---

# 3. Problem Statement

Traditional self-service retail systems often require customers to manually interact with interfaces, scan products, or rely heavily on predefined workflows.

A smarter kiosk could reduce friction by allowing the system to understand the customer's actions through cameras, sensors, voice interaction, and AI.

However, building such a system introduces several technical problems.

### 3.1 Customer Understanding

The system needs to determine when a customer approaches the kiosk.

Example:

```text
Camera
   ↓
Person Detection
   ↓
Customer Detected
```

---

### 3.2 Product Understanding

The system needs to identify products that the customer interacts with.

Example:

```text
Camera
   ↓
Object Detection
   ↓
Product Classification
   ↓
Product ID
```

---

### 3.3 Product Verification

The system must compare the customer's expected cart with what the computer vision system observes.

Example:

```text
Expected Cart

Coca Cola × 1
Water × 1

        ↓

Detected Products

Coca Cola
Coca Cola
Water

        ↓

Mismatch Detected

        ↓

Suspicious Transaction
```

This demonstrates how Computer Vision can be connected to a real business workflow.

---

### 3.4 Human-Machine Interaction

The kiosk should provide a natural interaction mechanism.

Potential interfaces include:

- Touch UI
- Voice
- Gesture
- Camera
- AI assistant

The system should be able to explain what is happening to the customer.

---

### 3.5 Smart Machine Control

The kiosk is not only a software application.

It also interacts with physical or simulated devices:

```text
Backend
   ↓
MQTT
   ↓
Smart Device
   ├── LCD
   ├── LED
   ├── Servo
   └── Sensors
```

The project therefore needs an architecture that separates business logic from hardware implementation.

---

# 4. Project Vision

The long-term vision is to build a modular Smart Retail Kiosk platform where:

> **Computer Vision understands the physical environment, AI understands the customer interaction, backend services manage the business workflow, and embedded systems execute physical actions.**

The complete system should behave like an integrated smart machine rather than a collection of independent demos.

The target architecture is:

```text
                    CUSTOMER
                       │
             ┌─────────┴─────────┐
             │                   │
           Camera              Voice
             │                   │
             ▼                   ▼
      Computer Vision       AI Assistant
             │                   │
             └─────────┬─────────┘
                       │
                       ▼
                KIOSK CONTROLLER
                       │
                       ▼
                STATE MACHINE
                       │
                       ▼
                   BACKEND
                       │
          ┌────────────┼────────────┐
          │            │            │
          ▼            ▼            ▼
      Database       AI Service    MQTT
                                     │
                                     ▼
                              SMART MACHINE
                                     │
                       ┌─────────────┼─────────────┐
                       │             │             │
                      LCD           LED          Servo
```

---

# 5. Project Objectives

The project has five major objectives.

## 5.1 AI Engineering

Demonstrate the ability to integrate AI models into a production-oriented software system.

Expected capabilities:

- Object Detection
- Face Detection
- Face Recognition
- Object Tracking
- Gesture Recognition
- Speech Recognition
- LLM-based AI Assistant
- Text-to-Speech

---

## 5.2 Computer Vision Engineering

Demonstrate the complete Computer Vision pipeline:

```text
Camera
   ↓
Frame Capture
   ↓
Preprocessing
   ↓
Detection
   ↓
Tracking
   ↓
Recognition
   ↓
Decision
```

The project should measure model performance rather than simply displaying predictions.

---

## 5.3 Smart Machine Engineering

Demonstrate how software can control embedded systems.

The architecture should support:

```text
Simulator
     │
     │
     ▼
SmartDevice Interface
     ▲
     │
     │
   ESP32
```

The backend should communicate with an abstraction rather than directly depending on a specific hardware implementation.

---

## 5.4 Product Engineering

The project should demonstrate engineering practices beyond AI modeling.

These include:

- Modular architecture
- API design
- Database design
- State-machine design
- Error handling
- Logging
- Testing
- CI/CD
- Docker
- Documentation
- Benchmarking

---

## 5.5 Portfolio Development

The final project should provide evidence of the ability to:

```text
Design
   ↓
Build
   ↓
Integrate
   ↓
Test
   ↓
Measure
   ↓
Optimize
   ↓
Document
   ↓
Demonstrate
```

This is the primary portfolio objective.

---

# 6. Target Users

The prototype is designed primarily as an engineering and research demonstration.

Potential real-world users include:

### Retail Stores

Self-service retail environments where customers purchase products without direct cashier interaction.

### Convenience Stores

Small stores requiring automated checkout and product verification.

### Smart Vending Systems

Automated retail systems combining cameras, payment, and physical product handling.

### Smart Campus / Office Stores

Unmanned or semi-automated retail environments.

### AI / Robotics Research

The architecture can also serve as a prototype platform for experimenting with:

- Computer Vision
- Human-machine interaction
- Edge AI
- IoT
- Smart Machines

---

# 7. Core User Experience

The intended customer workflow is:

```text
                 CUSTOMER
                    │
                    ▼
             Approach Kiosk
                    │
                    ▼
          Customer Detection
                    │
                    ▼
            Kiosk Greets User
                    │
                    ▼
              Start Shopping
                    │
                    ▼
            Select Products
                    │
                    ▼
          Product Verification
                    │
             ┌──────┴──────┐
             │             │
          Valid          Invalid
             │             │
             ▼             ▼
          Checkout     Suspicious
             │          Transaction
             ▼             │
          Payment           │
             │              │
             ▼              ▼
         Completed       Alert / Review
```

---

# 8. Core Product Capabilities

The initial product vision includes the following capabilities.

## 8.1 Customer Detection

Detect when a person approaches the kiosk.

Input:

```text
Camera Frame
```

Output:

```text
Customer Detected
```

---

## 8.2 Customer Recognition

Optional identity recognition can be used to provide personalized experiences.

Potential workflow:

```text
Face
 ↓
Face Detection
 ↓
Face Embedding
 ↓
Identity Matching
 ↓
Customer Profile
```

This feature will be treated carefully because biometric data introduces privacy and security considerations.

---

## 8.3 Product Detection

The system identifies products visible to the camera.

Example:

```text
Frame
 ↓
YOLO
 ↓
Bounding Boxes
 ↓
Product Classes
```

Example result:

```text
CocaCola: 1
Water: 1
Snack: 2
```

---

## 8.4 Product Tracking

Tracking allows the system to understand product movement across frames.

Example:

```text
Frame 1 → Product A
Frame 2 → Product A
Frame 3 → Product A
Frame 4 → Product A
```

Instead of treating each frame as a completely independent observation, the system maintains object identity across frames.

---

## 8.5 Product Verification

The system compares:

```text
Expected Cart
      vs
Detected Products
```

Example:

```text
Expected:

Water × 1
Coca Cola × 1

Detected:

Water × 1
Coca Cola × 2
```

Result:

```text
VERIFICATION_FAILED
```

The kiosk can then transition into:

```text
SUSPICIOUS_TRANSACTION
```

---

## 8.6 AI Assistant

The AI assistant provides conversational support.

Potential capabilities:

- Product information
- Shopping guidance
- Checkout explanation
- Error explanation
- Product recommendations
- Frequently asked questions

Example:

```text
Customer:
"What is this drink?"

        ↓

AI Assistant

        ↓

Product Database
        ↓

LLM
        ↓

Response

"This is Coca Cola. It is a carbonated soft drink..."
```

---

## 8.7 Voice Interaction

Potential voice pipeline:

```text
Microphone
    ↓
Speech Recognition
    ↓
Text
    ↓
LLM
    ↓
Response
    ↓
TTS
    ↓
Speaker
```

---

## 8.8 Embedded Device Control

The kiosk can control simulated physical components.

Example:

```text
Payment Successful
       ↓
MQTT Command
       ↓
Embedded Simulator
       ├── LCD → "Payment Successful"
       ├── LED → ON
       └── Servo → OPEN
```

---

# 9. System Boundaries

The initial project will focus on software simulation and controlled environments.

## In Scope

```text
Computer Vision
AI Assistant
Product Detection
Product Verification
Kiosk State Machine
Backend API
Database
MQTT
Embedded Simulator
Testing
Benchmarking
Docker
Documentation
```

## Out of Scope — Initial Version

```text
Real payment processing
Real banking integration
Production retail deployment
Physical product dispensing hardware
Large-scale cloud deployment
Real biometric identity management
Full regulatory certification
Production-grade anti-fraud system
```

These may be considered in future versions.

---

# 10. Independent Project Declaration

This project is an **independent portfolio project**.

It is inspired by the general problem domain of AI-powered Smart Retail Kiosks and Smart Machines.

The project is not:

- An internal product of NaviX.
- An official implementation of RoShopSmart.
- A NaviX proprietary project.
- A reproduction of proprietary source code.
- A representation of NaviX's internal architecture.

No proprietary source code, confidential information, or internal technology from any company will be used.

The project is independently designed and implemented for educational, engineering, research, and portfolio purposes.

---

# 11. Engineering Philosophy

The project follows a product engineering mindset.

The primary principle is:

> **AI is a component of the product, not the product itself.**

A successful system must combine:

```text
AI
+
Software
+
Hardware
+
Data
+
Infrastructure
+
UX
+
Testing
```

The project therefore avoids building isolated AI notebooks without integration.

Instead, every AI component should eventually connect to a real system workflow.

Example:

```text
YOLO
 ↓
Product Detection
 ↓
Product Service
 ↓
Shopping Cart
 ↓
Verification
 ↓
Checkout
```

Rather than:

```text
YOLO Demo
```

---

# 12. Architecture Principles

The project follows several architectural principles.

## 12.1 Modularity

Each major capability should be independently replaceable.

Example:

```text
Object Detection
     │
     ├── YOLO
     ├── Alternative Model
     └── Future Edge Model
```

---

## 12.2 Hardware Abstraction

The business logic should not directly depend on hardware.

Instead:

```text
Business Logic
      ↓
SmartDevice Interface
      ↓
 ┌────┴────┐
 ▼         ▼
Simulator ESP32
```

This allows the simulator to be replaced by physical hardware later.

---

## 12.3 API-First Design

Backend capabilities should be exposed through well-defined APIs.

Example:

```text
POST /customers/detect
POST /products/detect
POST /cart/items
POST /verification
POST /checkout
POST /payment
GET  /products/{id}
```

The exact API design will be defined in later documentation.

---

## 12.4 Testability

Each component should be testable independently.

```text
AI Model
Backend
Database
State Machine
MQTT
Embedded Simulator
```

The system should also support integration and end-to-end testing.

---

## 12.5 Observability

The system should produce structured logs and measurable metrics.

Example:

```text
Detection latency
Inference latency
API latency
MQTT latency
Transaction duration
Verification accuracy
Error rate
```

---

# 13. Technology Direction

The initial technology direction is:

### AI / Computer Vision

```text
Python
OpenCV
YOLO
MediaPipe
InsightFace
OpenAI API
Speech Recognition
TTS
```

### Backend

```text
FastAPI
PostgreSQL
REST API
WebSocket where appropriate
```

### IoT

```text
MQTT
ESP32-compatible architecture
Web-based Embedded Simulator
```

### Infrastructure

```text
Docker
Docker Compose
GitHub Actions
Pytest
```

### Documentation

```text
Markdown
Architecture Diagrams
State Machine Diagrams
AI Pipeline Diagrams
```

Technology choices are not considered final until documented and evaluated through technical decision records.

---

# 14. Development Strategy

Development will be incremental.

## Phase 0 — Project Initialization

Focus:

```text
Documentation
Requirements
Architecture
GitHub
Technical Decisions
```

No major implementation yet.

---

## Phase 1 — Software Simulation

Build the core retail workflow without hardware.

```text
Camera
 ↓
Computer Vision
 ↓
Kiosk UI
 ↓
Backend
 ↓
Database
```

Goal:

> Validate the product workflow and architecture.

---

## Phase 2 — AI Integration

Integrate:

```text
Object Detection
Face Recognition
Gesture Recognition
Tracking
Voice
AI Assistant
TTS
```

Goal:

> Demonstrate AI Engineering capability.

---

## Phase 3 — Smart Machine Simulation

Implement:

```text
FastAPI
 ↓
MQTT
 ↓
Embedded Simulator
 ↓
LCD
LED
Servo
Sensors
```

Goal:

> Demonstrate Embedded / IoT / Smart Machine architecture.

---

## Phase 4 — Productization

Add:

```text
Docker
CI/CD
Testing
Logging
Security
Benchmarking
Error Handling
Documentation
```

Goal:

> Transform the prototype into a portfolio-level engineering project.

---

## Phase 5 — Portfolio and Demonstration

Prepare:

```text
Demo Video
Architecture Presentation
Technical Blog
Benchmark Results
Interview Questions
Engineering Decisions
Known Limitations
Future Roadmap
```

Goal:

> Clearly communicate the engineering capability demonstrated by the project.

---

# 15. Success Criteria

The project will be considered successful when it demonstrates an integrated workflow such as:

```text
Customer
   ↓
Camera
   ↓
Customer Detection
   ↓
Shopping
   ↓
Product Detection
   ↓
Product Verification
   ↓
Checkout
   ↓
Payment Simulation
   ↓
MQTT
   ↓
Embedded Simulator
   ↓
LCD / LED / Servo
   ↓
Transaction Completed
```

Additionally, the system should have:

- Automated tests
- Documented architecture
- Measurable AI performance
- Measurable system performance
- Reproducible development environment
- Clear technical decisions
- Error handling
- Security considerations
- Demonstrable Git history

---

# 16. Portfolio Evaluation Criteria

The project should be evaluated across five dimensions.

## AI Engineering

Can the project demonstrate:

```text
Model Integration
Inference
Evaluation
Optimization
```

---

## Computer Vision

Can the system:

```text
Detect
Track
Recognize
Verify
```

objects or people in a controlled environment?

---

## Software Engineering

Does the project demonstrate:

```text
Architecture
API Design
Database
Testing
Error Handling
CI/CD
```

?

---

## Embedded / IoT Engineering

Can the system:

```text
Backend
 ↓
MQTT
 ↓
Device Abstraction
 ↓
Simulator / ESP32
```

?

---

## Product Engineering

Does the project demonstrate:

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
Iteration
```

?

---

# 17. Expected Final Demonstration

The final demonstration should be approximately 2–3 minutes.

Example:

```text
00:00
Customer approaches kiosk

00:15
Camera detects customer

00:25
Kiosk greets customer

00:40
Customer selects product

01:00
Computer Vision verifies product

01:20
Customer asks AI assistant about product

01:30
AI responds using voice/TTS

01:45
Customer starts checkout

02:00
Payment simulation succeeds

02:05
Backend publishes MQTT command

02:10
Embedded simulator reacts

       LCD → Payment Successful
       LED → ON
       Servo → OPEN

02:20
Transaction completed

02:30
System returns to IDLE
```

A second scenario should demonstrate an error:

```text
Expected Product
       ≠
Detected Product

       ↓

Verification Failed

       ↓

SUSPICIOUS_TRANSACTION

       ↓

Alert / Explanation

       ↓

Manual Resolution
```

This second scenario is important because it demonstrates that the system handles failure conditions rather than only the happy path.

---

# 18. Known Limitations

The initial project will have several limitations.

### Computer Vision

Performance may depend on:

- Camera quality
- Lighting
- Product appearance
- Occlusion
- Dataset quality
- Hardware capability

---

### Face Recognition

Face recognition introduces:

- Privacy concerns
- False positives
- False negatives
- Spoofing risks
- Data protection requirements

Therefore, face recognition should initially be treated as an experimental capability rather than a production authentication mechanism.

---

### Product Verification

A camera-based system may not perfectly determine:

- Hidden products
- Products outside the camera view
- Multiple identical products
- Occluded products
- Very similar product packages

The benchmark should explicitly measure these limitations.

---

### Embedded Simulation

The simulator does not perfectly represent real-world hardware constraints.

For example:

```text
Real ESP32
    ≠
Web Simulator
```

The simulator is primarily intended to validate:

- Protocol
- Commands
- State transitions
- Device abstraction
- Integration architecture

---

# 19. Future Vision

Future versions may extend the system toward:

```text
Edge AI
   ↓
Real ESP32
   ↓
Physical Sensors
   ↓
Physical Servo
   ↓
Real Product Dispensing
```

Potential future capabilities:

- Edge inference
- Real ESP32 hardware
- RFID
- Weight sensors
- Smart shelves
- Physical product dispensing
- Real payment integration
- Multi-kiosk management
- Remote monitoring
- Fleet management
- Cloud analytics
- Advanced anomaly detection
- Digital twin of the kiosk

The architecture should therefore remain modular enough to support these extensions.

---

# 20. Project Roadmap

```text
┌──────────────────────────────────────────────┐
│ PHASE 0                                      │
│ Project Vision & Documentation               │
└───────────────────────┬──────────────────────┘
                        │
                        ▼
┌──────────────────────────────────────────────┐
│ PHASE 1                                      │
│ Software Retail Simulation                   │
└───────────────────────┬──────────────────────┘
                        │
                        ▼
┌──────────────────────────────────────────────┐
│ PHASE 2                                      │
│ AI & Computer Vision Integration             │
└───────────────────────┬──────────────────────┘
                        │
                        ▼
┌──────────────────────────────────────────────┐
│ PHASE 3                                      │
│ Embedded / IoT Smart Machine Simulation      │
└───────────────────────┬──────────────────────┘
                        │
                        ▼
┌──────────────────────────────────────────────┐
│ PHASE 4                                      │
│ Productization                               │
└───────────────────────┬──────────────────────┘
                        │
                        ▼
┌──────────────────────────────────────────────┐
│ PHASE 5                                      │
│ Portfolio / Demo / Interview Preparation    │
└──────────────────────────────────────────────┘
```

---

# 21. Definition of Done — Project Vision

This document is considered complete when the following questions can be answered clearly:

### Problem

- What problem are we solving?
- Why is it technically interesting?

### Product

- What does the kiosk do?
- Who uses it?
- What is the customer workflow?

### AI

- Where is AI used?
- What role does Computer Vision play?
- What role does the AI Assistant play?

### Smart Machine

- How does software communicate with devices?
- How can the simulator eventually be replaced by ESP32?

### Engineering

- How will the system be tested?
- How will performance be measured?
- How will failures be handled?

### Portfolio

- What engineering skills does the project demonstrate?
- What evidence will be produced?

---

# 22. Final Project Principle

The central principle of RoShopSmart AI Retail Kiosk is:

> **Build a complete intelligent system, not just an AI demo.**

The project should demonstrate the ability to take a real-world problem and transform it into an engineered system:

```text
Real-World Problem
        ↓
Product Vision
        ↓
Requirements
        ↓
System Architecture
        ↓
AI Architecture
        ↓
Software Implementation
        ↓
Computer Vision
        ↓
IoT / Embedded Integration
        ↓
Testing
        ↓
Benchmark
        ↓
Optimization
        ↓
Documentation
        ↓
Production-Oriented Demo
```

The ultimate goal is to demonstrate practical capability across:

```text
AI
+
Computer Vision
+
Software Engineering
+
Embedded / IoT
+
System Architecture
+
Product Engineering
```

This project is intended to serve as an independent engineering portfolio demonstrating the ability to **design, build, integrate, test, evaluate, and improve an AI-powered Smart Machine system**.