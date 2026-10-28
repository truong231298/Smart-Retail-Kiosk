# RoShopSmart AI Retail Kiosk — AI Architecture

**Document:** `04-ai-architecture.md`  
**Version:** `1.0`  
**Status:** Draft / Initial  
**Project:** `roshopsmart-ai-retail-kiosk`

---

# 1. Purpose

This document defines the AI architecture of the **RoShopSmart AI Retail Kiosk**.

The AI layer is responsible for enabling the kiosk to:

- Perceive customers and products.
- Understand visual information.
- Track objects and interactions.
- Recognize customers where appropriate.
- Understand voice input.
- Provide an AI shopping assistant.
- Generate natural-language responses.
- Convert responses into speech.
- Support product verification.
- Provide intelligent error explanations.

The architecture is designed to keep AI models independent from the core business logic.

The system should be able to replace an AI model without rewriting the kiosk's business workflow.

---

# 2. AI Engineering Goals

The AI architecture has the following goals:

1. **Model independence**
2. **Low coupling**
3. **Measurable inference performance**
4. **Modular AI pipelines**
5. **Replaceable providers**
6. **Local-first inference where practical**
7. **Cloud AI where it provides additional value**
8. **Testability**
9. **Observability**
10. **Graceful degradation**

The AI layer should not simply demonstrate that an AI model works.

It should demonstrate the engineering process:

```text
Problem
   ↓
AI Requirement
   ↓
Data
   ↓
Model
   ↓
Inference
   ↓
Post-processing
   ↓
Application Integration
   ↓
Evaluation
   ↓
Optimization
```

---

# 3. AI System Overview

The AI system consists of several major capabilities:

```text
AI SYSTEM
│
├── Computer Vision
│   ├── Person Detection
│   ├── Face Detection
│   ├── Face Recognition
│   ├── Product Detection
│   ├── Object Tracking
│   └── Gesture Recognition
│
├── Speech
│   ├── Speech Recognition
│   └── Text-to-Speech
│
├── AI Assistant
│   ├── Intent Understanding
│   ├── Product Q&A
│   ├── Shopping Guidance
│   ├── Error Explanation
│   └── Tool Calling
│
└── AI Infrastructure
    ├── Model Registry
    ├── Inference Runtime
    ├── Provider Abstraction
    ├── Metrics
    └── Logging
```

---

# 4. High-Level AI Architecture

```text
                         CUSTOMER
                            │
              ┌─────────────┴─────────────┐
              │                           │
              ▼                           ▼
           CAMERA                      MICROPHONE
              │                           │
              ▼                           ▼
      ┌───────────────┐          ┌────────────────┐
      │ Vision        │          │ Speech         │
      │ Pipeline      │          │ Pipeline       │
      └───────┬───────┘          └───────┬────────┘
              │                          │
              ▼                          ▼
      Structured Events             Text Input
              │                          │
              └────────────┬─────────────┘
                           ▼
                  ┌──────────────────┐
                  │ AI Orchestrator  │
                  └────────┬─────────┘
                           │
             ┌─────────────┼─────────────┐
             ▼             ▼             ▼
        CV Services    AI Assistant    TTS
             │             │
             │             ▼
             │        AI Gateway
             │             │
             │             ▼
             │        LLM Provider
             │
             ▼
       Application Layer
             │
             ▼
        State Machine
```

---

# 5. AI Architecture Principle

The core principle is:

> **AI produces intelligence signals; application logic decides what the kiosk does.**

For example:

```text
YOLO
 ↓
"Water bottle detected"
```

The AI system should not directly execute:

```text
checkout()
```

Instead:

```text
YOLO
 ↓
ProductDetectedEvent
 ↓
VerificationService
 ↓
Application Decision
 ↓
Checkout
```

This prevents AI models from becoming the source of business logic.

---

# 6. AI Boundary

The AI layer has a clear interface with the application.

```text
┌──────────────────────────────┐
│         AI LAYER             │
│                              │
│ Detection                    │
│ Recognition                  │
│ Tracking                     │
│ Speech                       │
│ LLM                          │
└──────────────┬───────────────┘
               │
         Structured Events
               │
               ▼
┌──────────────────────────────┐
│      APPLICATION LAYER       │
│                              │
│ State Machine                │
│ Cart                         │
│ Verification                 │
│ Checkout                     │
│ Transaction                  │
└──────────────────────────────┘
```

The application should consume stable contracts rather than model-specific outputs.

---

# 7. AI Capability Matrix

| Capability | Input | Output | Initial Technology |
|---|---|---|---|
| Person Detection | Image | Person bounding boxes | YOLO |
| Product Detection | Image | Product detections | YOLO |
| Object Tracking | Video | Track IDs | ByteTrack / similar |
| Face Detection | Image | Face boxes | InsightFace |
| Face Recognition | Face | Identity embedding | InsightFace |
| Gesture Recognition | Video | Gesture event | MediaPipe |
| Speech Recognition | Audio | Text | STT provider |
| AI Assistant | Text | Response | OpenAI API |
| TTS | Text | Audio | TTS provider |

Technologies are implementation choices rather than permanent architectural dependencies.

---

# 8. Computer Vision Pipeline

The CV pipeline processes camera input.

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
Post-processing
  ↓
Event Generation
```

The pipeline should be configurable depending on the task.

For example, product verification may use:

```text
Detection
+
Tracking
```

while customer identification may use:

```text
Face Detection
+
Face Recognition
```

---

# 9. Frame Processing

Raw camera frames should not automatically be sent through every AI model.

Instead:

```text
Frame
 ↓
Frame Scheduler
 ↓
Select Processing Pipeline
```

Possible strategies:

```text
10 FPS detection
30 FPS display
5 FPS heavy recognition
```

This reduces unnecessary inference cost.

---

# 10. Detection Architecture

The object detection interface should be model-independent.

Conceptual interface:

```python
class ObjectDetector:
    def detect(self, frame):
        ...
```

Possible implementations:

```text
ObjectDetector
│
├── YOLODetector
├── FutureDetector
└── MockDetector
```

The rest of the application depends on:

```text
ObjectDetector
```

rather than:

```text
YOLODetector
```

---

# 11. Detection Output

Raw model output should be normalized.

Example:

```json
{
  "class_id": 1,
  "class_name": "water_bottle",
  "confidence": 0.94,
  "bbox": {
    "x1": 120,
    "y1": 80,
    "x2": 240,
    "y2": 350
  }
}
```

Normalized output:

```text
Detection
├── object_id
├── class_name
├── confidence
├── bounding_box
└── timestamp
```

This provides a stable interface between AI and application layers.

---

# 12. Product Detection

Product detection is one of the core AI capabilities.

Example:

```text
Camera Frame
      ↓
YOLO
      ↓
┌───────────────────────────┐
│ Coca Cola   0.96          │
│ Water       0.93          │
│ Chips       0.91          │
└───────────────────────────┘
```

The detector should identify products relevant to the kiosk inventory.

---

# 13. Product Detection Dataset

The project should eventually maintain a dedicated product dataset.

Example structure:

```text
datasets/
└── products/
    ├── images/
    ├── labels/
    ├── train/
    ├── val/
    └── test/
```

Potential dataset variations:

```text
Lighting
Angles
Occlusion
Distance
Background
Multiple Products
Partial Visibility
```

The goal is not only high benchmark accuracy but robust real-world kiosk performance.

---

# 14. Product Detection Evaluation

Product detection should be evaluated using:

```text
Precision
Recall
mAP@50
mAP@50:95
Inference Latency
FPS
```

Example benchmark:

```text
Model: ProductDetector-v1

mAP50:             XX
Precision:         XX
Recall:            XX
CPU latency:       XX ms
GPU latency:       XX ms
FPS:               XX
```

Actual numbers must come from measured experiments.

They must never be fabricated.

---

# 15. Object Tracking

Detection alone is insufficient for video-based product verification.

The system needs temporal consistency.

```text
Frame 1
  ↓
Detection
  ↓
Track ID = 01

Frame 2
  ↓
Detection
  ↓
Track ID = 01

Frame 3
  ↓
Detection
  ↓
Track ID = 01
```

Tracking allows the system to reason about:

```text
Object appeared
Object remained
Object moved
Object disappeared
```

---

# 16. Tracking Architecture

```text
Detector
   │
   ▼
Detection Results
   │
   ▼
Tracker
   │
   ▼
Track Objects
   │
   ├── track_id
   ├── class
   ├── bbox
   ├── confidence
   └── timestamps
```

Potential implementation:

```text
YOLO
+
ByteTrack
```

or another suitable tracker.

The tracker must remain replaceable.

---

# 17. Product Interaction Events

Tracking can generate higher-level events.

Examples:

```text
PRODUCT_APPEARED
PRODUCT_MOVED
PRODUCT_REMOVED
PRODUCT_STABLE
PRODUCT_OCCLUDED
```

Example:

```text
Product appears
      ↓
Track created
      ↓
Customer interaction
      ↓
Track disappears
      ↓
ProductRemovedEvent
```

This can later support automatic cart updates.

---

# 18. Face Detection

Face detection identifies faces in the camera frame.

Pipeline:

```text
Camera
  ↓
Face Detector
  ↓
Face Bounding Box
  ↓
Face Quality Check
  ↓
Face Recognition
```

Face detection and face recognition should be separate modules.

---

# 19. Face Recognition

Face recognition should not be treated as a direct application decision.

Architecture:

```text
Face
 ↓
Embedding Model
 ↓
Face Embedding
 ↓
Similarity Search
 ↓
Identity Candidate
 ↓
Confidence Validation
 ↓
Recognition Result
```

Example:

```text
Identity:
customer_123

Similarity:
0.91

Decision:
MATCH
```

Thresholds must be experimentally evaluated.

---

# 20. Face Recognition Privacy

Face recognition introduces privacy concerns.

The system should:

- Avoid storing raw face images unnecessarily.
- Protect face embeddings.
- Minimize retention.
- Clearly separate identity data from transaction data.
- Avoid exposing embeddings through public APIs.
- Provide configurable recognition behavior.

For the portfolio prototype, face recognition should be treated as an optional module.

The kiosk must remain functional without it.

---

# 21. Gesture Recognition

Gesture recognition provides an alternative interaction method.

Potential gestures:

```text
OPEN_HAND
THUMBS_UP
THUMBS_DOWN
POINT_LEFT
POINT_RIGHT
CONFIRM
CANCEL
```

Pipeline:

```text
Camera
  ↓
Hand Detection
  ↓
Landmarks
  ↓
Gesture Classifier
  ↓
Gesture Event
```

Example:

```text
THUMBS_UP
   ↓
GestureDetectedEvent
   ↓
Application
   ↓
Confirm Selection
```

---

# 22. MediaPipe Integration

MediaPipe may be used for hand landmarks and gesture-related processing.

Architecture:

```text
GestureRecognizer
       │
       └── MediaPipeGestureRecognizer
```

The application receives:

```text
GestureEvent
```

rather than MediaPipe-specific structures.

---

# 23. Speech Recognition

Voice interaction pipeline:

```text
Microphone
   ↓
Audio Capture
   ↓
Voice Activity Detection
   ↓
Speech Recognition
   ↓
Text
   ↓
Intent / AI Assistant
```

Example:

```text
Customer:
"What drinks do you have?"

       ↓

Speech-to-Text

       ↓

"What drinks do you have?"

       ↓

AI Assistant
```

---

# 24. Speech Recognition Interface

Use a provider abstraction:

```text
SpeechRecognizer
│
├── CloudSpeechRecognizer
├── LocalSpeechRecognizer
└── MockSpeechRecognizer
```

Application dependency:

```text
SpeechRecognizer
```

not:

```text
Specific Provider SDK
```

---

# 25. AI Assistant Architecture

The AI Assistant provides natural-language interaction.

Primary responsibilities:

```text
Product Q&A
Shopping Guidance
Recommendation
Error Explanation
General Kiosk Assistance
```

The AI Assistant should not directly control the kiosk.

Instead:

```text
Customer
 ↓
AI Assistant
 ↓
Intent / Tool Request
 ↓
Application Service
 ↓
Validated Result
 ↓
AI Response
```

---

# 26. LLM Architecture

```text
Customer Input
      │
      ▼
AI Assistant
      │
      ▼
Prompt / Context Builder
      │
      ▼
AI Gateway
      │
      ▼
LLM Provider
      │
      ▼
Structured Response
      │
      ▼
Response Validator
      │
      ▼
Customer
```

---

# 27. AI Gateway

The AI Gateway abstracts the LLM provider.

```text
AIProvider
│
├── OpenAIProvider
├── MockAIProvider
└── FutureProvider
```

Responsibilities:

- Request creation.
- Model configuration.
- Timeout.
- Retry.
- Token tracking.
- Error handling.
- Structured output validation.

---

# 28. Structured AI Output

The AI Assistant should prefer structured outputs for application-facing operations.

Example:

```json
{
  "intent": "PRODUCT_SEARCH",
  "query": "cold drinks",
  "response": "I found three cold drinks.",
  "tool_calls": [
    {
      "tool": "search_products",
      "arguments": {
        "category": "drink"
      }
    }
  ]
}
```

The application validates the structure before executing any action.

---

# 29. Tool Calling

The LLM can request tools.

Example tools:

```text
search_products
get_product_details
check_inventory
get_cart
add_to_cart
remove_from_cart
get_order_status
```

Architecture:

```text
LLM
 │
 ▼
Tool Request
 │
 ▼
Tool Validator
 │
 ▼
Application Service
 │
 ▼
Tool Result
 │
 ▼
LLM
```

The LLM should never receive unrestricted database access.

---

# 30. AI Assistant Example

Customer:

```text
"Do you have any drinks without sugar?"
```

Flow:

```text
Voice
 ↓
Speech-to-Text
 ↓
AI Assistant
 ↓
Intent Detection
 ↓
search_products()
 ↓
Product Service
 ↓
PostgreSQL
 ↓
Product Results
 ↓
LLM
 ↓
Natural Language Response
 ↓
TTS
 ↓
Speaker
```

---

# 31. AI Assistant Context

The AI Assistant may receive controlled context such as:

```text
Current kiosk state
Current cart
Available products
Current transaction
Detected errors
```

Example:

```json
{
  "state": "SHOPPING",
  "cart": [
    {
      "product_id": "P001",
      "quantity": 1
    }
  ]
}
```

Only the minimum required context should be provided.

---

# 32. Prompt Architecture

Prompts should be separated from application code.

Suggested structure:

```text
ai/
└── prompts/
    ├── system/
    ├── product_assistant/
    ├── error_explanation/
    └── recommendations/
```

Prompts should be versioned.

Example:

```text
product_assistant_v1
product_assistant_v2
```

This allows prompt experiments and regression testing.

---

# 33. Hallucination Control

The AI Assistant must not invent:

```text
Product Price
Inventory
Product Availability
Transaction Status
```

Instead:

```text
LLM
 ↓
Tool
 ↓
Authoritative Service
 ↓
Validated Data
 ↓
LLM
```

For example:

```text
Bad:

"Water costs $1.20."

```

unless the Product Service actually returned:

```text
price = 1.20
```

---

# 34. AI Safety Boundary

The LLM should not directly perform sensitive operations.

The following operations require application validation:

```text
Payment
Checkout
Inventory Modification
Transaction Completion
Device Control
Identity Changes
```

The AI may request an action.

The application decides whether that action is valid.

---

# 35. TTS Architecture

Text-to-Speech pipeline:

```text
AI Response
    ↓
Text Validation
    ↓
TTS Provider
    ↓
Audio
    ↓
Speaker
```

Interface:

```text
TextToSpeech
│
├── CloudTTS
├── LocalTTS
└── MockTTS
```

---

# 36. AI Orchestrator

The AI Orchestrator coordinates AI capabilities.

Responsibilities:

```text
Route AI requests
Manage context
Call AI services
Normalize responses
Handle failures
Publish AI events
```

Conceptual architecture:

```text
                    AI ORCHESTRATOR
                          │
          ┌───────────────┼───────────────┐
          ▼               ▼               ▼
       Vision          Assistant         Voice
          │               │               │
          ▼               ▼               ▼
      CV Models          LLM             STT/TTS
```

The orchestrator should not contain business rules.

---

# 37. AI Event Architecture

AI services generate structured events.

Examples:

```text
PersonDetectedEvent
FaceRecognizedEvent
ProductDetectedEvent
GestureDetectedEvent
SpeechRecognizedEvent
AIResponseGeneratedEvent
```

Example:

```json
{
  "event_type": "PRODUCT_DETECTED",
  "timestamp": "2026-10-28T10:00:00Z",
  "payload": {
    "product_id": "P001",
    "confidence": 0.94
  }
}
```

---

# 38. AI → Application Integration

The recommended integration is:

```text
AI
 │
 ▼
AI Event
 │
 ▼
Application Event Handler
 │
 ▼
Application Service
 │
 ▼
State Machine
```

Example:

```text
ProductDetected
      ↓
Product Event Handler
      ↓
Cart / Verification Service
      ↓
Application Decision
```

---

# 39. AI Inference Modes

The system should support three inference modes.

## Local

```text
Camera
 ↓
Local AI
```

Advantages:

- Low latency.
- Privacy.
- No network dependency.

---

## Cloud

```text
Camera / Text
 ↓
Cloud AI
```

Advantages:

- Powerful models.
- Easier model updates.
- Potentially better accuracy.

---

## Hybrid

```text
Edge AI
   +
Cloud AI
```

Recommended future architecture:

```text
Real-time CV
    ↓
Edge

Complex reasoning
    ↓
Cloud
```

---

# 40. Edge AI Strategy

Real-time tasks should preferably run locally when hardware allows.

Examples:

```text
Person Detection
Product Detection
Tracking
Gesture Recognition
```

Cloud AI may be used for:

```text
Natural Language Reasoning
Complex Recommendations
AI Assistant
```

This minimizes latency and network dependency.

---

# 41. AI Performance Budget

Every AI component should have a measurable performance target.

Example target categories:

```text
Detection latency
Tracking latency
Face recognition latency
Speech recognition latency
LLM latency
TTS latency
```

Example:

```text
Camera → Detection:
< target latency

Voice → Response:
< target latency

Product verification:
< target latency
```

Actual targets will be established after baseline benchmarking.

---

# 42. AI Resource Management

The AI pipeline should consider:

```text
CPU
GPU
RAM
VRAM
Power
Network
```

Potential optimization strategies:

```text
Frame skipping
Batch inference
Model quantization
Resolution reduction
Model caching
GPU acceleration
Pipeline parallelism
```

Optimization should be based on measured bottlenecks.

---

# 43. Model Configuration

Model configuration should not be hardcoded.

Example:

```yaml
vision:
  detector:
    model: product_detector_v1
    confidence_threshold: 0.5
    input_size: 640

tracking:
  enabled: true

face:
  enabled: false

assistant:
  provider: openai
```

This allows experiments without modifying application code.

---

# 44. Model Versioning

Models must be versioned.

Example:

```text
product_detector_v1
product_detector_v2
face_recognition_v1
gesture_model_v1
```

Every inference event should optionally record:

```text
model_name
model_version
inference_time
confidence
```

This enables reproducibility.

---

# 45. AI Experiment Tracking

AI experiments should record:

```text
Experiment ID
Model
Dataset
Configuration
Metrics
Hardware
Latency
Accuracy
```

Example:

```text
Experiment:
EXP-001

Model:
product_detector_v1

Dataset:
products_v1

Hardware:
RTX / CPU

mAP50:
XX

Latency:
XX ms
```

Actual measurements must be collected from experiments.

---

# 46. AI Observability

Important AI metrics:

```text
Inference latency
Confidence distribution
Detection count
False positives
False negatives
Model errors
Provider errors
Token usage
LLM latency
TTS latency
```

These metrics help identify system bottlenecks.

---

# 47. Confidence Handling

AI confidence should not automatically become a binary truth.

Example:

```text
Confidence > 0.90
    ↓
High confidence

0.60–0.90
    ↓
Medium confidence

< 0.60
    ↓
Low confidence
```

Thresholds must be calibrated using validation data.

---

# 48. Uncertainty Handling

When the AI is uncertain:

```text
AI uncertain
     ↓
Application receives uncertainty
     ↓
Request additional evidence
```

Example:

```text
Product confidence = 0.42
        ↓
Do not immediately add product
        ↓
Wait for additional frames
        ↓
Re-evaluate
```

This is preferable to blindly trusting a single inference.

---

# 49. Temporal Verification

For important decisions, the system should use multiple frames.

Example:

```text
Frame 1:
Water 0.71

Frame 2:
Water 0.84

Frame 3:
Water 0.92

Frame 4:
Water 0.94
```

The system can establish:

```text
Stable Detection
```

instead of relying on a single frame.

---

# 50. Product Verification AI Pipeline

The complete verification pipeline:

```text
Camera
  ↓
Frame Sampling
  ↓
Product Detection
  ↓
Object Tracking
  ↓
Temporal Aggregation
  ↓
Product Identity
  ↓
Detected Product Set
  ↓
Verification Service
  ↓
Expected vs Detected
  ↓
Verification Result
```

---

# 51. Suspicious Transaction Detection

Suspicious behavior is initially rule-based rather than fully AI-based.

Example:

```text
Expected:
Coke × 1

Detected:
Coke × 2
```

Result:

```text
MISMATCH
```

Future versions may incorporate:

```text
Behavior Analysis
Object Interaction Analysis
Anomaly Detection
```

The project should not claim advanced fraud detection before implementing and evaluating it.

---

# 52. AI Failure Handling

AI failures should not crash the entire kiosk.

Example:

```text
AI unavailable
      ↓
Fallback
      ↓
Manual / UI interaction
```

Possible fallback:

```text
Product Detection unavailable
        ↓
Display product catalog
        ↓
Customer selects manually
```

Another:

```text
LLM unavailable
        ↓
Use predefined FAQ responses
```

---

# 53. AI Test Strategy

AI testing has multiple layers.

## Unit Tests

Test:

```text
Preprocessing
Post-processing
Threshold Logic
Event Conversion
Prompt Construction
Response Parsing
```

## Integration Tests

Test:

```text
Model → AI Service
AI Service → Backend
LLM → Tool
STT → Assistant
```

## End-to-End Tests

Test:

```text
Camera
 ↓
Detection
 ↓
Verification
 ↓
Checkout
```

---

# 54. Mock AI Components

Development should support mocks.

Example:

```text
MockDetector
MockRecognizer
MockSpeechRecognizer
MockAIProvider
MockTTS
```

This allows:

```text
Backend tests
State-machine tests
Checkout tests
```

without requiring actual AI models.

---

# 55. Deterministic AI Testing

AI-dependent workflows should have deterministic fixtures.

Example:

```json
{
  "detections": [
    {
      "product": "water",
      "confidence": 0.95
    }
  ]
}
```

The test can then verify:

```text
Expected water × 1
Detected water × 1
→ MATCH
```

This is more reliable than using a live camera during every test.

---

# 56. AI Benchmarking

Each major model should have a benchmark.

Minimum benchmark categories:

```text
Accuracy
Latency
Throughput
Memory
CPU/GPU Usage
```

Example:

| Model | Accuracy | Latency | FPS | Memory |
|---|---:|---:|---:|---:|
| Detector v1 | TBD | TBD | TBD | TBD |
| Detector v2 | TBD | TBD | TBD | TBD |

No benchmark number should be added until it has been measured.

---

# 57. AI Optimization Workflow

Optimization follows:

```text
Baseline
   ↓
Profile
   ↓
Identify Bottleneck
   ↓
Optimize
   ↓
Benchmark
   ↓
Compare
   ↓
Accept / Reject
```

Example:

```text
Baseline:
120 ms/frame

Optimization:
Lower input resolution

Result:
70 ms/frame

Accuracy:
-2%

Decision:
Evaluate trade-off
```

---

# 58. AI Architecture Trade-offs

## Local AI

Advantages:

```text
Low latency
Privacy
Offline operation
```

Disadvantages:

```text
Hardware requirements
Model deployment complexity
```

---

## Cloud AI

Advantages:

```text
Powerful models
Easy model upgrades
Less local compute
```

Disadvantages:

```text
Latency
Network dependency
API cost
Privacy considerations
```

---

## Hybrid

Advantages:

```text
Low-latency perception
Powerful reasoning
Flexible architecture
```

Disadvantages:

```text
More complex deployment
```

The initial project should use a pragmatic hybrid architecture.

---

# 59. Recommended Initial AI Stack

Initial implementation:

```text
Computer Vision
├── OpenCV
├── YOLO
├── Tracking
└── InsightFace

Gesture
└── MediaPipe

AI Assistant
└── OpenAI API

Speech
├── Speech-to-Text
└── Text-to-Speech
```

These technologies are implementation choices and may be replaced after benchmarking.

---

# 60. AI Repository Structure

Recommended structure:

```text
ai/
│
├── detection/
│   ├── base.py
│   ├── yolo_detector.py
│   └── schemas.py
│
├── tracking/
│   ├── base.py
│   └── tracker.py
│
├── face/
│   ├── detector.py
│   ├── recognizer.py
│   └── embedding.py
│
├── gesture/
│   ├── recognizer.py
│   └── schemas.py
│
├── speech/
│   ├── stt.py
│   └── tts.py
│
├── assistant/
│   ├── agent.py
│   ├── tools.py
│   ├── prompts/
│   └── schemas.py
│
├── inference/
│   ├── pipeline.py
│   ├── scheduler.py
│   └── metrics.py
│
└── config/
    └── models.yaml
```

---

# 61. AI Dependency Direction

Preferred dependency direction:

```text
Application
    ▲
    │
AI Interfaces
    ▲
    │
AI Implementations
    ▲
    │
External Libraries / Providers
```

For example:

```text
VerificationService
       ↓
ObjectDetector
       ↓
YOLODetector
       ↓
YOLO Library
```

The Verification Service should never directly import the YOLO library.

---

# 62. AI Security

AI-related security requirements include:

- API keys stored in environment variables.
- No API keys in source code.
- Input validation.
- Output validation.
- Tool-call authorization.
- Prompt injection resistance.
- Rate limiting where applicable.
- Sensitive data minimization.
- Audit logging.

The LLM must be treated as an untrusted reasoning component rather than a privileged system component.

---

# 63. Prompt Injection Boundary

Customer input must not automatically become a trusted instruction.

Example:

```text
Customer Input
      ↓
Untrusted Data
      ↓
Prompt Construction
      ↓
LLM
```

The AI system must maintain separation between:

```text
System Instructions
Application Context
Customer Input
Tool Results
```

Tool permissions must be enforced outside the LLM.

---

# 64. AI Cost Control

The project should minimize unnecessary external AI calls.

Strategies:

```text
Cache product information
Use local CV
Use deterministic FAQ
Limit context size
Use smaller models where appropriate
Avoid repeated requests
```

Example:

```text
Product Price
```

should come from:

```text
Product Service
```

rather than asking an LLM every time.

---

# 65. AI Data Privacy

The AI architecture should follow data minimization.

Avoid sending unnecessary information to external providers.

For example:

```text
Raw Camera Stream
```

should not automatically be uploaded to a cloud LLM.

Instead:

```text
Camera
 ↓
Local CV
 ↓
Structured Information
 ↓
AI Assistant
```

This reduces privacy exposure.

---

# 66. AI Architecture Evolution

### Version 1

```text
Local CV
+
Cloud LLM
+
Cloud STT/TTS
```

### Version 2

```text
Edge CV
+
Cloud AI Assistant
+
Local TTS
```

### Version 3

```text
Edge AI
+
Cloud AI
+
Hardware Acceleration
```

The architecture should evolve based on measured requirements rather than assumptions.

---

# 67. Definition of Done

The AI architecture is considered complete for this phase when:

```text
☐ AI components identified
☐ CV pipeline defined
☐ Detection abstraction defined
☐ Tracking architecture defined
☐ Face architecture defined
☐ Gesture architecture defined
☐ Speech architecture defined
☐ AI Assistant architecture defined
☐ LLM gateway defined
☐ Tool-calling boundary defined
☐ TTS architecture defined
☐ AI event model defined
☐ AI/application boundary defined
☐ Mock strategy defined
☐ Benchmark strategy defined
☐ AI security boundary defined
☐ AI cost strategy defined
☐ AI privacy strategy defined
```

---

# 68. Final AI Architecture

The complete AI architecture can be summarized as:

```text
                         CUSTOMER
                            │
              ┌─────────────┴─────────────┐
              │                           │
              ▼                           ▼
           CAMERA                    MICROPHONE
              │                           │
              ▼                           ▼
       ┌───────────────┐           ┌──────────────┐
       │ COMPUTER      │           │ SPEECH       │
       │ VISION        │           │ PIPELINE     │
       ├───────────────┤           ├──────────────┤
       │ Detection     │           │ STT          │
       │ Tracking      │           │ TTS          │
       │ Face          │           └──────┬───────┘
       │ Gesture       │                  │
       └───────┬───────┘                  │
               │                          │
               └────────────┬─────────────┘
                            ▼
                   ┌──────────────────┐
                   │  AI ORCHESTRATOR │
                   └────────┬─────────┘
                            │
              ┌─────────────┼─────────────┐
              │             │             │
              ▼             ▼             ▼
          CV Events    AI Assistant    AI Events
                            │
                            ▼
                       AI Gateway
                            │
                            ▼
                       LLM Provider
                            │
                            ▼
                    Validated Response
                            │
                            ▼
                  Application Services
                            │
                            ▼
                     State Machine
                            │
                            ▼
                     Kiosk Actions
```

The fundamental AI engineering loop is:

```text
PERCEIVE
   ↓
UNDERSTAND
   ↓
GENERATE SIGNAL
   ↓
APPLICATION DECISION
   ↓
ACT
   ↓
OBSERVE RESULT
   ↓
EVALUATE
   ↓
IMPROVE
```

This architecture allows the RoShopSmart AI Retail Kiosk to demonstrate not only the use of AI models, but the complete engineering process of integrating **Computer Vision, LLMs, Speech AI, application logic, and smart-machine control** into a single product.