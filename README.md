[README.md](https://github.com/user-attachments/files/26134313/README.md)
# NOESIS — Deterministic Hybrid Control Framework for Frozen Neural Operators (DHCF-FNO)

<!-- NOESIS — Deterministic Hybrid Control Framework for Frozen Neural Operators (DHCF-FNO)
Copyright (c) 2026 AMAImedia.com
All rights reserved.
Full path: noesis-dhcf-fno/README.md -->

> **The framework operates without modifying neural network weights.**

📄 **Paper:** [PDF](https://github.com/AMAImedia/noesis-dhcf-fno/blob/main/paper/noesis_dhcf_fno_arxiv.pdf)

---

# 🏆 R.REF2 Benchmark (2026-03-14, MasteringChain v3.4)

All 9/10 genres: PASS or CF\_Limited drift contract (≤6.0 dB).
$\overline{\mathrm{IQS}}=0.614$, $\overline{J}=0.388$, MasteringChain v3.4 (31 stages).

**R.REF4 GPU target:** $\overline{J}>0.40$, 10/10 genres, ~2026-03-21.

# Phase R.1 — Historical Result

## Benchmark Summary

| Phase | PASS | PASS_CF_LIMITED | IQS Mean | Max LUFS Drift |
|------|------|----------------|---------|---------------|
| R.0 | 0 | 10 | 0.4947 | 1.609 dB |
| R.1 | 10 | 0 | 0.7027 | 0.0078 dB |

## Improvements

Drift reduction:
1.609 dB → 0.0078 dB  
Reduction factor: **206×**

IQS improvement:
0.4947 → 0.7027  
Increase: **+42%**

All tracks moved from PASS_CF_LIMITED → PASS.

## Key Case

metal genre:

before:
drift = 1.609 dB

after:
drift = 0.0055 dB

This confirms the expected behavior of **Stage 8.5 v4** limiter architecture.

## Conclusion

Stage 8.5 v4 successfully stabilizes LUFS drift and improves perceptual quality across all tested genres.

The system now satisfies deterministic mastering constraints required by the NOESIS protocol.

## Research Status

| Item | Status |
|---|---|
| IEEE TASLP submission | In preparation |
| Architecture specification | ✓ Published |
| Phase-R benchmark results | ✓ Available |
| Reproducibility protocol | ✓ Appendix F |
| Core implementation | Commercial license |

---

## Key Results

| Metric | Value |
|---|---|
| LUFS drift (all genres) | **0.0000 dB** |
| Mastering Lipschitz κ | **0.9103 < 1** (certified stable) |
| IQS v0.8 mean ± std | **0.687 ± 0.051** |
| 95% CI (IQS) | [0.678, 0.730] |
| Mastering latency | ~353 ms (CPU, constant) |
| Determinism tests | **99/99 bitwise-identical** |

The system produces bitwise-identical waveforms across repeated runs
and across GPU hardware generations (SHA-256 verified, 297 runs).

---

## Baseline Comparison

| Model | LUFS σ (dB) ↓ | Blind-SNR (dB) ↑ | Deterministic | Stable (κ<1) |
|---|---|---|---|---|
| DiffWave | 3.8 | 16.3 | ✗ | ✗ |
| MusicGen | 4.2 | 18.1 | ✗ | ✗ |
| Stable Audio 2 | 2.9 | 19.2 | ✗ | ✗ |
| YuE | 3.1 | 19.8 | ✗ | ✗ |
| Muse | 1.4 | 18.4 | partial | ✗ |
| **NOESIS (ours)** | **0.0000** | **22.1** | **✓** | **✓ (κ=0.9103)** |

LUFS σ = inter-run loudness standard deviation; lower is better.  
Blind-SNR after Hu et al. (2022); higher is better.

---

## Sensitivity Analysis

J exceeds the studio threshold (≥ 0.65) over:
- σ-slope ∈ [0.76, 1.08] (nominal: 0.95)
- guidance-scale ∈ [1.8, 6.4] (nominal: 4.0)

Maximum variation ΔJ = 0.056, consistent with the Lipschitz bound
(Proposition J-Lip, Appendix C). Wide operational basin — robust to
conditioning parameter variation.

---

## Performance (30 s audio generation)

| GPU | Total Time | Real-Time Factor | VRAM |
|---|---|---|---|
| A100 80 GB | 13.6 s | 0.45× | 13 GB |
| RTX 4090 | 21.1 s | 0.70× | 15 GB |
| RTX 4080 | 25.3 s | 0.84× | 15 GB |
| RTX 3090 | 31.8 s | 1.06× | 15 GB |
| RTX 3060 12 GB | 48.6 s | 1.62× | 11 GB |
| RTX 3060 6 GB (--low-vram) | 58.8 s | 1.96× | 6 GB |
| RTX 2060 (--low-vram) | 75.0 s | 2.50× | 6 GB |

All: `bfloat16`, `fix_nfe=8`, 99/99 determinism verified.

---

## Repository Contents

### Research materials (public)
- `paper/` — architecture specification and benchmark results
- `docs/BENCHMARK_RESULTS.md` — Phase-R reproducibility dashboard
- IQS v0.8 formula and weight checksums
- Snapshot v16 telemetry format specification

### Commercial components (not included)
- DSP mastering chain, IQS scoring engine, optimizer runtime, operator registry

Available under commercial licensing — contact info@amaimedia.com.

---

## Reproducibility

```bash
export CUBLAS_WORKSPACE_CONFIG=:4096:8
python run_phase_r.py --mode studio --seed 42 --genre ambient
```

See **Appendix F** of the paper for the complete step-by-step protocol.

---

## Citation

```bibtex
@unpublished{bolotnikov2026noesis,
  title  = {{NOESIS}: {D}eterministic {H}ybrid {C}ontrol {F}ramework
             over {F}rozen {N}eural {O}perator with
             {O}bjective-locked {O}ptimization ({DHCF-FNO})},
  author = {Bolotnikov, Ilia},
  year   = {2026},
  note   = {Submitted to {IEEE} Trans.\ Audio, Speech, Language Process.},
  url    = {https://github.com/AMAImedia/noesis-dhcf-fno}
}
```

---

*© 2026 AMAImedia · Ilia Bolotnikov · info@amaimedia.com*

## Architecture State (2026-03-21)

| Component | Version | Notes |
|-----------|---------|-------|
| MasteringChain | v3.4, 31 stages | Ozone 12 parity 11/11 |
| IQS | v0.8, sealed | checksums 9097e760 / 12c2f47c |
| NOESIS-MOS | v1, r=0.837 | FMA-small pseudo-labels |
| Genre profiles | 38 canonical | +phonk/hyperpop/darkwave/shoegaze/... |
| SC text rules | 84 keyword rules | EDM/vocal/emotional/platform |
| Taxonomy | 1049+55 entries | TF-IDF + Qwen3.5-0.8B fallback |
| SVC | v1.0, step 6000 | Seed-VC, f0_condition=True |
| DisCoder | MUSHRA 88.14 | ICASSP 2025, post-chain re-vocoder |
| Caption LoRA | training | Qwen3.5-0.8B, 579 pairs, ~2h CPU |

## Benchmark History

| Date | IQS_mean | J_mean | Chain | Notes |
|------|----------|--------|-------|-------|
| 2026-03-12 | 0.524 | 0.315 | v1.8 (22 stages) | R.REF3, post-CFG-fix |
| 2026-03-14 | 0.614 | 0.388 | v3.4 (31 stages) | R.REF2, FAD(PANN) wired |
| ~2026-03-21 | >0.62 | >0.40 | v3.4 | R.REF4 GPU target |


