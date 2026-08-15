# Avatar Runtime

## Purpose

Realtime anime/VTuber character renderer with stable identity. Normal speech must use realtime animation, not generative video.

## Recommended MVP

Primary: Unity + VRM.

Alternative: Live2D.

The runtime exposes a provider-independent avatar control interface.

## Runtime input

```json
{
  "emotion": "happy",
  "emotion_intensity": 0.7,
  "gesture": "wave",
  "gaze": "viewer",
  "tts": {
    "audio_path": "audio/tts_001.wav",
    "phonemes": []
  }
}
```

## Animation layers

```text
Base pose
Breathing
Eye movement
Blink
Head motion
Emotion
Gesture
Lip-sync
```

Higher priority layers override lower layers only when necessary.

## Lip-sync

Drive mouth animation from audio plus phoneme/viseme timing. Do not regenerate the whole face for each speech segment.

Minimum mouth states should cover neutral, open/close and common vowel/consonant shapes with blend weights.

## Identity stability

Keep face proportions, hair, eyes, clothing, body proportions, colors and accessories stable throughout the session.

## Expressions

MVP expressions:

```text
neutral
happy
sad
surprised
angry
shy
```

Expressions support intensity from 0.0 to 1.0 and blending.

## Eyes and gaze

Support:

```text
viewer
camera
left
right
up
down
random gaze
```

Gaze changes must be smoothly interpolated.

## Blink

Use randomized timing rather than a fixed interval. Suggested normal range: 2.4-6.8 seconds. Support normal, double and reaction blinks.

## Head motion

Support yaw, pitch and roll. Use smooth interpolation and subtle amplitudes. Avoid high-frequency random noise.

## Idle

Minimum idle behavior:

```text
breathing
blink
eye movement
head micro movement
posture variation
```

## Gestures

MVP:

```text
wave
nod
shake_head
point
laugh
surprised
shy
celebrate
```

Gestures are interruptible according to priority.

## Gift reaction

```text
Gift -> emotion -> gesture -> optional camera effect -> speech -> restore previous state
```

Large reactions should preserve previous state where possible.

## Synchronization

Audio and mouth animation share one timeline:

```text
audio_start
phoneme_start
phoneme_end
audio_end
```

Do not independently estimate timing when accurate phoneme timing exists.

## Rendering target

```text
1920x1080
30 FPS minimum
60 FPS preferred
```

Final composition is handled by OBS.

## OBS

Preferred path:

```text
Unity -> capture/virtual output -> OBS
```

OBS owns background, avatar layer, chat, gifts, alerts, music and overlays.

## Performance telemetry

Monitor FPS, GPU usage, VRAM, CPU usage, frame time, audio latency and queue latency.

If GPU pressure occurs, reduce effects/quality/animation complexity before sacrificing realtime frame rate.

## Recovery

If runtime crashes:

```text
detect -> restart -> reload character -> restore safe state -> notify dashboard
```

## Acceptance Criteria

- VRM character loads
- stable 30 FPS minimum
- lip-sync works
- blink works
- gaze works
- head motion works
- breathing works
- six expressions work
- gestures work
- gift reactions work
- animation layers blend correctly
- runtime state is inspectable
- TTS audio stays synchronized
- basic runtime recovery works