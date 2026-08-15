# AI VTuber LIVE Vietnam — Codex Development Rules

## 1. Project Goal

Build an AI VTuber live-streaming system for Vietnamese TikTok LIVE.

The MVP must support TikTok LIVE event ingestion, comment/gift processing, viewer context, character personality, behavior/state machine, Vietnamese TTS, phoneme/timing data, realtime avatar animation, lip-sync, facial expressions, idle/micro movements, gift reactions, OBS composition, and end-to-end event-to-avatar testing.

Primary principle: stable realtime interaction is more important than photorealistic rendering.

## 2. Architecture Principles

### Event-driven architecture

All external events enter through the Event Gateway.

Required flow:

TikTok → Gateway → Normalized Event → Queue → Processor → Behavior Engine → TTS / Avatar Action → Avatar Runtime → OBS → TikTok LIVE

Never allow application modules to directly depend on the TikTok connector.

### Shared event schema

All internal events must conform to `docs/event-schema.md`. Do not create ad-hoc event formats.

### Behavior Engine owns decisions

The Behavior Engine decides whether to respond, what to say, emotion, gesture, priority, interruption, and idle behavior. It must output abstract commands and must not directly manipulate avatar implementation details.

### Replaceable providers

LLM and TTS providers must be behind interfaces/adapters. Business logic must not depend on a specific provider.

### Independent Avatar Runtime

Avatar Runtime must not depend directly on TikTok, Redis, LLM, or a specific TTS provider. It consumes normalized avatar commands.

### Priority

1. Emergency/system safety
2. High-value gift reactions
3. Current speech
4. Important comments
5. Normal comments
6. Idle behavior

## 3. MVP Constraints

Target hardware: NVIDIA RTX 5060 Ti 16GB, 32GB RAM recommended, Windows or Linux.

MVP must not require Kafka, Kubernetes, distributed GPU infrastructure, or video diffusion for every frame. Redis Streams is sufficient for MVP queues.

## 4. Video Generation Policy

Do not use generative video models as the primary realtime avatar renderer. Normal speaking uses audio + phoneme timing + avatar state + animation parameters → realtime avatar renderer.

Generative video may later be used for special reactions, short transitions, or pre-generated content, but must not be required for normal speech.

## 5. Required Modules

```text
apps/
├── gateway/
├── behavior/
├── tts/
├── avatar/
├── compositor/
└── dashboard/

packages/
├── event-schema/
├── state-machine/
├── character/
├── queue/
└── telemetry/
```

## 6. Queue Rules

Use Redis Streams for MVP. Recommended streams:

```text
event.raw
event.comment
event.gift
behavior.input
tts.input
tts.output
avatar.action
telemetry
```

Consumers must support retry, timeout, idempotency, structured logging, and graceful shutdown.

## 7. Idempotency

Every external event requires a unique event ID. Duplicate events must not be processed twice. Gift events are especially important.

## 8. Failure Handling

LLM unavailable → predefined fallback response.

TTS unavailable → predefined reaction or skip speech.

Avatar unavailable → safe/static scene and reconnect path.

TikTok connection lost → mark disconnected, stop new event consumption, preserve safe state, reconnect when possible.

Redis unavailable → fail visibly and safely; never silently discard events.

## 9. Emergency Stop

Provide an emergency stop that immediately stops TTS playback, new behavior generation, and gift reactions, and optionally freezes the avatar while keeping OBS alive where possible.

## 10. Logging

Use structured logs containing timestamp, event_id, stream_id, module, action, latency, status, and error where relevant. Never log API keys, tokens, cookies, authentication headers, or other credentials.

## 11. Testing

Every module requires unit/integration tests as appropriate. Required test categories include schema validation, state transitions, queue behavior, event simulator tests, and end-to-end tests.

## 12. Event Simulator

Before real TikTok integration, support simulated comments, gifts, follows, joins, likes, and shares, including low/normal/high traffic, gift bursts, duplicates, and malformed events.

## 13. Development Workflow

1. Inspect repository.
2. Read `AGENTS.md` and all `docs/` files.
3. Understand current architecture.
4. Produce an implementation plan.
5. Implement one roadmap phase only.
6. Run tests.
7. Fix failures.
8. Report changed files and test results.
9. Wait for the next phase.

Do not implement the entire roadmap in one uncontrolled change.

## 14. Definition of Done

A feature is complete only when implementation, tests, error handling, appropriate logging, documented interfaces, and acceptance criteria are satisfied.

## 15. Code Quality

Prefer small modules, explicit interfaces, typed data, deterministic behavior where possible, dependency injection, and configuration over hard-coded values.

Avoid global mutable state, hidden singleton dependencies, direct cross-module imports that violate architecture, provider-specific business logic, and duplicated event schemas.

## 16. Product Principle

The goal is not to make an AI that talks constantly. The goal is to make a character that behaves like a coherent live performer. The system must support silence, idle behavior, selective responses, contextual reactions, emotional continuity, gift reactions, viewer memory, and natural timing.

Optimize for latency + continuity + behavior quality + viewer interaction, not response speed alone.