"""NOESIS — Deterministic Hybrid Control Framework for Frozen Neural Operators (DHCF-FNO)
Copyright (c) 2026 AMAImedia.com
All rights reserved.
docs/NOESIS_MASTERING_SPEC_v0_9.md"""

# NOESIS MASTERING SPEC v0.9

```
Version:    v0.9  (2026-03-07)
Author:     Ilia Bolotnikov / AMAImedia.com (2026)
Status:     ACTIVE
Supersedes: NOESIS_MASTERING_SPEC_v0_8.md
Changes v0.9:
  FIXED SS2:  Canonical stage order clarified — MonoBass was always Stage 0,
              but "SpectralTilt -> LUFS" was being documented as the first two
              steps. Stage order in code (streaming_mastering_engine.py) is
              authoritative; documentation now matches.
  FIXED SS6:  TruePeak abort threshold -0.1 dBTP (was incorrectly shown as 0)
  NEW SS4:    Frame bounds path to near-tight (Theorem C.1, P1.3)
              N_tap=8192 -> L_M_fixed <= 4.2 vs current 14.44
  NEW SS8:    Phase-Aligned Mono Bass spec (Theorem C.3, Phase P2.1)
  NEW SS9:    True-Peak 4x oversampling spec (Theorem, Phase P1.1)
  NEW SS10:   Stage 8.5 v2 pre-limiter spec (two-pass adaptive)
  NEW SS11:   Stage 11 hard-clamp spec (final WAV safety)
  Updated SS2: Generation Artifact Guard (Stage -1) added
  Updated SS5: Lipschitz chain extended with Stages 8.5, 9, 11
  Updated SS7: IRC-5 limiter theory (Theorem C.2, Phase P4.1)
```

---

## SS1. Architecture

```
M(x) = C o T o S o L_phi o N(x)

C = hard-clamp (Stage 11, mandatory WAV safety)
T = LUFS trim (Stage 8, mechanical correction)
S = softclip + LUFS correction (Stage 9)
L_phi = mastering chain (Stages 0-7)
N(x) = frozen neural operator (DiT + LM + VAE)

Guard (Stage -1): GenerationArtifactGuard (pre-mastering LUFS gate)
Pre-limiter (Stage 8.5): two-pass adaptive pre-limiter (optional)
VST3 (Stage 10): FrozenVSTOperator post-trim (optional)

Guarantee: reset() on entry -> all nodes from zero -> determinism (Axiom A1)
```

---

## SS2. CANONICAL STAGE ORDER (IMMUTABLE — CONTRACT SS8.1, Invariant #4)

### SS2.1 Complete Pipeline

```
Stage -1 | GenerationArtifactGuard   | Pre-mastering LUFS gate        | MANDATORY
──────────────────────────────────────────────────────────────────────────────────
Stage  0 | MonoBassAlignmentStage    | Sub-bass mono/phase, f<120 Hz  | L <= 1.0
Stage  1 | AdaptiveSpectralTiltStage | Frequency slope correction      | L ~= 17.89
Stage  2 | SubbandLUFSSlopeNode      | Integrated loudness targeting   | L <= 2.0
Stage  3 | MultibandTransientBlock   | 4-band dynamics                 | L <= 4.0
Stage  4 | GlueBusCompressor         | Glue compression + iso226 EQ    | L <= 2.0
Stage  5 | BarkMaskingNode           | Perceptual masking, 24 bands    | L <= 2.0
Stage  6 | ModulationCoherenceNode   | Modulation coherence norm       | L <= 1.5
Stage  7 | PredictiveDualStageLimiter| 4x ISP-safe cascade             | L_core=1.0214
──────────────────────────────────────────────────────────────────────────────────
Stage 8  | LoudnessVerification      | Post-limiter LUFS trim          | L <= 1.0
Stage 8.5| TwoPassPreLimiter         | Adaptive pre-limiter (optional) | L <= 1.0
Stage  9 | PostSoftclip              | Softclip + LUFS correction      | L <= 1.0
Stage 10 | FrozenVSTOperator         | VST3 post-trim (optional)       | L <= 1.0
Stage 11 | HardClamp                 | Final clip to ceiling_lin       | L <= 1.0
```

**IMMUTABLE:** Stages 0-7 cannot be reordered. Stages 8-11 are post-processing.

### SS2.2 Stage -1: Generation Artifact Guard

Applied before Stage 0 for every track:
```
ARTIFACT_FLOOR_LUFS  = -55.0   (below -> GenerationArtifactError, retry generation)
PRE_LIFT_TARGET_LUFS = -35.0   (lift if below this)
GAIN_CAP_DB          = +60.0   (max gain in rms_pre_lift)
FLOOR_LUFS           = -69.0   (true silence floor for normalization gate)

Root cause: ACE-Step characteristic output range is -62 to -67 LUFS.
Without guard, Stage 2 applies 50+ dB gain to near-silence, amplifying noise.
File: mastering/generation_artifact_guard.py
```

### SS2.3 Key Correction vs v0.8

v0.8 documentation header wrote "SpectralTilt -> LUFS -> Multiband -> ..."
suggesting SpectralTilt was first. The code has always had MonoBass first
(order=10 vs SpectralTilt order=10 but MonoBass is inserted before in graph).
v0.9 documentation matches code (streaming_mastering_engine.py is authoritative).

---

## SS3. FILE MAP

```
mastering/
  streaming_mastering_engine.py      StreamingMasteringEngine v4.4+ (orchestrator)
  streaming_graph.py                 GraphEngine (sequential node execution)
  mastering_chain.py                 MasteringChain orchestrator (Stages 8-11)
  base_stage.py                      BaseMasteringStage (abstract)
  dsp_node.py                        DSPNode contract (interface)
  generation_artifact_guard.py       Stage -1 (LUFS gate)
  mono_bass_alignment_stage.py       Stage 0 (MonoBass + phase-align, P2.1)
  adaptive_spectral_tilt_stage.py    Stage 1 (SpectralTilt)
  subband_lufs_slope_node.py         Stage 2 (LUFSSlope)
  multiband_transient_block.py       Stage 3 (Multiband)
  glue_bus.py                        Stage 4 (GlueBus v2.0 + iso226)
  bark_masking_node.py               Stage 5 (BarkMask)
  modulation_coherence_node.py       Stage 6 (ModCoherence)
  limiter_core.py                    Stage 7 (PredictiveDualStageLimiter + true-peak, P1.1)
  mastering_normalize.py             Stage 8 (LUFSTrim helper)
  mastering_pre_limiter.py           Stage 8.5 (TwoPassPreLimiter helper)
  mastering_post_limit.py            Stage 9/safety/drift helper
  compressor.py                      Stage 3+6 helper
  linear_phase_fir_crossover.py      LinearPhaseFIRCrossover (shared, P1.3)
  loudness_stage.py                  LoudnessStage (BS.1770 + EBU R128, P1.1)
  bs1770_filters.py                  BS.1770-4 K-weighting filters
  iso226.py                          Equal-loudness contours (ISO 226:2003)
  mastering_primitives.py            Low-level DSP primitives
  spectral_tools.py                  Spectral utilities + biquad cascade (P2.3)
  stability_contract.py              Runtime stability checks
  stem_aware_node.py                 Stem-aware processing base
  stem_mastering.py                  Per-stem mastering + StemAdaptiveEQ (P2.3)
  mono_bass.py                       MonoBass wrapper (legacy alias)
  mid_side_stage.py                  M/S processing
  dc_removal.py                      DC offset removal
  finalizer.py                       Output normalization
  upward_compress.py                 Upward compressor
  true_peak_filter.py                True-peak FIR coefficients (P1.1)
  irc5_limiter.py                    IRC-5 N-candidate limiter (P4.1, PLANNED)
  mono_bass_alignment_stage.py       Phase-aligned variant (P2.1, PLANNED)
```

---

## SS4. FRAME BOUNDS (Theorem B.15)

### SS4.1 Current Values (N_tap=2048, SEALED 2026-03-01)

```
A_f    = 0.51969725   lower frame bound
B_f    = 1.40730381   upper frame bound
B_linf = 3.60972762   L-infinity Young's bound

Verification: A_f <= 1 <= B_f  ->  0.5197 <= 1 <= 1.4073  OK
Tightness ratio: B_f / A_f = 2.708  (not near-tight)
Crossover: LinearPhaseFIRCrossover(sr=48000, block_size=1024,
                                   kernel_size=2048, low=120.0, high=5000.0)
```

### SS4.2 Target Values After P1.3 (N_tap=8192, Theorem C.1)

```
A_f_target    ~= 0.9756   (to be empirically verified)
B_f_target    ~= 1.0248
B_linf_target <= 1.05     (near-tight, theory bound)
B_f/A_f       <= 1.05     (vs 2.708 current — significant improvement)

L_M_fixed_current = B_linf * G_max = 3.6097 * 4.0 = 14.44
L_M_fixed_target  = 1.05 * 4.0 = 4.2             (Theorem C.1 consequence)

Theorem C.1 proof sketch:
  Kaiser N_tap=8192, beta=8.6 -> stopband > 80 dB -> delta_s < 1e-4
  Power-complementary: ||H_LP^2 + H_HP^2 - 1||_inf < 0.05
  Near-tight: B_f/A_f <= (1+delta)/(1-delta) <= 1.05

SEAL after verification: update A_f, B_f, B_linf in:
  mastering/mastering_chain.py
  mastering/streaming_mastering_engine.py
  docs/NOESIS_MASTERING_SPEC_v0_9.md (this file)
  NOESIS_DHCF_FNO_PROTOCOL_v0_7.md SS B.3 (base theory)
  NOESIS_DHCF_FNO_PROTOCOL_v0_8_ADDENDUM.md §C.1-C.3 (new theorems)
```

---

## SS5. LIPSCHITZ CONSTANTS

### SS5.1 Core (Theorem B.16(i))

```
L_core_empirical  = 1.0214   M_core_pure, 50 trials, seed=42
G_max_design      = 4.0      limiter gain bound
C_M_empirical     = 0.8912   actual ceiling
L_M_fixed_theory  = 14.44    B_linf * G_max (current, N_tap=2048)
L_M_fixed_target  = 4.20     after P1.3 (Theorem C.1)
Margin: G_max / L_core = 4.0 / 1.0214 = 3.92x (23.0 dB safety)
```

### SS5.2 Full Chain

```
L_N_empirical    = 17.89   pre_graph Lipschitz (Stages 0-6)
L_full_theory    = 258.25  L_N * L_M_fixed = 17.89 * 14.44
L_full_empirical ~= 258.25 (measured, consistent with theory)
```

### SS5.3 Root Cause L_N = 17.89

```
AdaptiveSpectralTiltStage:
  slope ~= -0.4 dB/oct -> DC bin gain = +19.5 dB = x9.47
  All other stages have L <= 4.0
  Total chain L_N ~= 17.89 (measured, NOT product of individual bounds)
```

### SS5.4 Full Lipschitz Chain (SS2.1 stages)

```
Input x (stereo float32)
  [- 1]  ArtifactGuard     L <= 1.0  (gain <= +60 dB, capped)
  [ 0 ]  MonoBass          L <= 1.0
  [ 1 ]  SpectralTilt      L ~= 17.89 (DC boost dominates)
  [ 2 ]  LUFSSlope         L <= 2.0
  [ 3 ]  Multiband         L <= 4.0
  [ 4 ]  GlueBus           L <= 2.0
  [ 5 ]  BarkMask          L <= 2.0
  [ 6 ]  ModCoherence      L <= 1.5
  =====================================
  N(x) total: L_N ~= 17.89 (measured, not product)
  [ 7 ]  Limiter           L_core=1.0214, C_M<=1.0, G_max=4.0
  [ 8 ]  LUFSTrim          L <= 1.0  (linear gain only)
  [8.5]  PreLimiter        L <= 1.0  (level control, no expansion)
  [ 9 ]  PostSoftclip      L <= 1.0  (tanh + gain correction)
  [10 ]  VST3 (optional)   L <= 1.0  (deterministic, reset per track)
  [11 ]  HardClamp         L <= 1.0  (clip, never expands signal)
  =====================================
  M_core:     L <= 14.44 (theory), L_emp=1.0214
  M_full:     L ~= 258.25 (current)
  M_full_P1.3: L <= 75 (after near-tight crossover, Theorem C.1 + Lemma B.16-C)
Output y (stereo float32)
```

---

## SS6. STABILITY METRICS (CONTRACT SS2)

```
kappa           = g_max * L_core = 0.8912 * 1.0214 = 0.910
stability_margin = 1 - kappa = 0.090  (9.0%, GREEN zone)
lambda_decay    = 0.094  per step (Theorem E.1)
C_M_empirical   = 0.8912  (limiter ceiling, Theorem B.14 <= 1.0  OK)

Conditions:
  kappa < 1       ->  0.910 < 1   OK (GES, Theorem B.26 Case I)
  margin > 0      ->  0.090 > 0   OK
  E[log g] + log L_core < 0 = -0.094 < 0  OK (Theorem E.1)

TruePeak constraint (UPDATED v0.9 from v0.8):
  TruePeak_dBTP < -0.1 dBTP  (Invariant #6, CONTRACT SS9.7)
  Measured via 4x polyphase FIR (Invariant #19)
  After Stage 11 hard-clamp: peak <= ceiling_lin = 10^(-1.0/20) ~= 0.891
```

---

## SS7. LIMITER THEORY

### SS7.1 Current: PredictiveDualStageLimiter

```
4-stage ISP-safe cascade:
  Stage A: lookahead = 5 ms, attack = 0.5 ms, release = 50 ms
  Stage B: lookahead = 2 ms, attack = 0.2 ms, release = 20 ms
  Stage C: lookahead = 1 ms, attack = 0.1 ms, release = 10 ms
  Stage D: hard clip at ceiling_lin = 10^(ceiling_db/20)

ceiling_db = -1.0 dBTP (default, tightened to -0.1 dBTP after Stage 7)
All IIR poles < 1 (assert at init, Invariant #10)
Determinism: lookahead via O(N) monotonic deque sliding window
```

### SS7.2 Stage 8.5 v2: Two-Pass Adaptive Pre-Limiter

```
Purpose: prevent LUFS drift for sparse/ambient material before limiter

Pass 1 (profiling):
  - Transient density estimation (block size=256, hop=128)
  - Spectral spread analysis (BarkStereo 24 bands)
  - Sparse flag: density < 0.15 OR spread < 0.25

Pass 2 (compression):
  threshold_base  = -12.0 dBFS
  sparse_benefit  = +4.0 dB (RAISE threshold for sparse material)
    -> corrected from original "sparse_penalty" (inverted bug, fixed v0.9)
  lookahead_ms    = 5.0 ms
  lookahead_samples = round(5e-3 * sr)
  sliding_max: O(N) monotonic deque (not block-based envelope)

Why block-based was insufficient:
  attack~0.995, 256-sample blocks -> need ~150 consecutive high-peak blocks
  Sparse piano produces only 2-3 consecutive blocks -> never converges
  Solution: sample-level lookahead + deque sliding max

Files: mastering/mastering_pre_limiter.py, mastering/pre_limiter_core.py
```

### SS7.3 Planned: IRC-5 Multiband Limiter (Theorem C.2, Phase P4.1)

```
N=8 candidate release schedules evaluated simultaneously
Perceptual distortion functional:
  D_perc(G, x) = Sum_k w_k * ||X_k - G*X_k||^2 / T_mask(k)^2
  k = Bark band index
  w_k = ISO 226 equal-loudness weight
  T_mask(k) = simultaneous masking threshold

Theorem C.2 suboptimality bound: <= 1/N = 12.5% at N=8
Benefit: -4 LUFS headroom without pumping/distortion
File: mastering/irc5_limiter.py  (PLANNED, Phase P4.1)
```

---

## SS8. PHASE-ALIGNED MONO BASS (Theorem C.3, Phase P2.1)

```
Current mode (before P2.1): "sum_to_mono" (M/S summation, lossy at phi != 0)

Target mode (after P2.1): "phase_aligned"

Algorithm (Theorem C.3):
  phi_diff(f) = angle(X_L(f)) - angle(X_R(f)),  f < f_c = 120 Hz
  X_L'(f) = X_L(f) * exp(+j*phi_diff/2)         [rotate L toward center]
  X_R'(f) = X_R(f) * exp(-j*phi_diff/2)         [rotate R toward center]
  Mono(f)  = (X_L'(f) + X_R'(f)) / 2

Energy conservation (Theorem C.3):
  E_aligned(f) = |X_L(f)|^2   (invariant under rotation)
  E_sum(f)     = |X_L(f)|^2 * cos^2(phi/2)  (M/S sum, lossy)
  E_aligned / E_sum = 1/cos^2(phi/2) >= 1  always

Extreme cases:
  phi=0:   E_aligned = E_sum  (no difference)
  phi=pi/2: E_aligned/E_sum = 2.0  (+3 dB recovered)
  phi=pi:   E_sum = 0 (total cancellation in M/S!), E_aligned = |X_L|^2

Lipschitz: L <= 1.0 (rotation is unitary, energy-preserving)
IIR poles: N/A (FFT-based, no feedback)
Crossover: f_c = 120 Hz (canonical mono bass boundary, Invariant #4)

Genre routing (to be added in genre_profiles_v4.py):
  phase_aligned: edm, hip_hop, neurofunk, trap, drill, phonk
  sum_to_mono:   classical, ambient (stereo imaging preferred)

File: mastering/mono_bass_alignment_stage.py  (P2.1)
```

---

## SS9. TRUE-PEAK OVERSAMPLING (Invariants #6/#19, Phase P1.1)

```
Specification (BS.1770-4 SS4.2):
  TruePeak_dBTP = 20 * log10(max|x_upsampled|)
  Upsampling: 4x polyphase FIR
  Kaiser window: beta=8.0, N_taps >= 256
  Stopband attenuation: > 80 dB
  Passband ripple: < 0.1 dB

Determinism:
  FIR coefficients: h = scipy.signal.firwin(N_taps, 0.4*sr, window=('kaiser',8.0))
  h_checksum = SHA-256(h.tobytes()) -> sealed at init (Invariant #19)

Contract (Invariant #6, UPDATED v0.9):
  TruePeak_dBTP < -0.1 dBTP
  Abort CONTRACT SS9.7 if TruePeak >= -0.1 dBTP

EBU R128 measurements (non-hash snapshot extension, SS6.2):
  integrated_lufs:    from Stage 2 (BS.1770-4 gated)
  lra_db:             P95 - P10 of 3s windows
  momentary_max_lufs: max 400ms window
  short_term_max_lufs: max 3s window
  true_peak_dbtp:     from 4x upsampling

File: mastering/limiter_core.py  (true-peak mode, P1.1)
      mastering/bs1770.py        (EBU R128 extensions, P1.1)
      mastering/true_peak_filter.py  (FIR coefficients, P1.1)
```

---

## SS10. STAGE 8.5 v2 SPEC (Two-Pass Pre-Limiter)

```
Full specification (production state, v0.9):

Parameters:
  THRESHOLD_BASE_DB = -12.0
  SPARSE_BENEFIT_DB = +4.0   (raise threshold for sparse material)
  RATIO_BASE        = 3.5    (normal material)
  RATIO_SPARSE      = 5.0    (sparse material — harder knee)
  ATTACK_MS         = 0.5    (fast, catches transients)
  RELEASE_MS        = 80.0   (slow, holds gain reduction)
  LOOKAHEAD_MS      = 5.0    (forward lookahead)
  DENSITY_THRESHOLD = 0.15   (below = sparse)
  SPREAD_THRESHOLD  = 0.25   (below = sparse)

Gain computation:
  gain_db = 0.0 if level_db < threshold
  gain_db = -(level_db - threshold) * (1 - 1/ratio)  otherwise

Lookahead (O(N) deque):
  x_look = sliding_max(|x|, window=lookahead_samples)
  Using collections.deque monotonic sliding window (O(1) amortized per sample)
  NOT block-based envelope (block-based fails for sparse material, see SS7.2)

File: mastering/mastering_pre_limiter.py, mastering/pre_limiter_core.py
```

---

## SS11. STAGE 11: HARD-CLAMP (Final WAV Safety)

```
Purpose: prevent inter-sample peaks from exceeding ceiling after Stage 9 gain correction
Required: ALWAYS applied before WAV export

Implementation:
  ceiling_lin = 10.0 ** (ceiling_db / 20.0)  (= 0.891 for ceiling_db=-1.0)
  y = np.clip(x, -ceiling_lin, +ceiling_lin)

Lipschitz: L <= 1.0 (clip is non-expansive, ||clip(x)-clip(y)|| <= ||x-y||)
IIR poles: N/A (memoryless)
LUFS delta: log delta from clamp ~= 0.05-0.15 dB (logged, non-hash snapshot)

Rule: output_lufs (pre-clamp) is the drift contract value.
      Hard-clamp is applied AFTER LUFS measurement and drift verification.
      This allows LUFS to be verified correctly while ensuring WAV peak safety.

File: mastering/mastering_chain.py (_final_hard_clamp method)
```

---

## SS12. OPERATOR TAXONOMY

```
Method          | Operator          | Theorem   | Lipschitz      | Stage
----------------|-------------------|-----------|----------------|-------
process()       | M_full(x)         | B.16-A    | L ~= 258.25    | 0-11
process_core()  | M_core(x, phi)    | B.16(i)   | L_emp=1.0214   | 7
M_fixed(x)      | L_phi o N(x)      | B.16(ii)  | L <= 14.44     | all
M_adaptive(x)   | L_{phi(x)} o N(x) | B.16-A    | piecewise Lip. | all
lufs_trim()     | T(x)              | linear    | L <= 1.0       | 8
softclip_corr() | S(x)              | tanh+gain | L <= 1.0       | 9
hard_clamp()    | C(x)              | clip      | L <= 1.0       | 11
```

After P1.3 (Theorem C.1, Lemma B.16-C):
```
L_M_fixed_new <= 1.05 * 4.0 = 4.2
L_full_new    ~= 17.89 * 4.2 ~= 75  (vs 258.25 current)
```

---

## SS12.1 THREE-TIER DRIFT CONTRACT

```
drift_db = abs(integrated_lufs_output - lufs_target)

PASS:             drift_db <= 0.01 dB         (ideal convergence)
PASS_CF_LIMITED:  drift_db > 0.01 dB AND
                  lufs_stalled=True (CF-physically-limited) AND
                  drift_db <= 2.0 dB           (physically constrained)
FAIL:             drift_db > 2.0 dB OR
                  (drift_db > 0.01 and not CF_LIMITED)

_LUFS_CF_LIMITED_WARN_DB = 1.5 dB  (warning threshold within CF_LIMITED zone)

Rule: output_lufs is measured PRE-clamp (Stage 9 output before Stage 11 hard-clamp).
      Hard-clamp (Stage 11) is applied AFTER drift contract evaluation.

Typical values (Phase R benchmark, 10 genres):
  PASS:           ambient, classical, lofi, jazz, pop  (drift=0.0000)
  PASS_CF_LIMITED: EDM, hip-hop, metal, neurofunk      (drift=0.0000 after 8 passes)
  FAIL:           none (Phase R: 10/10 within contract)
```

---



```
kappa            = 0.910   (g_max * L_core)
stability_margin = 0.090   (9.0%, GREEN)
L_core_empirical = 1.0214
L_N_empirical    = 17.89
L_full           ~= 258.25 (current) / ~75 (after P1.3)
operator_graph_checksum = f1d8a82c7d6e859667cddb7732b869ba3b82926501719e6da78b947edc3dd2ba
                          (30 operators, SEALED 2026-03-01)
```

---

## SS14. TEST COVERAGE

```
test_snapshot_v16.py:     62+35 checks  (sections A-M)
freeze_baseline.bat:      11/11 PASS
test_lipschitz_mastering: 20/20 PASS  (cross-platform)
test_e2e_chain:           20/20 PASS  (cross-platform)

Pending (after P1.1-P1.3):
  test_true_peak:         T1-T5 (BS.1770-4 compliance)
  test_crossover_8192:    T1-T5 (near-tight verification)
  test_phase_aligned_bass: T1-T5 (Theorem C.3 validation)
```

---

## CROSS-REFERENCES

```
PROTOCOL v0.7 (base, sealed):
  Theorems B.15, B.16(i/ii/A/B), B.26, E.1 -- formal proofs (full text)

PROTOCOL v0.8 ADDENDUM:
  Theorem C.1 -- near-tight crossover (N_tap=8192)
  Theorem C.2 -- IRC-5 perceptual distortion (Phase P4.1)
  Theorem C.3 -- phase alignment energy conservation (Phase P2.1)

CONTRACT v0.8:
  SS8.1 -- canonical stage order (authoritative)
  SS9 -- abort conditions (TruePeak, LUFS, NaN)
  SS30 -- True-Peak Contract
  SS31 -- Phase Alignment + Crossover + Merkle

MODULE v1.0:
  mastering/ file tree, import DAG
```

---

*Document: NOESIS_MASTERING_SPEC_v0_9.md*
*Version: v0.9 (2026-03-07)*
*Author: Ilia Bolotnikov / AMAImedia.com (2026)*
*Supersedes: NOESIS_MASTERING_SPEC_v0_8.md*
*Status: ACTIVE*
