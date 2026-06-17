<div align="center">

# NOESIS — by amaimedia

### Cinema-grade AI dubbing & a full creative media platform.

[![Platform](https://img.shields.io/badge/platform-amaimedia.com-7c3aed)](https://amaimedia.com)
[![Dubbing](https://img.shields.io/badge/AI%20dubbing-600%2B%20languages-a78bfa)](docs/dubbing.md)
[![Speakers](https://img.shields.io/badge/multi--speaker-up%20to%2020-f472b6)](docs/dubbing.md)
[![Lip-sync](https://img.shields.io/badge/lip--sync-%C2%B150ms-f59e0b)](docs/dubbing.md)
[![License](https://img.shields.io/badge/license-proprietary-0a0a0b)](LICENSE)

**Dub any video into 600+ languages — keeping every actor's real voice, emotion, and timing.**
Plus voice cloning, music, image, and video tools in one platform.

[Watch the dubbing demo](#) · [Platform](docs/platform.md) · [Benchmarks](docs/benchmarks.md) · [API](docs/api-reference.md)

</div>

---

> **What this repository is.** A public overview of the NOESIS platform by amaimedia —
> capabilities, quality results, and the public API. It is **not** the source code.
> The models, pipelines, training methods, and engines are proprietary and not published here.
> See [LICENSE](LICENSE) and [NOTICE](NOTICE).

---

## 🎯 The hero: AI dubbing

NOESIS dubs film, series, and online video at a quality bar built for real distribution —
not a robotic voiceover. The dubbing engine:

- **Preserves each actor's voice.** Per-speaker voice cloning keeps timbre and identity in the target language.
- **Handles real scenes.** Automatic multi-speaker separation, up to **20 speakers** per title.
- **Stays in sync.** Timing locked to the source within **±50 ms** for natural lip-sync feel.
- **Speaks the world.** **600+ languages** out of the box.
- **Keeps the mix.** Music and ambience are preserved; only the dialogue is re-voiced.

→ Full overview: **[docs/dubbing.md](docs/dubbing.md)**

## 🧩 The platform

| Surface | What it does |
|---|---|
| 🎬 **Dubbing** | Multi-speaker, lip-synced video dubbing in 600+ languages |
| 🗣️ **Voice** | Clone a voice from 5–30 s and synthesize 48 kHz speech, multilingual |
| ✂️ **Video edit** | Auto-cut long video into vertical Shorts/Reels with hooks, titles, subtitles |
| 🎵 **Music** | Generate original music from a text prompt |
| 🖼️ **Image** | Generate images from a text prompt |
| 💬 **Assistant** | Orchestrates the above through one conversational interface |

→ Details: **[docs/platform.md](docs/platform.md)**

## 📊 Quality, measured

We publish **results**, not recipes.

| Capability | Result |
|---|---|
| Translation quality | **COMET ≈ 0.88** on FLORES-200, competitive with leading MT systems |
| Languages | **600+** |
| Speakers per title | up to **20** |
| Lip-sync accuracy | **±50 ms** to source |
| Voice output | **48 kHz**, identity-preserving |

→ Methodology & full table: **[docs/benchmarks.md](docs/benchmarks.md)**

## 🔌 API

A simple job-based API: submit, poll, retrieve. Dubbing, voice, video-edit, music, image.

```
POST  dub_video      { file_url, target_lang, quality }      → job_id
GET   job_status     { job_id }                              → progress + result_url
```

→ Full reference: **[docs/api-reference.md](docs/api-reference.md)**

## 🏛️ How it fits together

A high-level view of the platform — engines are black boxes by design.

→ **[docs/architecture.md](docs/architecture.md)**

---

## About

NOESIS is built by **[amaimedia](https://amaimedia.com)**. Product names, engine names,
designs, and methods are trademarks and trade secrets of amaimedia. This repository
documents the product; it does not license the underlying technology.

**Contact:** info@amaimedia.com · **Press:** see [PRESS_KIT/](PRESS_KIT/)

© 2026 amaimedia. All rights reserved.
