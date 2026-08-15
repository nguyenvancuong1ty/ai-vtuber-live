# AI VTuber LIVE Vietnam — Architecture

## 1. Overview

Event-driven AI VTuber runtime for Vietnamese TikTok LIVE, designed for a single workstation.

Primary flow:

TikTok LIVE → Event Gateway → Event Normalizer → Redis Streams → Comment/Gift Processor → Viewer Memory → Behavior Engine → TTS / Avatar Actions → Avatar Runtime → OBS → TikTok LIVE

Viewer Memory is a first-class MVP subsystem. It persists bounded viewer facts and gift history across live sessions and supplies relevant context to the Behavior Engine.

## 2. High-Level Architecture

```text
TikTok LIVE
    │
    ▼
TikTok Connector
    │
    ▼
Event Gateway
(normalize / validate / dedupe)
    │
    ▼
Redis Streams
    ├── Comment Processor
    └── Gift Processor
             │
             ▼
       Viewer Memory
       resolve / update / retrieve
             │
             ▼
      Behavior Engine
      personality
      context
      viewer memory
      priority
      state machine
          │       │
          ▼       ▼
         TTS   Avatar Actions
          │       │
          └──┬────┘
             ▼
       Avatar Runtime
       lip-sync / expression
       gaze / blink / gesture / idle
             │
             ▼
            OBS
             │
             ▼
         TikTok LIVE
```

## 3. Services

### Gateway
Receive external events, normalize, validate, deduplicate, and publish to Redis. It must not call LLM, generate TTS, control the avatar, or implement viewer-memory business logic.

### Comment Processor
Normalize text, detect language, moderate, classify intent, score priority, detect duplicates, and decide response eligibility.

### Gift Processor
Normalize gifts, calculate tier, aggregate repeated gifts, map gifts to actions, assign priority, and emit memory updates.

### Viewer Memory
Persistent subsystem defined in `docs/memory-system.md`. It resolves viewers by stable platform user ID, records bounded interactions, aggregates gift history, maintains lightweight derived context, and exposes memory through a repository/service interface.

Memory must not be coupled to a specific LLM provider.

### Behavior Engine
Maintain character state, select events, retrieve relevant viewer memory, generate responses, determine emotion/gesture, manage interruptions, and idle behavior. It outputs abstract commands and does not query the database directly from prompt-generation code.

### TTS Service
Convert text to audio and return duration and phoneme/viseme timing when supported. Expose a provider-independent API.

### Avatar Runtime
Recommended MVP: Unity + VRM. Alternative: Live2D. Responsibilities: realtime rendering, lip-sync, expressions, gaze, blinking, head motion, breathing, gestures, and idle animation.

### Compositor
OBS is responsible for final composition: background, avatar, chat, gift overlays, alerts, music, and UI.

## 4. Data Flow

Comment:

```text
TikTok → Gateway → event.comment → Comment Processor → Viewer Memory → behavior.input → Behavior Engine → tts.input → TTS → tts.output → Avatar Runtime → OBS
```

Gift:

```text
TikTok → Gateway → event.gift → Gift Processor → Viewer Memory → behavior.input → Behavior Engine → Avatar Action → Avatar Runtime → OBS
```

Viewer memory update should be idempotent and keyed by the stable platform user ID plus event ID where event-level deduplication is required.

## 5. Runtime State

Runtime state includes:

```text
current_state
current_emotion
current_speech
current_gesture
current_viewer
current_priority
last_interaction
idle_timer
viewer_memory_context
```

Keep hot state in memory/Redis. Persistent records belong in SQLite initially, with PostgreSQL as a later replacement.

## 6. Latency Targets

Targets are measured, not assumed:

```text
event ingestion        < 300 ms
event processing       < 300 ms
memory lookup          < 100 ms target
LLM first response     < 1500 ms
TTS first audio        < 1000 ms
avatar start           < 300 ms
```

Target end-to-end comment-to-avatar-start latency: 1–3 seconds depending on external provider latency.

Memory lookup must not become a blocking bottleneck for the live response path. A cached/empty context is acceptable when the memory subsystem is temporarily unavailable.

## 7. Persistence

Initial tables:

```text
viewer
viewer_memory
viewer_gift
interaction
live_session
character_memory
analytics_event
```

Detailed viewer-memory requirements are defined in `docs/memory-system.md`.

## 8. Configuration

Use environment variables for provider/model/Redis/database/avatar/OBS configuration. Memory retention limits must also be configurable. Never commit secrets.

## 9. Security and Privacy

Never store TikTok cookies, access tokens, API keys, or session credentials in source code or logs.

Viewer memory must follow data minimization. Do not store unnecessary personal information. Raw interaction text retention must be bounded and configurable. Provide a deletion path for a viewer's stored memory.

## 10. Scalability

MVP is single-machine. Future separation may include Gateway, Behavior Server, Memory Server, TTS Server, Avatar GPU Server, and OBS/Streaming Server. Do not implement distributed infrastructure until a real bottleneck exists.
