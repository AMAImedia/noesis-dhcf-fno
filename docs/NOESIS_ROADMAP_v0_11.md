"""NOESIS — Deterministic Hybrid Control Framework for Frozen Neural Operators (DHCF-FNO)
Copyright (c) 2026 AMAImedia.com
All rights reserved.
docs/NOESIS_ROADMAP_v0_11.md"""

# NOESIS ROADMAP AND FORMULAS v0.11

```
Version:    v0.11  (2026-03-07)
Author:     Ilia Bolotnikov / AMAImedia.com (2026)
Status:     ACTIVE
Supersedes: NOESIS_ROADMAP_AND_FORMULAS_v0_10.md
Changes v0.11:
  FIXED: IQS formula synced to canonical 6-term v0.9 across all sections
  FIXED: All formula references to IQS_max=0.75 (was 0.50 in v0.9 roadmap)
  NEW §3.1a: Lemma — formal proof of IQS range [-0.25, 0.75]
  NEW §4: Scientific Paper Integration (Muse 2026, KAD, MAD, Survey 2025)
  NEW §5: PER metric in benchmark suite
  NEW §6: IQS_edit mode (epsilon-term, editFLAM)
  NEW §7: Distance metric hierarchy (KAD > FAD-CLAP-MA > FAD-VGGish)
  NEW §8: Generator Agnosticism in roadmap context
  NEW Phase P5: /sleep Memory Consolidation (mylm-inspired)
  Updated Phase table: R.GPU -> P1-P3 -> P4 -> D1-D5 -> P5 (future)
  Updated frozen constants: IQS_VERSION="0.9", pass_gate=0.55
  Added P1.1-P4.4 full technical context
  Added academic paper citations (bibtex)
```

---

## SECTION 0. SEALED — COMPLETED FULLY

All phases closed, code in repository, tests green.

```
Phase P    Snapshot v16 + fingerprint + 62 tests               DONE
Phase 0    HarmonicDensity + LatentEntropy -> dit_runtime v9.0  DONE
Phase 1    QA Ensemble (UTMOS+DNSMOS+FAD), QualityFusion         DONE
Phase 2    CSO + MutationController v2 + GenerationRouter         DONE
Phase S    Code hygiene: 139+ files, zero legacy, all headers     DONE
Phase 1-PQ PerceptualAnalysisLayer v2.0 (Bark+ERB+phase)         DONE
Phase 2-MQ IQS v0.7 unified, iso226 EQ, GlueBus v2.0            DONE
Phase R.1  Benchmark infrastructure (10 genres, synthetic)        DONE
Phase R.2  Three-tier drift contract (PASS/PASS_CF_LIMITED/FAIL)  DONE
Phase R.FIX1 Generation Artifact Guard + Stage 11 hard-clamp     DONE
Phase R.FIX2 IQS v0.8->v0.9 recalibration + MOS proxy rebuild   DONE
             TinyMOS v2 lock + Genre-aware HD normalization
```

**Sealed system constants (2026-03-01):**

```
B_f                       = 1.40730381    upper frame bound (Theorem B.15, N_tap=2048)
A_f                       = 0.51969725    lower frame bound
B_linf                    = 3.60972762    L-inf Young's bound (current)
L_core_empirical          = 1.0214        M_core_pure (50 trials, seed=42)
L_N_empirical             = 17.89         pre_graph full chain
kappa_empirical           = 0.910         g_max * L_core (stability margin 9.0%)
operator_graph_checksum   = f1d8a82c7d6e859667cddb7732b869ba3b82926501719e6da78b947edc3dd2ba
IQS_VERSION               = "0.9"         (weights identical to v0.8, Sum=1.00)
IQS_WEIGHTS_CHECKSUM      = [see iqs.py]
```

---

## SECTION 1. FORMULA AUDIT STATUS (v0.11)

Three critical inconsistencies found and resolved:

| # | Document | Problem | Fix Applied |
|---|----------|---------|-------------|
| 1 | TZ v0.7 §9 | IQS formula 5-term, alpha=0.40, IQS_max=0.50 | Canonical 6-term v0.9 in all docs |
| 2 | MASTERING_SPEC v0.8 §2 | "SpectralTilt first" (wrong) | v0.9 §2: MonoBass is Stage 0 |
| 3 | PROTOCOL v0.7 Invariant #16 | FAD-VGGish "preferred" | Updated: KAD(PANN) primary |

All canonical formulas now live in a single source of truth per document:
- IQS formula: `metrics/iqs.py` (IQS_VERSION="0.9") + PROTOCOL v0.8 ADDENDUM §FORMULA
- Stage order: `mastering/streaming_mastering_engine.py` + MASTERING_SPEC v0.9 §2
- Invariants: PROTOCOL v0.7 base (#1-#18) + ADDENDUM (#19-#23) + CONTRACT v0.8 §Full-Invariant-Index

---

## SECTION 2. PHASE ROADMAP

### Priority Framework

```
R.GPU  CURRENT   GPU validation with real generation (before any P1)
P1     CRITICAL  breaks standards or metrics if missing
P2     HIGH      measurable IQS improvement, streaming compatibility
P3     MEDIUM    extends system, does not block current
P4     LOW       long-term, external dependencies
D1-D5  DOCS      one doc session each, after corresponding code phases
P5     FUTURE    memory consolidation, requires large session history
```

**Dependency rule:** each Px phase starts only after all P(x-1) CRITICAL phases are done.

---

### Phase R.GPU — GPU Validation (CURRENT PRIORITY)

```
Status:   PENDING (first session)
Goal:     10 real tracks on GPU, IQS mean >= 0.60
Files:    benchmark_runner.py, generation_loop_v20.py, benchmark_ab.py
Criterion: all PASS, snapshots in JSONL, checksums verified
After:    Update CANONICAL_BASELINE.json -> proceed to P1.1
```

Checklist:
```
[ ] Real generation (not synthetic) on CUDA GPU
[ ] 5 genres x seed=42, 2 tracks each = 10 tracks
[ ] IQS mean >= 0.60, no track < 0.40
[ ] LUFS drift = 0.000 per track (three-tier contract)
[ ] TruePeak < -0.1 dBTP per track
[ ] WAV checksum stable (2 runs -> identical)
[ ] Snapshot v16 all fields present (including IQS, J)
[ ] operator_graph_checksum = f1d8a82c... (unchanged)
[ ] Write results to benchmark_results_gpu.jsonl
[ ] Verify §LYRIC-SCALE log lines appear (NaN/prompt fix deployed)
```

---

### P1.1 — True-Peak Oversampling + EBU R128 [1 session, CRITICAL]

**Rationale:** Without True-Peak oversampling, inter-sample peaks exceed 0 dBTP
on downsampling. Spotify/Apple Music/YouTube reject content with TP > -1 dBTP.
EBU R128 = de-facto standard 2026 for all streaming platforms.
Reference: ITU-R BS.1770-4 (2015) formal loudness measurement standard.

**Scientific basis:** Theorem C.2 (IRC-5) uses Bark-weighted D_perc which
requires correct TruePeak for limiter ceiling verification.

**Files modified:**
```
mastering/limiter_core.py     <- 4x oversampling true-peak detection
mastering/loudness_stage.py   <- EBU R128: LRA + momentary + short-term
mastering/bs1770.py           <- short-term LUFS (3s window) + LRA
mastering/true_peak_filter.py <- FIR coefficients (Kaiser beta=8.0, N_taps>=256)
mastering/mastering_chain.py  <- integrate true_peak field into stage output
tools/benchmark_runner.py     <- add TruePeak to benchmark metrics
```

**Technical spec:**
```python
# TruePeak via 4x polyphase upsampling (Invariant #19)
# scipy.signal.upfirdn(h, x, up=4, down=1)
# Kaiser window: beta=8.0, N_taps >= 256, stopband > 80 dB
# TruePeak_dBTP = 20*log10(max|x_upsampled|)
# Invariant #6: TruePeak < -0.1 dBTP (tightened from 0 dBTP)
# FIR coefficients: SHA-256 locked at init (deterministic)

# EBU R128 extensions (bs1770.py):
# LRA = P95(LUFS_3s) - P10(LUFS_3s)  [EBU R128 SS3.5]
# M_max = max(400ms windows)           [EBU R128 SS3.3]
# ST_max = max(3s windows)             [EBU R128 SS3.4]
```

**Tests (5/5):**
```
T1: 997 Hz sinusoid at 0 dBFS -> TruePeak < -0.1 dBTP
T2: 4x upsampling determinism: 2 runs -> bit-identical
T3: LRA of pink noise: LRA in [0, 20] dB
T4: EBU R128 fields present in snapshot non-hash extension
T5: FIR coefficients immutable: SHA-256 of h == same on restart
```

**Dependency:** R.GPU done | **Unblocks:** P3.4, P4.1

---

### P1.2 — FAD Backend: KAD(MMD) + FAD-CLAP-MA Upgrade [1 session, CRITICAL]

**Rationale (2025-2026 research):**
KAD (Chung et al., ICML 2025): distribution-free, unbiased, O(1/sqrt(n))
convergence vs FAD O(d/n). N >= 50 sufficient (vs FAD N >= 500).
FAD-VGGish has Gaussian assumption bias (Theorem F.1 violation) and
Spearman rho = 0.51 vs KAD(PANN) rho = 0.74 on DCASE 2023 benchmark.
This directly fixes Invariant #16 (FAD was "preferred", now KAD is primary).

**Files modified:**
```
metrics/fad_backend.py   <- add KADBackend, make primary
metrics/fad_types.py     <- add KADResult, EmbeddingChoice enum
metrics/iqs.py           <- Distance_n: prefer KAD if available
```

**Technical spec:**
```python
# KAD = MMD^2 with RBF kernel (Theorem F.1)
# sigma = median pairwise distance bandwidth (data-driven)
# Embedding priority: pann -> clap_ma -> vggish (graceful fallback)
# KAD(P,P) = 0 analytically, verified in T1
# Distance_n = clip(KAD / KAD_max, 0, 1), KAD_max = 0.5
```

**Invariant update:** #16: KAD(PANN) primary; FAD-CLAP-MA secondary; VGGish fallback

**Tests (5/5):**
```
T1: KAD(P, P) = 0.0  (identical inputs)
T2: KAD monotone: white noise vs music > KAD(music vs music)
T3: Fallback chain: pann unavailable -> clap_ma -> vggish
T4: IQS Distance_n: uses KAD when available
T5: KAD determinism: same seed -> same result
```

**Dependency:** R.GPU done | **Unblocks:** P2.3, P3.3

---

### P1.3 — Crossover kernel 8192 (Theorem C.1) [1 session, CRITICAL]

**Rationale:** Current N_tap=2048 gives B_f/A_f = 2.708 (not near-tight).
Theorem C.1: N_tap >= 8192 -> B_f/A_f <= 1.05, L_M_fixed drops
from 14.44 -> 4.2. Closes FabFilter Pro-MB tightness gap.
Latency: 4096/48000 = 0.085 s (offline mastering, acceptable).

**Files modified:**
```
mastering/linear_phase_fir_crossover.py  <- kernel_size 2048 -> 8192
mastering/mastering_chain.py             <- update frame bound constants
```

**New frame bounds (SEAL after empirical verification):**
```
A_f_8192 ~= 0.9756,  B_f_8192 ~= 1.0248,  B_linf_8192 <= 1.05
L_M_fixed <= 1.05 * 4.0 = 4.2  (vs 14.44 current)
```

**Tests (5/5):**
```
T1: ||H_LP^2 + H_HP^2 - 1||_inf < 0.05 at kernel=8192
T2: B_f/A_f <= 1.05 (near-tight, Theorem C.1)
T3: Determinism: 2 runs -> bit-identical filter output
T4: L_M_fixed <= 4.2 (measured, 50 trials)
T5: Constants A_f, B_f, B_linf updated and sealed in mastering_chain.py
```

**Dependency:** R.GPU done | **Unblocks:** P2.1

---

### P2.1 — Phase-Aligned Mono Bass (Theorem C.3) [1 session, HIGH]

**Rationale:** Current M/S sum loses energy at phase misalignment:
at phi=pi -> total cancellation. Sub-bass energy preservation critical
for EDM/neurofunk/hip-hop IQS (Theorem C.3: E_aligned/E_sum = 1/cos^2(phi/2)).

**Files modified:**
```
mastering/mono_bass_alignment_stage.py  <- add phase_aligned mode
mastering/mono_bass.py                  <- expose mode parameter
mastering/mastering_chain.py            <- use mode per genre profile
```

**Dependency:** P1.3 (frame bounds updated) | **Unblocks:** —

---

### P2.2 — Segment-Level Structure Control (Muse 2026) [1 session, HIGH]

**Rationale (Muse 2026):** Segment-level style conditioning (Intro/Verse/Chorus/
Bridge/Outro) not available in open-source song generation models until Muse.
Current StructurePlanner has ~8 structural tags; Muse/MiniMax use 14 canonical.
PER (Phoneme Error Rate) added as benchmark metric.

**Files modified:**
```
planner/structure_planner.py         <- 14 SegmentTag enum
planner/planner_deterministic.py     <- PlanDelta tracking
planner/planner_base.py              <- SegmentStyle dataclass
generation/generation_loop_v20.py   <- pass segment_styles to caption builder
prompt_utils.py                      <- build_caption() with segment conditioning
```

**Technical spec:**
```python
# 14-tag vocabulary (CONTRACT SS32):
# INTRO VERSE PRE_CHORUS CHORUS POST_CHORUS BRIDGE
# BREAKDOWN BUILD DROP OUTRO INTERLUDE SOLO AD_LIB TRANSITION

# Checksums (enter snapshot_checksum core):
# seg.checksum = SHA256(tag||duration||style||energy||density)
# segment_styles_checksum = SHA256(sorted([s.checksum for s in plan]))
# plan_delta_checksum = SHA256(sorted(added+removed+changed))
```

**Dependency:** P1.1 (EBU R128 in snapshot) | **Unblocks:** P3.1, P3.3

---

### P2.3 — Per-Stem Adaptive EQ [1 session, HIGH]

**Rationale:** Frequency conflicts between stems (bass masking kick, vocal
masking lead) reduce IQS Phase_n term. Deterministic checksum-locked EQ
profiles eliminate masking and improve perceptual clarity.

**Files modified:**
```
mastering/stem_mastering.py    <- StemAdaptiveEQ (8-band biquad per stem)
mastering/stem_aware_node.py   <- integrate EQ profiles
mastering/spectral_tools.py    <- biquad_cascade() helper
```

**Technical spec:**
```python
# H(z) = prod_j (b0_j + b1_j*z^-1 + b2_j*z^-2) / (1 + a1_j*z^-1 + a2_j*z^-2)
# All IIR poles < 1 (assert at init, Invariant #10)
# L_EQ <= 2.0 per stem (measure at init, assert)
# EQ_PROFILES_CHECKSUM = SHA256(json.dumps(EQ_PROFILES_V1, sort_keys=True))
# Invariant #23: version bump on any coefficient change

# 12-stem profiles: kick, snare_clap, hihats, percussion,
#   subbass, bass, lead, synth, pad, vocals, fx, other
```

**Dependency:** P1.2 (KAD in place) | **Unblocks:** P3.3

---

### P3.1 — Chained JSONL + Stage Merkle Tree [1 session, MEDIUM]

**Rationale:** Theorem H.4 covers snapshot_checksum but not full audit trail.
Chained JSONL adds tamper-evidence to all sessions. Merkle Tree enables
selective disclosure (single-stage proof without revealing full pipeline).

**Files modified:**
```
telemetry/telemetry_logger.py      <- prev_entry_hash in each entry
telemetry/chain_validator.py       <- NEW: O(n) chain validation
reproducibility/snapshot_utils.py  <- NEW: MerkleTree class
reproducibility/reproducibility_v16.py <- mastering_merkle_root in core hash
```

**Technical spec:**
```python
# Chained JSONL (Invariant #21):
# entry_n.prev_entry_hash = SHA256(entry_{n-1}_bytes)
# entry_0.prev_entry_hash = "GENESIS"

# Merkle Tree (Theorem H.4 extension):
# leaf_i = SHA256(stage_name_i || stage_checksum_i)
# parent  = SHA256(left_child || right_child)
# root    = mastering_merkle_root -> enters snapshot_checksum core
# 8 leaves: MonoBass, SpectralTilt, LUFSSlope, Multiband,
#            GlueBus, BarkMask, ModCoherence, Limiter
```

**Dependency:** P2.2 | **Unblocks:** P3.2

---

### P3.2 — Event Bus + Performance Profiling [1 session, MEDIUM]

**Files modified:**
```
telemetry/event_bus.py                   <- NEW: thread-safe pub/sub
generation/generation_loop_v20.py        <- emit() at key points
mastering/streaming_mastering_engine.py  <- stage_timer context manager
```

**Technical spec:**
```python
# TelemetryEventBus: thread-safe, non-blocking
# emit() overhead: < 0.1 ms guarantee
# EVENT_TYPES: generation_started, diffusion_step, mastering_stage,
#              iqs_computed, generation_complete, abort_triggered
# Snapshot non-hash extension: perf_profile{diffusion_ms, mastering_ms, vram_peak_mb}
```

**Dependency:** P3.1 | **Unblocks:** —

---

### P3.3 — editFLAM IQS epsilon-Term Extension [1 session, MEDIUM]

**Rationale (AudioChat 2025, arXiv:2602.17097):** IQS v0.9 measures output
quality but not edit quality. For segment regeneration (P2.2), need metric
for "was the edit executed correctly". OpenFLAM = frame-level audio-language.

**Files modified:**
```
metrics/flam_iqs_adapter.py  <- NEW: editFLAM -> epsilon normalization
metrics/iqs.py               <- add IQS_edit mode (epsilon term, optional)
metrics/mos_backends.py      <- optional OpenFLAM backend
```

**Technical spec:**
```python
# IQS_edit = IQS_base - epsilon * EditPenalty
# epsilon = 0.08  (separate from base weights, Sum_base=1.00 UNCHANGED)
# EditPenalty in [0, 1] via editFLAM normalization
# IQS_edit NOT passed to BFGS (Invariant #12 preserved)
# task=None -> IQS_edit = IQS_base (epsilon not applied)
```

**Dependency:** P2.3, P2.2 | **Unblocks:** —

---

### P3.4 — VST3 Integration via Pedalboard [1 session, MEDIUM]

**Rationale:** Stage 10 optional post-mastering VST3 processing.
FrozenVSTOperator pattern = analogous to TinyMOS (frozen, inference only).
Spotify pedalboard (Apache 2.0). Theorem B.1 compliant: VST is post-mastering.

**Files modified:**
```
vst/frozen_vst_operator.py   <- NEW: deterministic VST3 wrapper (Invariant #22)
vst/__init__.py               <- NEW
mastering/mastering_chain.py  <- optional Stage 10
operator_registry.py          <- register FrozenVSTOperator
```

**Invariant #22:** `plugin.reset()` called before `process()` per track.
Violation -> DetachabilityViolation abort.

**Dependency:** P1.1 | **Unblocks:** P3.5

---

### P3.5 — MIDI Extraction via basic-pitch [1 session, MEDIUM]

**Rationale:** MIDI extraction -> reference audio -> ACE-Step melody conditioning
(CONTRACT SS18 timbre_fix_frame). Closes melody control gap.

**Files modified:**
```
midi/midi_extractor.py       <- NEW: basic-pitch wrapper
midi/midi_to_reference.py    <- NEW: MIDI -> WAV -> reference conditioning
midi/__init__.py              <- NEW
generation/reference_audio.py <- add from_midi() path
```

**Dependency:** P3.4 | **Unblocks:** D5

---

### P4.1 — IRC-5 Style Multiband Limiter (Theorem C.2) [2 sessions, LOW]

**Rationale (Theorem C.2):** N=8 candidates evaluated simultaneously.
Suboptimality <= 1/N = 12.5% at N=8. Enables -4 LUFS headroom without
pumping/distortion. Closes iZotope Ozone 12 IRC 5 gap.

**Files:** `mastering/irc5_limiter.py` (NEW), `mastering/limiter_core.py`
**Dependency:** P1.1, P2.3

---

### P4.2 — Multiband Stereo Width + Correlation Guard [1 session, LOW]

**Files:** `mastering/stereo.py` (add multiband_width, correlation_guard),
`mastering/mid_side_stage.py`

**Spec:** 3-band width profiles per genre (bass always mono, f < 120 Hz).
Correlation guard: if corr(L,R) < 0.3 -> reduce Side by 6 dB.
**Dependency:** P4.1

---

### P4.3 — Demucs Neural Stem Separation [5-8 sessions, FUTURE]

```
Replaces: STFT 12-band -> HTDemucs neural source separation (~83M params)
Benefit:  ~85% -> ~95% separation quality (vocal bleed reduction)
Same 12 canonical stems + processing order (immutable standard)
File:  mastering/stem_separator.py (rewrite backend)
Ref:   Defossez et al. 2023, Hybrid Transformers for Music Source Separation
```

---

### P4.4 — Caption Optimizer LLM [5+ sessions, FUTURE]

```
Goal:    user_prompt -> better_caption -> Qwen3 (frozen) -> DiT (frozen)
Does NOT violate CONTRACT SS18: DiT sees only caption
Requires: 1000+ generation pairs with IQS labels
Ref:     Phase L from TZ v0.7
```

---

### P5 — /sleep Memory Consolidation (mylm-inspired) [3-5 sessions, FUTURE]

**Rationale (Hannun 2024, https://github.com/awni/mylm):**
In batch mastering of 100+ tracks (freelance workflows), session knowledge
(client LUFS preferences, genre tilt profiles, IQS-optimal parameter clusters)
dissipates at session end. The /sleep mechanism from mylm provides a design
pattern for consolidating session knowledge into persistent Control Plane
memory without violating Theorem B.1 (Frozen Operator Boundary) or
Axiom A6 (No Hidden Learning). See PROTOCOL v0.8 ADDENDUM §M.10 for full theory.

**Key insight:** The /sleep mechanism modifies ONLY the Control Plane:
  - Caption builder genre preferences (phi_clients)
  - Genre HD floor adjustments (Phi_genre)
  - BFGS warm-start vectors (theta_warmstart)
Signal/Param layers (DiT/LM/VAE/Qwen3) remain strictly frozen (Theorem B.1).

**Algorithm:**
```
1. Collect session snapshots (N tracks, JSONL)
2. Extract high-IQS clusters (IQS >= 0.65) per genre
3. Generate Q&A pairs:
     Q: "Optimal phi for genre=ambient, LUFS=-14, H_n=0.30?"
     A: "tilt=-1.2, multiband_hf=+0.8, glue_ratio=2.5, IQS=0.72, cs=abc123"
4. Update Phi_genre: new_floor = P10(H_n | IQS >= 0.65, genre)
5. Update theta_warmstart = mean(phi* | IQS >= 0.65)
6. Append to cpmmemory.jsonl (chained, Invariant #21)
7. Assert: operator_graph_checksum unchanged (frozen weights)
```

**Theorem M.10.3 compatibility:**
```
d(theta_N)/dt = 0       (Axiom A2, DiT/LM/VAE/Qwen3 frozen)
Sum(IQS weights) = 1.00 (Lemma B.5-A unchanged)
BFGS receives IQS_base  (Invariant #12 preserved)
No hidden learning      (Axiom A6, all CPM changes explicit + logged)
```

**LoRA extension (research, beyond P5):**
Fine-tune ONLY caption_optimizer LoRA adapters (not DiT/LM/VAE/Qwen3).
Training data: Q&A pairs from /sleep logs.
Requires: >= 200 session snapshots + offline supervised training.

**Files:** `personalization/sleep_consolidator.py`, `personalization/__init__.py`
**Dependency:** P4.4 (caption system mature), >= 200 session snapshots

---

## SECTION 3. CANONICAL FORMULA BASE v0.11

### SS3.1 IQS v0.9 — Single Source of Truth

```
IQS = alpha*MOS_n + eta*HD_n_genre - beta*Distance_n
      - gamma*Phase_n - delta*Drift_n - zeta*Bark_n

alpha = 0.50  (MOS proxy, dominant quality signal)
eta   = 0.25  (Harmonic Density, genre-aware positive term)
beta  = 0.08  (Distance penalty, KAD primary from v1.1)
gamma = 0.07  (Phase coherence penalty)
delta = 0.06  (Loudness drift penalty: |integrated_LUFS - target_LUFS|)
zeta  = 0.04  (Bark-band stereo coherence penalty)
              Sum = 1.00  (Invariant #7, IQS_WEIGHTS_CHECKSUM sealed)
```

**Lemma SS3.1a (IQS range proof):**
```
All normalized terms in [0, 1].
IQS_max = alpha + eta = 0.50 + 0.25 = 0.75
  (achieved when MOS_n=1, HD_n=1, all penalties=0)
IQS_min = -(beta+gamma+delta+zeta) = -(0.08+0.07+0.06+0.04) = -0.25
  (achieved when MOS_n=0, HD_n=0, all penalties=1)
IQS in [-0.25, 0.75]  TIGHT (both bounds achievable)
Proof: IQS is affine in all terms, each in [0,1]. QED.
```

```
HD_n_genre = min(1.0, H_raw / floor_genre)
Genre HD floors:
  ambient:    0.25  classical: 0.20  jazz:       0.30
  lofi:       0.30  pop:       0.45  rnb:        0.40
  hip_hop:    0.50  trap:      0.50  edm:        0.65
  house:      0.60  techno:    0.58  dubstep:    0.55
  metal:      0.60  rock:      0.55  punk:       0.50
  neurofunk:  0.62  dnb:       0.58  phonk:      0.52
  default:    0.45  (all other genres)

MOS_n calibration (calibrated multi-factor proxy):
  MOS_raw = 2.5 + 2.0 * q_composite,  q_composite in [0, 1]
  MOS_n   = (MOS_raw - 1.0) / 4.0     in [0.375, 0.875]
  q = 0.30*q_loudness + 0.30*q_dynamics + 0.20*q_spectral + 0.20*q_stereo
```

### SS3.2 Quality Fusion J

```
J = 0.60 * IQS + 0.40 * QA_external

Studio threshold: J >= 0.65
  At QA_external=0.82: IQS >= (0.65 - 0.328)/0.60 = 0.537
  At QA_external=0.70: IQS >= (0.65 - 0.280)/0.60 = 0.617
  At QA_external=0.60: IQS >= (0.65 - 0.240)/0.60 = 0.683

IQS pass_gate (pre-filter): IQS >= 0.55
  Math: J = 0.60*0.55 + 0.40*0.82 = 0.660 >= 0.65  OK
  (ensures J >= 0.65 at typical QA_external = 0.82)

IQS_edit (edit mode only, P3.3):
  IQS_edit = IQS_base - 0.08 * EditPenalty
  Sum_base weights unchanged = 1.00 (Invariant #7)

delta_user_minus_iqs = IQS_user_rating - IQS  (logged, non-hash snapshot)
```

### SS3.3 Typical Track IQS Calculation

```
Typical good track: MOS_n=0.85, HD_n_genre=0.65, D_n=0.20, P_n=0.08, L_n=0, B_n=0
  IQS = 0.50*0.85 + 0.25*0.65 - 0.08*0.20 - 0.07*0.08 = 0.566
  J   = 0.60*0.566 + 0.40*0.82 = 0.668 >= 0.65  PASS (studio)

Worst acceptable case (to reach J=0.65 threshold):
  IQS = 0.537  (at QA_external=0.82)
  IQS = 0.617  (at QA_external=0.70)

Improvement v0.7 -> v0.9 (empirical per-genre estimates):
  Genre      IQS v0.7  IQS v0.9  Delta
  ambient    0.12      0.52      +0.40
  classical  0.11      0.51      +0.40
  edm        0.22      0.64      +0.42
  hip-hop    0.20      0.61      +0.41
  jazz       0.14      0.54      +0.40
  metal      0.22      0.63      +0.41
  neurofunk  0.23      0.63      +0.40
  lofi       0.15      0.55      +0.40
  pop        0.18      0.59      +0.41
  Mean       0.18      0.58      +0.40
```

### SS3.4 EBU R128 (P1.1)

```
Integrated_LUFS = -0.691 + 10*log10(Sum_i G_i*E_i)  [ITU-R BS.1770-4 SS4.1]
  G_i = K-weighting gain per channel
  E_i = mean square of K-weighted channel signal
  Gating: 400ms blocks, -70 LUFS absolute gate, -10 LU relative gate

LRA = P95(LUFS_3s) - P10(LUFS_3s)  [EBU R128 SS3.5]
  Window: 3s, hop: 100ms

TruePeak_dBTP = 20*log10(max|x_upsampled|)  [BS.1770-4 SS4.2]
  4x polyphase FIR, Kaiser beta=8.0, N_taps >= 256
  Invariant #6: TruePeak_dBTP < -0.1 dBTP
```

### SS3.5 KAD (Theorem F.1)

```
KAD(P,Q) = MMD^2_k(P,Q)
  = E_{x,x'~P}[k(x,x')] - 2*E_{x~P,y~Q}[k(x,y)] + E_{y,y'~Q}[k(y,y')]
  k(x,y) = exp(-||phi(x)-phi(y)||^2 / (2*sigma^2))
  sigma = median({||phi(x_i)-phi(x_j)|| : i<j})  (data-driven bandwidth)
  phi: PANN embedding (primary), CLAP-MA (secondary), VGGish (fallback)

Properties (Theorem F.1):
  KAD(P,Q) = 0 <=> P = Q  (no Gaussian assumption, distribution-free)
  E[KAD_hat] = KAD(P,Q)   (unbiased U-statistic)
  KAD_hat converges at O(1/sqrt(n))  (vs FAD O(d/n))
  N >= 50 sufficient  (vs FAD N >= 500)
```

### SS3.6 Mastering Lipschitz Chain

```
Current (N_tap=2048):
  L_M_fixed_theory = B_linf * G_max = 3.6097 * 4.0 = 14.44
  L_full = L_N * L_core ~= 17.89 * 1.0214 = 18.27 (empirical)

After P1.3 (N_tap=8192, Theorem C.1):
  B_linf_new <= 1.05
  L_M_fixed_new <= 1.05 * 4.0 = 4.2  (-70% improvement)
  L_full_new <= 17.89 * 4.2 ~= 75    (-71% vs current 258)
```

### SS3.7 PER (Phoneme Error Rate, Muse 2026)

```
PER = (S + D + I) / N
  S = phoneme substitutions
  D = phoneme deletions
  I = phoneme insertions
  N = total reference phonemes (from forced alignment)
  Lower PER = better lyric fidelity
  Typical target: PER < 0.15 for good lyric generation
```

### SS3.8 Phase Alignment (Theorem C.3)

```
X_L'(f) = X_L(f) * exp(+j * phi_diff(f) / 2)
X_R'(f) = X_R(f) * exp(-j * phi_diff(f) / 2)
Mono(f)  = (X_L' + X_R') / 2,  for f < f_c = 120 Hz

E_aligned(f) = |X_L(f)|^2  (rotation-invariant)
E_sum(f)     = |X_L(f)|^2 * cos^2(phi/2)  (M/S, lossy)
Improvement: E_aligned/E_sum = 1/cos^2(phi/2) >= 1
```

---

## SECTION 4. SCIENTIFIC PAPERS 2025-2026

### SS4.1 Muse (Jiang et al., 2026)

**Key results for NOESIS:**
- Segment-level style conditioning -> Phase P2.2 (SegmentTag, 14-tag vocab)
- PER metric as benchmark measure -> added to benchmark suite (SS3.7)
- Qwen3-0.6B + MuCodec: architectural reference (alternative backend, SS34)
- Multi-turn conversational style prompting -> P2.2 SegmentStyle

**NOESIS advantage over Muse:**
  Cryptographic reproducibility (SHA-256 + Merkle tree), per-genre LUFS targets,
  deterministic mastering pipeline (Muse has no mastering pipeline),
  IQS quality scoring (Muse has no quality gate).

```bibtex
@article{jiang2026muse,
  title   = {Muse: Towards Reproducible Long-Form Song Generation
             with Fine-Grained Style Control},
  author  = {Jiang, Changhao and others},
  journal = {arXiv preprint arXiv:2601.03973},
  year    = {2026}
}
```

### SS4.2 KAD: No More FAD! (Chung et al., ICML 2025)

**Key results:**
- KAD = MMD-based, distribution-free, unbiased (Theorem F.1)
- N >= 50 sufficient (vs FAD N >= 500) - critical for NOESIS N=10 benchmark
- Lower compute: O(n^2) MMD vs O(d^2) eigendecomposition for FAD
- PANN variant: Spearman rho = 0.74 (vs VGGish 0.51 on DCASE 2023)

**Impact:** P1.2 replaces Invariant #16. Distance_n uses KAD instead of FAD.
Improves IQS beta-term calibration at small benchmark sizes.

```bibtex
@inproceedings{chung2025kad,
  title     = {KAD: No More FAD! An Effective and Efficient
               Evaluation Metric for Audio Generation},
  author    = {Chung, Yoonjin and Eu, Pilsun and Lee, Junwon and
               Choi, Keunwoo and Nam, Juhan and Chon, Ben Sangbae},
  booktitle = {ICML 2025},
  year      = {2025},
  url       = {https://arxiv.org/abs/2502.15602}
}
```

### SS4.3 MAD: Aligning Text-to-Music Evaluation (Huang et al., 2025)

**Key results:**
- MAUVE Audio Divergence (MAD) > FAD-CLAP in human preference alignment
- SRC(MAD, human) >= SRC(FAD-VGGish, human) + 0.15 (Theorem F.2)
- Human preference data: 7800 pairwise comparisons on 6000 AI-generated tracks
- Warning: context window issues in MAD with audio > 10s

**Impact:** Theorem F.2 in PROTOCOL v0.8. Future tertiary metric in fad_backend.py.

```bibtex
@article{huang2025mad,
  title   = {Aligning Text-to-Music Evaluation with Human Preferences},
  author  = {Huang, Yichen and Novack, Zachary and Saito, Koichi and others},
  journal = {arXiv preprint arXiv:2503.16669},
  year    = {2025}
}
```

### SS4.4 Music Evaluation Survey (Kader & Gong, 2025)

**Key results:**
- 50+ metrics across 6 categories taxonomized
- FAD, CLAP-score, KLD align poorly with human preferences
- KAD (MMD) and MAD (MAUVE) better alternatives (confirms P1.2)
- PER = standard for vocal/lyric quality evaluation (confirms P2.2)
- Western-centric bias in 5.7% non-Western genres (NOESIS 33-genre taxonomy mitigates)

```bibtex
@article{kader2025survey,
  title   = {A Survey on Evaluation Metrics for Music Generation},
  author  = {Kader, Faria Binte and Gong, Yuan},
  journal = {arXiv preprint arXiv:2509.00051},
  year    = {2025}
}
```

### SS4.5 Benchmarking Music Generation (Retkowski et al., ICASSP 2025)

**Key results:**
- FAD-CLAP-MA: best correlation with human music quality (confirms P1.2 secondary)
- LAION-MA: best for text-audio alignment (cosine similarity)
- Suno v3.5: highest human preference (commercial baseline for NOESIS benchmarking)
- Elo/Bradley-Terry ratings for model comparison (implement in benchmark_ab.py)

```bibtex
@inproceedings{retkowski2025benchmarking,
  title     = {Benchmarking Music Generation Models and Metrics
               via Human Preference Studies},
  author    = {Retkowski, Fabian and others},
  booktitle = {ICASSP 2025},
  year      = {2025}
}
```

### SS4.6 mylm /sleep Memory Consolidation (Hannun, 2024)

**Key results for NOESIS:**
- KV-cache analogy: session knowledge in Control Plane memory
- /sleep command: consolidates session -> persistent weight updates
- Q&A generation: auto-extracts key parameter patterns
- LoRA online fine-tuning: fast (minutes on M-series) vs full training

**NOESIS adaptation (Phase P5):** Applied to caption builder preferences only.
DiT/LM/VAE/Qwen3 remain frozen (Theorem B.1). All CPM changes SHA-256 logged.
See PROTOCOL v0.8 ADDENDUM §M.10 for formal theory.

```bibtex
@misc{hannun2024mylm,
  title  = {mylm: A minimal language model with /sleep memory consolidation},
  author = {Hannun, Awni},
  year   = {2024},
  url    = {https://github.com/awni/mylm}
}
```

---

## SECTION 5. DISTANCE METRIC HIERARCHY v0.11

```
Tier 1 (PRIMARY):   KAD(PANN)
  Theorem F.1: distribution-free, unbiased, O(1/sqrt(n)) convergence
  Use for: IQS Distance_n (beta term), benchmark comparison
  N_min = 50, sigma = median bandwidth (data-driven)

Tier 2 (SECONDARY): FAD-CLAP-MA
  Theorem F.2 corollary: best human-preference correlation among tractable metrics
  Use for: validation when KAD unavailable, text-music alignment
  N_min = 200 for stable estimates

Tier 3 (FALLBACK):  FAD-VGGish
  Preserved for audit continuity (Invariant #16 legacy fallback)
  Use for: backward compatibility only
  Limitations: Gaussian assumption, N_min >= 500, rho=0.51

Tier 4 (FUTURE):    MAD (MAUVE Audio Divergence)
  Theorem F.2: SRC(MAD, human) >= SRC(FAD-VGGish, human) + 0.15
  Use for: research evaluation when CLAP-MA embeddings available
  Limitation: expensive computation, audio > 10s context issues

Selection logic in fad_backend.py:
  if pann_available: return KAD(pann)
  elif clap_ma_available: return FAD(clap_ma)
  else: return FAD(vggish)  # legacy fallback, log WARNING
```

---

## SECTION 6. BENCHMARK SUITE v0.11

### SS6.1 10 Genre Track Suite

| # | Genre | LUFS Target | Drift Contract | IQS v0.7 (old) | IQS v0.9 (expected) |
|---|-------|-------------|----------------|-----------------|----------------------|
| 1 | ambient | -14.0 | PASS | 0.12 | 0.52+ |
| 2 | edm | -9.0 | PASS | 0.22 | 0.62+ |
| 3 | lofi | -16.0 | PASS | 0.15 | 0.55+ |
| 4 | pop | -14.0 | PASS | 0.18 | 0.58+ |
| 5 | hip-hop | -12.0 | PASS | 0.20 | 0.60+ |
| 6 | jazz | -14.0 | PASS | 0.14 | 0.54+ |
| 7 | classical | -18.0 | PASS | 0.11 | 0.50+ |
| 8 | neurofunk | -8.0 | PASS | 0.23 | 0.63+ |
| 9 | metal | -8.0 | PASS | 0.22 | 0.62+ |
| 10 | (variable) | genre-specific | PASS | variable | 0.55+ |

### SS6.2 New Metrics After P1.x

After P1.1 (EBU R128):
- `true_peak_dbtp < -0.1` per track
- `lra_db in [3, 15]` (healthy dynamics)

After P1.2 (KAD):
- `kad_score` per track (lower = better reference alignment)
- `distance_metric = KAD_PANN` in snapshot

After P2.2 (Segments):
- `per_metric` per track (lyric fidelity, lower = better)
- `segment_styles_checksum` in snapshot core

### SS6.3 GPU Validation Criteria (R.GPU)

```
IQS mean >= 0.60  (all 10 tracks)
No track IQS < 0.40
LUFS drift = 0.000 per track
TruePeak < -0.1 dBTP per track
WAV checksum stable (2 runs = identical)
operator_graph_checksum unchanged
```

---

## SECTION 7. COMPLETE PRIORITY MATRIX v0.11

```
Phase           Sessions  Priority  Dependency
R.GPU           CURRENT   CRITICAL  -
P1.1 True-Peak  1         CRITICAL  R.GPU
P1.2 KAD        1         CRITICAL  R.GPU
P1.3 Crossover  1         CRITICAL  R.GPU
P2.1 Phase-Bass 1         HIGH      P1.3
P2.2 Segments   1         HIGH      P1.1
P2.3 Stem EQ    1         HIGH      P1.2
P3.1 Merkle+Ch  1         MEDIUM    P2.2
P3.2 EventBus   1         MEDIUM    P3.1
P3.3 editFLAM   1         MEDIUM    P2.3, P2.2
P3.4 VST3       1         MEDIUM    P1.1
P3.5 MIDI       1         MEDIUM    P3.4
D1 Protocol v0.8 1 doc    CRITICAL  P1.1+P1.2+P1.3
D2 Contract v0.8 1 doc    HIGH      D1
D3 Roadmap v0.11 1 doc    HIGH      D1  (THIS DOCUMENT)
D4 Mastering v0.9 1 doc   HIGH      P1.1+P1.3
D5 Table I+Index 1 doc    MEDIUM    P3.5
P4.1 IRC-5 Lim  2         LOW       P1.1, P2.3
P4.2 MB Width   1         LOW       P4.1
P4.3 Demucs     5-8       FUTURE    P3.5
P4.4 Caption LLM 5+       FUTURE    1000+ generations
P5   /sleep CPM  3-5       FUTURE    P4.4, >=200 snapshots
```

---

## SECTION 8. GENERATOR AGNOSTICISM NOTE

NOESIS DHCF-FNO is NOT restricted to ACE-Step v1.5 Turbo.
The mastering pipeline (Stages 0-11), IQS scoring, LUFS control,
and reproducibility system are backend-agnostic.

Any backend satisfying Definition G.1 (PROTOCOL v0.8 ADDENDUM §G.1) is supported:
```
generate(caption: str, seed: int, duration_s: float, sr: int = 48000)
    -> np.ndarray  # float32, stereo or mono
```

Tested or planned alternative backends:
- ACE-Step v1.5 Turbo (reference, current)
- MusicGen (Meta)
- Stable Audio (Stability AI)
- AudioLDM 2 (HKUST)
- Muse 2026 (Jiang et al., Qwen3+MuCodec)

See CONTRACT v0.8 SS34, PROTOCOL v0.8 Part G.

---

*Document: NOESIS_ROADMAP_v0_11.md*
*Version: v0.11 (2026-03-07)*
*Author: Ilia Bolotnikov / AMAImedia.com (2026)*
*Supersedes: NOESIS_ROADMAP_AND_FORMULAS_v0_10.md*
*Status: ACTIVE*
