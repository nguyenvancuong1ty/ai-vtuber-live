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
comments -> normalize -> moderate -> deduplicate -> classify -> score -> select -> LLM
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

## Viewer context

Maintain only useful context:

```text
viewer_id
display_name
first_seen
last_seen
interaction_count
gift_count
last_topics
last_response
```

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

## Idle behavior

```text
10-20s -> micro movement
20-40s -> idle action
30+s   -> optional short monologue
```

Idle speech must be short and context-aware. Avoid endless repetitive monologues.

## Repetition control

Track recent responses and reduce repeated sentence patterns, gestures, and reactions using a rolling context window.

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

If the LLM fails, use contextual predefined response templates. The avatar remains operational.

## Acceptance Criteria

- selected comments receive responses
- comments can be ignored
- priority works
- high-priority gifts can interrupt speech
- state transitions are deterministic and tested
- personality is configurable
- viewer context works
- repeated responses are reduced
- idle behavior works
- LLM failure has a fallback
- unit tests cover every state transition