# AI VTuber LIVE Vietnam — Roadmap

## Phase 0 — Repository Bootstrap

Goal: project skeleton and shared contracts.

Tasks:
- repository structure
- configuration
- logging
- test framework
- Redis connection
- shared event schema
- event simulator skeleton

Acceptance:
- repository builds
- tests run
- Redis connection works
- event schema validates
- simulator emits basic events
- no secrets are committed

## Phase 1 — Event Gateway

Goal: reliable event ingestion.

Tasks:
- TikTok connector interface and implementation
- normalization
- validation
- deduplication
- Redis publishing
- reconnect logic

Acceptance:
- simulated events enter gateway
- normalized events reach Redis
- duplicates are rejected
- malformed events are rejected
- reconnect works
- structured logs exist
- downstream services do not depend directly on the connector

## Phase 2 — Event Simulator + Queue

Goal: development without TikTok LIVE.

Scenarios:

```text
normal_chat
high_chat
gift_burst
mixed_live
duplicate_events
malformed_events
idle_live
```

Acceptance:
- supports at least 40 comments/min and 100 comments/min
- supports gift bursts
- handles duplicate/malformed events without crashing
- exposes queue depth, processed, failed, retry and latency metrics

## Phase 3 — Comment Processor

Goal: turn raw comments into useful behavior candidates.

Tasks:
- nickname normalization
- language detection
- moderation
- spam filtering
- intent classification
- priority scoring
- duplicate detection
- response eligibility

Acceptance:
- candidate selection works under burst traffic
- does not send every comment to the LLM
- tests cover spam, duplicate, compliment, question, greeting, irrelevant and unsafe cases

## Phase 4 — Behavior Engine

Goal: character brain.

Tasks:
- state machine
- personality configuration
- viewer context
- comment selection
- response generation
- emotion and gesture selection
- interruption
- cooldown
- idle behavior
- repetition control

Acceptance:
- selected comments receive responses
- comments can be ignored
- gifts can interrupt
- previous state can be restored
- structured output is validated
- personality is configurable
- viewer context works
- repetition is reduced
- idle behavior works
- LLM fallback works

## Phase 5 — TTS

Goal: synchronized Vietnamese speech.

Tasks:
- provider interface
- provider implementation
- audio normalization
- duration detection
- phoneme/viseme timing
- cache
- playback queue

Acceptance:
- BehaviorCommand produces audio, duration and timing data
- target latency is measurable
- provider can be replaced without modifying Behavior Engine

## Phase 6 — Avatar Runtime

Goal: persistent realtime VTuber character.

Tasks:
- Unity project
- VRM loading
- expression system
- lip-sync
- blink
- gaze
- head movement
- breathing
- gestures
- state inspection
- audio synchronization

Acceptance:
- 30 FPS minimum
- stable for at least 30 minutes
- no identity drift or animation corruption
- speech, lip-sync, blink, gaze, emotion, gesture and idle all work

## Phase 7 — Gift Engine

Goal: meaningful gift reactions.

Tasks:
- gift tier configuration
- aggregation
- priority
- reaction mapping
- cooldown
- combo detection
- milestone system

Acceptance:

```text
small gift  -> short reaction
medium gift -> stronger reaction
large gift  -> interrupt + special reaction
combo       -> unlock/milestone event
```

Duplicate gift events must not cause duplicate reactions.

## Phase 8 — OBS Integration

Goal: final livestream output.

Tasks:
- Unity output
- OBS scene
- avatar layer
- background
- chat overlay
- gift overlay
- alerts
- audio mixing
- emergency stop

Acceptance:
- 1920x1080 output
- stable 30 FPS minimum
- avatar and audio remain synchronized
- emergency stop stops active speech/reactions

## Phase 9 — End-to-End Runtime

Goal: connect all modules.

```text
TikTok -> Gateway -> Redis -> Processor -> Behavior -> TTS -> Avatar -> OBS
```

Acceptance:
- simulated or real comment triggers response, TTS, lip-sync, expression and gesture
- target comment-to-avatar-start latency is 1-3 seconds depending on external provider latency
- failures degrade safely

## Phase 10 — Long-Running Stability

Run 1h, 2h and 4h tests.

Monitor:

```text
FPS
GPU
VRAM
RAM
queue depth
LLM latency
TTS latency
audio latency
event rate
errors
```

Acceptance:
- no unusable memory growth
- no queue deadlock
- no duplicate gift reactions
- no uncontrolled speech loop
- no persistent avatar freeze
- no sustained audio desynchronization

## Phase 11 — Analytics

Track viewers, comments/min, gifts/min, gift value, unique viewers, repeat viewers, response latency, response rate, average session duration, and character behavior metrics.

Acceptance:
- every live session produces an analytics record
- core metrics are queryable

## Phase 12 — Optimization

Only optimize after real data exists:

```text
personality
comment selection
response timing
gift reactions
idle behavior
animation
TTS
```

Do not optimize based only on assumptions.

## Final MVP Definition

```text
TikTok LIVE
  ↓
Comment/Gift
  ↓
Event Gateway
  ↓
Redis
  ↓
Behavior Engine
  ↓
Vietnamese TTS
  ↓
VRM Avatar
  ↓
Lip-sync + Emotion + Gesture
  ↓
OBS
  ↓
LIVE
```

The MVP is complete when this path works continuously for several hours under simulated and real event load.

## Post-MVP

Possible future work: multiple characters, multiple voices, long-term viewer memory, relationship/progression, outfit system, mini games, dance system, special-event animations, multilingual support, multi-machine deployment, analytics dashboard, and A/B testing.