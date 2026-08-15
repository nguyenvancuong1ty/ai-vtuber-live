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

## 3. Event Types

MVP: `comment`, `gift`, `follow`, `join`, `like`, `share`, `subscribe`, `system`.

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

## 11. Priority

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

## 12. Idempotency

Every event has a unique ID. Processed IDs must be retained for a configurable deduplication window. Duplicate events are ignored. Gift events must never trigger duplicate reactions.

## 13. Event Simulator

Support `single comment`, `comment burst`, `gift`, `gift burst`, `follow`, `join`, `mixed traffic`, `duplicate event`, and `malformed event` scenarios.

Example:

```json
{
  "scenario": "gift_burst",
  "duration_seconds": 60,
  "comments_per_minute": 40,
  "gifts_per_minute": 10
}
```

## 14. Schema Versioning

Every event contains `version`. Breaking semantic changes require a new schema version. Do not silently change existing semantics.