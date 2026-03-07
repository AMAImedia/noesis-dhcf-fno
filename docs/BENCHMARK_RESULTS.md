<!-- NOESIS — Deterministic Hybrid Control Framework for Frozen Neural Operators (DHCF-FNO)
Copyright (c) 2026 AMAImedia.com
All rights reserved.
Full path: docs/BENCHMARK_RESULTS.md
Version: v1.1 (2026-03-07) — adds real Phase R.1 GPU results
Supersedes: v1.0 (synthetic pipeline evaluation only) -->

# Phase-R Benchmark Results

**Research Reproducibility Dashboard**
*NOESIS DHCF-FNO v1.3 — Deterministic AI Music Generation + Mastering*

---

## Phase R.1 — Real GPU Results (2026-03-07)

```
Framework:    NOESIS DHCF-FNO v1.3 (mastering_chain.py v1.3)
Model:        ACE-Step v1.5 Turbo DiT (2.4B params, bfloat16, fix_nfe=8)
Text enc.:    Qwen3-Embedding-0.6B (595M params, hidden=1024)
Lyric enc.:   acestep-5Hz-lm-0.6B (662M params, vocab=217204, CPU offload)
Hardware:     CUDA GPU (bfloat16)
Python:       3.11.9 (embedded CPython)
IQS:          v0.8 (α=0.50 β=0.08 γ=0.07 δ=0.06 η=0.25 ζ=0.04)
Snapshot:     v16 (Merkle tree + crypto audit trail)
Mode:         studio  |  Seeds: 42 (×9 genres), 99 (ambient 60s)
```

### Per-Genre Results (seed=42, 30s tracks)

| Genre     | IQS    | LUFS  | Drift (dB) | Drift Tier  | Peak (dBFS) | Pass |
|-----------|--------|-------|------------|-------------|-------------|------|
| ambient   | 0.4964 | -14.4 | 0.4260     | PASS_CF_LIM | -1.0        | ✅   |
| edm       | 0.4727 |  -9.9 | 0.9160     | PASS_CF_LIM | -1.0        | ✅   |
| hip-hop   | 0.5206 | -11.6 | 0.6310     | PASS_CF_LIM | -1.2        | ✅   |
| jazz      | 0.5006 | -14.6 | 0.6370     | PASS_CF_LIM | -2.4        | ✅   |
| classical | 0.5254 | -16.1 | 0.1350     | PASS_CF_LIM | -1.0        | ✅   |
| neurofunk | 0.4622 |  -9.3 | 1.2660     | PASS_CF_LIM | -1.1        | ✅   |
| lofi      | 0.5074 | -14.6 | 0.6310     | PASS_CF_LIM | -2.5        | ✅   |
| metal     | 0.4626 |  -9.6 | 1.6090     | PASS_CF_LIM | -1.0        | ✅   |
| pop       | 0.4839 | -13.1 | 1.0720     | PASS_CF_LIM | -1.8        | ✅   |
| ambient99 | 0.5155 | -18.0 | 0.0000     | PASS        | -3.1        | ✅   |

### Aggregate Statistics

```
IQS:         mean=0.4947  std=0.0222  min=0.4622  max=0.5254
L_emp:       1.0214 (uniform across all genres, Theorem B.16(i))
κ:           0.9103  |  Stability margin: 0.090 (§EC2 GREEN)
Drift tiers: PASS=1  PASS_CF_LIMITED=9  FAIL=0
Pass rate:   10/10
Snapshot:    v16  |  99/99 QA PASS
```

### Determinism Verification

```
§DIAG-LATENT: all diff_from_silence in [0.597, 0.813]  →  healthy DiT output ✓
Operator graph checksum: f1d8a82c7d6e859667cddb7732b869ba3b82926501719e6da78b947edc3dd2ba
Baseline freeze: 11/11 invariants intact (noesis_baseline.json)
```

---

## IQS Note: CF-Limited Drift Penalty

The observed IQS mean (0.4947) is lower than the studio-quality gate
(J ≥ 0.65 requires IQS ≥ 0.537 with QA_ext=0.82) because 9/10 tracks are
**CF-limited**: the mastering pipeline cannot reach the aggressive LUFS targets
for hot genres (EDM -9, neurofunk -8, metal -8 dBLUFS) due to crest-factor physics.

High drift → high L_n term → IQS penalty. This is correct behavior: IQS
accurately reflects that the mastering target was not fully achieved.
The three-tier drift contract correctly classifies these as PASS_CF_LIMITED
(not FAIL) since CF-limitation is physically justified.

**To improve IQS beyond 0.537:** tune Stage 8.5 pre-limiter to reduce CF
before normalization, creating more headroom for hot genres.

---

## A/B Test Summary (Phase R.2–R.6)

| Test           | Verdict | ΔIQS   | Conclusion                              |
|----------------|---------|--------|-----------------------------------------|
| R2: PAL v1→v2  | B_WIN   | +0.020 | PAL v2 (FFT) adopted ✅                 |
| R4: TinyMOS v1→v2 | A_WIN | −0.273 | TinyMOS v1 retained; v2 reverted ✅    |
| R5: ISO226 0→0.5 | NEUTRAL | +0.006 | Pending online dedicated A/B run       |
| R6: GlueBus    | NEUTRAL | +0.000 | Pending online dedicated A/B run       |

---

## Stability

```
L_emp = 1.0214  |  G_max = 4.0  |  κ = L_emp/G_max = 0.9103
Stability margin = 1 − κ = 0.0897 ≈ 9.0%  (§EC2: GREEN)
Risk envelope = ln(κ) + ln(L_emp) = −0.094 < 0  (Theorem E.1 satisfied)
Theorem B.26: κ < 1  ✓
```

---

## Snapshot v16 — Sample Record (ambient seed=42)

```json
{
  "snapshot_version": 16,
  "seed": 42,
  "IQS": 0.4964,
  "integrated_lufs": -14.4,
  "loudness_drift": 0.426,
  "operator_graph_checksum": "f1d8a82c7d6e859667cddb7732b869ba3b82926501719e6da78b947edc3dd2ba",
  "stability_margin": 0.090,
  "empirical_Lipschitz": 1.0214,
  "stability_status": "GREEN"
}
```

---

## Validation Suite

```
test_snapshot_v16.py:  99/99 PASSED  (53.8 ms)
noesis_freeze_baseline --verify:  11/11 ALL INVARIANTS INTACT
Phase R.1 pass rate:  10/10
```

---

## Historical Reference: Synthetic Pipeline Baseline (pre-GPU)

> The following data was measured on `phase_r_synthetic.py` (offline, no DiT).
> It served as a pre-GPU calibration target and is retained for audit trail only.
> **Do not use for production benchmarking — use Phase R.1 GPU results above.**

| Genre     | IQS (synthetic) | LUFS  | Drift | UTMOS | DNSMOS |
|-----------|-----------------|-------|-------|-------|--------|
| Ambient   | 0.7398          | -14.3 | 0.000 | 3.672 | 3.581  |
| Classical | 0.7631          | -16.1 | 0.000 | 3.729 | 3.638  |
| Lo-Fi     | 0.7135          | -13.7 | 0.000 | 3.814 | 3.723  |
| Jazz      | 0.7673          | -14.1 | 0.000 | 3.495 | 3.411  |
| Pop       | 0.6984          | -11.8 | 0.000 | 3.538 | 3.443  |
| Hip-Hop   | 0.6953          | -10.6 | 0.000 | 3.486 | 3.382  |
| EDM       | 0.6616          |  -8.7 | 0.000 | 3.666 | 3.562  |
| Neurofunk | 0.6296          |  -8.3 | 0.000 | 3.784 | 3.676  |
| Metal     | 0.6242          |  -7.5 | 0.000 | 3.486 | 3.381  |

*Synthetic mean IQS=0.699 reflects zero-drift ideal conditions (no CF-limitation).*
*Real GPU IQS=0.4947 reflects CF-limited mastering — both are physically correct.*

---

*© 2026 AMAImedia · Ilia Bolotnikov · info@amaimedia.com*
*Version: v1.1 (2026-03-07)*
*Supersedes: BENCHMARK_RESULTS.md v1.0 (synthetic data only)*
