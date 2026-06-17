# Benchmarks

We publish **results, not methods.** Numbers below are product-quality indicators measured
on public, industry-standard test sets. The models and techniques that produce them are proprietary.

## Translation quality (FLORES-200)

Localization quality measured with **COMET** (`wmt22-comet-da`), the neural metric used to
rank state-of-the-art machine translation — it correlates with human judgment far better
than BLEU.

| Test | Metric | NOESIS |
|---|---|---|
| FLORES-200 devtest (EN↔RU, EN↔ZH) | **COMET** | **≈ 0.88** |
| — | chrF++ | ≈ 50 |

*Competitive with leading dedicated MT systems. Evaluated on 100 sentences per direction,
greedy decoding, neural-metric primary.*

## Dubbing capabilities

| Property | Value |
|---|---|
| Languages | **600+** |
| Speakers per title | up to **20** (auto-separated) |
| Lip-sync accuracy | **±50 ms** to source |
| Voice fidelity | **48 kHz**, identity-preserving |
| Music / ambience | preserved (dialogue-only re-voicing) |

## Quality philosophy

Every generated line passes automated gates before delivery:

- **Language correctness** — verified, not assumed.
- **Speaker identity** — the cloned voice matches the actor.
- **Timing** — within the lip-sync tolerance.
- **Audio integrity** — no truncation, artifacts, or drift.

Failing lines are automatically retried; the output is a distribution-ready master.

---

*Methodology notes: public test sets (FLORES-200) and standard metrics (COMET, chrF++) are
used so results are independently reproducible at the metric level. Internal model details,
prompts, and pipelines are not part of these figures and are not disclosed.*
