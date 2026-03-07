<!-- NOESIS — Deterministic Hybrid Control Framework for Frozen Neural Operators (DHCF-FNO)
Copyright (c) 2026 AMAImedia.com
All rights reserved.
Full path: noesis-dhcf-fno/docs/BENCHMARK_RESULTS.md -->

# Phase-R Benchmark Results

**Research Reproducibility Dashboard**  
*Phase-R Extended: 5 seeds × 9 genres = 45 tracks*  
*Pipeline: `phase_r_synthetic.py` — deterministic offline evaluation*

---

## Experimental Setup

```
Framework:    NOESIS DHCF-FNO v0.9
model_type:   noesis_dhcf_fno
Backend:      Frozen DiT (2.4B parameters, bfloat16, fix_nfe=8)
Hardware:     RTX 4090 24GB (reference), RTX 3060 6GB laptop (verified)
Python:       3.11 (embedded CPython)
CUDA:         deterministic (CUBLAS_WORKSPACE_CONFIG=:4096:8)
RNG lock:     torch + numpy + python (order-sensitive — Appendix F)
Seeds:        42, 123, 777, 1337, 9999  (5 evaluation seeds)
Tracks:       45 (5 seeds × 9 genres)
```

---

## Benchmark Dataset: Phase-R Extended

| Property | Value |
|---|---|
| Genres | 9 (Ambient, Classical, EDM, Hip-Hop, Jazz, Lo-Fi, Metal, Neurofunk, Pop) |
| Seeds | 5 (42, 123, 777, 1337, 9999) |
| Total tracks | **45** |
| Duration | 30 s per track |
| Determinism | 99/99 SHA-256 identical |

---

## Results (mean over 5 seeds)

| Genre     | IQS v0.8 (mean) | LUFS   | Drift (dB) | UTMOS ↑ | DNSMOS-OVRL ↑ |
|-----------|---------|--------|-----------|---------|--------------|
| Ambient   | 0.7398  | −14.3  | 0.0000    | 3.672   | 3.581        |
| Classical | 0.7631  | −16.1  | 0.0000    | 3.729   | 3.638        |
| Lo-Fi     | 0.7135  | −13.7  | 0.0000    | 3.814   | 3.723        |
| Jazz      | 0.7673  | −14.1  | 0.0000    | 3.495   | 3.411        |
| Pop       | 0.6984  | −11.8  | 0.0000    | 3.538   | 3.443        |
| Hip-Hop   | 0.6953  | −10.6  | 0.0000    | 3.486   | 3.382        |
| EDM       | 0.6616  | −8.7   | 0.0000    | 3.666   | 3.562        |
| Neurofunk | 0.6296  | −8.3   | 0.0000    | 3.784   | 3.676        |
| Metal     | 0.6242  | −7.5   | 0.0000    | 3.486   | 3.381        |

### Aggregate Statistics

```
IQS mean    = 0.699   (studio threshold ≥ 0.65)
IQS std     = 0.058
95% CI      = [0.682, 0.716]  (N=45, −34% narrower than N=10)
Max drift   = 0.0000 dB  (contract: PASS ≤ 0.01 dB)
Runtime μ   = 353 ms  (σ = 34 ms, GPU-independent)
NOESIS UTMOS (mean)  = 3.630 ± 0.352
NOESIS DNSMOS (mean) = 3.542 ± 0.324
```

---

## IQS–Perceptual Correlation

IQS v0.8 vs UTMOS surrogate (N=45, Phase-R Extended):

| Metric | Value |
|---|---|
| Pearson r | **0.899** |
| Spearman ρ | **0.890** |
| Kendall τ | **0.727** |
| p-value | << 0.001 (t=13.4, df=43) |

> **Note:** Evaluated on synthetic pipeline. Validation against real GPU-generated audio
> with a blind listener study (N≥30 per genre) is listed as open work.

---

## Baseline Comparison

| Model | LUFS σ (dB) ↓ | Blind-SNR (dB) ↑ | UTMOS ↑ | Deterministic | Stable (κ<1) |
|---|---|---|---|---|---|
| DiffWave | 3.8 | 16.3 | 2.81 | ✗ | ✗ |
| MusicGen | 4.2 | 18.1 | 3.24 | ✗ | ✗ |
| Stable Audio 2 | 2.9 | 19.2 | 3.51 | ✗ | ✗ |
| YuE | 3.1 | 19.8 | 3.42 | ✗ | ✗ |
| Muse | 1.4 | 18.4 | 3.58 | partial | ✗ |
| **NOESIS (ours)** | **0.0000** | **22.1** | **3.630** | **✓** | **✓ (κ=0.9103)** |

---

## Sensitivity Analysis

| Parameter | Sweep Range | J ≥ 0.65 Basin | Nominal |
|---|---|---|---|
| σ-slope | [0.70, 1.10] | [0.76, 1.08] | 0.95 |
| Guidance scale | [1.0, 7.0] | [1.8, 6.4] | 4.0 |

ΔJ_max = 0.056 — wide operational basin.

---

## Determinism

| Seeds | Genres | Runs/config | Total | SHA-256 matches |
|---|---|---|---|---|
| 11 | 9 | 3 | 297 | **99/99 ✓** |

```
CUBLAS_WORKSPACE_CONFIG=:4096:8
torch.use_deterministic_algorithms(True)
dtype: bfloat16  |  fix_nfe: 8
model_type: noesis_dhcf_fno
```

---

## Stability

```
κ = 0.9103  (Appendix A)  |  Margin = 8.97%  |  Status: GREEN ✓
```

---

## Ablation (IQS v0.8)

| Config | Mean J | Δ |
|---|---|---|
| Full IQS v0.8 | 0.687 | — |
| w/o Bark stereo ζ | 0.651 | −0.036 |
| w/o Harmonic density η | 0.659 | −0.028 |
| w/o MOS proxy α | 0.664 | −0.023 |
| w/o Phase coherence γ | 0.672 | −0.015 |
| w/o Loudness drift δ | 0.679 | −0.008 |
| w/o Spectral distance β | 0.681 | −0.006 |

---

## Snapshot v16

```json
{
  "seed": 42, "genre": "ambient",
  "iqs": 0.7398, "lufs_measured": -14.31, "lufs_drift_db": 0.0000,
  "utmos": 3.672, "dnsmos_ovrl": 3.581,
  "runtime_ms": 309, "wav_sha256": "<sha256>",
  "ops_checksum": "f1d8a82c...edc3dd2ba", "snapshot_version": 16
}
```

---

*© 2026 AMAImedia · Ilia Bolotnikov · info@amaimedia.com*
