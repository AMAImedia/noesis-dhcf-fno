# Architecture (high level)

A black-box view of how the platform fits together. Internal engines, models, and methods
are proprietary and represented here only as opaque components.

```
                ┌─────────────────────────────────────────────┐
   User / API ─►│            Orchestration & Jobs              │
                │   submit → queue → poll → deliver            │
                └───────────────┬─────────────────────────────┘
                                │
        ┌───────────┬───────────┼───────────┬───────────┐
        ▼           ▼           ▼           ▼           ▼
   ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐
   │ Dubbing │ │  Voice  │ │  Video  │ │  Music  │ │  Image  │
   │ engine  │ │ engine  │ │  edit   │ │ engine  │ │ engine  │
   └────┬────┘ └─────────┘ └─────────┘ └─────────┘ └─────────┘
        │
        ▼  (dubbing internals, black-box)
   understand ─► translate ─► re-voice ─► align & mix
        │
        ▼
   ┌─────────────────────────────┐
   │  Quality gates (every job)  │  language · identity · timing · audio
   └─────────────┬───────────────┘
                 ▼
            Delivered master
```

## Design principles

- **Job-based** — every request is submit → poll → retrieve; nothing blocks.
- **Quality-gated** — outputs are verified before delivery, with automatic retries.
- **Engine-agnostic surface** — the public API never exposes which engine or model runs underneath.
- **Identity-preserving** — voices, actors, and brand stay intact across languages.

> The diagram intentionally stops at the component boundary. Model architectures, training
> methods, prompts, and pipeline internals are trade secrets of amaimedia and are not published.
