# Viewer Memory System

## 1. Purpose

Viewer Memory gives the VTuber persistent, bounded memory across live sessions so returning viewers can be recognized and treated consistently.

The system stores useful interaction history, gift history, lightweight preferences, and a compact memory summary. It must not become an unrestricted transcript archive.

Primary goals:

- recognize returning viewers
- remember useful conversation context
- remember gift history
- personalize responses
- maintain continuity across live sessions
- provide deterministic data to the Behavior Engine

---

## 2. Design Principles

### Persistent identity

A viewer is identified by the stable platform user ID supplied by the connector. Nicknames are display attributes and may change.

### Bounded memory

Store only information useful for live interaction. Do not retain unlimited raw chat history by default.

### Structured facts first

Prefer structured fields such as:

- first seen
- last seen
- interaction count
- gift count
- total gift value
- favorite topics
- last interaction
- relationship score

A compact `memory_summary` may contain higher-level context.

### Privacy and minimization

Do not store unnecessary personal information. Do not store passwords, tokens, cookies, payment credentials, or authentication data.

---

## 3. Data Model

### viewer

```text
id
platform
platform_user_id
display_name
nickname
first_seen_at
last_seen_at
interaction_count
gift_count
total_gift_value
last_interaction_at
created_at
updated_at
```

`platform_user_id` must be unique per platform.

### viewer_memory

```text
id
viewer_id
memory_summary
favorite_topics
relationship_score
last_topics
last_response
last_emotion
created_at
updated_at
```

`viewer_memory` contains derived context rather than an unlimited conversation transcript.

### viewer_gift

```text
id
viewer_id
gift_id
gift_name
count
total_coin_value
first_received_at
last_received_at
```

Aggregate repeated gifts where appropriate.

### interaction

```text
id
viewer_id
live_session_id
type
text
intent
priority
created_at
```

Raw text retention must be configurable. MVP should support retention limits.

---

## 4. Memory Lifecycle

### First interaction

```text
new viewer
→ create viewer
→ create viewer_memory
→ record interaction
```

### Returning viewer

```text
incoming event
→ resolve platform_user_id
→ load viewer memory
→ update last_seen
→ provide relevant context to Behavior Engine
```

### Gift

```text
gift event
→ record viewer_gift
→ increment viewer.gift_count
→ update total_gift_value
→ update relationship/context if configured
```

---

## 5. Memory Retrieval

Before generating a response, Behavior Engine may request a compact viewer context:

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

Only relevant fields should be passed to the LLM.

---

## 6. Memory Update Rules

Memory should be updated after meaningful interactions, not after every internal animation event.

Update candidates:

- meaningful comment
- completed response
- gift
- follow
- explicit preference
- recurring topic
- meaningful return visit

Do not treat arbitrary speculation from the LLM as a fact.

Facts explicitly provided by the viewer may be stored only when the application has a clear rule for doing so.

---

## 7. Relationship Score

A lightweight relationship score may be used for response prioritization.

It is an application metric, not a claim about the real relationship with the viewer.

Example inputs:

```text
visit frequency
interaction frequency
gift history
recency
positive interaction signals
```

The score must be bounded and explainable.

Example:

```text
0.0 → 1.0
```

Do not let relationship score alone determine whether a viewer receives a response.

---

## 8. Gift Memory

Gift history should support:

- total gift count
- total value
- last gift
- gift frequency
- gift type history

Example context:

```text
Hoàng Long
3 gift events
120 total coins
last gift: Rose
```

The system must not fabricate gift history.

---

## 9. Conversation Memory

MVP should retain a bounded recent interaction window.

Example:

```text
last 10 meaningful interactions
```

The exact limit must be configurable.

Older context should be summarized into `memory_summary` only when supported by deterministic or explicitly validated logic.

---

## 10. Cross-Session Memory

Memory survives the end of a LIVE session.

At the beginning of a new session:

```text
viewer event
→ persistent lookup
→ restore lightweight context
→ Behavior Engine
```

The system must distinguish:

```text
new viewer
returning viewer
currently active viewer
```

---

## 11. Memory Safety

Never allow memory to cause the character to claim facts that do not exist.

The Behavior Engine must treat memory as context, not truth beyond the stored data.

If memory is unavailable:

```text
viewer context = empty
```

The character must continue operating normally.

---

## 12. Retention

Retention must be configurable.

Recommended MVP defaults:

```text
raw interaction text: bounded retention
aggregated viewer facts: retained
aggregated gift facts: retained
memory summary: retained
```

The implementation must make it possible to delete a viewer's stored memory by platform user ID.

---

## 13. API Contract

### Resolve viewer

```text
GET /viewers/:platform/:platformUserId
```

### Get memory

```text
GET /viewers/:platform/:platformUserId/memory
```

### Record interaction

```text
POST /viewers/:platform/:platformUserId/interactions
```

### Record gift

```text
POST /viewers/:platform/:platformUserId/gifts
```

### Delete memory

```text
DELETE /viewers/:platform/:platformUserId/memory
```

The exact transport may change; domain interfaces must remain stable.

---

## 14. Behavior Engine Integration

Behavior Engine receives memory context as an explicit input:

```text
BehaviorInput
├── event
├── character_state
├── conversation_context
└── viewer_memory
```

It must not query the database directly from prompt-generation code.

Use a `ViewerMemoryRepository` or equivalent interface.

---

## 15. Acceptance Criteria

Memory MVP is complete when:

- a viewer is created on first interaction
- returning viewers are resolved by stable platform user ID
- interaction counts persist across sessions
- gift counts persist across sessions
- total gift value persists
- last interaction persists
- bounded conversation context works
- memory context reaches Behavior Engine
- memory is not directly coupled to LLM provider code
- missing memory does not break the live runtime
- duplicate events do not double-count interactions/gifts
- viewer memory can be deleted
- tests cover first visit, returning viewer, gift update, duplicate event, cross-session restore, and deletion
