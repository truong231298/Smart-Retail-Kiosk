# RoShopSmart AI Retail Kiosk — Benchmark Strategy

**Document:** `10-benchmark-strategy.md`  
**Version:** `1.0`  
**Status:** Draft / Initial  
**Project:** `roshopsmart-ai-retail-kiosk`

---

# 1. Purpose

This document defines the benchmarking strategy for the RoShopSmart AI Retail Kiosk.

The purpose of benchmarking is to answer:

> **How well does the system perform, under what conditions, and compared with which baseline?**

The project must not make performance claims such as:

```text
"Very fast"
"Highly accurate"
"Real-time"
"Production-ready"
```

without measurable evidence.

Instead, performance should be expressed using quantitative metrics:

```text
Accuracy
Precision
Recall
F1
mAP
FPS
Latency
p50
p95
p99
Throughput
CPU
GPU
Memory
MQTT latency
API latency
Recovery time
Success rate
```

---

# 2. Benchmark Philosophy

The benchmark process follows:

```text
Baseline
   ↓
Measure
   ↓
Analyze
   ↓
Optimize
   ↓
Measure Again
   ↓
Compare
   ↓
Document
```

The most important principle is:

> Never claim an optimization without comparing it against a measured baseline.

---

# 3. Benchmark Scope

Benchmarking covers six major areas:

```text
1. Computer Vision
2. AI / LLM
3. Backend
4. Database
5. MQTT / IoT
6. End-to-End Smart Machine
```

Additional measurements:

```text
Resource Usage
Reliability
Recovery
Scalability
```

---

# 4. Benchmark Categories

| Category | Main Metrics |
|---|---|
| CV Detection | mAP, precision, recall, FPS |
| CV Tracking | ID stability, FPS, latency |
| Face Recognition | FAR, FRR, accuracy, latency |
| Gesture Recognition | accuracy, latency |
| AI Assistant | latency, tool accuracy, groundedness |
| API | p50, p95, p99 |
| Database | query latency, throughput |
| MQTT | round-trip latency, delivery success |
| Device | command latency, response time |
| E2E | total transaction latency |
| Reliability | recovery time, failure rate |
| Resources | CPU, GPU, RAM |

---

# 5. Benchmark Environment

Every benchmark must record its environment.

Minimum information:

```text
Operating System
CPU
GPU
RAM
Python version
Model version
Backend version
Database version
MQTT broker version
Docker version
Input resolution
Dataset version
Configuration
```

Example:

```text
OS:
Windows 11

CPU:
Intel XXXXX

GPU:
NVIDIA XXXXX

RAM:
16 GB

Python:
3.x

Model:
YOLO-product-v1

Input:
640 × 640
```

---

# 6. Reproducibility

A benchmark should be reproducible.

Record:

```text
Code commit
Model version
Dataset version
Configuration
Hardware
Software versions
Random seed
Benchmark duration
```

Example:

```text
Commit:
a81f92c

Model:
yolo-product-v1

Dataset:
product-test-v1

Input:
640×640

Confidence:
0.5
```

---

# 7. Benchmark Baseline

Before optimization, establish a baseline.

Example:

```text
Baseline

CV inference:
120 ms

API p95:
180 ms

MQTT round trip:
45 ms

E2E verification:
280 ms
```

These numbers are illustrative only.

Actual values must be measured.

---

# 8. Computer Vision Benchmark

Computer Vision is one of the most important benchmark areas.

The primary questions are:

```text
How accurate is detection?

How fast is inference?

How stable is detection?

How does performance change under difficult conditions?
```

---

# 9. Object Detection Metrics

For product detection measure:

```text
Precision
Recall
F1 Score
mAP50
mAP50-95
```

Definitions:

```text
Precision =
TP / (TP + FP)
```

```text
Recall =
TP / (TP + FN)
```

```text
F1 =
2 × Precision × Recall
/
(Precision + Recall)
```

---

# 10. Product Detection Benchmark

Example benchmark table:

| Model | Input | Precision | Recall | mAP50 | mAP50-95 |
|---|---:|---:|---:|---:|---:|
| Model A | 640 | XX | XX | XX | XX |
| Model B | 640 | XX | XX | XX | XX |

The actual values must be generated from the project's evaluation dataset.

---

# 11. FPS Benchmark

Measure:

```text
FPS =
Processed Frames / Time
```

Example:

```text
1000 frames
/
20 seconds
=
50 FPS
```

Measure separately:

```text
CPU FPS
GPU FPS
```

if both environments are available.

---

# 12. Inference Latency

FPS alone is not sufficient.

Measure:

```text
Preprocessing
Inference
Postprocessing
Total
```

Example:

| Stage | Latency |
|---|---:|
| Preprocess | XX ms |
| Inference | XX ms |
| Postprocess | XX ms |
| Total | XX ms |

---

# 13. Percentile Latency

Average latency can hide spikes.

Measure:

```text
p50
p90
p95
p99
```

Example:

```text
p50 = 35 ms
p95 = 60 ms
p99 = 110 ms
```

For interactive systems, p95/p99 can be more informative than average latency.

---

# 14. Batch vs Single Inference

If applicable, compare:

```text
Batch = 1
Batch = 2
Batch = 4
Batch = 8
```

Measure:

```text
Latency
Throughput
Memory
```

For a kiosk camera pipeline, batch size 1 will likely be the primary real-time configuration.

---

# 15. Input Resolution Benchmark

Compare:

```text
320 × 320
640 × 640
960 × 960
```

Measure:

```text
Accuracy
FPS
Latency
Memory
```

The objective is to find the best accuracy/performance trade-off.

---

# 16. Confidence Threshold Benchmark

Test:

```text
0.30
0.40
0.50
0.60
0.70
0.80
```

Compare:

```text
Precision
Recall
False positives
False negatives
Verification accuracy
```

The final threshold must be evidence-based.

---

# 17. Product Counting Benchmark

The kiosk needs quantity verification.

Example:

```text
Expected:
Coke × 3

Detected:
Coke × 2
```

Measure:

```text
Count Accuracy
Under-count Rate
Over-count Rate
Exact Count Accuracy
```

---

# 18. Exact Product Verification Accuracy

Define:

```text
Exact Verification Accuracy
=
Correctly verified transactions
/
Total verification attempts
```

Example:

```text
950 correct
/
1000 attempts

= 95%
```

This metric is highly relevant because it maps directly to the business workflow.

---

# 19. Difficult-Condition Benchmark

Evaluate under:

```text
Normal lighting
Low light
Backlight
Occlusion
Multiple products
Product overlap
Fast movement
Different camera distances
Different camera angles
```

Example:

| Condition | Accuracy | FPS |
|---|---:|---:|
| Normal | XX | XX |
| Low Light | XX | XX |
| Occlusion | XX | XX |
| Multiple Products | XX | XX |

---

# 20. Temporal Stability Benchmark

Measure whether detections remain stable across frames.

Example:

```text
Frame 1 → Coke
Frame 2 → Coke
Frame 3 → Coke
Frame 4 → Unknown
Frame 5 → Coke
```

Metrics:

```text
Detection Stability
Frame-to-frame consistency
Track continuity
False state transitions
```

---

# 21. Object Tracking Benchmark

If tracking is implemented, measure:

```text
Track persistence
ID switches
Tracking FPS
Tracking latency
Lost track rate
```

Possible metrics:

```text
IDF1
MOTA
ID switches
```

depending on the tracking implementation.

---

# 22. Face Recognition Benchmark

Measure:

```text
Recognition Accuracy
False Acceptance Rate
False Rejection Rate
Latency
```

Test:

```text
Different lighting
Different angles
Different distances
Multiple people
Partial occlusion
```

---

# 23. Face Recognition Trade-off

The system should analyze the relationship:

```text
Security
     ↕
Recognition Convenience
```

Increasing the recognition threshold may reduce false acceptance but increase false rejection.

The selected threshold must be documented.

---

# 24. Gesture Recognition Benchmark

Measure:

```text
Accuracy
Precision
Recall
Latency
False activation rate
```

Test:

```text
Valid gesture
Invalid gesture
No hand
Multiple hands
Fast movement
Partial visibility
```

---

# 25. AI Assistant Benchmark

The AI assistant should not be evaluated only by response speed.

Measure:

```text
Latency
Tool-call accuracy
Groundedness
Task completion
Hallucination rate
Refusal correctness
```

---

# 26. AI Latency

Measure:

```text
User input
   ↓
Speech recognition
   ↓
LLM
   ↓
Tool calls
   ↓
Response generation
   ↓
TTS
```

Record:

```text
Speech recognition latency
LLM latency
Tool latency
TTS latency
Total latency
```

---

# 27. AI Tool Accuracy

Example:

```text
User:
"What is the price of Coca Cola?"
```

Expected tool:

```text
ProductService.get_product()
```

Measure:

```text
Correct tool selection
Correct parameters
Correct result usage
```

---

# 28. AI Groundedness Benchmark

Test questions whose answers exist in the database.

Example:

```text
Product:
Coca Cola
Price:
10,000 VND
```

Expected:

```text
AI returns:
10,000 VND
```

Measure:

```text
Grounded Answer Rate
```

---

# 29. AI Hallucination Benchmark

Create questions containing unavailable information.

Example:

```text
"What is the price of Pepsi?"
```

when Pepsi does not exist in the database.

Expected:

```text
The assistant states that the product is unavailable or unknown.
```

Measure:

```text
Hallucination Rate
```

---

# 30. AI Safety Benchmark

Test:

```text
Prompt injection
Tool abuse
Unauthorized device command
Unauthorized price modification
Database manipulation
Credential extraction
```

Measure:

```text
Attack rejection rate
Unsafe action rate
```

---

# 31. Backend Benchmark

Important APIs:

```text
POST /sessions
GET /products
POST /cart/items
POST /verification
POST /checkout
POST /payment
GET /transactions
```

Measure:

```text
Requests/sec
p50
p95
p99
Error rate
CPU
Memory
```

---

# 32. API Latency Benchmark

Example:

| Endpoint | p50 | p95 | p99 |
|---|---:|---:|---:|
| Create Session | XX ms | XX ms | XX ms |
| Get Products | XX ms | XX ms | XX ms |
| Add Cart Item | XX ms | XX ms | XX ms |
| Verification | XX ms | XX ms | XX ms |
| Checkout | XX ms | XX ms | XX ms |

---

# 33. Backend Throughput

Measure:

```text
Requests per second
```

under increasing concurrency:

```text
1
5
10
25
50
100
```

Record:

```text
Latency
Error rate
CPU
Memory
```

---

# 34. Database Benchmark

Important queries:

```text
Product lookup
Product search
Inventory lookup
Create session
Create transaction
Transaction history
Verification event insertion
```

Measure:

```text
Query latency
Rows scanned
Index usage
Connection pool utilization
```

---

# 35. Database Optimization Benchmark

Before optimization:

```text
Query:
XX ms
```

After adding an appropriate index:

```text
Query:
XX ms
```

Document:

```text
Before
Change
After
Improvement
Trade-offs
```

---

# 36. MQTT Benchmark

MQTT is part of the Smart Machine control loop.

Measure:

```text
Publish latency
Round-trip latency
Message delivery success
Reconnect time
Throughput
```

---

# 37. MQTT Round-Trip

Measure:

```text
Backend
 ↓
Publish Command
 ↓
MQTT Broker
 ↓
Simulator
 ↓
Execute
 ↓
Status Event
 ↓
MQTT
 ↓
Backend
```

Metric:

```text
Round Trip Time
```

---

# 38. MQTT Reliability Benchmark

Run:

```text
1000 commands
```

Measure:

```text
Successful commands
Failed commands
Duplicate commands
Timeouts
Lost messages
```

Example:

```text
Success:
99.8%

Timeout:
0.1%

Failure:
0.1%
```

Actual results must be measured.

---

# 39. Device Command Latency

Measure:

```text
Command issued
      ↓
Device receives
      ↓
Device executes
      ↓
Device reports status
```

Record:

```text
Command latency
Execution latency
Status latency
Total latency
```

---

# 40. Simulator Benchmark

Measure simulator performance:

```text
CPU
Memory
Browser FPS
MQTT latency
UI response time
```

The simulator should not become the bottleneck.

---

# 41. Device State Synchronization

Measure:

```text
Desired State
     vs
Reported State
```

Example:

```text
Desired:
SERVO = OPEN

Reported:
SERVO = OPEN
```

Metric:

```text
State convergence time
```

---

# 42. End-to-End Benchmark

The most important benchmark measures the complete business workflow.

```text
Customer detected
       ↓
Product detected
       ↓
Verification
       ↓
Checkout
       ↓
Payment
       ↓
Device command
       ↓
Device action
       ↓
Transaction completed
```

---

# 43. E2E Latency

Define:

```text
T_start =
customer interaction begins

T_end =
device action successfully confirmed
```

Then:

```text
E2E Latency =
T_end - T_start
```

Measure:

```text
p50
p95
p99
```

---

# 44. E2E Transaction Success Rate

Define:

```text
Transaction Success Rate =
Successful completed transactions
/
Valid transaction attempts
```

This is one of the most meaningful product-level metrics.

---

# 45. Suspicious Transaction Detection Benchmark

Measure:

```text
True Positive
False Positive
False Negative
True Negative
```

From these calculate:

```text
Precision
Recall
F1
```

The goal is not simply:

```text
"Detect more suspicious transactions."
```

The system must balance:

```text
Fraud detection
vs
Customer friction
```

---

# 46. False Positive Cost

A false positive can cause:

```text
Customer interruption
Manual verification
Transaction delay
Poor user experience
```

Therefore measure:

```text
False Positive Rate
```

and document its business impact.

---

# 47. False Negative Cost

A false negative may allow:

```text
Incorrect product verification
Revenue loss
Inventory mismatch
Potential fraud
```

Therefore the system should evaluate both:

```text
False Positive
False Negative
```

rather than optimizing only one.

---

# 48. Reliability Benchmark

Measure:

```text
System uptime
Failure rate
Recovery time
Session recovery
Device recovery
MQTT reconnect time
```

---

# 49. Recovery Time

Example:

```text
MQTT disconnect
      ↓
Failure detected
      ↓
Reconnect
      ↓
Device state synchronized
```

Measure:

```text
Recovery Time
```

---

# 50. Fault Injection Benchmark

Run controlled failures:

```text
MQTT unavailable
Database unavailable
AI timeout
Camera failure
Device failure
Simulator restart
Backend restart
```

For each scenario record:

```text
Failure detected?
Safe state?
Data consistency?
Recovery successful?
Recovery time?
```

---

# 51. Resource Benchmark

Monitor:

```text
CPU
RAM
GPU
VRAM
Disk
Network
```

during:

```text
Idle
CV inference
AI interaction
Checkout
MQTT traffic
Full E2E transaction
```

---

# 52. Resource Profile

Example:

| Scenario | CPU | RAM | GPU | VRAM |
|---|---:|---:|---:|---:|
| Idle | XX% | XX MB | XX% | XX MB |
| CV | XX% | XX MB | XX% | XX MB |
| AI | XX% | XX MB | XX% | XX MB |
| Full Flow | XX% | XX MB | XX% | XX MB |

---

# 53. Edge Resource Constraints

The kiosk may eventually run on limited hardware.

Therefore evaluate:

```text
CPU-only
GPU-enabled
Low-memory environment
```

Possible optimization techniques:

```text
Model quantization
Lower resolution
Frame skipping
Model selection
Inference scheduling
Caching
```

---

# 54. Accuracy vs Latency Trade-off

A major engineering decision is:

```text
Higher Accuracy
      ↕
Lower Latency
```

Example:

| Configuration | mAP | FPS |
|---|---:|---:|
| Model A | XX | XX |
| Model B | XX | XX |
| Quantized Model | XX | XX |

Choose based on the actual product requirement.

---

# 55. Model Optimization Benchmark

If optimization is applied:

```text
Baseline Model
      ↓
Optimization
      ↓
Optimized Model
```

Compare:

```text
Accuracy
Latency
FPS
Memory
Model Size
```

---

# 56. Quantization Benchmark

If supported:

```text
FP32
FP16
INT8
```

Measure:

```text
Accuracy difference
Latency
Memory
Throughput
```

Do not assume quantization is automatically better.

---

# 57. Frame Skipping Benchmark

For real-time CV:

```text
Process every frame
```

versus:

```text
Process every 2nd frame
```

versus:

```text
Process every 3rd frame
```

Compare:

```text
CPU/GPU usage
FPS
Detection stability
Verification accuracy
```

---

# 58. Caching Benchmark

Potential caching areas:

```text
Product information
Product embeddings
AI responses
Static configuration
```

Measure:

```text
Before cache
After cache
```

and verify that stale data does not compromise transaction correctness.

---

# 59. Benchmark Comparison Format

Every optimization experiment should use:

```text
Experiment
Baseline
Change
Result
Improvement
Trade-off
Decision
```

Example:

```text
Experiment:
YOLO input resolution

Baseline:
960×960

Change:
640×640

Result:
Latency ↓ XX%
mAP ↓ XX%

Decision:
Use 640×640
```

---

# 60. Statistical Reliability

A benchmark should not rely on a single measurement.

For latency:

```text
Run:
N ≥ 100
```

where practical.

For model evaluation:

```text
Use a representative test dataset.
```

For load testing:

```text
Run multiple trials.
```

---

# 61. Warm-up

Performance benchmarks should account for warm-up.

Example:

```text
Model initialization
GPU initialization
Database connection pool
MQTT connection
```

Discard initial warm-up measurements where appropriate.

---

# 62. Benchmark Isolation

Do not benchmark while unrelated workloads are consuming resources.

Record:

```text
Background processes
Docker containers
GPU processes
Network conditions
```

when relevant.

---

# 63. Benchmark Automation

Benchmarks should eventually be executable using scripts.

Example:

```text
benchmarks/
├── cv/
├── backend/
├── database/
├── mqtt/
├── e2e/
└── reports/
```

---

# 64. Benchmark Output

A benchmark should produce structured output.

Example:

```json
{
  "benchmark": "product_detection",
  "model": "yolo-product-v1",
  "dataset": "product-test-v1",
  "input_size": "640x640",
  "precision": 0.94,
  "recall": 0.91,
  "map50": 0.93,
  "fps": 42.7,
  "latency_ms": {
    "p50": 22,
    "p95": 29,
    "p99": 35
  }
}
```

---

# 65. Benchmark Report

Each major benchmark should generate a report containing:

```text
Benchmark Name
Date
Commit
Environment
Configuration
Dataset
Baseline
Result
Comparison
Conclusion
```

---

# 66. Benchmark Versioning

Benchmark results should be associated with:

```text
Git Commit
Model Version
Dataset Version
Software Version
```

Example:

```text
Benchmark:
BENCH-2026-001

Commit:
a81f92c

Model:
yolo-product-v1

Dataset:
product-test-v1
```

---

# 67. Benchmark Regression

Performance can regress even when functionality still works.

Example:

```text
Previous:
FPS = 45

Current:
FPS = 28
```

Tests may still pass.

Therefore performance regression testing is required.

---

# 68. Regression Thresholds

Define acceptable degradation.

Example:

```text
API latency:
maximum +20%

CV FPS:
maximum -10%

mAP:
maximum -2%

Memory:
maximum +15%
```

These are initial policy examples and should be adjusted based on measured requirements.

---

# 69. Benchmark Gates

A build may fail if:

```text
Critical accuracy drops below threshold.

Latency exceeds threshold.

Security benchmark fails.

Transaction success rate drops.

Device command failure rate exceeds threshold.
```

---

# 70. Product-Level KPIs

The project should eventually report:

```text
Customer Detection Latency
Product Verification Accuracy
Suspicious Transaction Detection F1
Checkout Latency
Payment Success Rate
Device Command Success Rate
E2E Transaction Success Rate
```

These metrics connect engineering performance to product behavior.

---

# 71. Engineering Metrics vs Product Metrics

Not every metric has equal product meaning.

### Engineering

```text
FPS
CPU
RAM
MQTT latency
API latency
```

### Product

```text
Verification accuracy
Transaction success
False positive rate
Customer interaction latency
Recovery success
```

The project should report both.

---

# 72. Benchmark Dashboard

A future dashboard may display:

```text
┌──────────────────────────────────────────────┐
│          ROSHOPSMART BENCHMARK               │
├──────────────────────────────────────────────┤
│ Product mAP50                 XX%            │
│ Product Verification          XX%            │
│ CV FPS                        XX             │
│ CV p95                        XX ms          │
│ API p95                       XX ms          │
│ MQTT RTT p95                  XX ms          │
│ E2E p95                       XX ms          │
│ Transaction Success           XX%            │
│ Device Success                XX%            │
│ System Recovery               XX ms          │
└──────────────────────────────────────────────┘
```

---

# 73. Recommended Initial Benchmark Targets

Targets should be treated as engineering goals, not fabricated achievements.

Initial targets:

```text
Product detection:
mAP50 ≥ 0.85

Product verification:
≥ 95% on controlled test dataset

CV pipeline:
≥ 20 FPS on target hardware

API:
p95 < 300 ms for normal operations

MQTT:
p95 round-trip < 100 ms on local network

Device command:
≥ 99% successful commands

E2E:
≥ 95% successful valid transactions
```

These values must be validated against the actual hardware and requirements.

---

# 74. Benchmark Priorities by Phase

## Phase 1

Focus:

```text
API latency
Database latency
State transition latency
E2E transaction latency
```

---

## Phase 2

Focus:

```text
Detection accuracy
Detection FPS
Inference latency
Face recognition
Gesture recognition
AI latency
AI groundedness
```

---

## Phase 3

Focus:

```text
MQTT latency
Device command success
Simulator response
State synchronization
Fault recovery
```

---

## Phase 4

Focus:

```text
Full E2E latency
Load
Reliability
Resource usage
Security
Regression
```

---

# 75. Benchmark-to-Optimization Workflow

When performance is poor:

```text
Observe
 ↓
Measure
 ↓
Profile
 ↓
Identify Bottleneck
 ↓
Change One Variable
 ↓
Benchmark
 ↓
Compare
```

Do not optimize blindly.

---

# 76. Bottleneck Categories

Possible bottlenecks:

```text
CPU
GPU
Memory
Database
Network
MQTT
AI API
Model inference
Serialization
Frontend rendering
```

The benchmark should identify the actual bottleneck.

---

# 77. Example Optimization

Suppose:

```text
CV latency = 150 ms
```

Profiling shows:

```text
Preprocessing = 20 ms
Inference = 100 ms
Postprocessing = 30 ms
```

Optimization should focus first on inference.

Potential experiments:

```text
Smaller model
FP16
INT8
Lower resolution
Hardware acceleration
```

Then benchmark again.

---

# 78. Benchmark Documentation Rule

Never write:

```text
"Model B is faster."
```

Instead write:

```text
"Model B reduced p95 inference latency
from 74 ms to 51 ms (-31.1%)
on the same hardware, dataset,
input resolution and software environment."
```

This demonstrates engineering rigor.

---

# 79. Benchmark Evidence

Portfolio evidence should include:

```text
Benchmark tables
Graphs
Raw results
Configuration
Screenshots
Performance videos
Commit references
```

Recommended directory:

```text
docs/
└── evidence/
    └── benchmarks/
        ├── cv/
        ├── ai/
        ├── backend/
        ├── mqtt/
        └── e2e/
```

---

# 80. Benchmark Visualization

Useful charts include:

```text
Latency distribution
FPS comparison
Accuracy vs threshold
Accuracy vs latency
Model comparison
API p95 over load
MQTT latency
Resource utilization
```

The purpose of visualization is to make engineering trade-offs obvious.

---

# 81. Final Benchmark Matrix

| System | Primary Metric | Secondary Metrics |
|---|---|---|
| Product Detection | mAP50 | FPS, latency |
| Product Verification | Exact Accuracy | Precision, Recall |
| Face Recognition | FAR/FRR | Accuracy, latency |
| Gesture | Accuracy | Latency |
| AI Assistant | Groundedness | Latency, tool accuracy |
| API | p95 | Throughput, errors |
| Database | Query latency | CPU, connections |
| MQTT | RTT p95 | Reliability |
| Device | Command success | Response latency |
| E2E | Transaction success | p95 latency |
| System | Reliability | Recovery time |

---

# 82. Definition of Done

The benchmark strategy is complete when:

```text
☐ Benchmark environment defined
☐ Reproducibility defined
☐ CV metrics defined
☐ AI metrics defined
☐ Backend metrics defined
☐ Database metrics defined
☐ MQTT metrics defined
☐ Device metrics defined
☐ E2E metrics defined
☐ Resource metrics defined
☐ Reliability metrics defined
☐ Baseline strategy defined
☐ Optimization methodology defined
☐ Regression thresholds defined
☐ Benchmark evidence structure defined
☐ Phase benchmark gates defined
```

---

# 83. Final Engineering Principle

The benchmark philosophy of RoShopSmart is:

```text
DON'T SAY:
"It works."

SHOW:

Accuracy
+
Latency
+
FPS
+
Reliability
+
Resource Usage
+
Failure Recovery
+
Business Success Rate
```

The objective is not to produce impressive numbers in isolation.

The objective is to demonstrate:

> **A measurable engineering improvement under a controlled and reproducible environment.**

That is the standard required to transform a prototype into a credible **AI Product Engineering & Smart Machine portfolio project**.