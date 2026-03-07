"""NOESIS — Deterministic Hybrid Control Framework for Frozen Neural Operators (DHCF-FNO)
Copyright (c) 2026 AMAImedia.com
All rights reserved.
docs/NOESIS_ENGINEERING_CONTRACT_v0_8.md"""

# NOESIS ENGINEERING CONTRACT v0.8

```
Version:    v0.8  (2026-03-07)
Author:     Ilia Bolotnikov / AMAImedia.com (2026)
Status:     ACTIVE
Supersedes: NOESIS_ENGINEERING_CONTRACT_v0_7.md
Changes v0.8:
  FIXED §1:   M_core now references Theorem B.16(i) explicitly
  FIXED §4:   IQS formula synced to 6-term v0.9 canonical
  FIXED §6:   snapshot core += plan_delta_checksum, mastering_merkle_root,
              segment_styles_checksum
  FIXED §8:   canonical stage order — MonoBass is Stage 0 (MidSideHP removed)
  FIXED §9.7: TruePeak abort = −0.1 dBTP (was incorrectly 1.0)
  FIXED §16:  external models include KAD backend
  NEW §29:    KAD Invariant (distribution-free distance, Theorem F.1)
  NEW §30:    True-Peak Contract (4x oversampling FIR, BS.1770-4)
  NEW §31:    Phase Alignment + Crossover + Merkle Tree (C.1/C.3/H.4)
  NEW §32:    Segment Control Contract (14-tag, PlanDelta, checksums)
  NEW §33:    Memory Consolidation Contract (/sleep, CPM)
  NEW §34:    Generator Agnosticism Contract (multi-backend)
  NEW §35:    Coding Architecture Rule (orchestrator + helpers)
  Updated §21/§27: noesis_dhcf_fno model_type, renamed model files
```

---

# PART A — CORE STABILITY ASSERTIONS (SS1-SS10)

## SS1. Scope

This contract governs:
- **M_core** — linear mastering operator [**Theorem B.16(i)**, not generic B.16]
- **M_pre** — adaptive stages (pre_graph)
- **IQS optimizer** — theta-update [Theorem B.10, trust-region BFGS]
- **Snapshot telemetry** — immutable JSONL [Theorem H.4]
- **Switching logic** — finite regime partition [Axiom A3]

Any violation of SS1-SS35 conditions means the system is incorrect.

---

## SS2. Stability Contract

### SS2.1 Core Stability Condition (Theorem B.26 Case I)

```
g_max * L_core < 1
```

- L_core = 1.0214 (empirical, 50 trials, seed=42)
- g_max = 0.8912 (measured ceiling)

### SS2.2 Runtime Enforcement

```python
stability_margin = 1 - g_max * L_core
# ABORT if stability_margin <= 0
```

Sealed values (2026-03-01):
```
kappa = 0.8912 * 1.0214 = 0.910
stability_margin = 0.090  (9.0%, GREEN zone)
```

### SS2.3 Logged Metric

Snapshot MUST contain: `{"stability_margin": float}`

---

## SS3. Adaptive Gain Contract

```
g(x, theta) in [g_min, g_max]
g_max = 0.8912 (measured ceiling, design: 4.0)
```

Prohibited: cumulative gain multiplication without cap, dynamic g_max growth,
recursive amplification.

---

## SS4. Optimizer Contract

### SS4.1 Trust-Region BFGS (Inner, Theorem B.10)

```
Trust region radius:  r = 0.5
Step bound:           eta <= 1.9 / lambda_max(H)
Convergence:          |IQS_k - IQS_{k-1}| < 1e-4  OR  k = k_max = 6
```

Separation invariant (HARD, Invariant #12):
BFGS receives IQS_base ONLY. NOT QA_external. NOT IQS_edit. NOT J.

### SS4.2 IQS Formula — Canonical 6-term v0.9 (SINGLE SOURCE OF TRUTH)

```
IQS = alpha*MOS_n + eta*HD_n_genre - beta*Distance_n
      - gamma*Phase_n - delta*Drift_n - zeta*Bark_n

alpha = 0.50  (MOS proxy)
eta   = 0.25  (Harmonic Density, genre-aware)
beta  = 0.08  (Distance penalty, KAD primary)
gamma = 0.07  (Phase coherence penalty)
delta = 0.06  (Loudness drift penalty)
zeta  = 0.04  (Bark stereo coherence penalty)
              Sum = 1.00  (Invariant #7, IQS_WEIGHTS_CHECKSUM sealed)

IQS_max = alpha + eta = 0.75   (Lemma B.5-A)
IQS_min = -(beta+gamma+delta+zeta) = -0.25
IQS in [-0.25, 0.75]

HD_n_genre = min(1.0, H_raw / floor_genre)
Genre floors: {ambient:0.25, classical:0.20, jazz:0.30, lofi:0.30,
               edm:0.65, metal:0.60, neurofunk:0.62, default:0.45}
```

Implementation file: `metrics/iqs.py` (IQS_VERSION="0.9")

### SS4.3 Quality Fusion J

```
J = 0.60 * IQS + 0.40 * QA_external
Studio threshold: J >= 0.65
```

CoordinateSearch optimizes J. BFGS optimizes IQS_base only (SS11.3).

### SS4.4 Gradient Norm Monitoring

Snapshot MUST contain: `{"gradient_norm": float}`

---

## SS5. Switching Contract (Theorem H.2)

- Finite modes (Axiom A3: finite partition)
- No Zeno behavior (dwell time >= T_block = 256/sr)
- Snapshot MUST contain: `{"switching_count": int}`

---

## SS6. Snapshot v16 Contract

### SS6.1 Core Block (ALL fields mandatory — enter SHA-256)

```json
{
    "schema_version":              16,
    "seed":                        "int",
    "structure_plan_checksum":     "SHA-256",
    "sigma_checksum_scheduler":    "SHA-256",
    "latent_checksum_initial":     "SHA-256",
    "latent_checksum_final":       "SHA-256",
    "wav_checksum":                "SHA-256",
    "IQS":                         "float in [-0.25, 0.75]",
    "J":                           "float in [0, 1]",
    "iqs_weights_checksum":        "SHA-256",
    "operator_graph_checksum":     "SHA-256",
    "operator_versions":           "dict {name: version}",
    "plan_delta_checksum":         "SHA-256",
    "segment_styles_checksum":     "SHA-256",
    "mastering_merkle_root":       "SHA-256",
    "stability_margin":            "float",
    "gradient_norm":               "float",
    "theta_norm":                  "float",
    "switching_count":             "int",
    "loudness_drift":              "float (<=0.01 dB)",
    "integrated_lufs":             "float",
    "prompt_hash":                 "SHA-256",
    "diffusion_lipschitz_bound":   "float",
    "torch_version":               "str",
    "dtype":                       "float32",
    "sample_rate":                 48000,
    "deterministic_lock_enabled":  true,
    "runtime_fingerprint":         {"...fingerprint_checksum": "SHA-256"},
    "quality_fusion_J":            "float in [0,1] | null",
    "quality_fusion_confidence":   "float in [0,1] | null",
    "quality_fusion_weights_checksum": "SHA-256 | null"
}

snapshot_checksum = SHA-256(json_sort(all_core_fields_above))
```

New fields in v0.8 (all enter core SHA-256 hash):
- `plan_delta_checksum` — SHA-256 of PlanDelta (SS32)
- `segment_styles_checksum` — SHA-256 of segment style list (SS32)
- `mastering_merkle_root` — Merkle root over 8 mastering stage checksums (SS31)

### SS6.2 Non-Hash Extensions

Appended to snapshot JSON, excluded from SHA-256:
```json
"ebu_r128":              {"integrated_lufs": -14.0, "lra_db": 7.3, "true_peak_dbtp": -0.8},
"distance_metrics":      {"primary_metric": "KAD_PANN", "kad_score": 0.043},
"stem_eq":               {"version": "v1.0", "checksum": "..."},
"perf_profile":          {"diffusion_ms": 1840, "mastering_ms": 12, "vram_peak_mb": 4812},
"mastering_merkle_tree": {"root": "...", "leaves": {"mono_bass": "cs0", ...}},
"delta_user_minus_iqs":  "float"
```

### SS6.3 Three-Tier LUFS Drift Contract

```
PASS:             drift <= 0.01 dB
PASS_CF_LIMITED:  0.01 < drift <= 2.0 dB  AND  CF-physically-limited
FAIL:             drift > 2.0 dB  OR  unexplained
```

### SS6.4 Validation Aborts

```
NaN or Inf in audio          -> ValueError
IQS not in [-0.25, 0.75]     -> RuntimeError  (Lemma B.5-A)
loudness_drift > 0.01 dB     -> RuntimeError  (Invariant #5)
J not in [0, 1] when present -> RuntimeError
```

---

## SS7. Risk Envelope

```
E[log g_q] + log L_core = -0.094 < 0  -> almost-sure stable (Theorem E.1)
```

---

## SS8. Determinism Clause

**One seed -> one WAV -> one snapshot_checksum.**

### SS8.1 Canonical Stage Order (IMMUTABLE — Invariant #4)

CORRECTED v0.8: MonoBass is Stage 0 (previously MidSideHP was incorrectly listed).

```
[0]  MonoBass              sub-bass mono enforcement (f < 120 Hz)
[1]  SpectralTilt          frequency slope correction
[2]  LUFSSlope             integrated loudness targeting
[3]  Multiband             4-band dynamics processing
[4]  GlueBus               glue compressor, bus processing
[5]  BarkMask              perceptual masking, 24 Bark bands
[6]  ModCoherence          modulation coherence normalization
[7]  Limiter (4x ISP-safe) true-peak limiting, 4 cascaded stages
[8]  LUFSTrim              final LUFS target normalization
[8.5] Pre-Limiter          two-pass adaptive pre-limiter (optional)
[9]  PostLimit             softclip + LUFS correction
[10] VST3 (optional)       FrozenVSTOperator post-trim
[11] Hard-Clamp (always)   clip to ceiling_lin = -1.0 dBTP
```

### SS8.2 Generation Artifact Guard (Stage -1, pre-mastering)

```python
ARTIFACT_FLOOR_LUFS  = -55.0  # below -> GenerationArtifactError
PRE_LIFT_TARGET_LUFS = -35.0  # lift if below this
GAIN_CAP             = +60.0  # dB, maximum lift
FLOOR_LUFS           = -69.0  # true silence floor
```

If integrated_LUFS < ARTIFACT_FLOOR_LUFS: raise GenerationArtifactError (retry up to 3x).

---

## SS9. Abort Conditions

| ID    | Condition                            | Action                   | Reference  |
|-------|--------------------------------------|--------------------------|------------|
| SS9.1 | stability_margin <= 0                | RuntimeError at init     | B.26       |
| SS9.2 | g(x,theta) > g_max                   | Clamp + warning          | B.14       |
| SS9.3 | norm(grad_IQS) > 100*median(history) | Rollback theta           | B.10       |
| SS9.4 | kappa(H) > 1e5                       | Hessian reset H = I      | B.8        |
| SS9.5 | NaN or Inf in audio                  | ValueError               | A7         |
| SS9.6 | LUFS drift > 0.01 dB                 | RuntimeError             | B.14       |
| SS9.7 | TruePeak >= -0.1 dBTP  [FIXED v0.8]  | RuntimeError             | Inv. #6    |
| SS9.8 | IQS not in [-0.25, 0.75]             | RuntimeError             | B.5-A      |
| SS9.9 | IIR pole >= 1 at init                | ValueError               | Inv. #10   |
|SS9.10 | GenerationArtifactError              | Retry up to 3x new seed  | SS8.2      |

---

## SS10. Core Guarantees

If SS1-SS9 hold:
- Boundedness (Theorem B.14)
- Exponential stability (Theorem B.26, margin > 0)
- Almost-sure convergence theta -> theta* (Theorem E.3)
- Reproducibility (one seed -> one wav -> one checksum, Theorem H.4)

---

# PART B — EXTENDED HIERARCHY AND MODES (SS11-SS17)

## SS11. Hierarchical Optimization Contract

### SS11.1 Inner Optimizer (BFGS, Theorem B.10)

```
Input:  theta in Omega = [5.0,7.5] x [-0.2,0.2] x [0.35,0.55]
Output: theta* = argmax IQS(theta)
Method: Trust-region BFGS, r=0.5, k_max=6
File:   optimization/bfgs_optimizer.py
```

### SS11.2 Outer Meta-Optimizer (CoordinateSearch)

```
Domain: |Theta_grid| <= 450 (6x5x5x3 seeds)
Sweep:  guidance -> slope -> entropy (sequential)
Method: Deterministic grid + inner BFGS per point
File:   optimization/coordinate_search.py
```

### SS11.3 Separation Invariant (HARD — Invariant #12)

```
BFGS       -> IQS_base ONLY  (not QA_external, not IQS_edit, not J)
CSO        -> J (outer, sees QA_external)
CLEngine   -> orchestrator only (no formula logic)
```

### SS11.4 Hierarchy

```
Level 3 — CoordinateSearch  (outer, optimizes J)
    |
Level 1 — BFGS              (inner, optimizes IQS_base)
    |
Level 0 — M_fixed o D_theta (signal processing)
```

---

## SS12. Rule-Based Correction Contract

```
Delta_theta = f_rule(metrics, genre_profile)
Bounds:  Delta_theta_i in [-0.1, +0.1] per step
No stochastic component, no gradient estimation
File: optimization/rule_based_corrector.py
```

---

## SS13. Mode Contract

### SS13.1 Studio Mode

```
J_threshold = 0.65,  C_threshold = 0.60,  seed_iterations <= 3
Full hierarchical: CoordinateSearch -> BFGS -> QualityFusion -> Snapshot v16
```

### SS13.2 Fast Mode

```
J_threshold = 0.50 (relaxed), no rejection
Single-pass: fixed theta -> DiT -> Mastering -> IQS
```

Mode selected at startup. No runtime switching. File: `cli/generation_router.py`

---

## SS14. Snapshot v16 Contract (shorthand)

See SS6 for complete schema. Key rules:
- `snapshot_checksum = SHA-256(json_sort(core_block))`
- Non-hash extensions appended after core
- `loudness_drift <= 0.01 dB` (three-tier contract, SS6.3)

---

## SS15. Extended Abort

| ID     | Condition                            | Action            |
|--------|--------------------------------------|-------------------|
| SS15.1 | J < threshold AND mode=Studio        | Reject + seed++   |
| SS15.2 | C < 0.60 (confidence low)            | Warning + log     |
| SS15.3 | seed_iterations > 3                  | Hard stop         |
| SS15.4 | Outer loop > 450 evaluations         | Terminate CSO     |
| SS15.5 | External model non-deterministic     | Fallback to internal MOS |

---

## SS16. External Model Safety Contract (updated v0.8)

```
Model            Metric    Device    Range         Status
TinyMOS v1       MOS       CPU       [1.0, 5.0]    PRODUCTION_FORCE_V1=True
DNSMOS v4        OVRL      CPU       [1.0, 5.0]    active
KAD (PANN)       KAD       GPU/CPU   >= 0          PRIMARY distance (v0.8)
FAD-CLAP-MA      FAD       GPU       >= 0          secondary
FAD-VGGish       FAD       GPU       >= 0          legacy fallback

All backend checksums in snapshot:
  external_model_checksums.kad_pann, .fad_clap, .tinymos, .dnsmos, .fad_ref

Fallback chain: KAD(PANN) -> FAD-CLAP-MA -> FAD-VGGish (Invariant #16)
Determinism check at startup: same input -> same output (3 trials).
```

---

## SS17. Extended Guarantees (SS11-SS16)

If SS1-SS16 hold, additionally:
- Hierarchical convergence (Theorem A.5)
- Finite termination outer loop (Proposition A.4)
- Quality gate: J < threshold -> reject (SS15.1)
- External model determinism verified at startup (SS16)

---

# PART C — FROZEN NEURAL OPERATOR BOUNDARY (SS18-SS28)

## SS18. Architecture Constants (ACE-Step reference backend — FROZEN)

```
sample_rate:         48000 Hz   (FROZEN in VAE hop_length=1920)
latent_frame_rate:   25 Hz      (48000/1920)
acoustic_dim:        64         (VAE latent channels)
text_hidden_dim:     1024       (DiT conditioning dim)
fix_nfe:             8          (TURBO ONLY, FROZEN — 32 steps = pure noise)
dtype DiT:           bfloat16   (NOT float16 — NaN overflow risk)
dtype mastering:     float32    (NOESIS, strict, Invariant #10)
shift values:        {1.0, 2.0, 3.0} ONLY
silence_latent:      [1, 15000, 64] after transpose from [1, 64, 15000]
```

## SS19. Prompt Format (FROZEN — ACE-Step reference)

```
# Instruction
Fill the audio semantic mask based on the given conditions:

# Caption
{caption_text}

# Metas
- bpm: {bpm_or_NA}
- timesignature: {timesig_or_NA}
- keyscale: {key_or_NA}
- duration: {int_seconds} seconds
<|endoftext|>
```

Rules: exact headers, `- key: value\n` format, `<|endoftext|>` immediately after last meta.

Lyrics format:
```
# Languages
{language_code}

# Lyric
{lyrics_or_[instrumental]}<|endoftext|>
```

## SS20. Text Encoder Contract (FROZEN)

```
Caption encoding:
  tokenizer(text, max_length=256)
  outputs = text_encoder(input_ids=...)  # NO attention_mask in forward call
  text_attention_mask = mask.bool()      # must be bool dtype

Lyric encoding:
  tokenizer(lyrics, max_length=2048)
  lyric_hidden = text_encoder.embed_tokens(input_ids)  # embed_tokens ONLY
  lyric_attention_mask = mask.bool()                   # must be bool dtype
```

## SS21. Diffusion Model — v0.8 Note

```
model_type: "noesis_dhcf_fno"  (renamed from "acestep" in Phase S)
File: models/noesis_v15_turbo/modeling_noesis_v15_turbo.py

For alternative backends (SS34):
  Any backend satisfying Definition G.1 (PROTOCOL v0.8 SS G.1) is valid.
  Interface: generate(caption, seed, duration_s, sr=48000) -> np.ndarray
```

## SS22-SS25. VAE, dtype, Mutable/Immutable

*Unchanged from CONTRACT v0.7 SS22-SS25. See that document for full tables.*

Key rules:
- VAE input is channels-first [B, 64, T] — must transpose from model output [B, T, 64]
- VAE output [B, 2, samples] — stereo, 48 kHz
- float16 in DiT inference -> NaN overflow (use bfloat16, Invariant #10)

## SS26. Debugging Checklist (ACE-Step)

If silence/noise: check prompt headers, meta format `- key: value\n`,
mask dtypes (bool for text/lyric, float for attention), silence_latent
transposition, bfloat16 dtype. Quick test:
```python
diff = (target_latents - src_latents).abs().mean()
assert diff > 0.01, "Model generated silence — check prompt format"
```

## SS27. Frozen File Map (v0.8 naming)

```
models/
  noesis_v15_turbo/                    (renamed from acestep-v15-turbo)
    config.json                        FROZEN (model_type: "noesis_dhcf_fno")
    model.safetensors                  FROZEN (4.8 GB DiT weights)
    silence_latent.pt                  FROZEN [1, 64, 15000]
    modeling_noesis_v15_turbo.py       FROZEN (renamed from modeling_acestep*)
    configuration_noesis_v15.py        FROZEN (renamed from configuration_acestep*)
  vae/
    diffusion_pytorch_model.safetensors  FROZEN (674 MB)
  qwen3-embedding-0.6B/
    model.safetensors                  FROZEN (1.2 GB)
    tokenizer.json                     FROZEN
```

## SS28. Phase S Migration

```
Legacy "acestep" imports in production code: zero  ✓
model_type "acestep" -> "noesis_dhcf_fno":          ✓
Migration map table: NOESIS_MODULE_v0_9.md          ✓
HF Hub aliases preserved for compatibility          ✓
```

---

# PART D — EXTENDED SPECIFICATIONS (SS29-SS35)

## SS29. KAD Invariant (Theorem F.1, Invariant #16)

```
Distance metric hierarchy:
  Primary:   KAD(PANN)    unbiased MMD, distribution-free (Theorem F.1)
  Secondary: FAD-CLAP-MA  best human-preference correlation
  Tertiary:  FAD-VGGish   legacy fallback (preserved for audit continuity)

IQS Distance_n:
  Distance_n = clip(KAD(P_ref, P_gen) / KAD_max, 0, 1)
  KAD_max = 0.5  (calibrated: music vs white noise)
  Fallback: FAD_n = clip(FAD / FAD_max, 0, 1)

File: metrics/fad_backend.py (KADBackend class)
Snapshot (non-hash): distance_metrics.primary_metric = "KAD_PANN"
```

---

## SS30. True-Peak Contract (BS.1770-4, Invariants #6/#19)

```
TruePeak_dBTP = 20*log10(max|x_upsampled|)   [BS.1770-4 SS4.2]

Measurement method:
  4x polyphase FIR upsampling
  Kaiser window: beta=8.0, N_taps >= 256
  Stopband attenuation: > 80 dB
  FIR coefficients: SHA-256 locked at init (deterministic, Invariant #19)
  Lipschitz: L_TP <= 1.0 (linear upsampling, no gain)

Contract (Invariant #6, UPDATED v0.8):
  TruePeak_dBTP < -0.1 dBTP
  Abort SS9.7 if TruePeak >= -0.1 dBTP

EBU R128 additional measurements (non-hash snapshot extension):
  LRA     = P95(LUFS_3s) - P10(LUFS_3s)  [EBU R128 SS3.5]
  M_max   = max(400ms window LUFS)         [EBU R128 SS3.3]
  ST_max  = max(3s window LUFS)            [EBU R128 SS3.4]

Files: mastering/limiter_core.py, mastering/bs1770.py
Phase: P1.1
```

---

## SS31. Phase Alignment + Crossover + Merkle Tree (Invariants #20, #21)

### SS31.1 Near-Tight Crossover (Theorem C.1, Invariant #20)

```
After P1.3 (N_tap=8192, Kaiser beta=8.6):
  ||H_LP^2 + H_HP^2 - 1||_inf < 0.05
  B_f / A_f <= 1.05  (near-tight)
  L_M_fixed <= 1.05 * G_max = 4.2  (improved from 14.44 at N_tap=2048)
  Latency: 4096/48000 = 0.085 s

Before P1.3 (N_tap=2048):
  B_f/A_f = 2.708,  L_M_fixed <= 14.44

Frame bounds: SEALED after empirical P1.3 verification
File: mastering/linear_phase_fir_crossover.py
```

### SS31.2 Phase-Aligned Mono Bass (Theorem C.3)

```
Algorithm:
  phi_diff(f) = angle(X_L(f)) - angle(X_R(f)),  f < f_c = 120 Hz
  X_L'(f) = X_L(f) * exp(+j*phi_diff/2)
  X_R'(f) = X_R(f) * exp(-j*phi_diff/2)
  Mono'(f) = (X_L' + X_R') / 2

Energy guarantee (Theorem C.3):
  E_aligned(f) = |X_L(f)|^2  (rotation-invariant, unitary)
  E_aligned / E_sum = 1/cos^2(phi/2) >= 1 always

Mode "phase_aligned" is default from P2.1.
File: mastering/mono_bass_alignment_stage.py
```

### SS31.3 Mastering Merkle Tree (Theorem H.4, Invariant #21)

```
leaf_i = SHA-256(stage_name_i || stage_checksum_i)
parent  = SHA-256(left_child || right_child)
root    = mastering_merkle_root -> enters snapshot_checksum core (SS6.1)

Stages: MonoBass, SpectralTilt, LUFSSlope, Multiband,
        GlueBus, BarkMask, ModCoherence, Limiter

1-bit change in any stage -> root changes (Theorem H.4 avalanche)
File: reproducibility/snapshot_utils.py  (MerkleTree class)
Phase: P3.1
```

---

## SS32. Segment Control Contract (Phase P2.2)

```
Canonical 14-tag vocabulary (SegmentTag enum):
  INTRO, VERSE, PRE_CHORUS, CHORUS, POST_CHORUS, BRIDGE,
  BREAKDOWN, BUILD, DROP, OUTRO, INTERLUDE, SOLO, AD_LIB, TRANSITION

SegmentStyle checksum (enters snapshot core, SS6.1):
  seg.checksum = SHA-256(tag || duration_beats || style_desc || energy || density)
  segment_styles_checksum = SHA-256(sorted([s.checksum for s in plan]))

PlanDelta:
  plan_delta_checksum = SHA-256(sorted(added+removed+changed)) -> snapshot core

PER metric (Muse 2026):
  PER = (S + D + I) / N
  S=substitutions, D=deletions, I=insertions, N=total reference phonemes
  Lower PER -> better lyric fidelity (add to benchmark suite)

Tags outside canonical 14 -> ValueError (no silent fallback, Axiom A6)
File: planner/structure_planner.py, planner/planner_deterministic.py
```

---

## SS33. Memory Consolidation Contract (/sleep, PROTOCOL SS M.10)

```
Scope: Control Plane ONLY. Signal/Param layers REMAIN FROZEN (Theorem B.1).
  - DiT / LM / VAE / Qwen3 weights: delta(theta_N) = 0 always
  - Mastering signal stages: immutable (SS8.1)

CPM = (phi_clients, Phi_genre, theta_warmstart):
  phi_clients: per-client mastering parameter history
  Phi_genre:   genre HD floor adjustments from session data
  theta_warmstart: BFGS initial state for next session

Invariants during /sleep:
  d(theta_N)/dt = 0           (Axiom A2)
  Sum(IQS weights) = 1.00     (Lemma B.5-A)
  All CPM entries: SHA-256 keyed, append-only log (Axiom A6)
  BFGS sees IQS_base ONLY      (Invariant #12 preserved)

Algorithm:
  1. Extract IQS >= 0.65 parameter clusters per genre
  2. Generate Q&A pairs: Q: "optimal phi for genre/LUFS/H_n?"
                         A: "tilt, ratio, glue, IQS, checksum"
  3. Update Phi_genre: new floor = P10(H_n | IQS >= 0.65, genre)
  4. Update theta_warmstart = mean(phi* | IQS >= 0.65)
  5. Append to cpmmemory.jsonl (chained, Invariant #21)
  6. Assert: operator_graph_checksum unchanged (frozen weights)

LoRA extension (Phase P5, research only):
  Target: caption_optimizer LoRA adapters ONLY
  NOT DiT/LM/VAE/Qwen3 (Theorem B.1 boundary)
  Requires: >= 200 session snapshots, offline training
  License: Apache 2.0 or MIT for LoRA library only

File: personalization/sleep_consolidator.py  (Phase P5)
```

---

## SS34. Generator Agnosticism Contract (PROTOCOL SS G)

```
NOESIS DHCF-FNO is compatible with any audio generation backend
satisfying Definition G.1 (PROTOCOL v0.8 SS G.1).

Required interface (any backend):
  generate(caption: str, seed: int, duration_s: float,
           sr: int = 48000) -> np.ndarray  # float32, [2,N] or [N]
  get_backend_checksum() -> str            # SHA-256 of frozen weights

NOESIS invariants apply to ALL backends (SS1-SS9 + pipeline SS8.1):
  - Generation Artifact Guard (SS8.2): LUFS gates
  - Mastering Stages 0-11: unchanged regardless of backend
  - IQS formula SS4.2: identical
  - Snapshot v16 SS6: backend_name added to core fields
  - Determinism: seed -> identical wav -> identical checksum

Reference: ACE-Step v1.5 Turbo (this document SS18-SS28)

Alternative registered backends:
  "acestep_v15_turbo"  reference backend (default)
  "musicgen_large"     Meta autoregressive LM
  "stable_audio"       Stability AI latent diffusion
  "audioldm2"          HKUST latent diffusion
  "muse_2026"          Jiang et al. 2026 (Qwen3+MuCodec)

File: generation/generation_router.py  (SUPPORTED_BACKENDS dict)
```

---

## SS35. Coding Architecture Rule

```
MANDATORY MODULE DECOMPOSITION:

If a module exceeds 200 lines (guideline, not hard ceiling):
  -> Refactor as: thin orchestrator + 2-3 helper modules (~150-200 lines each)
  -> Orchestrator: imports helpers, exposes public API, no DSP/math logic
  -> Helpers:      self-contained, independently testable, <= 200 lines each
  -> __init__.py:  mandatory for every new package (exports public API)

When NOT to split (exceptions):
  - File contains irreducible mathematical formulas/constants (keep intact)
  - Splitting would break functional cohesion (DSP stage = atomic unit)
  - Orchestrator itself needs more lines for correct wiring -> acceptable

Priority refactor candidates:
  modeling_noesis_v15_turbo.py   (2246 lines) CRITICAL + zero-legacy violation
  benchmark_ab_engine.py         (883  lines) HIGH
  fad_backend.py                 (769  lines) HIGH
  sdk.py                         (547  lines) MEDIUM

Pattern example (already implemented):
  mastering_chain.py      = orchestrator
  mastering_normalize.py  = helper (Stage 8)
  mastering_pre_limiter.py = helper (Stage 8.5)
  mastering_post_limit.py  = helper (Stages 9/9.5/safety/drift)
  compressor.py            = helper (Stages 3+6)

HEADER: every file must start with:
  \"\"\"NOESIS - Deterministic Hybrid Control Framework for Frozen Neural Operators (DHCF-FNO)
  Copyright (c) 2026 AMAImedia.com
  All rights reserved.
  <FULL PATH>\"\"\"
```

---

# FULL INVARIANT INDEX v0.8

Cross-reference to PROTOCOL v0.8 (complete definitions):

```
#1  DiT/VAE/Qwen3 weights FROZEN (B.1)
#2  Sigma scheduler: monotonic + checksum-locked (B.31)
#3  Deterministic 3x RNG lock (A1)
#4  Stage order IMMUTABLE: MonoBass->SpectralTilt->...->LUFSTrim  [FIXED v0.8]
#5  LUFS drift <= 0.01 dB (three-tier contract)
#6  TruePeak < -0.1 dBTP  [UPDATED v0.8]
#7  IQS weights versioned + checksum-locked, Sum=1.00
#8  operator_graph_checksum updated on every operator change
#9  One seed -> one wav -> one snapshot_checksum (H.4)
#10 float32 explicit dtype; IIR poles < 1 (assert at init)
#11 No rand() without seed-lock; no dropout in mastering/QA
#12 BFGS sees IQS_base ONLY (SS4.1 separation)
Invariant #13 Phase C: max 4 seeds per generation attempt
Invariant #14 Cosine guard: cos_sim >= 0.70
Invariant #15 T_adapt in [-20, -8] dB
Invariant #16 Distance: KAD(PANN) primary; FAD-CLAP-MA secondary; VGGish fallback [UPDATED v0.8]
Invariant #17 Post-limiter trim: L <= 1.0 (linear gain only)
Invariant #18 MOSResult unified wrapper (mos_types.py)
Invariant #19 True-Peak via 4x polyphase FIR (Kaiser beta=8.0, N_taps>=256)  [NEW v0.8]
Invariant #20 FIR crossover N_tap=8192; B_f/A_f <= 1.05 (Theorem C.1)        [NEW v0.8]
Invariant #21 Chained JSONL: entry[n].prev = SHA-256(entry[n-1])              [NEW v0.8]
Invariant #22 VST3: plugin.reset() before process() per track                 [NEW v0.8]
Invariant #23 EQ profiles versioned + checksum-locked (EQ_PROFILES_CHECKSUM)  [NEW v0.8]
```

---

*Document: NOESIS_ENGINEERING_CONTRACT_v0_8.md*
*Version: v0.8 (2026-03-07)*
*Author: Ilia Bolotnikov / AMAImedia.com (2026)*
*Supersedes: NOESIS_ENGINEERING_CONTRACT_v0_7.md*
*Status: ACTIVE*
