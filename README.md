# """NOESIS — Deterministic Hybrid Control Framework for Frozen Neural Operators (DHCF-FNO)
# Copyright (c) 2026 AMAImedia.com
# All rights reserved."""

License: NOESIS Research License v1.0 (Research-only)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## License

NOESIS is released under the **NOESIS Research License v1.0**.

The source code is provided for research and academic inspection only.

> License: NOESIS Research License v1.0 (Research-only)
> If you use ideas from this work in academic publications,
> please cite the NOESIS paper.
> ![License](https://img.shields.io/badge/license-NOESIS%20Research-blue)

Commercial use, redistribution, or derivative implementations of the system require written permission from AMAImedia.com.

See LICENSE file for details.
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

**Deterministic Hybrid Control Framework over Frozen Neural Operator  
with Objective-locked Optimization**

> **Paper:** Submitted to IEEE Transactions on Audio, Speech, and Language Processing  
> **Contact:** info@amaimedia.com · [AMAImedia.com](https://amaimedia.com)

---

## Research Status

| Item | Status |
|------|--------|
| Paper (IEEE TASLP submission) | ✓ Submitted |
| Architecture specification | ✓ Published |
| Benchmark results (Phase-R) | ✓ [Available](docs/BENCHMARK_RESULTS.md) |
| Reproducibility tests | ✓ 99/99 SHA-256 verified |
| Core implementation | Commercial license |

---

## What is NOESIS?

NOESIS is a **production-grade control wrapper** that transforms *any* frozen AI music-generation model into a cryptographically reproducible, professionally mastered audio engine — without touching a single neural weight.

The key principle: the neural model is treated as an **immutable operator**. NOESIS adds a complete control and mastering layer *around* it.

```
Text Prompt + Seed + Genre
        │
        ▼
┌──────────────────────────────────────────────┐
│  NOESIS DHCF-FNO Control Layer               │
│  3× RNG Lock · SHA-256 Audit · IQS v0.8 QA  │
│                                              │
│  ┌─────────────────┐  ┌───────────────────┐  │
│  │  Frozen D_θ     │  │ 11-Stage Mastering│  │
│  │  (ANY backend)  │  │ κ = 0.9103 < 1   │  │
│  └─────────────────┘  └───────────────────┘  │
│  Trust-region L-BFGS · Snapshot v16 telemetry│
└──────────────────────────────────────────────┘
        │
        ▼
  Mastered WAV + Snapshot v16 JSON
  (SHA-256 sealed · bitwise reproducible · Merkle tree)
```

---

## Key Results

| Metric | Value |
|--------|-------|
| LUFS drift | 0.0000 dB (all 9 genres) |
| Mastering Lipschitz κ | 0.9103 < 1 |
| IQS v0.8 | 0.687 ± 0.051 (95% CI: [0.678, 0.730]) |
| Mastering latency | 353 ± 4 ms (CPU, GPU-independent) |
| Determinism | 99/99 SHA-256 tests (11 seeds × 9 genres) |

---

## GPU Performance (30 s audio, fix_nfe=8, bfloat16)

| GPU | Total | RTF | VRAM | Mode |
|-----|-------|-----|------|------|
| A100 80GB | 13.6 s | 0.45× | 13.1 GB | Standard |
| RTX 4090 | 21.1 s | 0.70× | 14.8 GB | Standard |
| RTX 4080 | 25.3 s | 0.84× | 14.6 GB | Standard |
| RTX 3090 | 31.8 s | 1.06× | 14.8 GB | Standard |
| RTX 3060 12GB | 48.6 s | 1.62× | 11.2 GB | Standard |
| **RTX 3060 6GB (laptop)** | **~59 s** | **~2.0×** | **5.9 GB** | **`--low-vram`** |
| **RTX 2060 6GB** | **~75 s** | **~2.5×** | **5.8 GB** | **`--low-vram`** |
| GTX 1080 8GB | ~95 s | ~3.2× | 7.6 GB | fp16 fallback† |

> **6 GB VRAM:** Use `--low-vram`. Full determinism + mastering guarantees hold.  
> **† GTX 1080 (Pascal):** No native bfloat16 → float16 fallback. Minimum recommended: RTX 2060.

---

## Repository Contents

**Published (free):**
- Research paper (PDF, see Releases)
- Architecture specification and diagrams
- [Phase-R benchmark results](docs/BENCHMARK_RESULTS.md)
- IQS v0.8 formula + sealed weight checksums (Σ = 1.00)
- Snapshot v16 audit format specification

**Commercial license only:**
- DSP mastering chain implementation
- IQS scorer and TinyMOS predictor
- Closed-loop L-BFGS optimizer
- DHCF runtime and operator registry

---

## Citation

```bibtex
@unpublished{bolotnikov2026noesis,
  title     = {{NOESIS}: {D}eterministic {H}ybrid {C}ontrol {F}ramework
               over {F}rozen {N}eural {O}perator with
               {O}bjective-locked {O}ptimization ({DHCF-FNO})},
  author    = {Bolotnikov, Ilia},
  year      = {2026},
  note      = {Submitted to {IEEE} Trans.\ Audio, Speech, Language Process.},
  url       = {https://github.com/AMAImedia/noesis-dhcf-fno}
}
```

---

## Contact

📧 **info@amaimedia.com**  
🌐 **[AMAImedia.com](https://amaimedia.com)**

---

*© 2026 AMAImedia. All rights reserved.*
