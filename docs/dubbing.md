# NOESIS Dubbing

Cinema-grade AI dubbing: turn any video into another language while keeping every
actor's real voice, emotion, and timing. Built for content that actually ships —
films, series, courses, and creator video at scale.

## Why it's different

Most "AI dubbing" is a single robotic voice read over the original. NOESIS treats a
title the way a professional dubbing studio does — per actor, per scene, with the mix intact.

| | Typical AI voiceover | **NOESIS Dubbing** |
|---|---|---|
| Voices | one generic TTS voice | **each actor's own voice, cloned** |
| Speakers | mono | **multi-speaker, up to 20** |
| Timing | drifts | **locked to source ±50 ms** |
| Music / ambience | lost or muddied | **preserved** |
| Languages | a handful | **600+** |

## What it does (black-box overview)

```
   video in  ─►  understand  ─►  translate  ─►  re-voice  ─►  align & mix  ─►  video out
               (who speaks,     (localized,   (each actor's   (±50 ms sync,
                when, what)      natural)      cloned voice)    music kept)
```

1. **Understand** — detect who speaks and when; separate up to 20 speakers; capture timing.
2. **Translate** — localize the dialogue naturally (not word-for-word), tuned for spoken delivery.
3. **Re-voice** — synthesize each line in the target language **in that actor's voice**.
4. **Align & mix** — lock timing to the source for lip-sync feel; re-blend with the original music and ambience.

*The specific models, engines, and methods behind each step are proprietary and not disclosed.*

## Quality controls

Every dubbed line passes automated quality gates before it ships — language correctness,
speaker identity, timing, and audio integrity — with automatic retries. The goal is a
distribution-ready master, not a draft.

## Languages & scale

- **600+ languages** supported.
- **Up to 20 speakers** per title, separated automatically.
- Long-form ready: full episodes and films, not just clips.

## Use it

Via the platform UI at [amaimedia.com](https://amaimedia.com), or the API:

```
POST dub_video { file_url, target_lang, quality }  → job_id
GET  job_status { job_id }                          → progress + result_url
```

See [api-reference.md](api-reference.md) · [benchmarks.md](benchmarks.md).
