# Behavior Engine

## Purpose

Central decision layer for the Vietnamese AI VTuber. It decides whether to respond, what to say, emotion, gesture, gaze, priority, interruption, and idle behavior. It never renders or directly controls the avatar.

## State machine

```text
IDLE -> READING -> THINKING -> SPEAKING -> COOLDOWN -> IDLE
SPEAKING -> INTERRUPTED -> REACTION -> SPEAKING / IDLE
```

States:
- IDLE: breathing, blinking, gaze, subtle posture changes.
- READING: short processing reaction; target 200-1200ms.
- THINKING: optional short processing state; use sparingly.
- SPEAKING: TTS, lip-sync, expression, gaze and subtle head motion.
- REACTION: gifts, surprises, jokes and major events.
- COOLDOWN: prevents repetitive immediate reactions.

## Comment selection

Never send every comment to the LLM.

```text
comments -> normalize -> moderate -> deduplicate -> classify -> score -> select -> memory lookup -> LLM
```

Conceptual score:

```text
relevance + novelty + conversation_context + user_relationship + entertainment_value + gift_relation
```

Comments may be ignored when duplicate, spam, low relevance, unsafe, already answered, repeated, or lower priority than the current activity.

## Timing

Do not use one fixed delay. Suggested ranges:

```text
simple comment: 1-2s
normal question: 2-4s
complex question: 3-6s
small gift: 0.5-2s
large gift: 0-1.5s
```

Actual timing depends on state and queue pressure.

## Personality

Character personality must be configuration-driven:

```json
{
  "name": "Yuki",
  "language": "vi",
  "personality": {
    "traits": ["playful", "friendly"],
    "speech_style": "casual Vietnamese",
    "energy": 0.7,
    "humor": 0.6
  }
}
```

## Viewer Memory

Viewer memory is a first-class input to Behavior Engine and is defined in `docs/memory-system.md`.

The engine receives a bounded `viewer_memory` context through a repository/service interface.

Input shape:

```text
BehaviorInput
├── event
├── character_state
├── conversation_context
└── viewer_memory
```

Example:

```json
{
  "viewer_id": "user_123",
  "display_name": "Hoàng Long",
  "is_returning": true,
  "interaction_count": 8,
  "gift_count": 3,
  "total_gift_value": 120,
  "favorite_topics": ["anime", "games"],
  "last_topics": ["One Piece"],
  "memory_summary": "Returning viewer who often talks about anime."
}
```

Memory should be used to improve continuity, for example:

```text
viewer returns
→ recognize viewer
→ retrieve relevant context
→ naturally reference stored context when appropriate
```

Do not force memory into every response. The character should not repeatedly announce that it remembers the viewer.

Memory is context, not unrestricted truth. The engine must not invent facts that are absent from stored memory.

Prompt-generation code must not query the database directly. Use a `ViewerMemoryRepository` or equivalent interface.

If memory is unavailable, continue with an empty memory context rather than blocking the live response path.

## Viewer context

Maintain only useful context:

```text
viewer_id
display_name
first_seen
last_seen
interaction_count
gift_count
total_gift_value
last_topics
last_response
memory_summary
```

Persistent storage and retention rules are defined in `docs/memory-system.md`.

## Nickname normalization

Examples:

```text
hoang_long_99 -> Hoàng Long
abc123 -> ABC
nguyen_van_a -> Nguyễn Văn A
```

Never blindly send raw usernames to TTS.

## Gift behavior

Gift mappings are configuration-driven:

```text
SMALL  -> happy + wave + short thanks
MEDIUM -> stronger emotion + gesture + thanks
LARGE  -> surprise + special animation + high-priority speech
COMBO  -> milestone/progression event
```

Every accepted gift must update viewer gift memory idempotently.

## Idle behavior

```text
10-20s -> micro movement
20-40s -> idle action
30+s   -> optional short monologue
```

Idle speech must be short and context-aware. Avoid endless repetitive monologues.

## Repetition control

Track recent responses and reduce repeated sentence patterns, gestures, and reactions using a rolling context window.

Viewer memory can provide historical context, but recent-session repetition control must remain separate from persistent memory.

## LLM contract

The LLM must return structured output:

```json
{
  "should_speak": true,
  "speech": "Ủa anh Long mới vào hả?",
  "emotion": "happy",
  "emotion_intensity": 0.65,
  "gesture": "wave",
  "gaze": "viewer",
  "priority": 70
}
```

Invalid output is rejected and routed to fallback behavior.

## Fallback

If the LLM fails, use contextual predefined response templates. The avatar remains operational. Missing viewer memory must also fall back to normal behavior.

## Acceptance Criteria

- selected comments receive responses
- comments can be ignored
- priority works
- high-priority gifts can interrupt speech
- state transitions are deterministic and tested
- personality is configurable
- first-time and returning viewers are distinguishable
- viewer memory is retrieved through an explicit interface
- interaction and gift history can influence context
- memory is not mandatory for the response path
- repeated responses are reduced
- idle behavior works
- LLM failure has a fallback
- memory failure has a fallback
- unit tests cover every state transition
- tests cover new viewer, returning viewer, memory retrieval, and gift-history context
