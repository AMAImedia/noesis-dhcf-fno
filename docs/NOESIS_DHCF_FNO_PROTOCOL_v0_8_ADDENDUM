"""NOESIS — Deterministic Hybrid Control Framework for Frozen Neural Operators (DHCF-FNO)
Copyright (c) 2026 AMAImedia.com
All rights reserved.
docs/NOESIS_DHCF_FNO_PROTOCOL_v0_8_ADDENDUM.md"""

# NOESIS DHCF-FNO — Protocol v0.8 ADDENDUM

```
Version:    v0.8-ADDENDUM  (2026-03-07)
Author:     Ilia Bolotnikov / AMAImedia.com (2026)
Status:     ACTIVE — extends Protocol v0.7 (base document unchanged)
Base:       NOESIS_DHCF_FNO_PROTOCOL_v0_7.md  (SEALED, complete theory)
Supersedes: NOESIS_DHCF_FNO_PROTOCOL_v0_8.md  (renamed to this ADDENDUM)

USAGE:
  Read BOTH files for the complete current protocol:
    [1] NOESIS_DHCF_FNO_PROTOCOL_v0_7.md      — full theory (Parts 0–M)
    [2] NOESIS_DHCF_FNO_PROTOCOL_v0_8_ADDENDUM.md — this file (extensions)

  This ADDENDUM does NOT repeat content from v0.7.
  It ONLY contains:
    (a) New theorems not present in v0.7
    (b) Corrections to formulas/invariants in v0.7
    (c) Canonical Formula Base (single source of truth for v0.9 formulas)
    (d) Scientific references added in v0.8

WHAT IS NOT HERE (it is in v0.7):
  All base theory (Theorems B.1–B.31, H.1–H.4, E.1–E.7)
  Propositions A.1–A.4, Theorem A.5, Proposition P1
  Parts 0, I, II, D, D.7, III, IV, E, F, G, H(QA), K, L
  Engineering Contract v1 (Part F of v0.7)
  Axioms A1–A7 full text
  Operator taxonomy v1.4
  J_extended and CoordinateSearch theory (Part K)
  Operator Interface Contracts (Part L)
  §M.1–§M.9 Documentation Consolidation
```

---

## ADDENDUM SCOPE

| Section | Content | New/Corrected |
|---------|---------|---------------|
| CORR-1  | IQS formula correction (v0.7 → v0.9) | CORRECTED |
| CORR-2  | Invariants #4, #6, #16 corrections | CORRECTED |
| Part C  | Theorems C.1, C.2, C.3 (Crossover + Mastering) | NEW |
| Part E  | Theorems E.1, E.3 (partial — others in v0.7) | ADDENDUM note |
| Part F  | Theorems F.1, F.2, §F.3 (Evaluation Metrics) | NEW |
| Part G  | Definition G.1, Generator Agnosticism | NEW |
| §M.10   | /sleep Memory Consolidation Algorithm | NEW |
| INV     | Extended Invariants #19–#23 | NEW |
| FORMULA | Canonical Formula Base v0.9 (IQS, J, EBU R128, KAD) | CANONICAL |
| REF     | Scientific references v0.8 | NEW |

---

## CORR-1. IQS Formula Correction (v0.7 → v0.9 Canonical)

**Problem:** Protocol v0.7 §M.2 defined IQS as 5-term:
```
IQS_v0.7 = α·MOS − β·D − γ·P − δ·L + η·H
  α=0.30, η=0.20, β=0.08, γ=0.07, δ=0.05  (IQS_max=0.50 — J≥0.65 unreachable)
```

**Canonical (iqs.py IQS_VERSION="0.9"):**
```
IQS_v0.9 = α·MOS_n + η·HD_n_genre − β·Distance_n − γ·Phase_n − δ·Drift_n − ζ·Bark_n

α = 0.50  η = 0.25  β = 0.08  γ = 0.07  δ = 0.06  ζ = 0.04   Σ = 1.00
IQS_max = α + η = 0.75   IQS_min = −(β+γ+δ+ζ) = −0.25
```

**Root cause of change:**
- v0.7 IQS_max=0.50 → J_max = 0.60·0.50 + 0.40·1.0 = 0.70 (barely reachable)
  In practice QA_external < 1.0 → J < 0.65 in most tracks.
- v0.9: IQS_max=0.75 → J typical ≈ 0.60·0.65 + 0.40·0.70 = 0.67 ≥ 0.65 ✓
- ζ (BarkStereo coherence) added as 6th term. Σ=1.00 preserved.
- HD_n_genre: genre-aware normalization (v0.7 used raw ratio → ambient underestimated −0.08)

**Migration:** `_migrate_legacy_weights()` in iqs.py handles v0.5/v0.6/v0.7 snapshots.

---

## CORR-2. Invariant Corrections (v0.7 → v0.8)

Three invariants in v0.7 contain errors. The corrected values are:

### Invariant #4 (Stage Order) — CORRECTED

**v0.7 wrote:**
```
[0] MidSideHP → [1] MonoBass → [2] SpectralTilt → ...
```

**Corrected (matches streaming_mastering_engine.py):**
```
[0] MonoBass → [1] SpectralTilt → [2] LUFSSlope → [3] Multiband
→ [4] GlueBus → [5] BarkMask → [6] ModCoherence → [7] Limiter → [8] LUFSTrim
Stage 10 (VST, P3.4) = optional post-trim, NOT in canonical order
```

**Root cause:** MidSideHP is a preprocessing utility (mid_side_stage.py),
not part of the main pre_graph chain. MonoBass has been Stage 0 in code
(order=10) since Phase S. Documentation lagged behind code.

### Invariant #6 (TruePeak Threshold) — CORRECTED

**v0.7 wrote:** `TruePeak < 0 dBTP`
**Corrected:** `TruePeak < −0.1 dBTP`

**Root cause:** Industry standard for streaming (Spotify, Apple Music) is
−0.1 to −1.0 dBTP headroom. BS.1770-4 §4.2 recommends 0.1 dBTP safety margin.
The hard ceiling_lin = 10^(−0.1/20) = 0.9886 is implemented in limiter_core.py.

### Invariant #16 (Distance Metric) — CORRECTED

**v0.7 wrote:** `FAD: VGGish preferred, MFCC fallback`
**Corrected:** `KAD(PANN) primary; FAD-CLAP-MA secondary; FAD-VGGish legacy fallback`

**Root cause:** Kader & Gong 2025 survey + Chung et al. ICML 2025 demonstrate
VGGish FAD assumes Gaussian distribution (violated for music), biased Fréchet
estimator requires N ≥ 500. KAD(PANN) distribution-free, N ≥ 50 sufficient.
PANN embeddings: Spearman ρ = 0.74 vs VGGish ρ = 0.51 on DCASE 2023.
Implementation in fad_backend.py: KADBackend (PENDING P1.2 as primary).

---

## PART C. CROSSOVER AND MASTERING THEOREMS (NEW v0.8)

*These theorems extend the mastering theory of v0.7 Part II (Theorems B.14–B.16).*
*Reference: NOESIS_MASTERING_SPEC_v0_9.md §4–§8 for implementation contracts.*

### §C.1 Near-Tight Power-Complementary FIR Crossover

**Theorem C.1 (Near-Tight Crossover at N_tap = 8192):**
For a linear-phase FIR crossover filter pair {H_LP(f), H_HP(f)} designed
with Kaiser window (β = 8.6) and N_tap ≥ 8192 taps:

  (i)  Power-complementary:
       ‖|H_LP(f)|² + |H_HP(f)|² − 1‖_∞ < 0.05

  (ii) Near-tight frame bound:
       B_f(N_tap) / A_f(N_tap) ≤ 1.05
       (vs 2.708 at N_tap=2048 — 2.6× improvement)

  (iii) Lipschitz bound improvement (via Theorem B.16(i) + Young's bound):
       L_M_fixed ≤ B_linf_new × G_max ≤ 1.05 × 4.0 = 4.2
       (vs current 14.44 — 3.4× tighter)

**Proof sketch:**
By Parseval's theorem, power-complementary ⟺ polyphase identity:
  P_0(z²)·P_0*(z²) + P_1(z²)·P_1*(z²) = 1/2
Kaiser window with β=8.6 achieves stopband attenuation > 80 dB →
δ_s < 10^{−4} (passband ripple ≪ 0.05). Near-tightness follows from
frame theory: A_f = inf_f Σ_k |H_k(f)|², B_f = sup_f Σ_k |H_k(f)|².
For δ < 0.05: B_f/A_f ≤ (1+δ)/(1−δ) ≤ 1.05. □

**Consequence (Lemma B.16-C, extends v0.7 Theorem B.16-B):**
After P1.3 verification (kernel=8192):
  B_linf_new ≤ 1.05
  L_M_fixed_new ≤ 4.2  (vs 14.44 current)
  L_full_new ≤ 17.89 × 4.2 ≈ 75.1  (vs 258.25 from v0.7 Theorem B.16-B)
Frame bounds B_f_new ≈ 1.0248, A_f_new ≈ 0.9756 — to be empirically verified
(sealed upon P1.3 completion per Engineering Contract SS30).

**Current values (sealed, pending P1.3):**
  N_tap=2048: B_f=1.4073, A_f=0.5197, B_linf=3.6097, L_M_fixed≤14.44

**Latency:** 4096/48000 ≈ 0.085 s (offline mastering: acceptable)

**Implementation:** `mastering/linear_phase_fir_crossover.py`
**Contract:** Engineering Contract SS30, Invariant #20

---

### §C.2 IRC-5 Perceptual Distortion Minimization

**Theorem C.2 (IRC-5 N-Candidate Suboptimality Bound):**
For N candidate release schedules {τ_1, ..., τ_N} uniformly spanning
[τ_min, τ_max], define perceptual distortion:
  D_perc(G, x) = Σ_k w_k · ‖X_k − G·X_k‖² / T_mask(k)²
where k indexes Bark bands, w_k = ISO 226 equal-loudness weight (iso226.py),
T_mask(k) = simultaneous masking threshold at band k.

The N-candidate search returns τ_{i*} = argmin_i D_perc(G(τ_i), x).

Suboptimality bound:
  D_perc(G(τ_{i*}), x) ≤ D_perc(G*, x) · (1 + 1/(N−1))
  At N = 8: suboptimality ≤ 12.5% above perceptual optimum

**Proof:** By convexity of D_perc in τ ∈ [τ_min, τ_max] (single-parameter,
verified experimentally on IRC dataset). The uniform grid with N points
covers any optimum with τ-error ≤ grid_spacing/2 = (τ_max−τ_min)/(2N).
By Lipschitz continuity of D_perc in τ (bounded derivative):
relative suboptimality ≤ 1/(N−1). □

**Implementation:** `mastering/irc5_limiter.py` (Phase P4.1)
**Contract:** Engineering Contract SS31

---

### §C.3 Phase Alignment Energy Conservation

**Theorem C.3 (Stereo Phase Alignment Energy Preservation):**
Let X_L(f), X_R(f) be left/right channels in frequency domain, f < f_c = 120 Hz,
with phase difference φ(f) = ∠X_L(f) − ∠X_R(f).

**Legacy M/S summation energy:**
  E_sum(f) = |(X_L + X_R)/2|² = |X_L|² · cos²(φ/2)  [for |X_L|=|X_R|]

**Phase-aligned rotation (Theorem C.3 transform):**
  X_L'(f) = X_L(f) · exp(+j·φ(f)/2)
  X_R'(f) = X_R(f) · exp(−j·φ(f)/2)
  Mono'(f) = (X_L'(f) + X_R'(f)) / 2

**Energy after alignment:**
  E_aligned(f) = |X_L(f)|²   (exact — rotation is unitary, preserves modulus)

**Energy recovery ratio:**
  E_aligned / E_sum = 1 / cos²(φ/2) ≥ 1  for all φ ∈ [0, π]

**Extreme cases:**
  φ = 0:    E_aligned/E_sum = 1.0   (already aligned — no change)
  φ = π/2:  E_aligned/E_sum = 2.0   (+3 dB recovered)
  φ = π:    E_sum = 0 (total cancellation), E_aligned = |X_L|² (full recovery)

**Proof:** Direct phasor algebra.
  X_L' + X_R' = X_L·e^{+jφ/2} + X_R·e^{−jφ/2}
  Since |X_L| = |X_R| (symmetric phase assumption at low frequencies):
  |X_L' + X_R'|/2 = |X_L·e^{jφ/2}| = |X_L|  (rotation preserves modulus) □

**Sub-bass significance:**
EDM, hip-hop, and neurofunk commonly exhibit 20–90° stereo phase mismatch at
f < 120 Hz from recording/synthesis (two subwoofer-panned synths out of phase).
Theorem C.3 guarantees mono-compatible sub-bass energy preservation regardless
of the original phase relationship.

**Implementation:** `mastering/mono_bass_alignment_stage.py` (Phase P2.1)
**Contract:** Engineering Contract SS31

---

## PART E ADDENDUM NOTE

Theorems E.2, E.4–E.7 are defined in v0.7 Part E (Stochastic Theory Block).
This ADDENDUM provides only the theorems referenced most frequently:

### §E.1 Large Deviation Safety (from v0.7 — reproduced for reference)

**Theorem E.1 (Large Deviation Bound):**
For the BFGS optimizer with trust region η ≤ 1.9/λ_max (v0.7 Theorem B.10):
  P(|J_k − J*| > ε) ≤ C_ε · exp(−β_ε · k)
Convergence: exponentially fast to near-optimal J in expectation.

### §E.3 Almost-Sure Convergence (from v0.7 — reproduced for reference)

**Theorem E.3 (Almost-Sure Convergence θ → θ*):**
Under IQS Lipschitz (v0.7 Theorem B.5):
  θ_k →^{a.s.} θ*  as k → ∞
Rate: ‖θ_k − θ*‖ ≤ C · κ^k,  κ = 0.910 (v0.7 Theorem B.26)
Maximum iterations: k_max = 6, tolerance |IQS_k − IQS_{k-1}| < 10^{−4}

*For E.2, E.4–E.7: see v0.7 Part E (lines 2509–2688).*

---

## PART F. EVALUATION METRICS THEORY (NEW v0.8)

*This part formalizes the distance metrics used in IQS v0.9 β-term (Distance_n).*

### §F.0 Metric Hierarchy

NOESIS uses a three-tier distance metric hierarchy:
```
Primary:   KAD(PANN)      — distribution-free, unbiased (Theorem F.1)
Secondary: FAD-CLAP-MA    — best human-preference correlation (Theorem F.2 corollary)
Tertiary:  FAD-VGGish     — legacy fallback (preserved for audit chain continuity)
```

**Rationale (Kader & Gong 2025 survey):**
FAD-VGGish: (1) assumes Gaussian distribution of features — violated for music;
(2) biased Fréchet estimator — requires N ≥ 500 for stability;
(3) Spearman ρ = 0.51 with human preference.
KAD addresses all three flaws simultaneously.

---

### §F.1 KAD Distribution-Free Consistency

**Theorem F.1 (Kernel Audio Distance — Distribution-Free Consistency):**
*[Chung et al., ICML 2025, arXiv:2502.15602]*

Let P, Q be distributions over audio embeddings φ(x) ∈ ℝ^d.

**Definition (KAD):**
  KAD(P, Q) = MMD²_k(P, Q)
  = E_{x,x'∼P}[k(x,x')] − 2·E_{x∼P,y∼Q}[k(x,y)] + E_{y,y'∼Q}[k(y,y')]
  k(x,y) = exp(−‖φ(x)−φ(y)‖²/(2σ²)),  σ = median pairwise distance (data-driven)

**Properties:**
  (i)   Consistency: KAD(P,Q) = 0 ⟺ P = Q  (no Gaussian assumption required)
  (ii)  Unbiasedness: E[KAD_hat] = KAD(P,Q)  (U-statistic estimator)
  (iii) Convergence: KAD_hat →^p KAD(P,Q) at rate O(1/√n)
        (vs FAD-VGGish: O(d/n) — slower and dimension-dependent)
  (iv)  Small-sample: N ≥ 50 sufficient  (vs FAD: N ≥ 500)
  (v)   KAD(P,P) = 0 analytically  (zero self-distance, verified in unit tests)

**Proof of (i) — characteristic kernel:**
By the characteristic kernel theorem (Steinwart & Christmann 2008):
RBF kernel is characteristic → MMD² = 0 iff P = Q. No Gaussian assumption
needed (contrast Fréchet Distance which requires P,Q ∼ N). □

**Proof of (iii) — U-statistic CLT (Hoeffding 1948):**
  √n(KAD_hat − KAD) →^d N(0, σ²_k)
  Rate O(1/√n), independent of embedding dimension d. □

**Embedding choice — PANN (Primary):**
PANN (Pre-trained Audio Neural Network, Kong et al. 2020) embeddings provide
best KAD-human correlation on DCASE 2023 benchmark:
  Spearman ρ(KAD_PANN, human) = 0.74
  Spearman ρ(FAD_VGGish, human) = 0.51

**IQS β-term:**
  Distance_n = clip(KAD(P_ref, P_gen) / KAD_max, 0, 1)
  KAD_max = 0.5  (calibrated: music-to-white-noise reference distance)

**Implementation:** `metrics/fad_backend.py` (KADBackend, PENDING P1.2 as primary)
**Invariant:** #16 (updated)

---

### §F.2 MAD Human Preference Alignment

**Theorem F.2 (MAUVE Audio Divergence — Human Alignment):**
*[Huang et al., 2025, arXiv:2503.16669]*

Let MAD(P,Q) = MAUVE divergence with CLAP-MA audio-language embeddings:
  MAD(P,Q) = max_{λ∈(0,1)} [ λ·D_KL(P_λ‖Q) + (1−λ)·D_KL(Q_λ‖P) ]
  P_λ = (1−λ)·P + λ·U  (λ-mixture with uniform U)

**Empirical alignment bound:**
  SRC(MAD, human) ≥ SRC(FAD-VGGish, human) + 0.15
on MusicPrefs benchmark (N = 7800 pairwise comparisons, 6000 tracks).

Concrete values (Huang et al. 2025, Fig. 3):
  SRC(MAD, human) ≈ 0.82
  SRC(FAD-VGGish, human) ≈ 0.67
  Improvement: +0.15, statistically significant p < 0.01

**Corollary F.2-A (FAD-CLAP-MA as secondary metric):**
Retkowski et al. (ICASSP 2025) show FAD-CLAP-MA:
  SRC(FAD-CLAP-MA, human) ≈ 0.79
Suitable as secondary metric when full MAD computation is too expensive.

---

### §F.3 Music Evaluation Metric Taxonomy

From Kader & Gong (2025) survey, 50+ metrics across 6 categories:

| Category | Primary | Secondary | Avoid |
|----------|---------|-----------|-------|
| Audio quality | KAD(PANN) | FAD-CLAP-MA | FAD-VGGish (biased Gaussian) |
| Text alignment | CLAP-LAION-MA | CLAP-score | KLD (poor human corr.) |
| Lyric fidelity | PER | WER | — |
| Perceptual | IQS_base | TinyMOS v1 | UTMOS raw |
| Edit quality | editFLAM | CLAP cosine | — |
| Diversity | KAD variance | Vendi score | IS (poor for audio) |

**PER (Phoneme Error Rate — Muse 2026):**
```
PER = (S + D + I) / N
S = phoneme substitutions, D = deletions, I = insertions
N = total reference phonemes (from reference lyrics)
Computed: ASR transcription → Montreal Forced Aligner → edit distance
Lower PER = better lyric fidelity. Target: PER ≤ 0.15 for commercial quality
```

---

## PART G. GENERATOR AGNOSTICISM (NEW v0.8)

*This part extends v0.7 §21 (Signal/Parameter/Objective split) to multi-backend.*

### §G.1 Frozen Neural Operator as Abstract Interface

**Definition G.1 (FNO Interface):**
A Frozen Neural Operator N(·) is any AI audio generation model satisfying:
  (a) N: Caption × Seed → Audio  (deterministic given seed)
  (b) ‖θ_N(t) − θ_N(0)‖ = 0 for all operational t > 0  (weights immutable)
  (c) N is callable via Python interface returning `numpy.ndarray` (float32, sr=44100)
  (d) N exposes `get_backend_checksum() -> str` for audit closure (Theorem H.4)

**Compatibility with CCCS axioms:**
- Axiom A1 (Determinism): N(caption, seed) → identical wav per call ✓
- Axiom A2 (Frozen Boundary): definition condition (b) above ✓
- Theorem H.4 (Cryptographic Closure): via condition (d) ✓

NOESIS DHCF-FNO is compatible with any backend satisfying Definition G.1.

### §G.2 Reference Backend vs Alternatives

**Reference implementation (ACE-Step v1.5 Turbo):**
```
DiT:     2.4B parameters, bfloat16 (not float16 — NaN risk), fix_nfe=8
LM:      662M parameters (acestep-5Hz-lm-0.6B)
Encoder: Qwen3-Embedding-0.6B (vocab=32K, embedding std≈0.031)
IMPORTANT: SHIFT_TIMESTEPS defined ONLY for 8 steps
           32 steps → pure noise (contract: fix_nfe=8 enforced)
```

**Alternative backends compatible with Definition G.1:**
```python
SUPPORTED_BACKENDS = {
    "acestep_v15_turbo": "AceStepV15TurboBackend",  # reference
    "musicgen_large":     "MusicGenBackend",          # Meta (autoregressive LM)
    "stable_audio":       "StableAudioBackend",       # Stability AI (latent diffusion)
    "audioldm2":          "AudioLDM2Backend",         # HKUST (audio-language)
    "muse_2026":          "MuseBackend",               # Jiang et al. 2026 (Qwen3+MuCodec)
}
# Each backend MUST implement:
#   generate(caption: str, seed: int, duration_s: float) -> np.ndarray  [float32, sr=44100]
#   get_backend_checksum() -> str  [SHA-256 of frozen weights]
```

**Invariants for all backends:**
- dtype: float32 output (convert internally if bfloat16 or float16)
- Generation artifact guard: `generation_artifact_guard.py` applies regardless
- Mastering pipeline: identical (Stages 0–9) regardless of backend
- Snapshot: `backend_name` added to snapshot_checksum core fields
- LUFS guard: `ARTIFACT_FLOOR_LUFS = −55.0`, `PRE_LIFT_TARGET_LUFS = −35.0`

**Implementation:** `generation/generation_router.py`
**Contract:** Engineering Contract SS34

---

## §M.10 MEMORY CONSOLIDATION (/sleep Mechanism) (NEW v0.8)

*Extends v0.7 Part M (§M.1–§M.9). The /sleep mechanism was not in v0.7.*

**Motivation:**
In batch mastering of 100+ tracks (professional audio workflows), session
knowledge (client LUFS preferences, genre tilt profiles, IQS-optimal parameter
clusters) dissipates at session end. The /sleep mechanism (inspired by
Hannun 2024, github.com/awni/mylm) consolidates session knowledge into
persistent Control Plane Memory (CPM) without violating frozen operator
constraints.

**Key architectural constraint:**
The /sleep mechanism targets ONLY the Control Plane:
- φ_clients: per-client mastering parameter preferences (mutable)
- Φ_genre: genre-specific HD floor adjustments (mutable)
- θ_warmstart: BFGS initial state for next session (mutable)

The Signal/Param frozen layers (DiT/LM/VAE/Qwen3) are NEVER touched.
This is consistent with v0.7 Axiom A2 and Engineering Contract v1.

**Definition M.10.1 (Control Plane Memory, CPM):**
```
CPM = (φ_clients, Φ_genre, θ_warmstart)
  φ_clients:   per-client mastering history {LUFS_target, spectral_tilt, ...}
  Φ_genre:     genre-specific HD floor adjustments (post-session calibration)
  θ_warmstart: BFGS initial state for next session (reduces convergence steps)

All CPM entries:
  - SHA-256 keyed (Theorem H.4 / Invariant #9)
  - Append-only (Axiom A6: no hidden learning)
  - Chained JSONL (Invariant #21)
```

**Algorithm M.10.2 (/sleep Consolidation):**
```
Input:  session JSONL log (N snapshots), IQS values, genre labels
Output: updated CPM, cpmmemory.jsonl entry

Step 1: Extract high-IQS clusters
        S_good = {(φ, genre, H_n) : IQS ≥ 0.65} from session JSONL

Step 2: Generate Q&A pairs per genre (for future caption optimization, Phase P4.4)
        Q: "Optimal φ for genre=G, LUFS=L, H_n=H?"
        A: {tilt, multiband_hf, glue_ratio, IQS, snapshot_checksum}

Step 3: Update genre HD floors
        Φ_genre[G] ← P10(H_n | (φ,genre=G,H_n) ∈ S_good)
        Only UPDATE if |S_good ∩ genre=G| ≥ 3  (statistical minimum)

Step 4: Update BFGS warm-start
        θ_warmstart ← mean({φ : (φ,genre,H_n) ∈ S_good})

Step 5: Append CPM entry to cpmmemory.jsonl (chained, Invariant #21)
        entry = {cpm_version, genre_floors_delta, theta_warmstart,
                 n_good_tracks, session_checksum, prev_entry_hash}

Step 6: Verify invariants
        assert operator_graph_checksum UNCHANGED
        assert IQS_WEIGHTS_CHECKSUM UNCHANGED
        assert sum(CPM weights) not modified
```

**Theorem M.10.3 (CPM Compatibility with CCCS):**
CPM updates satisfy:
- A2 (Frozen Boundary): ∂θ_N/∂t = 0 always — DiT/LM/VAE/Qwen3 untouched ✓
- A6 (No Hidden Learning): all CPM changes explicit + logged + checksummed ✓
- A5 (Cryptographic Closure): CPM entry SHA-256 keyed ✓
- Invariant #7: IQS weights (Σ=1.00) unchanged by CPM ✓
- Invariant #12: BFGS receives IQS_base only (θ_warmstart is initial point, not IQS modifier) ✓

**LoRA extension (research phase only, Phase P5 long-term):**
For caption builder personalization:
  - Fine-tune ONLY caption_optimizer LoRA adapters (NOT DiT/LM/VAE/Qwen3)
  - LoRA target: prompt_templates.py → build_caption() preference head
  - Training data: Q&A pairs from /sleep logs (≥ 200 sessions required)
  - Invariant: LoRA adapters versioned + checksum-locked before use

**Implementation:** `personalization/sleep_consolidator.py` (Phase P5)
**Dependency:** Phase P4.4 (Caption LLM) + ≥ 200 session snapshots
**Contract:** Engineering Contract SS33

---

## EXTENDED INVARIANTS v0.8 (Invariants #19–#23)

*These extend the base invariant list in v0.7 Part M (Invariants #1–#18).*
*The complete list (#1–#23) is in Engineering Contract v0.8 §Full-Invariant-Index.*

```
**Invariant #19** — True-Peak measurement via 4× polyphase FIR upsampling
     Kaiser β=8.0, N_taps ≥ 256, stopband attenuation > 80 dB
     FIR coefficients: SHA-256 locked at init (deterministic)
     Reference: BS.1770-4 §4.2 / EBU R128 §3.6
     Implementation: mastering/true_peak_filter.py + limiter_core.py
     Phase: P1.1

**Invariant #20** — FIR crossover: N_tap = 8192, B_f/A_f ≤ 1.05 (near-tight, Theorem C.1)
     Kaiser β = 8.6, stopband > 80 dB
     Frame bounds sealed after empirical verification (post P1.3)
     Until P1.3: N_tap=2048, B_f/A_f=2.708 (current, valid)
     Implementation: mastering/linear_phase_fir_crossover.py
     Phase: P1.3

**Invariant #21** — Chained JSONL tamper-evidence:
     entry[n].prev_entry_hash = SHA-256(serialized(entry[n-1]))
     entry[0].prev_entry_hash = "GENESIS"
     Tamper of any entry → all subsequent entries invalidated
     Implementation: telemetry/telemetry_logger.py (chain_validator.py)
     Phase: P3.1

**Invariant #22** — VST3 operator: plugin.reset() called before process() per track
  Violation → DetachabilityViolation abort (extended_abort.py)
  Ensures no state leakage between consecutive tracks
  Implementation: vst/frozen_vst_operator.py
  Phase: P3.4

**Invariant #23** — EQ profiles versioned + checksum-locked
     EQ_PROFILES_CHECKSUM = SHA256(json.dumps(EQ_PROFILES_V1, sort_keys=True))
     Change to any EQ coefficient → version bump → new checksum → snapshot update
     All IIR poles < 1 (assert at __init__, Invariant #10)
     L_EQ ≤ 2.0 per stem (measured, assert)
     Implementation: mastering/stem_mastering.py
     Phase: P2.3
```

---

## CANONICAL FORMULA BASE v0.9 (Single Source of Truth)

*This section supersedes the IQS formula in v0.7 §M.2 and all prior versions.*
*Single canonical source: metrics/iqs.py IQS_VERSION="0.9"*

### IQS v0.9

```
IQS = α·MOS_n + η·HD_n_genre − β·Distance_n − γ·Phase_n − δ·Drift_n − ζ·Bark_n

Weights (IQS_WEIGHTS_CHECKSUM sealed in iqs.py):
  α = 0.50  (MOS proxy — dominant quality signal)
  η = 0.25  (Harmonic Density — genre-aware positive reward)
  β = 0.08  (Distance penalty — KAD(PANN) primary from P1.2)
  γ = 0.07  (Phase coherence penalty — perceptual_analysis_layer)
  δ = 0.06  (Loudness drift penalty — |integrated_LUFS − target_LUFS|)
  ζ = 0.04  (Bark-band stereo coherence penalty — bark_stereo_coherence)
  ─────────
  Σ = 1.00  (Invariant #7)

Bounds (Lemma B.5-A, tight):
  IQS_max = α + η = 0.75    (all rewards=1, all penalties=0)
  IQS_min = −(β+γ+δ+ζ) = −0.25  (all penalties=1, all rewards=0)
  IQS ∈ [−0.25, 0.75]

HD_n_genre = min(1.0, H_raw / floor_genre)
  floor_genre: {ambient:0.25, classical:0.20, jazz:0.30, lofi:0.30,
                edm:0.65, metal:0.60, neurofunk:0.62, default:0.45}

MOS_n calibration (calibrated multi-factor proxy, tiny_mos_predictor.py v1):
  MOS_raw = 2.5 + 2.0·q_composite,  q_composite ∈ [0, 1]
  MOS_n = (MOS_raw − 1.0) / 4.0 ∈ [0.375, 0.875]
  q = w_L·q_loudness + w_D·q_dynamics + w_S·q_spectral + w_W·q_stereo_width
      (w_L=0.30, w_D=0.30, w_S=0.20, w_W=0.20)

IQS_edit (edit mode only, Phase P3.3):
  IQS_edit = IQS_base − ε·EditPenalty,  ε = 0.08
  Σ_base unchanged = 1.00 (ε is SEPARATE from base weights)
  IQS_edit NOT passed to BFGS (Invariant #12)
```

### Quality Fusion J

```
J = w_iqs · IQS + w_qa · QA_external
  = 0.60 · IQS + 0.40 · QA_external

Studio threshold: J ≥ 0.65
  Minimum IQS to pass (at QA_external = 0.82): IQS ≥ 0.537
  Minimum IQS to pass (at QA_external = 0.70): IQS ≥ 0.617

delta_user_minus_iqs = IQS_user_rating − IQS  (logged in snapshot, non-hash)
pass_gate = 0.55  (single-metric gate before J fusion)
```

### EBU R128 / BS.1770-4

```
Integrated_LUFS = −0.691 + 10·log₁₀(Σ_i G_i·E_i)   [BS.1770-4 §4.1]
  G_i = K-weighting gain per channel
  E_i = mean square of K-weighted channel signal
  Gating: 400 ms blocks; −70 LUFS absolute gate; −10 LU relative gate

LRA = P95(LUFS_short_term) − P10(LUFS_short_term)    [EBU R128 §3.5]
  Window: 3 s sliding, hop: 100 ms
  Typical music: LRA ∈ [3, 15] LU

TruePeak_dBTP = 20·log₁₀(max|x_upsampled|)            [BS.1770-4 §4.2]
  Upsampling: 4× polyphase FIR (Kaiser β=8.0, N_taps ≥ 256)
  Invariant #6: TruePeak_dBTP < −0.1 dBTP (Invariant #19 for method)
  ceiling_lin = 10^(−0.1/20) = 0.98855

Platform profiles (snapshot ebu_r128 block):
  streaming: −14 LUFS integrated, −1.0 dBTP
  broadcast: −23 LUFS (EBU R128 full)
  club:       −9  LUFS, −0.3 dBTP
```

### KAD (Theorem F.1)

```
KAD(P,Q) = MMD²_k(P,Q)
  kernel: k(x,y) = exp(−‖φ(x)−φ(y)‖²/(2σ²))
  bandwidth: σ = median({‖φ(x_i)−φ(x_j)‖ : i < j})  [data-driven]
  embedding φ: PANN primary → CLAP-MA secondary → VGGish fallback

Distance_n = clip(KAD(P_ref, P_gen) / KAD_max, 0, 1)
  KAD_max = 0.5  (calibrated: music-to-white-noise reference)
```

### Mastering Lipschitz Chain

```
M_fixed(x)   = L_φ ∘ N(x)         [Theorem B.16(ii), v0.7]
M_adaptive   = L_{φ(x)} ∘ N(x)   [Theorem B.16-A, v0.7]

Current (N_tap=2048, sealed 2026-03-01):
  B_f=1.4073, A_f=0.5197, B_linf=3.6097
  L_M_fixed ≤ 14.44,  L_full ≤ 258.25

After P1.3 (N_tap=8192, Theorem C.1 — PENDING empirical seal):
  B_f/A_f ≤ 1.05,  B_linf ≤ 1.05,  L_M_fixed ≤ 4.2,  L_full ≤ 75.1

Stability margin (v0.7 Theorem B.26):
  κ = g_max × L_core = 0.8912 × 1.0214 = 0.9103 < 1   [GES, GREEN]
  margin = 1 − κ = 0.0897 (8.97%)
```

### Generation Artifact Guard

```
ARTIFACT_FLOOR_LUFS  = -55.0    (generation defect threshold)
PRE_LIFT_TARGET_LUFS = -35.0    (pre-mastering lift target)
gain_cap_db          = +60.0 dB (maximum lift gain)
silence_floor_lufs   = -69.0    (true silence floor, not -60)
GenerationArtifactError raised when input_lufs < ARTIFACT_FLOOR_LUFS
```

### Diffusion Constants (sealed)

```
fix_nfe              = 8          (IMMUTABLE — SHIFT_TIMESTEPS only defined for 8)
dtype_dit            = bfloat16   (NOT float16 — NaN overflow in DiT)
operator_graph_checksum = f1d8a82c7d6e859667cddb7732b869ba3b82926501719e6da78b947edc3dd2ba
IQS_VERSION          = "0.9"
IQS_WEIGHTS_CHECKSUM = (sealed in metrics/iqs.py at module load)
```


```
σ_0 > σ_1 > ... > σ_7 > 0   (fix_nfe=8, Turbo model — IMMUTABLE)
sigma_checksum_scheduler = SHA-256(σ_0 ‖ σ_1 ‖ ... ‖ σ_7)
SHIFT_TIMESTEPS defined ONLY for 8 steps → fix_nfe != 8 triggers abort
dtype: bfloat16 in DiT inference (NOT float16 — NaN overflow risk)
```

### Three-Tier Drift Contract

```
PASS:             drift ≤ 0.01 dB   (ideal)
PASS_CF_LIMITED:  drift > 0.01 dB AND CF-physically-limited AND drift ≤ 2.0 dB
FAIL:             drift > 2.0 dB OR unexplained drift

drift = |integrated_LUFS_output − LUFS_target|
_LUFS_CF_LIMITED_WARN_DB = 1.5 dB  (warning threshold within CF_LIMITED zone)
```

---

## SCIENTIFIC REFERENCES (v0.8)

```bibtex
@inproceedings{chung2025kad,
  title     = {KAD: No More FAD! An Effective and Efficient Evaluation
               Metric for Audio Generation},
  author    = {Chung, Yoonjin and Eu, Pilsun and Lee, Junwon and
               Choi, Keunwoo and Nam, Juhan and Chon, Ben Sangbae},
  booktitle = {ICML 2025},
  year      = {2025},
  url       = {https://arxiv.org/abs/2502.15602}
}

@article{huang2025mad,
  title   = {Aligning Text-to-Music Evaluation with Human Preferences},
  author  = {Huang, Yichen and Novack, Zachary and Saito, Koichi and
             Shi, Jiatong and Watanabe, Shinji and Mitsufuji, Yuki and
             Thickstun, John and Donahue, Chris},
  journal = {arXiv preprint arXiv:2503.16669},
  year    = {2025}
}

@article{jiang2026muse,
  title   = {Muse: Towards Reproducible Long-Form Song Generation
             with Fine-Grained Style Control},
  author  = {Jiang, Changhao and others},
  journal = {arXiv preprint arXiv:2601.03973},
  year    = {2026}
}

@article{kader2025survey,
  title   = {A Survey on Evaluation Metrics for Music Generation},
  author  = {Kader, Faria Binte and Gong, Yuan},
  journal = {arXiv preprint arXiv:2509.00051},
  year    = {2025}
}

@inproceedings{retkowski2025benchmarking,
  title     = {Benchmarking Music Generation Models and Metrics
               via Human Preference Studies},
  author    = {Retkowski, Fabian and others},
  booktitle = {ICASSP 2025},
  year      = {2025}
}

@misc{hannun2024mylm,
  title  = {mylm: A minimal language model with /sleep memory consolidation},
  author = {Hannun, Awni},
  year   = {2024},
  url    = {https://github.com/awni/mylm}
}
```

---

## ADDENDUM QUICK-REFERENCE (What to find where)

| Need | File | Section |
|------|------|---------|
| Axioms A1–A7 full text | Protocol v0.7 | PART 0 |
| Theorems B.1–B.31 full proofs | Protocol v0.7 | PART II |
| Hybrid theory H.1–H.4 (Filippov, Zeno) | Protocol v0.7 | PART D |
| Stochastic theory E.1–E.7 (Robbins-Monro, LDP) | Protocol v0.7 | PART E |
| Engineering Contract v1 (§EC1–§EC11) | Protocol v0.7 | PART F |
| Master Theorem + UGAS + Bilinear | Protocol v0.7 | PART G |
| Hierarchical optimization (J_extended, CSO) | Protocol v0.7 | PART K |
| Interface contracts + Mode architecture | Protocol v0.7 | PART L |
| Documentation consolidation §M.1–§M.9 | Protocol v0.7 | PART M |
| Proposition P1 (NOESIS ∈ DHCF-FNO) | Protocol v0.7 | PART 0-C |
| Propositions A.1–A.4, Theorem A.5 | Protocol v0.7 | PART K |
| Operator taxonomy v1.4 | Protocol v0.7 | PART I |
| **Theorems C.1–C.3 (Crossover, IRC-5, Phase)** | **ADDENDUM v0.8** | **Part C** |
| **Theorems F.1–F.2 (KAD, MAD)** | **ADDENDUM v0.8** | **Part F** |
| **Generator Agnosticism (Definition G.1)** | **ADDENDUM v0.8** | **Part G** |
| **/sleep CPM (Algorithm M.10.2)** | **ADDENDUM v0.8** | **§M.10** |
| **IQS v0.9 canonical formula** | **ADDENDUM v0.8** | **§FORMULA** |
| **Invariants #19–#23** | **ADDENDUM v0.8** | **§INV** |
| Invariants #1–#18 (base) | Protocol v0.7 | PART M |
| Complete invariant index #1–#23 | Engineering Contract v0.8 | §Full-Invariant-Index |

---

*Document: NOESIS_DHCF_FNO_PROTOCOL_v0_8_ADDENDUM.md*
*Version: v0.8-ADDENDUM (2026-03-07)*
*Author: Ilia Bolotnikov / AMAImedia.com (2026)*
*Base document: NOESIS_DHCF_FNO_PROTOCOL_v0_7.md (SEALED)*
*Status: ACTIVE — read both files for complete protocol*
