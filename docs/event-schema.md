# Event Schema

## 1. Purpose

All external and internal events use a common envelope for validation, deduplication, replay, debugging, queue processing, and future distributed deployment.

## 2. Event Envelope

```json
{
  "id": "evt_01JXYZ",
  "type": "comment",
  "version": 1,
  "timestamp": 1786800000123,
  "stream_id": "live_123",
  "source": "tiktok",
  "user": {
    "id": "user_123",
    "nickname": "hoang_long_99",
    "display_name": "Hoàng Long"
  },
  "payload": {}
}
```

Required fields: `id`, `type`, `version`, `timestamp`, `stream_id`, `source`, `payload`.

The stable `user.id` is the canonical viewer identity within a platform. Display names and nicknames are mutable attributes.

## 3. Event Types

MVP: `comment`, `gift`, `follow`, `join`, `like`, `share`, `subscribe`, `system`.

Internal memory-related events may include:

```text
viewer.resolved
viewer.created
viewer.interaction_recorded
viewer.gift_recorded
viewer.memory_updated
```

These internal events use the same envelope contract.

## 4. Comment Event

```json
{
  "id": "evt_comment_001",
  "type": "comment",
  "version": 1,
  "timestamp": 1786800000123,
  "stream_id": "live_123",
  "source": "tiktok",
  "user": {
    "id": "user_123",
    "nickname": "hoang_long_99",
    "display_name": "Hoàng Long"
  },
  "payload": {"text": "Yuki hôm nay xinh vậy"}
}
```

## 5. Normalized Comment

```json
{
  "event_id": "evt_comment_001",
  "type": "normalized_comment",
  "user": {"id": "user_123", "display_name": "Hoàng Long"},
  "text": "Yuki hôm nay xinh vậy",
  "language": "vi",
  "intent": "compliment",
  "toxicity": 0.0,
  "priority": 71,
  "should_reply": true
}
```

## 6. Gift Event

```json
{
  "id": "evt_gift_001",
  "type": "gift",
  "version": 1,
  "timestamp": 1786800010000,
  "stream_id": "live_123",
  "source": "tiktok",
  "user": {
    "id": "user_123",
    "nickname": "hoang_long_99",
    "display_name": "Hoàng Long"
  },
  "payload": {
    "gift_id": "rose",
    "gift_name": "Rose",
    "count": 1,
    "repeat_count": 3,
    "coin_value": 3
  }
}
```

## 7. Gift Action

```json
{
  "type": "gift_action",
  "event_id": "evt_gift_001",
  "user_id": "user_123",
  "tier": "small",
  "priority": 90,
  "emotion": "happy",
  "gesture": "wave",
  "animation": "gift_thanks",
  "speech_template": "thank_user"
}
```

## 8. Behavior Command

```json
{
  "id": "behavior_001",
  "type": "behavior_command",
  "priority": 70,
  "source_event_id": "evt_comment_001",
  "speech": {"text": "Anh Long ơi cảm ơn nha!", "enabled": true},
  "emotion": {"name": "happy", "intensity": 0.72},
  "gesture": {"name": "wave", "intensity": 0.5},
  "gaze": {"target": "viewer"}
}
```

## 9. TTS Output

```json
{
  "id": "tts_001",
  "type": "tts_output",
  "source_behavior_id": "behavior_001",
  "audio_path": "audio/tts_001.wav",
  "duration_ms": 2150,
  "phonemes": [
    {"phoneme": "a", "start_ms": 0, "end_ms": 120, "weight": 1.0}
  ]
}
```

## 10. Avatar Action

```json
{
  "id": "avatar_001",
  "type": "avatar_action",
  "emotion": {"name": "happy", "intensity": 0.72},
  "gesture": "wave",
  "gaze": "viewer",
  "speech": {"tts_id": "tts_001"}
}
```

## 11. Viewer Memory Context

The memory layer exposes a compact context to Behavior Engine:

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

This is context, not a free-form claim of fact. Only stored/derived data may be supplied.

## 12. Memory Update Event

```json
{
  "id": "evt_memory_001",
  "type": "viewer.interaction_recorded",
  "version": 1,
  "timestamp": 1786800020000,
  "stream_id": "live_123",
  "source": "memory",
  "user": {
    "id": "user_123",
    "display_name": "Hoàng Long"
  },
  "payload": {
    "source_event_id": "evt_comment_001",
    "interaction_type": "comment",
    "intent": "compliment"
  }
}
```

## 13. Priority

Recommended range:

```text
100 emergency/system
95 very large gift
90 large gift
80 medium gift
70 important comment
40 normal comment
20 follow/join
10 idle
```

Higher value means higher priority.

## 14. Idempotency

Every event has a unique ID. Processed IDs must be retained for a configurable deduplication window. Duplicate events are ignored.

Memory updates must also be idempotent. A duplicate comment or gift event must not double-count interaction/gift totals.

## 15. Event Simulator

Support `single comment`, `comment burst`, `gift`, `gift burst`, `follow`, `join`, `mixed traffic`, `duplicate event`, `malformed event`, and returning-viewer scenarios.

Example:

```json
{
  "scenario": "returning_viewer",
  "viewer_id": "user_123",
  "previous_interactions": 8,
  "previous_gifts": 3
}
```

## 16. Schema Versioning

Every event contains `version`. Breaking semantic changes require a new schema version. Do not silently change existing semantics.
