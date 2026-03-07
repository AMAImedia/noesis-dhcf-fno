
The framework operates **without modifying neural network weights**.

---

## Research Status

| Item | Status |
|-----|------|
| IEEE TASLP submission | ✓ Submitted |
| Architecture specification | ✓ Published |
| Phase-R benchmark results | ✓ Available |
| Reproducibility tests | ✓ Verified |
| Core implementation | Commercial license |

---

## Key Results

| Metric | Value |
|------|------|
| LUFS drift | 0.0000 dB |
| Mastering Lipschitz κ | < 1 |
| IQS v0.8 | 0.687 ± 0.051 |
| CPU mastering latency | ~350 ms |
| Determinism tests | 99/99 identical outputs |

The system demonstrates **bitwise reproducibility across runs** under fixed
seed and configuration.

---

## Performance (30s audio generation)

| GPU | Total Time | Real-Time Factor | VRAM |
|----|----|----|----|
| A100 | 13.6 s | 0.45× | 13 GB |
| RTX 4090 | 21.1 s | 0.70× | 15 GB |
| RTX 3090 | 31.8 s | 1.06× | 15 GB |
| RTX 3060 | 48.6 s | 1.62× | 11 GB |
| RTX 2060 | ~75 s | ~2.5× | 6 GB |

Low-VRAM mode is supported while preserving deterministic guarantees.

---

## Repository Contents

### Research materials (public)

- architecture documentation
- benchmark results
- reproducibility specification
- IQS scoring description
- telemetry format specification
- research paper

### Commercial components

The following components are not included in this repository:

- DSP mastering chain implementation
- IQS scoring engine
- closed-loop optimizer runtime
- operator registry system

These components are available under **commercial licensing**.

---

## Citation

If you use ideas from this work in research, please cite:

```bibtex
@unpublished{bolotnikov2026noesis,
  title     = {{NOESIS}: Deterministic Hybrid Control Framework for Frozen Neural Operators},
  author    = {Bolotnikov, Ilia},
  year      = {2026},
  note      = {Submitted to IEEE Transactions on Audio, Speech, and Language Processing},
  url       = {https://github.com/AMAImedia/noesis-dhcf-fno}
}
