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
- viewer/memory domain interfaces
- initial persistence schema for viewer memory

Acceptance:
- repository builds
- tests run
- Redis connection works
- event schema validates
- simulator emits basic events
- viewer and memory models validate
- repository interfaces are defined
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
- stable viewer identity extraction

Acceptance:
- simulated events enter gateway
- normalized events reach Redis
- duplicates are rejected
- malformed events are rejected
- reconnect works
- structured logs exist
- downstream services do not depend directly on the connector
- stable platform user ID is preserved for memory resolution

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
returning_viewer
```

Acceptance:
- supports at least 40 comments/min and 100 comments/min
- supports gift bursts
- handles duplicate/malformed events without crashing
- exposes queue depth, processed, failed, retry and latency metrics
- can simulate the same viewer returning across multiple sessions
- simulator can test duplicate events against memory counters

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
- interaction memory recording

Acceptance:
- candidate selection works under burst traffic
- does not send every comment to the LLM
- tests cover spam, duplicate, compliment, question, greeting, irrelevant and unsafe cases
- meaningful interactions are persisted against the stable viewer ID
- duplicate comments do not double-count viewer interactions

## Phase 4 — Viewer Memory

Goal: make persistent viewer memory available before the Behavior Engine becomes fully dependent on it.

Tasks:
- viewer repository
- viewer memory repository
- viewer gift repository
- stable platform user ID resolution
- first-seen / last-seen tracking
- interaction counters
- gift counters and total gift value
- bounded recent interaction context
- favorite-topic context
- memory summary field
- retention configuration
- memory deletion API/domain operation
- idempotent event updates

Acceptance:
- first interaction creates a viewer record
- returning viewer resolves to the same record across live sessions
- interaction count persists
- gift count persists
- total gift value persists
- last interaction persists
- bounded recent context is retrievable
- gift history is retrievable
- duplicate events do not double-count
- memory can be deleted by platform user ID
- missing memory does not crash the runtime
- tests cover first visit, returning viewer, gift update, duplicate event, cross-session restore, retention and deletion

## Phase 5 — Behavior Engine

Goal: character brain with persistent viewer context.

Tasks:
- state machine
- personality configuration
- viewer context
- viewer memory integration
- comment selection
- response generation
- emotion and gesture selection
- interruption
- cooldown
- idle behavior
- repetition control
- LLM fallback

Acceptance:
- selected comments receive responses
- comments can be ignored
- gifts can interrupt
- previous state can be restored
- structured output is validated
- personality is configurable
- first-time and returning viewers are distinguishable
- relevant viewer memory can influence responses
- the engine does not invent memory facts
- memory failure falls back to empty context
- repetition is reduced
- idle behavior works
- LLM fallback works

## Phase 6 — TTS

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
- viewer names supplied by memory are normalized before TTS

## Phase 7 — Avatar Runtime

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
- avatar can react to both comment behavior and gift behavior

## Phase 8 — Gift Engine

Goal: meaningful gift reactions and persistent gift memory.

Tasks:
- gift tier configuration
- aggregation
- priority
- reaction mapping
- cooldown
- combo detection
- milestone system
- gift history updates

Acceptance:

```text
small gift  -> short reaction
medium gift -> stronger reaction
large gift  -> interrupt + special reaction
combo       -> unlock/milestone event
```

Additional acceptance:
- gift totals persist across sessions
- returning viewers can be recognized through gift history
- duplicate gift events do not cause duplicate reactions or double-counting

## Phase 9 — OBS Integration

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

## Phase 10 — End-to-End Runtime

Goal: connect all modules.

```text
TikTok -> Gateway -> Redis -> Processor -> Memory -> Behavior -> TTS -> Avatar -> OBS
```

Acceptance:
- simulated or real comment triggers viewer resolution, response, TTS, lip-sync, expression and gesture
- returning viewer can receive relevant stored context
- gift updates persistent viewer gift history
- target comment-to-avatar-start latency is 1-3 seconds depending on external provider latency
- memory lookup does not become a blocking bottleneck
- failures degrade safely

## Phase 11 — Long-Running Stability

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
memory lookup latency
errors
```

Acceptance:
- no unusable memory growth
- no queue deadlock
- no duplicate gift reactions
- no uncontrolled speech loop
- no persistent avatar freeze
- no sustained audio desynchronization
- viewer memory remains consistent under sustained event load

## Phase 12 — Analytics

Track viewers, comments/min, gifts/min, gift value, unique viewers, repeat viewers, response latency, response rate, average session duration, and character behavior metrics.

Memory-related metrics:

```text
new viewers
returning viewers
returning viewer rate
interactions per viewer
gifts per viewer
memory lookup latency
memory update failures
```

Acceptance:
- every live session produces an analytics record
- core metrics are queryable
- returning-viewer metrics are available

## Phase 13 — Optimization

Only optimize after real data exists:

```text
personality
comment selection
response timing
gift reactions
idle behavior
animation
TTS
memory retrieval
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
Viewer Memory
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

The MVP is complete when this path works continuously for several hours under simulated and real event load, and a returning viewer can be resolved and receive relevant bounded memory context.

## Post-MVP

Possible future work: multiple characters, multiple voices, richer long-term viewer memory, relationship/progression, outfit system, mini games, dance system, special-event animations, multilingual support, multi-machine deployment, analytics dashboard, and A/B testing.
