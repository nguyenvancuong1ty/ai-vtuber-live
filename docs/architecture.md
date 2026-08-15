# AI VTuber LIVE Vietnam — Architecture

## 1. Overview

Event-driven AI VTuber runtime for Vietnamese TikTok LIVE, designed for a single workstation.

Primary flow:

TikTok LIVE → Event Gateway → Event Normalizer → Redis Streams → Comment/Gift Processor → Behavior Engine → TTS / Avatar Actions → Avatar Runtime → OBS → TikTok LIVE

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
      Behavior Engine
      personality
      context
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
Receive external events, normalize, validate, deduplicate, and publish to Redis. It must not call LLM, generate TTS, or control the avatar.

### Comment Processor
Normalize text, detect language, moderate, classify intent, score priority, detect duplicates, and decide response eligibility.

### Gift Processor
Normalize gifts, calculate tier, aggregate repeated gifts, map gifts to actions, and assign priority.

### Behavior Engine
Maintain character state, select events, generate responses, determine emotion/gesture, manage interruptions, and idle behavior. It outputs abstract commands.

### TTS Service
Convert text to audio and return duration and phoneme/viseme timing when supported. Expose a provider-independent API.

### Avatar Runtime
Recommended MVP: Unity + VRM. Alternative: Live2D. Responsibilities: realtime rendering, lip-sync, expressions, gaze, blinking, head motion, breathing, gestures, and idle animation.

### Compositor
OBS is responsible for final composition: background, avatar, chat, gift overlays, alerts, music, and UI.

## 4. Data Flow

Comment:

```text
TikTok → Gateway → event.comment → Comment Processor → behavior.input → Behavior Engine → tts.input → TTS → tts.output → Avatar Runtime → OBS
```

Gift:

```text
TikTok → Gateway → event.gift → Gift Processor → behavior.input → Behavior Engine → Avatar Action → Avatar Runtime → OBS
```

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
```

Keep hot state in memory/Redis. Persistent records belong in SQLite initially, with PostgreSQL as a later replacement.

## 6. Latency Targets

Targets are measured, not assumed:

```text
event ingestion        < 300 ms
event processing       < 300 ms
LLM first response     < 1500 ms
TTS first audio        < 1000 ms
avatar start           < 300 ms
```

Target end-to-end comment-to-avatar-start latency: 1–3 seconds depending on external provider latency.

## 7. Persistence

Initial tables:

```text
viewer
interaction
gift
live_session
character_memory
analytics_event
```

## 8. Configuration

Use environment variables for provider/model/Redis/database/avatar/OBS configuration. Never commit secrets.

## 9. Security

Never store TikTok cookies, access tokens, API keys, or session credentials in source code or logs.

## 10. Scalability

MVP is single-machine. Future separation may include Gateway, Behavior Server, TTS Server, Avatar GPU Server, and OBS/Streaming Server. Do not implement distributed infrastructure until a real bottleneck exists.