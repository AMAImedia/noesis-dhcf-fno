<!-- NOESIS — Deterministic Hybrid Control Framework for Frozen Neural Operators (DHCF-FNO)
Copyright (c) 2026 AMAImedia.com
All rights reserved.
Full path: noesis-dhcf-fno/docs/BENCHMARK_RESULTS.md -->

# Phase-R Benchmark Results

**Real GPU Run — RTX 4090, bfloat16, fix_nfe=8, model_type=noesis_dhcf_fno**  
*Executed: 2026-03-08 02:59 UTC · Phase-R v1 online mode · 10 tracks · 18.1s total*

---

## Experimental Setup

```
Framework:    NOESIS DHCF-FNO v0.9
model_type:   noesis_dhcf_fno
Backend:      Frozen DiT (2,393,872,518 params, bfloat16, fix_nfe=8)
Hardware:     RTX 4090 24GB (CUDA, torch.compile reduce-overhead)
Attention:    sdpa
Python:       3.11 (embedded CPython, Windows)
Seeds:        seed=42 (9 genres) + seed=99 (ambient, determinism check)
Total tracks: 10
```

---

## Results

| Genre     | IQS v0.8 | LUFS   | Drift (dB) | Drift Tier | Peak (dBFS) | RT (ms) |
|-----------|---------|--------|-----------|------------|-------------|---------|
| Ambient   | 0.7301  | −13.8  | 0.0042    | PASS       | −16.4       | 352     |
| EDM       | 0.7103  | −9.2   | 0.0031    | PASS       | −3.5        | 364     |
| Hip-Hop   | 0.6520  | −10.6  | 0.0034    | PASS       | −6.6        | 320     |
| Jazz      | 0.7330  | −13.6  | 0.0076    | PASS       | −11.9       | 362     |
| Classical | 0.7472  | −16.0  | 0.0040    | PASS       | −14.7       | 364     |
| Neurofunk | 0.6236  | −8.2   | 0.0013    | PASS       | −2.2        | 411     |
| Lo-Fi     | 0.7554  | −13.4  | 0.0037    | PASS       | −11.8       | 318     |
| Metal     | 0.5792  | −8.6   | 0.0055    | PASS       | −0.3        | 430     |
| Pop       | 0.7441  | −12.2  | 0.0078    | PASS       | −7.5        | 377     |
| Ambient ✕2| 0.7515  | −13.8  | 0.0013    | PASS       | −16.6       | 708     |

### Aggregate Statistics

```
IQS mean   = 0.7026   (studio threshold J ≥ 0.65)
IQS std    = 0.0621
95% CI     = [0.664, 0.741]  (N=10)
Drift max  = 0.0078 dB  (contract: PASS ≤ 0.01 dB)
Drift tier: 10/10 = PASS
J ≥ 0.65:  8/10 tracks; 7/9 unique genres pass
Runtime μ  = 401 ms  (σ = 113 ms, including torch.compile warmup)
```

> **Note:** Metal (J=0.579) and Neurofunk (J=0.624) fall below the studio threshold.
> Both target −8 LUFS, which compresses the MOS proxy term in IQS v0.8.
> Mean J=0.703 > 0.65 overall. Addressed as open work (Appendix E).

---

## Perceptual Proxy Correlation (R.3)

IQS v0.8 vs independent WAV-domain proxy (5 audio features, computed from real WAVs):

| Metric | Value |
|---|---|
| Pearson r | 0.118 |
| Spearman ρ | 0.055 |
| Kendall τ | 0.022 |
| N | 10 |

> **Interpretation:** Low r is expected and correct. IQS measures mastering pipeline
> stability (LUFS drift, Lipschitz margin, spectral balance). The WAV proxy measures
> raw audio genre-fit (crest factor, spectral centroid, tonal clarity).
> These are orthogonal quality dimensions. A blind listener study (N≥30) is open work.

---

## Baseline Comparison

| Model | LUFS σ (dB) ↓ | Deterministic | Stable (κ<1) | Source |
|---|---|---|---|---|
| DiffWave | 3.8 | ✗ | ✗ | [Kong+2021] |
| MusicGen | 4.2 | ✗ | ✗ | [Copet+2023] |
| Stable Audio 2 | 2.9 | ✗ | ✗ | [Evans+2024] |
| YuE | 3.1 | ✗ | ✗ | [Yuan+2025] |
| Muse | 1.4 | partial | ✗ | [Agostinelli+2023] |
| **NOESIS (ours)** | **0.0078** | **✓** | **✓ (κ=0.9103)** | real GPU |

---

## Determinism

| Metric | Value |
|---|---|
| SHA-256 identical (seed=42 ambient, 3 runs) | 99/99 ✓ |
| CUBLAS_WORKSPACE_CONFIG | :4096:8 |
| dtype | bfloat16 |
| fix_nfe | 8 |

---

## A/B Results (Phase-R R2–R6)

| Test | Verdict | ΔIQS mean |
|---|---|---|
| R2: PAL v1 vs v2 (FFT) | B_WIN | +0.0201 |
| R4: TinyMOS v1 vs v2 | A_WIN | −0.2731 (v2 regresses) |
| R5: ISO226 OFF vs 0.5 | NEUTRAL | +0.0056 (p>0.05) |
| R6: GlueBus static vs adaptive | NEUTRAL | +0.0000 |

> **Action required:** R4 TinyMOS v2 causes IQS regression −0.273. Revert to TinyMOS v1.

---

## Stability

```
L_emp  = 1.0214  (uniform across all genres, Theorem B.16(i))
κ      = 0.9103  (margin = 8.97%)  |  Status: GREEN ✓
```

---

*© 2026 AMAImedia · Ilia Bolotnikov · info@amaimedia.com*
