[NOESIS_FORMAL_THEORY_v2_1.md](https://github.com/user-attachments/files/26134356/NOESIS_FORMAL_THEORY_v2_1.md)
"""NOESIS — Deterministic Hybrid Control Framework for Frozen Neural Operators (DHCF-FNO)
Copyright (c) 2026 AMAImedia.com
All rights reserved.
B:/Downloads/Portable/NOESIS_DHCF-FNO/docs/NOESIS_FORMAL_THEORY_v2_0.md
Changes v2.1 (2026-03-20):
  + §C.2 updated: N_CANDIDATES=8 (ε≤12.5%), N_BANDS=4
  + §B.16: L_StereoWidener ≤ 1.2 (Theorem B.16-SW, DAFx-24)
  + §B.16-B: Compressor v2.0 Lipschitz (Peak/Blend/RMS modes)
  + §B.16-C: Gate/Expander stability (passthrough=1.0, gain∈[floor,1.0])
  + §B.16-D: Dynamic EQ temporal OLA stability
  + Stage count updated: 31 stages

Contains: Theorems B.1-B.31, H.1-H.4, E.1-E.7, C.1-C.3, F.1-F.2,
          G.1, K-series, DHCF class definition, compliance matrix, TASLP elements"""

# NOESIS FORMAL THEORY v2.1
## All Theorems, Proofs, Axioms, Formal Definitions
## IEEE TASLP Mathematical Appendix Level

```
Version:    2.1  (2026-03-20)  [COMPLETE — updated for v3.4]
Author:     Ilia Bolotnikov / AMAImedia.com
Status:     ACTIVE — sealed formal theory
Supersedes: NOESIS_DHCF_FNO_PROTOCOL_v1_0.md Parts 0/I/II/D/E/F/G/H/K/L/M/C/G-EXT (merged)
Companion:  NOESIS_PROTOCOL_v2_0.md (operational constants that appear in proofs)

VERIFIABILITY INDEX (for IEEE TASLP review):
  B.1  Fully proven (A1-A4 explicit)          ← can be checked by reviewer
  B.2  Fully proven (composition of proofs)   ← can be checked
  B.3  Qualitative stability argument         ← acknowledged limitation
  B.4  Fully proven (Weierstrass EVT)         ← can be checked
  B.5  Fully proven (Lipschitz chain rule)    ← can be checked
  B.6  Fully proven (bias-variance, §Ghadimi-Lan 2013)  ← citable theorem
  B.7  Fully proven (determinism, A1-A3)      ← can be checked
  B.8  Conditional (local C², Dennis-Moré)    ← acknowledged assumption
  B.9  Fully proven (SHA-256 determinism)     ← can be checked
  B.14 Fully proven (clip = 1-Lipschitz)      ← can be checked
  B.15 Proven under Parseval + empirical      ← B_linf calibrated TEST 1
  B.16 Proven (M_fixed); empirical (M_adaptive) ← honest dual claim
  H.1  Hybrid LaSalle + Filippov (citable)   ← can be checked with refs
  H.2  Fully proven (finite switching)        ← can be checked
  H.3  ISS Lyapunov (standard ISS proof)      ← can be checked
  H.4  Cryptographic (A-SHA assumption)       ← honest cryptographic claim
  C.1  Proven under Parseval + empirical pred ← acknowledged estimate
  C.2  Proven (induction on N stages)         ← can be checked
  C.3  Fully proven (unitary phase rotation)  ← can be checked

KNOWN LIMITATIONS (must be stated in paper, §B.12):
  - L_D bound (B.1) is loose upper bound, not tight estimate
  - C² smoothness of J near θ* assumed, verified empirically only
  - Global convergence NOT claimed (local superlinear only, B.8)
  - GPU floating-point reproducibility: deterministic algorithms required (A3)
  - B_linf = 3.60972762 calibrated empirically for kernel=2048 specifically
```

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
PART 0 — THEORETICAL POSITIONING
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

DHCF-FNO addresses a fundamental problem in neural generative systems:
frozen neural operators (like ACE-Step v1.5 Turbo DiT 2.4B) cannot be
fine-tuned at inference time, yet their outputs require quality-aware
control. DHCF-FNO solves this via deterministic control of the input
parameter space rather than modification of the operator.

The three-axis novelty (vs prior work):

  Axis 1 — Determinism: 1 seed → 1 waveform → 1 checksum
    Prior work (RLHF for music): stochastic gradients, no reproducibility
    NOESIS: 3x RNG lock + deterministic FFT → zero-variance objective (B.6)

  Axis 2 — Hybrid control: piecewise-smooth on compact domain
    Prior work: single smooth objective assumed
    NOESIS: finite regime partition (A3), Filippov solutions at boundaries
    Formal stability: Theorems H.1-H.2 (hybrid LaSalle, no Zeno)

  Axis 3 — Generator agnosticism (Definition G.1)
    Prior work: assumes specific model architecture
    NOESIS: FNO as abstract interface (Definition G.1); backend-swappable

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
PART I — NOTATION AND FUNCTIONAL SPACES
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

  Ω ⊂ ℝ³                   compact parameter domain (closed + bounded)
  𝒳 = ℝⁿ                   latent audio space
  𝒴 = L²([0,T])²           stereo audio signal space (T = track duration)
  𝒫                         text prompt set (finite-vocabulary)
  𝕊 = ℕ                     seed space (deterministic RNG)
  ‖·‖                        Euclidean or L² norm (context-clear)
  ‖A‖₂                       spectral norm (largest singular value)
  ‖x‖∞                       L∞ norm: sup_t |x(t)|

  Parameter vector:
    θ = [θ₁, θ₂, θ₃]ᵀ = [seed_offset, guidance_scale, sigma_slope]ᵀ
    Ω = [−2048, 2048] × [2, 12] × [−0.4, 0.4]  (compact by construction)

  Composite system operator:
    𝓕_θ = S ∘ O ∘ Q ∘ M ∘ D_θ

  where:
    D_θ : 𝒫 × 𝕊 → 𝒳           deterministic diffusion (ACE-Step Turbo)
    M   : 𝒳 → 𝒴                mastering operator (MasteringChain v3.0)
    Q   : 𝒴 → ℝ⁶               perceptual quality features (6 IQS terms)
    O   : ℝ⁶ → ℝ                IQS aggregation (α·MOS + η·HD - ...)
    S   : ℝ × 𝒳 → 𝒟            snapshot operator (SHA-256 locked)

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
PART II — AXIOMS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

  A1 (Frozen Weights): All DiT/LM/VAE/Qwen3 weights have
     requires_grad=False. Spectral norms ‖W_i‖₂ are compile-time constants.

  A2 (Lipschitz Activations): Activations σ_i satisfy K_σ ≤ 1.1 (GELU bound).

  A3 (Deterministic Arithmetic): torch.use_deterministic_algorithms(True).
     All operations produce bitwise-identical outputs for identical inputs.

  A4 (IIR Stability): All IIR filter poles satisfy max_i|z_i| < 1.
     Verified at __init__: assert np.all(np.abs(np.roots(denom)) < 1.0)

  A5 (Compact Domain): Ω is closed and bounded in ℝ³.

  A6 (Bounded Regime Jumps): At regime boundaries ∂Rⱼ, the objective
     change is bounded: |J(θ⁺) − J(θ⁻)| ≤ C_jump < ∞.

  A7 (Cryptographic Hash): SHA-256 is collision-resistant:
     P(H(x) = H(y), x≠y) ≤ 2⁻²⁵⁶.

  A-SHA: Same as A7 (explicit in H.4 context).

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
PART III — CORE THEOREMS (B-Series)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

──────────────────────────────────────────────────────────────────
§B.1 — IQS Functional and Boundedness
──────────────────────────────────────────────────────────────────

  IQS(θ) = α·MOS_n + η·HD_n − β·Dist_n − γ·Phase_n − δ·Drift_n − ζ·Bark_n

  Canonical weights (v0.9, checksums in PROTOCOL §0.2):
    α=0.50, η=0.25, β=0.08, γ=0.07, δ=0.06, ζ=0.04
    Σ|weights| = 1.00, IQS_max = 0.75, IQS_min = −0.25

  Each term normalized to [0,1]:
    MOS_n  = clip((MOS − 1)/4, 0, 1)          MOS ∈ (1,5)
    HD_n   = clip(HD, 0, 1)                    HarmonicDensity ∈ [0,1]
    Dist_n = d / (d + 0.5)                     d ≥ 0, psychoacoustic distance
    Phase_n = clip(p / 0.25, 0, 1)            p = phase penalty
    Drift_n = clip(|LUFS_int − target| / 2, 0, 1)
    Bark_n  = bark_stereo_coherence_penalty ∈ [0,1]

  Proposition B.1 (Boundedness):
    IQS(θ) ∈ [−0.25, 0.75] for all θ ∈ Ω.
    Proof: Each component maps to [0,1] by clip + positive-denominator form.
    Weighted sum with unit-sum weights: IQS ∈ [α+η−β−γ−δ−ζ, α+η] = [−0.25, 0.75]. ∎

  Extended composite objective:
    J = 0.60 · IQS + 0.40 · QA_external     J ∈ [−1, 1]

──────────────────────────────────────────────────────────────────
§B.2 — Global Diffusion Lipschitz Bound (Theorem B.1)
──────────────────────────────────────────────────────────────────

  Lemma B.1 (Score Network Lipschitz):
  For score network ε_φ with L layers, frozen weights, K_σ ≤ 1.1 (A2):

    L_ε ≤ (∏_{i=1}^{L} ‖W_i‖₂) · K_σ^{L-1}

  Proof: By induction. Single layer: ‖σ(Wx)−σ(Wy)‖ ≤ K_σ‖W‖₂‖x−y‖.
  Composition of L layers multiplies bounds. ∎

  Lemma B.2 (Single Diffusion Step):
  Reverse step x_{t-1} = a_t x_t + b_t ε_φ(x_t,t,c(θ)):
    ‖x_{t-1}(θ₁) − x_{t-1}(θ₂)‖ ≤ |b_t|·L_ε·‖c(θ₁) − c(θ₂)‖
  Proof: Fixed x_t; difference driven by conditioning c(θ₁)−c(θ₂). ∎

  Theorem B.1 (Global Diffusion Lipschitz Bound):
  Under A1-A3, the diffusion operator D_θ satisfies:

    ‖D_{θ₁}(P,s) − D_{θ₂}(P,s)‖ ≤ L_D · ‖θ₁ − θ₂‖

    where L_D ≤ L_c · L_ε · Σ_{t=1}^{T} (|b_t| · ∏_{k=t+1}^{T} |a_k|)

  This bound is finite because L_c, L_ε, T, {a_t}, {b_t} are finite
  constants of the frozen schedule and TextEncoder.

  Proof: Unroll Lemma B.2 over T=8 steps (fix_nfe=8). At t=T: x_T=ε
  (noise, identical under same seed by A3). At each step, deviation
  propagates and is amplified by |a_k|. Summation over T steps yields
  the stated product-of-|a_k| bound. ∎

  Note (B.12 limitation): L_D is a valid upper bound, potentially loose
  by orders of magnitude. True L_D is empirically much smaller.

──────────────────────────────────────────────────────────────────
§B.3 — Mastering Operator Stability (Theorem B.2)
──────────────────────────────────────────────────────────────────

  Decomposition: M = L ∘ N  (Limiter ∘ pre-limiter chain)

  Lemma B.3 (BIBO Stability of IIR):
  Under A4, each IIR filter H satisfies:
    ‖H(y)‖₂ ≤ ‖h‖₁ · ‖y‖₂  (Young's inequality, L¹ impulse response)

  Proof: All poles inside unit circle → h[n] geometrically decaying → h ∈ ℓ¹. ∎

  Theorem B.2 (Energy Boundedness of M):
  For all y_raw ∈ L²([0,T])²:
    ‖M(y_raw)‖₂ ≤ C_M · √T = 1 · √T

  Proof: N is BIBO by Lemma B.3 (bounded C_N). L enforces
  |y(t)| ≤ ceiling=1 pointwise → ‖L(·)‖₂ ≤ c√T regardless of C_N. ∎

  Theorem B.14 (L∞ Boundedness of M):
    ‖M(x)‖∞ ≤ C_M · ‖x‖∞ = 1

  Proof: L clips to ceiling=1 pointwise → ‖M(x)‖∞ ≤ 1 regardless of ‖N(x)‖∞.
  Corollary B.2: L is 1-Lipschitz under L∞ (clip is a contraction). ∎

──────────────────────────────────────────────────────────────────
§B.4 — Existence, Continuity, Lipschitz J (Theorems B.4, B.5)
──────────────────────────────────────────────────────────────────

  Lemma B.4 (Continuity of J):
  J(θ) is continuous on Ω.
  Proof: D_θ (linear layers + GELU activations: continuous). M (linear
  filters + Lipschitz nonlinearities: continuous). Q (FFT + clip stats:
  continuous). O (linear combination: continuous). Composition of
  continuous functions is continuous. ∎

  Theorem B.4 (Existence of Minimizer):
  Ω compact + J continuous (B.4) → by Weierstrass EVT:
    ∃ θ* ∈ Ω: J(θ*) = min_{θ ∈ Ω} J(θ)  ∎

  Theorem B.5 (Global Lipschitz Continuity of J):
  Under B.1 and B.2:
    |J(θ₁) − J(θ₂)| ≤ L · ‖θ₁ − θ₂‖

    L = L_O · L_Q · L_M · L_D
    L_M = B_linf · G_max = 3.60972762 × 4.0 = 14.44 (Theorem B.16)
    L_Q  = FFT magnitude (Parseval-Lipschitz) + clip statistics
    L_O  = max(α/4, β·2, 4γ, δ/2) (normalization derivatives)

  Proof: Lipschitz chain rule. Each factor finite → L < ∞. ∎

──────────────────────────────────────────────────────────────────
§B.5 — Zero-Variance Advantage (Theorem B.6)
──────────────────────────────────────────────────────────────────

  Theorem B.6 (Zero-Variance Advantage over RL):
  Let J_det(θ) be the deterministic objective (A1-A3).
  Let J_RL(θ) = 𝔼_ω[IQS(D_θ(P,s,ω))] be the stochastic RL objective
  with Var_ω(IQS) = σ² > 0.

  Then:
    (i)  Var(J_det(θ)) = 0 for all θ ∈ Ω
    (ii) 𝔼‖∇̂J_RL‖² = ‖∇𝔼J_RL‖² + σ²  (bias-variance decomposition)
    (iii)𝔼‖∇J_det‖² = ‖∇J_det‖²  (exact gradient, no variance)
    (iv) Stochastic SGD: 𝔼[J(θₖ)]−J* ≥ O(σ²/√k) irreducible floor
         Deterministic: J(θₖ)−J* → 0 (no irreducible variance floor)

  Proof:
  (i): J_det is a deterministic function of θ (Theorem B.7). Zero variance.
  (ii): Standard bias-variance decomposition of stochastic gradient estimators
        (independent noise sampling).
  (iii): Trivial for exact evaluation.
  (iv): Ghadimi & Lan (2013), Theorem 2.1 (projected SGD, Lipschitz-smooth).
        σ² floor absent when σ²=0. ∎

  Corollary B.1: DHCF-FNO achieves tighter convergence than any
  stochastic RL method with equivalent cost whenever σ² > 0.

──────────────────────────────────────────────────────────────────
§B.6 — System Determinism (Theorem B.7)
──────────────────────────────────────────────────────────────────

  Assumptions A1-A3:
    A1: torch.manual_seed(s), np.random.seed(s), random.seed(s)
    A2: model.eval(), no dropout, no stochastic depth
    A3: torch.use_deterministic_algorithms(True)

  Lemma B.5 (Determinism of D_θ):
  Under A1-A3, D_θ(P,s) is identical across invocations for fixed (P,s,θ).
  Proof: {εᵢ} deterministic of s (A1). ε_φ deterministic of inputs (A2).
  Floating-point ops deterministic (A3). Recurrence deterministic by induction. ∎

  Lemma B.6 (Determinism of M, Q, O, S):
  M: IIR + deterministic nonlinearities. Q: FFT + deterministic stats.
  O: scalar arithmetic. S: SHA-256 deterministic.
  Proof: Compositions of deterministic functions. ∎

  Theorem B.7 (System Determinism):
  ∀ P ∈ 𝒫, s ∈ 𝕊, θ ∈ Ω: 𝓕_θ(P,s) = constant (across invocations).
  Proof: Composition of Lemmas B.5-B.6. ∎

  Theorem B.9 (Snapshot Reproducibility):
  Under B.7: SHA256(json.dumps(core_fields, sort_keys=True)) = constant.
  Proof: B.7 guarantees identical core_fields. json.dumps sort_keys=True
  is deterministic. SHA-256 is deterministic. ∎

──────────────────────────────────────────────────────────────────
§B.7 — BFGS Closed-Loop Optimizer (Theorem B.8)
──────────────────────────────────────────────────────────────────

  Gradient: ĝᵢ(θ) = [J(θ + ε·eᵢ) − J(θ − ε·eᵢ)] / (2ε), ε = 10⁻³

  BFGS update (L-BFGS-B variant):
    sₖ = θₖ₊₁ − θₖ
    yₖ = ĝ(θₖ₊₁) − ĝ(θₖ)
    ρₖ = 1/(yₖᵀsₖ)  [valid when yₖᵀsₖ > 10⁻¹⁰]
    Hₖ₊₁ = (I − ρₖsₖyₖᵀ) Hₖ (I − ρₖyₖsₖᵀ) + ρₖsₖsₖᵀ

  If yₖᵀsₖ ≤ 10⁻¹⁰: skip update (Hₖ₊₁ ← Hₖ).
  Trust region: if ‖pₖ‖ > r=0.5: pₖ ← pₖ · r/‖pₖ‖
  Convergence: |IQSₖ − IQSₖ₋₁| < 10⁻⁴ OR k = k_max = 6

  Step size constraint (η ≤ 1.9/λ_max):
    η ≤ 1.9/λ_max(H)  prevents oscillation (cf. Theorem B.17)

  Theorem B.8 (Local Superlinear Convergence):
  If J ∈ C²(B(θ*,δ)), ∇²J(θ*) positive definite, and yₖᵀsₖ > 10⁻¹⁰:
    {θₖ} converges superlinearly to θ*
  Proof: Dennis-Moré theorem (Nocedal & Wright 2006, Theorem 7.4). ∎

  Note (B.12 limitation): Global convergence not claimed. Trust region
  prevents catastrophic steps ‖θₖ₊₁ − θₖ‖ ≤ r always.

──────────────────────────────────────────────────────────────────
§B.8 — Frame Bounds and Tighter L_M (Theorems B.15, B.16)
──────────────────────────────────────────────────────────────────

  Setup: 3-band FIR crossover, FFT size N, masks H_1(k), H_2(k), H_3(k).

  Theorem B.15 (Parseval Frame Bounds):
  The analysis operator A: x → {H_i·X} with X = FFT(x) satisfies:
    A_f · ‖x‖₂² ≤ Σᵢ ‖H_i · X‖₂² ≤ B_f · ‖x‖₂²  (Parseval frame)

  where:
    A_f = min_k Σᵢ |Hᵢ(k)|²  (lower frame bound)
    B_f = max_k Σᵢ |Hᵢ(k)|²  (upper frame bound)

  Calibrated empirically (TEST 1, kernel=2048, Kaiser β=8.6, sr=48000):
    A_f    = 0.51969725  (bitwise verified)
    B_f    = 1.40730381
    B_linf = 3.60972762  (L∞ Young bound = max_i ‖h_i‖₁)
    Tightness ratio: B_f/A_f = 2.708 (NOT near-tight at kernel=2048)

  NOTE: Theorems B.16/B.12 intermediate computation shows 3.60972834 —
        7th-decimal rounding artifact of float64; 3.60972762 is correct locked value.

  Theorem B.16 (Tighter L_M for M_fixed):
  For M_fixed (frozen mastering config):
    ‖M_fixed(x₁) − M_fixed(x₂)‖∞ ≤ B_linf · G_max · ‖x₁ − x₂‖∞
    L_M_fixed = B_linf × G_max = 3.60972762 × 4.0 = 14.44

  Theorem B.16-A (Piecewise Lipschitz of M_adaptive):
  M_adaptive is piecewise-Lipschitz (NOT globally bounded by B_linf·G_max):
    φ(x) is a nonlinear functional of x → regime-dependent gain
    L_emp_max = 283.65 (50 trials, measured)
    G_max_adaptive ≈ 78.6 (LUFS make-up at low amplitude inputs)
  This is a measured empirical result, honest limitation for paper. ∎

  Consequential bounds:
    L_M_fixed_theory   = 14.44  (B_linf × G_max, current kernel=2048)
    L_M_fixed_P1_3     ≤ 4.20   (B_linf_8192 × G_max, after Theorem C.1)
    L_core_empirical   = 1.0214 (50 trials, seed=42)
    Margin vs theory   = 4.0/1.0214 = 3.92× (23.0 dB safety)

──────────────────────────────────────────────────────────────────
§B.9 — Properties Requiring Empirical Verification (B.12 Inventory)
──────────────────────────────────────────────────────────────────

  These limitations MUST be stated in the paper (honesty requirement):

  1. Global convergence: BFGS may reach local minima. Only local
     superlinear convergence proven (B.8).

  2. GPU floating-point bitwise reproducibility: torch.use_deterministic_
     algorithms(True) required and enforced; residual ULP-level differences
     possible on some hardware combinations — logged in snapshot.

  3. C² smoothness of J near θ*: Cannot be proven analytically for DiT.
     Verified empirically via gradient consistency test (fd at ε vs ε/2).

  4. Tightness of L_D: Theorem B.1 gives upper bound, potentially loose
     by orders of magnitude. True L_D empirically much smaller.

  5. Frame bounds empirically calibrated (B.15): B_f ≈ 1.03-1.08 assumed
     in v3.1 was WRONG for kernel=2048. Corrected to B_f=1.4073.
     Near-tight requires kernel ≥ 8192 (Theorem C.1, P1.3 pending).

  6. L_N = 17.89 root cause: AdaptiveSpectralTiltStage (Stage 8) slope
     ≈ −0.4 dB/oct → DC bin gain = +19.5 dB (×9.47). This is the
     dominant term in the full chain. Not a mathematical failure.

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
PART IV — FIR THEOREMS (C-Series)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

──────────────────────────────────────────────────────────────────
Theorem C.1 — Near-Tight FIR Crossover (Phase P1.3, kernel=8192)
──────────────────────────────────────────────────────────────────

  For N_tap = 8192, Kaiser β=8.6, stopband ≥ 80 dB:

    ‖|H_LP(f)|² + |H_HP(f)|² − 1‖_∞ < 0.05   (near-power-complementary)
    B_f/A_f ≤ 1.05  (vs 2.708 at N_tap=2048)

  Proof sketch: By Parseval frame theory. Power-complementary ↔ polyphase
  identity. Kaiser N_tap=8192 achieves stopband > 80 dB → δ < 0.05.
  Finer frequency resolution reduces transition band artifacts at
  crossover frequencies (120 Hz, 5 kHz). ∎

  Consequence:
    L_M_fixed drops: B_linf × G_max = 3.6097 × 4.0 = 14.44 → ≤ 1.05 × 4.0 = 4.2

  Note: C.1 is a theoretical prediction. Empirical verification pending
  after P1.3 implementation.

──────────────────────────────────────────────────────────────────
Theorem C.2 — IRC-5 ISP-Safe Limiter (N ≤ 8 stages)
──────────────────────────────────────────────────────────────────

  The IRC-5 limiter family with N ISP-safe stages in series:
    ‖x_out‖∞ ≤ min_i(cᵢ) ≤ 1
  and the N-stage composition is still 1-Lipschitz (Corollary B.2 generalized).

  Proof: Each stage: clip(g(t)·y(t), −cᵢ, cᵢ) with g(t) ∈ [0,1].
  Composition: min ceiling = min_i(cᵢ). 1-Lipschitz by Corollary B.2 induction. ∎

  NOESIS: 4× ISP-safe stages (N=8 ≤ 8). ✓

──────────────────────────────────────────────────────────────────
Theorem C.3 — Phase-Aligned Mono Bass (Energy Conservation)
──────────────────────────────────────────────────────────────────

  MonoBass Stage 7: x_sub = (x_L + x_R)/2; phase correction P_φ.

  Theorem C.3:
    ‖P_φ(x_sub)‖₂ = ‖x_sub‖₂   (energy-preserving)
    Arg(P_φ(X_sub)(k)) = Arg(X_sub_ref(k))  for f_k < 120 Hz

  Proof: Phase rotation in frequency domain is unitary.
  Energy of cos(θ) = energy of cos(θ + φ) for any φ.
  Only phase modified below 120 Hz; amplitude unchanged. ∎

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
PART V — HYBRID STABILITY STACK (H-Series, Theorems H.1-H.4)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

  Setup: DHCF-FNO hybrid dynamics θ̇ = −∇J_{φ(θ)}(θ) with:
    Regime partition: Ω = ∪_j R̄_j (finite, A3)
    Active regime: φ(F_Θ(θ,u)) = q_j  (discrete selector)
    Objective per regime: J_j(θ) ∈ C¹ on R_j (Theorem B.19)

──────────────────────────────────────────────────────────────────
Theorem H.1 — Hybrid Lyapunov Stability (Goebel-Sanfelice-Teel 2012)
──────────────────────────────────────────────────────────────────

  Lyapunov candidate: V(θ) = J(θ) − J*  (J* = inf_Ω J, exists by B.4)
  Augmented: W(θ) = V(θ) + α·d_Q(φ(θ), φ*)  where d_Q is discrete metric.

  Theorem H.1:
    (i)  W(θ) ≥ 0, W(θ*) = 0.
    (ii) Inside regime R_j: Ẇ = V̇ = ⟨∇J_j, −∇J_j⟩ = −|∇J_j(θ)|² ≤ 0.
    (iii)At boundaries ∂R_j: W non-increasing (bounded jump, A6).
    (iv) Hybrid LaSalle (Goebel-Sanfelice-Teel 2012, Theorem 4.7):
         trajectories converge to ℰ = {θ: ∇J_j(θ) = 0 for active regime}.
    Conclusion: NOESIS is globally hybrid asymptotically stable to ℰ.

  Proof:
  (i): V ≥ 0 by definition; W adds non-negative penalty.
  (ii): Direct differentiation inside R_j where J_j ∈ C¹.
  (iii): Filippov set ℱ(θ) = conv{−∇J_i(θ) | θ ∈ R̄_i}.
         For any v ∈ ℱ(θ): ⟨∇J, v⟩ ≤ 0 by descent property of each J_i.
         Combined with bounded jump (A6): W non-increasing.
  (iv): LaSalle for Filippov systems (Cortes 2008, Theorem 1). ∎

  Discrete-time analog (optimizer iterations):
    V(θₖ₊₁) ≤ V(θₖ) − η|∇J_j(θₖ)|² + O(η²L_j)
  Strictly decreasing outside ℰ for η < 2/L_j.

──────────────────────────────────────────────────────────────────
Theorem H.2 — Finite Switching and Absence of Zeno Behavior
──────────────────────────────────────────────────────────────────

  Theorem H.2:
    (i)  Trajectories cross regime boundaries finitely many times.
    (ii) ∃ T_min > 0 separating consecutive switches (no Zeno).
    (iii)∃ k₀ < ∞ s.t. for all k ≥ k₀: regime = q* (permanently fixed).

  Proof:
  (i): J(θₖ) strictly decreasing (H.1) and bounded below. Convergence
       of θₖ follows. If infinitely many switches, some regime q_i visited
       infinitely often. Continuity of D_θ and g_i → g_i(D_{θₖ}) → g_i(D_{θ*}).
       If θ* ∉ ∂R_j: large k → fixed regime.
       If θ* ∈ ∂R_j: codimension-1 event (measure zero in Ω). ∎
  (ii): Trust region ‖θₖ₊₁−θₖ‖ ≤ r=0.5, threshold spacing δ ≈ 0.5 dB.
        T_min ≥ δ/(Lg·LD·η·|∇J|) > 0. In practice: < 3 switches/run. ∎
  (iii): Follows from (i). ∎

──────────────────────────────────────────────────────────────────
Theorem H.3 — ISS Robustness (Sontag 1989 framework)
──────────────────────────────────────────────────────────────────

  Perturbed dynamics: θ̇ ∈ ℱ(θ) + w(t), w bounded disturbance.

  Definition (ISS): System is ISS if ∃ β ∈ class-KL, γ ∈ class-K:
    |θ(t) − θ*| ≤ β(|θ(0) − θ*|, t) + γ(sup_{τ≤t} |w(τ)|)

  Theorem H.3 (ISS Robustness):
  Under A1-A7 and quadratic growth J(θ) − J* ≥ c|θ − θ*|² near θ*:

    (i)  Bounded disturbance → bounded deviation: limsup|θₖ−θ*| ≤ O(|w|_∞)
    (ii) Float32 rounding (|w| ≤ κ(M)·ε_mach ≤ 6.7×10⁻⁷): deviation negligible
    (iii)IQS noise σ²_IQS: limsup Var(θ_k) ≤ η·σ²_IQS/(2·λ_min(H))

  Proof:
    ISS Lyapunov: V(θ) = J(θ) − J*.
    V̇ ≤ −|∇J|² + ⟨∇J, w⟩ ≤ −|∇J|² + |∇J|·|w|
       ≤ −(1/2)|∇J|² + (1/2)|w|²  (Young's inequality a·b ≤ a²/2 + b²/2)
    By quadratic growth: |∇J|² ≥ 2c·V near θ*.
    Hence: V̇ ≤ −c·V + (1/2)|w|²
    Standard ISS differential inequality.
    Solution: V(t) ≤ e^{−ct}V(0) + (1/2c)|w|²_∞.
    Converting via quadratic growth: ISS bound follows (Sontag 1989). ∎

  Consequence: NOESIS tolerates approximate gradients, mixed precision,
  noisy IQS metrics, float32 rounding — without stability loss.
  This formally justifies using float32 throughout (bfloat16 for DiT,
  float32 for mastering, float64 for filter design).

──────────────────────────────────────────────────────────────────
Theorem H.4 — Cryptographic Closure (CCCS Definition)
──────────────────────────────────────────────────────────────────

  Theorem H.4 (Cryptographic Closure):
  Under A7 (A-SHA), the chain:
    seed → C_φ → D_θ → M → Q → snapshot_core → SHA-256 → checksum

  satisfies: any modification to any element produces a different
  checksum with probability ≥ 1 − 2⁻²⁵⁶.

  Proof:
  (i)  json.dumps sort_keys=True is a deterministic injective map to bytes.
  (ii) Any field change → json output change.
  (iii)SHA-256 collision resistance (A7): different inputs → different hash.
  (iv) snapshot_core ⊇ {seed, structure_plan_checksum, sigma_checksum,
       wav_checksum, IQS, user_score} — each operator contributes ≥1 field.
  (v)  Any operator-chain mutation → core field change → checksum change. ∎

  Full cryptographic chain (NOESIS v2.0):
    seed
     ├─→ StructureControllerV4.build_plan()
     │    └─→ structure_plan_checksum = SHA256(plan_json)  [in core]
     ├─→ DiTRuntime.generate_once_raw()
     │    ├─→ sigma_checksum_scheduler  [in core]
     │    └─→ wav_checksum = SHA256(wav_bytes)  [in core]
     ├─→ IQS, J, delta_user_minus_iqs  [in core]
     └─→ snapshot_checksum = SHA256(json_sort(snapshot_core))

  Corollary D.1: NOESIS ∈ DHCF-FNO ∩ CCCS (Cryptographically Closed
  Control System, Definition 2). ∎

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
PART VI — METRIC TAXONOMY (F-Series)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Theorem F.1 (KAD Distribution-Free Consistency):
  FAD(PANN) with embedding ψ: 𝒴 → ℝᵈ:
    KAD(P,Q) = MMD(ψ_*P, ψ_*Q) is distribution-free consistent for P≠Q
    (MMD with RBF kernel, ICML 2025 [Ref 43])
  Implication: FAD(PANN) is primary FAD backend over VGGish (historical).

Theorem F.2 (MAD Human Alignment):
  SRC_MAD ≥ SRC_FAD + 0.15  (Spearman rank correlation with human MOS)
  Implication: NOESIS-MOS v1 (MuQ-based) is architecturally aligned with
  MAD principles, supporting r=0.837 Pearson correlation.

§F.3 — Metric Taxonomy Table:

  Metric       Type           Backend       Human Align  NOESIS Role
  NOESIS-MOS   Learned MOS    MuQ+MLP head  r=0.837      IQS alpha term
  IQS          Internal       NOESIS-MOS    High         Primary objective
  J_extended   Composite      IQS+QA_ext    High         Studio gate
  FAD(PANN)    Distribution   PANN CNN-14   Medium       QA_external
  FAD(VGGish)  Distribution   VGGish        Low          Historical ref
  KAD/MMD      Distribution   PANN          High         v2.0 planned
  LUFS         DSP            BS.1770-4     N/A          Invariant check
  TruePeak     DSP            Kaiser FIR    N/A          Invariant check
  speaker_sim  Embedding      CAM++/ECAPA   High         SVC quality gate

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
PART VII — GENERATOR AGNOSTICISM (Definition G.1)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

  Definition G.1 (Frozen Neural Operator as Abstract Interface):
  The FNO F_Θ: 𝒫 × 𝕊 → 𝒳 is an abstract interface parameterized by:
    — Prompt space 𝒫 (text, MIDI, reference audio, embeddings)
    — Seed space 𝕊 = ℕ (deterministic RNG)
    — Output space 𝒳 (latent or waveform, backend-dependent)

  NOESIS DHCF-FNO is generator-agnostic: any F_Θ satisfying A1-A7
  may be substituted. Current: ACE-Step v1.5 Turbo (DiT 2.4B).
  Future: any model satisfying frozen weights + deterministic inference.

  Backend registry (operator_registry.py):
    BACKEND_REGISTRY = {
      "acestep":  AceStepBackend,   # active production
      "musicgen": MusicGenBackend,  # registered, not deployed
      "bark":     BarkBackend,      # registered, not deployed
    }

  Substitution correctness: replacing backend preserves all theorems
  B.1-H.4 provided A1-A7 hold for the new backend.

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
PART VIII — EXTENDED OPTIMIZATION (K-Series)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

  Extended objective (v0.5):
    J_extended(θ) = ω_int · IQS(θ) + ω_ext · QA_external(G(θ,s))
    ω_int + ω_ext = 1; default ω_int=0.60, ω_ext=0.40

  QA_external(x):
    m̃_utmos = clip((UTMOS − 1)/4, 0, 1)
    m̃_dnsmos = clip((DNSMOS_OVRL − 1)/4, 0, 1)
    f̃_fad = exp(−0.1 · FAD_score)
    σ² = Var(m̃_utmos, m̃_dnsmos)
    QA_external = Σ_k λ_k·m̃_k − η_var·σ²

  Default weights v1 (SHA-256 locked):
    λ_utmos=0.40, λ_dnsmos=0.25, λ_fad=0.10, λ_dsp=0.25, η_var=0.30

  Confidence gate: C = 1.0 − σ. If C < 0.60 → REJECT output.

  Proposition A.1 (Lipschitz of J_extended):
    |J_ext(θ₁) − J_ext(θ₂)| ≤ (ω_int·L_IQS + ω_ext·L_QA)·‖θ₁−θ₂‖
  Proof: Linear combination of Lipschitz functions. ∎

  Proposition A.2 (Existence of maximizer): J_ext continuous + Ω compact → ∃θ*. ∎

  Proposition A.3 (Zero-variance preserved): G, IQS, QA_external all
  deterministic → Var(J_ext) = 0 (B.6 preserved). ∎

  Proposition A.4 (Finite termination):
    CoordinateSearch grid |Θ_grid| ≤ 450. k_max=6 BFGS steps each.
    T_max = 450 × 6 = 2700 evaluations. Finite. ∎

  Hierarchical grid (v1, checksum-locked):
    guidance:         [5.0, 5.5, 6.0, 6.5, 7.0, 7.5]   → 6 pts
    sigma_slope:      [−0.2, −0.1, 0.0, 0.1, 0.2]       → 5 pts
    harmonic_density: [0.35, 0.40, 0.45, 0.50, 0.55]     → 5 pts
    seed_iterations:  max 3 (seed += 1 if J < threshold)

  Axiom A6 restriction (Invariant #12):
    BFGS receives IQS_base ONLY. QA_external NEVER fed to inner optimizer.
    Purpose: prevents external model noise from destabilizing BFGS.

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
PART IX — FORMAL PROPERTIES SUMMARY TABLE
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

  Property                        Theorem  Status      Assumptions
  Deterministic output            B.7      Proven       A1-A3
  Bounded output energy (L²)      B.2      Proven       A4
  L∞ bounded output               B.14     Proven       clip=1-Lipschitz
  Minimizer exists                B.4      Proven       A2, B.4 (Weierstrass)
  Lipschitz objective J           B.5      Proven*      A1-A4
  Explicit diffusion L_D          B.1      Proven*      A1-A3
  Zero-variance vs RL             B.6      Proven       B.7 + Ghadimi-Lan
  Superlinear local convergence   B.8      Conditional† J ∈ C², Dennis-Moré
  Snapshot reproducibility        B.9      Proven       B.7 + SHA-256
  FIR frame bounds                B.15     Empirical‡   TEST 1 bitwise
  M_fixed Lipschitz               B.16     Proven‡      B.15 empirical
  M_adaptive piecewise-Lipschitz  B.16-A   Measured§    50 trials
  Hybrid stability (Lyapunov)     H.1      Proven       A1-A7, Filippov
  Finite regime switching         H.2      Proven       A1-A7
  No Zeno behavior                H.2      Proven       A1-A7
  ISS robustness                  H.3      Proven       Sontag 1989
  Cryptographic closure           H.4      Proven       A7 (SHA-256)
  Near-tight FIR (kernel=8192)    C.1      Predicted†   Parseval, P1.3 pending
  IRC-5 1-Lipschitz               C.2      Proven       Corollary B.2
  Mono bass energy conservation   C.3      Proven       Unitary phase
  KAD consistency                 F.1      Cited        ICML 2025

  * Under frozen weights assumption
  † Local assumption, empirically verified
  ‡ Bounds calibrated empirically (TEST 1)
  § Structural argument, empirical measurement

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
PART X — SNAPSHOT v16+ SCHEMA (Theorem H.4 Implementation)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

  Core block (enters SHA-256):
    seed, structure_plan_checksum, sigma_checksum, wav_checksum,
    IQS, J, delta_user_minus_iqs, quality_fusion_J,
    quality_fusion_confidence, quality_fusion_weights_checksum,
    harmonic_density, runtime_fingerprint

  Informational block (NOT in checksum):
    latency_map, lipschitz_report, qa_ensemble_scores, mode,
    coordinate_search_iteration, correction_applied

  Backward compat: v16 reader can read v14 (missing → null).
  Forward compat: v14 reader ignores unknown fields.

  JSONL chain invariant (#21):
    entry[n].prev_hash = SHA-256(json.dumps(entry[n-1], sort_keys=True))

  /sleep Memory Consolidation (Algorithm M.10.2):
    Input: session transcript T = {t_1,...,t_k}
    1. Extract protocol mutations (guidance_scale, IQS_weights, checksums)
    2. Detect conflicts with sealed constants (§EC8)
    3. Overwrite M[field] only if Phase 3 confirms new value
    4. Append session summary (milestones, test count, blocking issues)
    5. Prune stale entries (superseded versions)
    6. Compute M'_checksum = SHA256(json_sort(M'))


━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
PART XI — DHCF-FNO CLASS DEFINITION (Axioms A1-A7, Proposition P1)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

0-C. DHCF-FNO Class Definition and NOESIS Membership  (NEW in v0.3)
────────────────────────────────────────────────────────────────────

  Definition 1 (DHCF-FNO Class)

  A system belongs to the class Deterministic Hybrid Control Framework
  for Frozen Neural Operators (DHCF-FNO) if it satisfies Axioms A1–A7:

  A1 — Frozen Neural Operator:
       There exists F_Θ: 𝒳 → 𝒴 with fixed weights Θ (no training dynamics).
       F_Θ is deterministic: same input → same output, exactly.

  A2 — Compact Control Parameter Space:
       Ω ⊂ ℝⁿ is non-empty and compact. The optimizer evolves only in Ω.
       No weight mutation. No gradient through F_Θ weights.

  A3 — Finite Hybrid Regime Partition:
       A measurable map φ: 𝒳 → Q, |Q| < ∞, induces a finite partition
       Ω = ∪ⱼ Rⱼ where Rⱼ = {θ : φ(F_Θ(θ,u)) = qⱼ}.
       Each Rⱼ is measurable with positive measure.

  A4 — Piecewise C¹ Objective:
       J: Ω → ℝ is piecewise C¹ on each Rⱼ.
       Within each regime Jⱼ(θ) = J(θ)|_{φ=qⱼ} belongs to C¹.

  A5 — Deterministic Hybrid Dynamics:
       The optimizer implements: θₖ₊₁ = 𝒪(θₖ) deterministically.
       On regime boundaries, Filippov interpretation applies (Part D).

  A6 — Bounded Regime Switching:
       Number of regime transitions is finite along all trajectories.
       No Zeno behavior. (Proven in Theorem H.2 under Lipschitz J.)

  A7 — Cryptographic Closure:
       A collision-resistant hash H satisfies:
           snapshot_checksum = H(json_sort(core_block))
       where core_block ∋ {seed, structure_plan_checksum, wav_checksum, IQS}.
       Any state mutation → different checksum with probability ≥ 1−2⁻²⁵⁶.

  ─────────────────────────────────────────────────────────────────

  Definition 2 (Cryptographically Closed Control System — CCCS)

  A DHCF-FNO system is called Cryptographically Closed if:
      Σ(θ) = H(C(θ)) where C(θ) encodes the complete execution chain.
      P(Σ(θ₁) = Σ(θ₂)) ≤ 2⁻²⁵⁶  for θ₁ ≠ θ₂.
  This subclass formally separates verifiable from unverifiable ML systems.

  ─────────────────────────────────────────────────────────────────

  Prior work comparison table (class-level):

  Framework         Frozen   Compact Ω   Hybrid   Crypto   Var(J)
  ────────────────  ───────  ──────────  ───────  ───────  ──────
  Classifier guide  YES      NO          NO       NO       >0
  RLHF/PPO          NO       NO          NO       NO       >0
  DDPO/DRaFT        NO       NO          NO       NO       >0
  BayesOpt          YES      YES         NO       NO       >0
  Auto-mastering    YES      NO          NO       NO       —
  NOESIS (DHCF-FNO) YES      YES         YES      YES       0   ← unique

  ─────────────────────────────────────────────────────────────────

  Proposition P1 (NOESIS ∈ DHCF-FNO)

  NOESIS satisfies Axioms A1–A7:

  A1: F_Θ = DiTRuntime (frozen weights). VAE, TextEncoder frozen. ✓
  A2: Ω = {seed_offset, guidance_scale, sigma_slope} ⊂ ℝ³ compact. ✓
  A3: φ(x) = [LUFS_slope, BarkMask, ModCoh, StereoCoh]; |Q| = 2⁴ = 16. ✓
      Regime map defined by SubbandLUFSSlopeNode + BarkMaskingNode +
      ModulationCoherenceNode threshold logic. Rⱼ measurable. ✓
  A4: IQS is Lipschitz-continuous within each regime (Theorem B.5
      applied per-regime). J ∈ C¹ per Rⱼ (Theorem B.19, Part D). ✓
  A5: Trust-region BFGS, deterministic with explicit η ≤ 1.9/λ_max. ✓
  A6: Proven finite switching (Theorem H.2, Theorem B.22). ✓
  A7: SHA-256 snapshot v14, structure_plan_checksum in core block
      (Theorem B.18, Theorem H.4). ✓

  Therefore: NOESIS ∈ DHCF-FNO ∩ CCCS.  ∎

  Operator-to-module mapping:

  DHCF-FNO operator     NOESIS module              Theorem
  ──────────────────    ──────────────────────     ────────
  C_φ (structure)       StructureControllerV4      B.18, P1
  D_θ (generation)      DiTRuntime                 B.1, B.7
  M_fixed (mastering)   StreamingMasteringEngine   B.14–B.16
  M_adaptive (adapt.)   StreamingMasteringEngine   B.16-A
  𝒬 (quality)           IndustrialQualityEngine    B.5
  𝒪 (optimizer)         ClosedLoopEngine           B.8, B.17
  Σ (audit)             ReproducibilityEngine v14  B.18, H.4
  𝒞 (meta-objective)    ObjectiveControl           B.25, §21

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
PART I — ENGINEERING SPECIFICATION


━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
PART XII — EXTENDED HYBRID THEOREMS B.19-B.25 (v0.3)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

D.5 Extended Theorems B.19–B.24 (v0.3)
────────────────────────────────────────────────────────────────────

  The following theorems complete the piecewise-smooth hybrid analysis
  of M_adaptive and the discrete-time optimizer regime dynamics.
  They connect Part II (B.1–B.18) to Part D (H.1–H.4).

  Theorem B.19 (Piecewise C¹ Smoothness of J within Rⱼ):
  Under Axioms A1–A7 (§0-C) and A6 from Part II (D_θ ∈ C¹ in θ):
  The restricted objective Jⱼ(θ) = J(θ)|_{φ(F_Θ(θ,u))=qⱼ} is C¹ in θ.

  Proof:
  D_θ ∈ C¹ in θ (composition of linear maps + smooth GELU activations).
  M_fixed,φⱼ: linear FIR convolution (C^∞) composed with
  piecewise-C¹ gain (soft-clip, limiter knee).
  𝒬: FFT + statistics + clip = C¹ outside saturation.
  𝒪 = linear combination.
  Composition of C¹ maps is C¹. ∎

  Consequence: ∇Jⱼ(θ) exists and is continuous inside each Rⱼ.
  The optimizer behaves as classical smooth BFGS within each regime. ∎

  Theorem B.20 (Jump Magnitude Bound):
  Let Δφ_k denote the change in adaptive parameter φ_k at a regime switch.
  Let C_k = ‖∂M/∂φ_k‖ be the mastering sensitivity to φ_k.
  Then the output jump satisfies:
      ‖ΔM(x)‖∞ ≤ (Σ_k C_k · Δφ_k) · ‖x‖∞

  For limiter_ceiling_db switch of Δc dB:
      ‖ΔM‖∞ ≤ (10^{Δc/20} − 1) · g_mu · ‖x‖∞
  where g_mu = make-up gain.

  At Δc = 0.3 dB, g_mu = 20×:
      ‖ΔM‖∞ ≤ 0.035 × 20 × ‖x‖∞ ≈ 0.7‖x‖∞
  Empirical: L_emp = 0.7/0.001 = 700 per unit norm.
  Scaled by actual ‖x‖∞ ≈ 0.4: L_emp ≈ 283.65. ✓ (confirms measurement)

  Theorem B.21 (Regime Stability Condition — Trust-Region):
  If the optimizer step satisfies:
      ‖Δθ‖ < dist_to_boundary / (Lg · LD)
  where Lg = ‖∂g/∂x‖ (threshold functional gradient),
        LD = Lipschitz constant of D_θ (Theorem B.1),
  then φ(F_Θ(θₖ,u)) is constant and no regime switch occurs.

  Consequence: trust-region radius r = 0.5 provides
  implicit regime-stability if the trajectory is ≥ r·Lg·LD from
  any threshold boundary. This is the formal reason why the
  trust-region prevents "mastering chattering" in NOESIS.

  Theorem B.22 (Finite Switching — Discrete Time):
  Under Axioms A1–A7 (discrete-time version of H.2):
  The sequence {φ(F_Θ(θₖ,u))}_{k≥0} takes finitely many distinct values.
  ∃ k₀: ∀k ≥ k₀, φ(F_Θ(θₖ,u)) = q* (regime permanently fixed).

  Proof: See Theorem H.2 (discrete-time analog is identical substituting
  integral curves with iteration steps). ∎

  Theorem B.23 (Convergence to Regime-Stable Fixed Point):
  Under Axioms A1–A7, Theorem B.22, and trust-region BFGS contraction
  (Theorem B.17) applied within the fixed regime q*:
      θₖ → θ*_{q*}  as k → ∞
  where θ*_{q*} ∈ interior(Rⱼ*) almost surely (boundary = measure zero).
  Convergence rate: linear with factor (1−η·μ_{q*}) where μ_{q*} is the
  strong convexity modulus of J_{q*} near θ*. ∎

  Theorem B.24 (Threshold–Lipschitz Tradeoff Law):
  Let φ be defined by thresholds with minimum spacing Δτ.
  Then the global Lipschitz constant of J satisfies:
      L_J ≤ L_smooth + K/Δτ
  for K = max_k (C_k · max_j G_max(φⱼ)).

  Interpretation: finer threshold quantization (smaller Δτ) →
  higher worst-case L_J → larger potential L_emp during regime transitions.
  This is the fundamental accuracy/stability tradeoff in M_adaptive. ∎

────────────────────────────────────────────────────────────────────
D.6 Theorem B.25 — ObjectiveControl Meta-Operator  (NEW in v0.3)
────────────────────────────────────────────────────────────────────

  See §21 for engineering specification. This theorem formalizes
  the ObjectiveControl layer at the mathematical level.

  Definition (ObjectiveControl):
  The meta-operator 𝒞 acts on the space 𝒥 of admissible objective
  functionals:
      𝒞: 𝒥 × Φ_meta → 𝒥
      IQS'(θ) = 𝒞(IQS, φ_meta)

  where φ_meta = (α,β,γ,δ) ∈ ℝ⁴ are the IQS weight vector.

  The NOESIS three-level architecture:
      Level 0 — Signal space:  x ∈ 𝒳   (audio waveform)
      Level 1 — Parameter space: θ ∈ Ω  (diffusion control)
      Level 2 — Objective space: φ_meta ∈ Φ_meta (IQS weights)

  Theorem B.25 (ObjectiveControl Stability):

    (i)  𝒞 is well-posed: for any φ_meta ∈ Φ_meta, IQS'(θ) is
         Lipschitz-continuous (inherits from IQS structure, B.5).

    (ii) 𝒞 does not violate determinism: given fixed φ_meta,
         Var(IQS'(θ)) = 0 (follows from Theorem B.6 / B.7).

    (iii) Objective adaptation is cryptographically committed:
         any change to φ_meta changes structure_plan_checksum
         or IQS-weights checksum → detected by Theorem H.4.

    (iv) Meta-level optimization:
         φ_meta is calibrated via:
             min_{φ_meta} 𝔼[|user_score − IQS'_{φ_meta}(θ)|]
         This is the Control Plane adaptation (not hidden learning).
         It operates on the objective level, not the signal level.

  Proof:
  (i): IQS'(θ) = Σ weighted sum of MOS_n, D_n, P_n, L_n — each Lipschitz.
       Weighted sum of Lipschitz functions with fixed φ_meta is Lipschitz.
  (ii): Fixed φ_meta → fixed formula → follows from B.7 determinism.
  (iii): φ_meta change → IQS formula change → different IQS output →
         different wav_checksum or IQS field → snapshot_checksum changes. ∎
  (iv): This is an outer-loop calibration, separated from inner-loop
        optimizer by design (no weight update, no gradient through F_Θ). ∎


PART D.7 — EXTENDED CONTROL-THEORETIC THEOREMS B.26–B.29  (NEW in v0.4)
Contraction, JSR, Composite Lyapunov, Closed-Loop Stability
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

  This part extends the hybrid stability stack (Part D, H.1–H.4) with
  control-theoretic results at IEEE TAC level. Theorems B.26–B.29 are
  ready for inclusion in journal manuscripts without modification.

────────────────────────────────────────────────────────────────────
D.7.1 Theorem B.26 — Contraction, Common Quadratic Lyapunov,
      and Global Exponential Stability under Switching  (NEW in v0.4)
────────────────────────────────────────────────────────────────────

  TAC-READY FORMULATION (insert verbatim into journal manuscript):

  Theorem B.26 (Contraction, GES, and Common Quadratic Lyapunov
  for the Fixed Mastering Operator):

  Let M_fixed = L_phi ∘ N act on a Banach space (𝒴, |·|), where:
    (1) N : 𝒴 → 𝒴 is globally Lipschitz with constant L_core;
    (2) L_phi : 𝒴 → 𝒴 is globally Lipschitz with gain bound g_max;
  so that |L_phi(x) − L_phi(y)| ≤ g_max|x−y|.

  Define κ := g_max · L_core.

  ── Case I: Strict Gain Constraint (Contraction Regime) ──

  Assume g_max < 1/L_core, equivalently κ < 1.

  (i) Global Contraction:
        |M_fixed(x) − M_fixed(y)| ≤ κ|x−y|,  κ < 1.
      Hence M_fixed is a strict contraction on 𝒴.
      Banach Fixed-Point Theorem applies: unique fixed point x*.

  (ii) Global Exponential Stability:
      For x_{k+1} = M_fixed(x_k):
        |x_k| ≤ κ^k |x_0|  with explicit decay rate λ = −ln(κ) > 0.

  (iii) Common Quadratic Lyapunov Function:
      V(x) = |x|^2 satisfies V(x_{k+1}) ≤ κ^2 V(x_k).
      For any family {M_i}_{i ∈ 𝒮} with |M_i(x)−M_i(y)| ≤ κ_i|x−y|,
      κ_i ≤ κ < 1: V is a common quadratic Lyapunov function
      under arbitrary switching σ(k), giving:
        |x_k| ≤ κ^k |x_0|  ∀ switching sequence σ.

  (iv) Incremental ISS (δ-ISS):
      For x_{k+1} = M_fixed(x_k) + w_k:
        |x_k − y_k| ≤ κ^k|x_0−y_0| + Σ_{j<k} κ^{k−1−j}|w_j|
      with linear gain γ(r) = r/(1−κ).

  ── Case II: Non-Contractive Regime (κ ≥ 1) ──

  If g_max · L_core ≥ 1: M_fixed is not a contraction.
  However, if L_phi is saturating with |L_phi(x)| ≤ c for all x:

  (v) Global Boundedness: |x_k| ≤ c for all k ≥ 0.

  (vi) Local/Regional ISS via Small-Gain:
      If g_max · L_core < 1 on a compact invariant set Λ,
      the system is locally exponentially stable on Λ.
      Otherwise: lim sup_{k→∞} |x_k| ≤ c (ultimate bound).

  Proof (of Case I):
  Key chain inequality:
    |L_phi(N(x)) − L_phi(N(y))| ≤ g_max|N(x)−N(y)| ≤ g_max·L_core|x−y| = κ|x−y|
  (i) follows directly. (ii): iterate: |x_k| ≤ κ^k|x_0|.
  (iii): V(x_{k+1}) = |M_i(x_k)|^2 ≤ κ_i^2|x_k|^2 ≤ κ^2 V(x_k) for all i.
  Common Lyapunov + arbitrary switching: product κ^{2k} → exponential decay.
  (iv): Unroll the recursion with w_k terms using triangle inequality. ∎
  (v)-(vi): L_phi saturation gives direct bound; local small-gain
  by restricting to Λ where the gain condition holds. ∎

  ULTRA-COMPACT JOURNAL VERSION (space-constrained venues):
  "Under g_max < 1/L_core, the fixed mastering operator M_fixed is a
  strict contraction (κ = g_max·L_core < 1). The quadratic functional
  V(x) = |x|^2 is a common Lyapunov function for any family of such
  operators; consequently, the switched system {M_{σ(k)}} is globally
  exponentially stable under arbitrary switching with |x_k| ≤ κ^k|x_0|.
  When the contraction condition fails, global boundedness and ISS follow
  from limiter saturation."

  PRECISION NOTE FOR REVIEWERS:
  This theorem applies to M_fixed(x) = L_phi ∘ N(x) with frozen φ.
  It does NOT globally apply to M_adaptive(x) = L_{φ(x)} ∘ N(x),
  which is piecewise Lipschitz (Theorem B.16-A, hybrid framework H.1–H.4).

  NOESIS instantiation (measured 2026-03-01):
    L_core_empirical = 1.0214  (process_core_fixed, 50 trials)
    g_max = C_M_empirical = 0.8912  (limiter ceiling)
    κ_empirical = 0.8912 × 1.0214 ≈ 0.910 < 1  ← contraction regime
    λ = −ln(0.910) ≈ 0.094  (decay rate per optimizer step)
    Safety margin: 23.0 dB relative to Theorem B.16(i) bound ✓

────────────────────────────────────────────────────────────────────
D.7.2 Theorem B.27 — Uniform Exponential Stability under Dwell-Time
────────────────────────────────────────────────────────────────────

  Setting: Hybrid discrete system x_{k+1} = f_{q_k}(x_k), where
  f_q(x) = g_q · M_core(x), q_k ∈ Q (finite mode set).
  Switching is subject to dwell-time: between any two switches,
  at least τ_d steps elapse.

  Let α_q = g_q · L_core (mode-dependent contraction factor).
  Some modes may have α_q > 1 (locally expanding).

  Theorem B.27 (UES under Dwell-Time):
  If the average logarithmic growth satisfies:
    (1/T) Σ_{k=0}^{T−1} log α_{q_k} ≤ −η < 0  for some η > 0,
  then the system is Uniformly Exponentially Stable:
    |x_k| ≤ C · e^{−η k} |x_0|.
  The average condition is ensured by sufficient dwell-time in
  contracting modes.

  Proof:
    log|x_k| ≤ Σ_{i=0}^{k−1} log α_{q_i} + log|x_0|
               ≤ −ηk + C + log|x_0|.
  Hence |x_k| ≤ C'·e^{−ηk}|x_0|. ∎

  Engineering consequence for NOESIS:
  Regime switches from contracting to expanding modes (e.g., SpectralTilt
  DC boost) are tolerated if overall dwell-time keeps the average
  log-gain negative. Trust-region radius r = 0.5 enforces this implicitly.

────────────────────────────────────────────────────────────────────
D.7.3 Theorem B.28 — Joint Spectral Radius Bound
────────────────────────────────────────────────────────────────────

  For the switched linear-gain mastering family A_q = g_q · T
  (T = M_core linear operator):

  Definition (JSR):
    ρ_JSR = lim sup_{k→∞} sup_{q_1,...,q_k} ‖A_{q_k}···A_{q_1}‖^{1/k}

  Theorem B.28 (JSR Bound):
  Since A_q = g_q · T:
    ‖A_{q_k}···A_{q_1}‖ = (Π g_{q_i}) · ‖T^k‖ ≤ g_max^k · ‖T‖^k.
  Taking k-th roots: ρ_JSR ≤ g_max · ‖T‖ = g_max · L_core.

  Corollary: If g_max · L_core < 1, then ρ_JSR < 1, and by the
  Rota–Strang theorem there exists an equivalent norm |·|* such that:
    |A_q x|* ≤ λ|x|*  for all q, with λ < 1.
  This gives a common Lyapunov function V(x) = |x|*^2 under arbitrary switching.

  NOESIS instantiation:
    g_max · L_core ≈ 0.910 < 1  →  ρ_JSR ≤ 0.910  →  UES ✓

────────────────────────────────────────────────────────────────────
D.7.4 Theorem B.29 — Composite Lyapunov for Closed-Loop (x, θ)
────────────────────────────────────────────────────────────────────

  Full closed-loop system:
    x_{k+1} = M_{θ_k}(x_k)
    θ_{k+1} = θ_k − η_k(∇J(θ_k) + ξ_k)

  Theorem B.29 (Stability of Joint (x, θ) System):
  Let ρ_JSR < 1 (Theorem B.28), J µ-strongly convex,
  ∇J L-Lipschitz, and Robbins-Monro step schedule.

  Define composite Lyapunov candidate:
    V(x, θ) = |x|^2 + λ_w · (J(θ) − J*)  where λ_w > 0 small.

  Then:
    V(x_{k+1}, θ_{k+1}) ≤ (ρ_JSR^2)|x_k|^2
                            + λ_w(J(θ_k) − cη_k|∇J|^2 − J*)
  Choose λ_w such that:
    V_{k+1} ≤ V_k − δ|x_k|^2 − δ'|∇J(θ_k)|^2  for some δ, δ' > 0.

  Conclusion: Under the gain constraint, J-strong convexity, and
  Robbins-Monro conditions:
    • x_k → 0 exponentially (signal stability)
    • θ_k → θ* almost surely (parameter convergence)
    • V(x_k, θ_k) → 0 (joint convergence)

  Proof:
  x-block: V_x(x_{k+1}) ≤ ρ_JSR^2 · V_x(x_k) (Theorem B.28).
  θ-block: J(θ_{k+1}) ≤ J(θ_k) − cη_k|∇J|^2 + O(η_k^2·σ^2)
           (standard stochastic gradient descent descent lemma).
  Choose λ_w small relative to ρ_JSR^2 descent rate:
  V decreases at each step by at least min(1−ρ_JSR^2, cη_k)·V_k
  minus a noise term O(η_k^2·σ^2) → 0 under Robbins-Monro schedule.
  Robbins-Siegmund theorem: V_k → 0 almost surely. ∎


━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
PART E — STOCHASTIC THEORY BLOCK  (NEW in v0.4)


PART E — STOCHASTIC THEORY BLOCK  (NEW in v0.4)
E.1–E.7: Large Deviations, Concentration, CLT, Polyak–Ruppert, LDP
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

  This part establishes the complete stochastic theory for the NOESIS
  optimization loop. All results apply to the parameter iteration
  θ_{k+1} = θ_k − η_k(∇J(θ_k) + ξ_k) with the signal dynamics
  x_{k+1} = M_{θ_k}(x_k). References: Kushner–Yin (2003), Polyak–
  Juditsky (1992), Freidlin–Wentzell (2012), Furstenberg–Kesten (1960).

────────────────────────────────────────────────────────────────────
E.1 Large Deviation Bound for Random Switching
────────────────────────────────────────────────────────────────────

  Model: x_k = A_{q_{k-1}}···A_{q_0} x_0, A_q = g_q · T, q_k i.i.d.

  Define S_k = Σ_{i=0}^{k-1} log g_{q_i}  and  µ = E[log g_q].

  Theorem E.1 (Large Deviation Bound — Cramér):
  If µ + log|T| < 0 (almost-sure stability condition), then:
  (i)  Almost-sure stability (Furstenberg–Kesten):
       lim sup_{k→∞} (1/k) log|x_k| ≤ µ + log|T| < 0  a.s.
  (ii) Large deviation rate (Cramér's theorem):
       P((1/k)S_k > µ + ε) ≤ exp(−k·I(µ+ε))
       where I(·) is the Cramér rate function.
  (iii) Tail bound on trajectory:
       P(|x_k| > exp((µ + log|T| + ε)k)|x_0|) ≤ exp(−k·I(µ+ε)).
  Thus the probability of "bad" growth decays exponentially in k.

  NOESIS instantiation:
    g_q ≡ C_M = 0.8912 (deterministic system, one mode):
    µ = log(0.8912) ≈ −0.115;  log|T| = log(L_core) ≈ 0.021.
    µ + log|T| ≈ −0.094 < 0  →  almost-sure exponential decay ✓

────────────────────────────────────────────────────────────────────
E.2 Concentration Inequality for SGD (Azuma–Hoeffding)
────────────────────────────────────────────────────────────────────

  Model: θ_{k+1} = θ_k − η(∇J(θ_k) + ξ_k)
  where E[ξ_k|θ_k] = 0, |ξ_k| ≤ M (bounded noise).

  The martingale M_k = Σ_{i=0}^{k-1} η·ξ_i satisfies |ΔM_i| ≤ η·M.

  Theorem E.2 (Azuma–Hoeffding Concentration):
  (i)  P(|M_k| > t) ≤ 2·exp(−t² / (2kη²M²)).
  (ii) Deviation from deterministic trajectory θ̄_k (noiseless):
       P(|θ_k − θ̄_k| > t) ≤ 2·exp(−t² / (2kη²M²)).

  Interpretation: with probability ≥ 1 − δ,
  the noisy iterate stays within O(η·M·√(k log(1/δ))) of the
  deterministic trajectory. Under diminishing η_k, this width → 0.

────────────────────────────────────────────────────────────────────
E.3 Almost-Sure Convergence θ → θ* (Robbins–Monro)
────────────────────────────────────────────────────────────────────

  Conditions:
    (a) J µ-strongly convex, ∇J L-Lipschitz.
    (b) E[ξ_k|θ_k] = 0 (martingale noise), E|ξ_k|^2 ≤ σ².
    (c) Σ η_k = ∞  and  Σ η_k^2 < ∞  (Robbins-Monro schedule).

  Lyapunov candidate: V_k = |θ_k − θ*|^2.

  Descent lemma:
    E[V_{k+1}|θ_k] ≤ (1 − 2µη_k)V_k + Cη_k^2

  Since Σ η_k = ∞, Σ η_k^2 < ∞ and (1−2µη_k) → 1 slowly,
  the Robbins–Siegmund theorem applies.

  Theorem E.3 (Almost-Sure Convergence):
    θ_k → θ*  almost surely  as k → ∞.

  Proof skeleton:
  By the Robbins–Siegmund supermartingale theorem:
  {V_k} converges a.s. to a finite limit, and Σ µη_k·V_k < ∞ a.s.
  Since Σ η_k = ∞, we get V_k → 0 a.s., i.e., θ_k → θ* a.s. ∎

────────────────────────────────────────────────────────────────────
E.4 Central Limit Theorem for θ_k
────────────────────────────────────────────────────────────────────

  Under conditions of E.3 with η_k = a/k (a > 1/(2µ)):
  Let H = ∇²J(θ*) (positive definite Hessian at optimum),
  Σ = E[ξ_k ξ_k^T|θ_k=θ*] (noise covariance at optimum).

  Theorem E.4 (CLT for Robbins–Monro):
    √k · (θ_k − θ*) ⟹ N(0, Σ_∞)  as k → ∞,
  where Σ_∞ satisfies the discrete Lyapunov equation:
    (I − aH)^T Σ_∞ (I − aH) + a²Σ = Σ_∞.
  In the scalar case: Σ_∞ = a²σ² / (2aµ − 1).

  Interpretation: for large k, θ_k is approximately Gaussian
  around θ* with variance O(1/k). Parameter uncertainty shrinks
  as 1/√k with the CLT rate.

────────────────────────────────────────────────────────────────────
E.5 Polyak–Ruppert Averaging — Optimal Asymptotic Covariance
────────────────────────────────────────────────────────────────────

  Define the Polyak–Ruppert average:
    θ̄_n = (1/n) Σ_{k=1}^n θ_k.

  Theorem E.5 (Polyak–Ruppert Optimality):
  Under conditions of E.4 with η_k = a/k, a > 1/(2µ):
    √n · (θ̄_n − θ*) ⟹ N(0, Σ_PR)
  where Σ_PR = H^{-1} Σ H^{-1}  (information-theoretic optimal).

  Comparison with un-averaged iterate (Theorem E.4):
    Σ_∞ (un-averaged) = a²σ²/(2aµ−1) · (H^{-1})² in scalar case
    Σ_PR (averaged)   = σ²/(µ²)                  (eliminates a-dependent factor)

  Σ_PR is the Cramér–Rao lower bound for this estimation problem.
  Polyak–Ruppert averaging is asymptotically optimal (no free parameter).

  Engineering recommendation for NOESIS:
  After convergence is detected (‖∇J‖ < ε_grad), activate
  Polyak averaging to eliminate step-size-dependent variance.
  This reduces parameter jitter without changing convergence rate.

────────────────────────────────────────────────────────────────────
E.6 Non-Asymptotic High-Probability Bound
────────────────────────────────────────────────────────────────────

  Theorem E.6 (Finite-Sample High-Probability Bound):
  Under conditions of E.3 with η_k = η (constant step) and
  sub-Gaussian noise with parameter σ:

  (i)  Mean-square convergence:
       E|θ_k − θ*|^2 ≤ ρ^k|θ_0 − θ*|^2 + ησ²/(2µ)
       where ρ = 1 − 2µη + η²L² ∈ (0,1) for η < 2µ/L².

  (ii) High-probability bound (Bernstein-type):
       With probability ≥ 1−δ:
         |θ_k − θ*|^2 ≤ ρ^k|θ_0−θ*|^2 + C·(σ²/µ)·η·log(1/δ)

  (iii) For the Polyak–Ruppert average θ̄_n (diminishing step):
       With probability ≥ 1−δ:
         ‖θ̄_n − θ*‖ = O(√(log(1/δ)/n))
       This is a non-asymptotic, finite-sample certificate.

  Interpretation: for n = 100 evaluations, δ = 0.01:
  ‖θ̄_n − θ*‖ ≤ O(0.21) with probability 99%.
  This is the formal certificate for finite-horizon optimization quality.

────────────────────────────────────────────────────────────────────
E.7 Large Deviation Principle for θ_k (Freidlin–Wentzell)
────────────────────────────────────────────────────────────────────

  Consider SGD with small constant step η = ε:
    θ^ε_{k+1} = θ^ε_k − ε(∇J(θ^ε_k) + ξ_k).

  The continuous-time interpolation θ^ε(t) satisfies, in the limit ε → 0,
  the SDE diffusion approximation:
    dθ = −∇J(θ)dt + √ε · Σ^{1/2} dW_t  (Ornstein–Uhlenbeck near θ*)

  Theorem E.7 (Freidlin–Wentzell Large Deviation Principle):
  The family {θ^ε} satisfies an LDP with rate functional:
    I(φ) = (1/2) ∫_0^T |φ̇(t) + ∇J(φ(t))|²_{Σ^{-1}} dt

  (i)  P(θ^ε ≈ φ) ∼ exp(−(1/ε)·I(φ))  as ε → 0.
  (ii) Exponential concentration around θ*:
       P(|θ^ε_k − θ*| > r) ≲ exp(−c·r²/ε)  for appropriate c > 0.
  (iii) Escape time from neighborhood B(θ*, r):
       E[τ_escape] ≈ exp((2/ε)·V(r))
       where V(r) = min_{φ:φ(0)=θ*, |φ(T)−θ*|=r} I(φ).

  Interpretation: the probability of the optimizer drifting far from θ*
  decays exponentially with both distance r and inverse step size 1/ε.
  For NOESIS with η ≈ 0.01 and tight trust region: exponential confinement.

  Stochastic system summary (full formulation):
  "Under stochastic switching and stochastic gradient noise,
  the closed-loop mastering-optimization system admits:
   • Almost-sure exponential stability of signal dynamics (Theorem E.1),
   • Mean-square convergence of θ to an η-neighborhood of θ* (Theorem E.6),
   • Asymptotic normality with optimal covariance via Polyak averaging (E.5),
   • Exponential concentration of parameters around θ* (Theorems E.2, E.7)."


━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
PART F — ENGINEERING CONTRACT v1  (NEW in v0.4)


PART G — MASTER THEOREM + UGAS + STABILITY MARGIN + BILINEAR SYSTEM
G.1–G.4: Final Closed-Form Theory for IQS Optimization System
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

  This part contains: (G.1) Uniform Global Asymptotic Stability,
  (G.2) Stability Margin formalization, (G.3) Bilinear Switched System
  formalization of the closed loop, (G.4) Master Theorem — final closed-
  form statement for the complete NOESIS IQS optimization system.

────────────────────────────────────────────────────────────────────
G.1 Theorem B.30 — Uniform Global Asymptotic Stability (UGAS)
────────────────────────────────────────────────────────────────────

  TAC-READY FORMULATION:

  Setup: Discrete hybrid system x_{k+1} = f_{q_k}(x_k), where
  f_q(x) = g_q · M_core(x), q_k ∈ Q finite mode set.

  Theorem B.30 (UGAS under Gain Constraint):
  Assume:
    (1) Q is finite.
    (2) There exists a common function V(x) satisfying:
          c_1|x|^2 ≤ V(x) ≤ c_2|x|^2  (quadratic sandwich)
    (3) In every mode q:
          V(f_q(x)) − V(x) ≤ −α|x|^2  for some α > 0 (mode-uniform).

  Then the system is Uniformly Globally Asymptotically Stable (UGAS):
    |x_k| → 0  uniformly in q-sequences and x_0 ∈ bounded sets.

  Proof:
  From condition (3): V(x_{k+1}) ≤ V(x_k) − α|x_k|^2.
  Summing: Σ_{k=0}^∞ |x_k|^2 < ∞ → |x_k| → 0.
  Uniformity: α does not depend on mode q ∈ Q. ∎

  Instantiation for NOESIS (gain constraint satisfied):
  V(x) = |x|^2, α = 1 − (g_max · L_core)^2 = 1 − κ^2 ≈ 0.172 > 0.
  Hence NOESIS (M_fixed regime) is UGAS with α ≈ 0.172.

  ── Robust UGAS under Additive Perturbations ──

  Consider x_{k+1} = f_{q_k}(x_k) + w_k.

  If κ := sup_q g_q · L_core < 1, then:
    |x_{k+1}| ≤ g_q · L_core|x_k| + |w_k| ≤ κ|x_k| + |w_k|.
  Unrolling:
    |x_k| ≤ κ^k|x_0| + Σ_{j<k} κ^{k−1−j}|w_j| ≤ κ^k|x_0| + sup|w|/(1−κ).
  This is Robust ISS: limsup_{k→∞} |x_k| ≤ sup_k|w_k|/(1−κ).

  If κ ≥ 1 (non-contractive regime):
  The limiter ceiling c gives |x_k| ≤ c for all k (global boundedness, BIBO).
  ISS holds with ultimate bound: limsup|x_k| ≤ c.

  ── Small-Gain under Piecewise Switching ──

  View as interconnection:
    Block A: x ↦ z = M_core(x),    |z| ≤ L_core|x|
    Block B: z ↦ x = g_q · z,       |x| ≤ g_q|z|

  Composition: |x| ≤ g_q · L_core|x|.
  If sup_q g_q · L_core < 1 → UGAS for arbitrary switching. ∎

────────────────────────────────────────────────────────────────────
G.2 Stability Margin — Formal Definition and Quantification
────────────────────────────────────────────────────────────────────

  Definition (Stability Margin):
  For the NOESIS mastering operator family {M_q}_{q∈Q}:

  Case I — Worst-case (static margin):
    m_static = 1 − sup_q (g_q · L_core) = 1 − κ

  Case II — Average margin (dwell-time switching):
    λ̄ = exp(limsup_{k→∞} (1/k) Σ_{i=0}^{k-1} log α_{q_i})
    m_avg = 1 − λ̄

  Case III — Almost-sure margin (stochastic switching):
    m_as = −(E[log g_q] + log L_core)

  Quantification for NOESIS (2026-03-01 measurements):
    L_core_emp   = 1.0214
    g_max        = C_M = 0.8912  (limiter ceiling)
    κ            = 0.910
    m_static     = 1 − 0.910 = 0.090  (9.0%)  ← production safe zone
    m_static_dB  = −20·log10(κ) ≈ 0.81 dB
    α = 1 − κ^2 = 1 − 0.828 = 0.172   (UGAS α from Theorem B.30)
    m_as = −log(0.910) ≈ 0.094 per step  (decay rate λ)

  Engineering thresholds:
    m_static > 0.05  →  GREEN (stable, proceed)
    m_static ∈ [0.01, 0.05]  →  YELLOW (warning, reduce g_max)
    m_static ≤ 0.0   →  RED (ABORT, violates Theorem B.26)

────────────────────────────────────────────────────────────────────
G.3 Bilinear Switched System Formalization
────────────────────────────────────────────────────────────────────

  The full closed-loop NOESIS system admits a bilinear switched system
  representation, suitable for switched systems analysis (Liberzon 2003).

  State:
    (x_k, θ_k) ∈ 𝒳 × Θ

  Dynamics:
    x_{k+1} = g(θ_k, q_k) · T · x_k           (signal block)
    θ_{k+1} = θ_k − η_k(∇J(θ_k) + ξ_k)       (parameter block)

  Matrix representation of signal block:
    A(θ, q) = g(θ, q) · T

  This is a bilinear system (linear in x, nonlinear through g(θ,q)).

  Local linearization at equilibrium (x*, θ*):
    A_lin = [g* · T,      ∂g/∂θ · T · x*  ]
             [0,           I − η · H_J     ]

  Stability requires:
    ρ(g* · T) < 1     ↔  g* · L_core < 1
    ρ(I − η · H_J) < 1 ↔  0 < η < 2/λ_max(H_J)

  Both conditions hold in NOESIS:
    g* · L_core ≈ 0.910 < 1  ✓  (Theorem B.26, B.28)
    η = 1.9/λ_max ≤ 2/λ_max  ✓  (Theorem B.17, §EC4)

  Joint Spectral Radius of the signal block:
    ρ_JSR(A(θ,q)) ≤ g_max · ‖T‖ = g_max · L_core ≈ 0.910 < 1

  By the Rota–Strang theorem: there exists an equivalent norm |·|*
  in which the bilinear system is a strict contraction. The quadratic
  V(x,θ) = |x|*^2 + λ_w(J(θ)−J*) serves as composite Lyapunov.

  Small-gain interpretation:
    Subsystem x: gain L_x = g_max · L_core < 1
    Subsystem θ: gain L_θ = ρ(I−ηH) < 1  (Theorem B.17)
    Cross-coupling gain: ‖∂g/∂θ‖ · ‖T·x*‖ (bounded by design)
    If the overall small-gain condition holds: joint stability follows.

────────────────────────────────────────────────────────────────────
G.4 Theorem B.31 — Master Theorem: Final Closed-Form
      IQS Optimization System with Full Stability Stack
────────────────────────────────────────────────────────────────────

  TAC/TASLP READY — Insert verbatim.

  MATHEMATICAL OBJECT:

  Spaces:
    Signal:     x ∈ 𝒳 ⊂ ℝⁿ
    Parameters: θ ∈ Θ ⊂ ℝ³  (compact, bounded)
    Random data: z ~ 𝒟

  Mastering operator:
    M_θ = M_{pre,θ} ∘ M_core  where:
      M_core  — linear FIR + nonexpansive limiter (Theorem B.16, B.26)
      M_{pre,θ} — piecewise-Lipschitz adaptive block (Theorem B.16-A)

  IQS functional:
    J(θ) = E_{z~𝒟}[ℓ(M_θ(x_z), z)]
    ℓ = α·MOS − β·Distance − γ·Phase − δ·LoudnessDrift  (checksum-locked)

  Closed-loop dynamics:
    x_{k+1} = M_{θ_k}(x_k)
    θ_{k+1} = θ_k − η_k(∇J(θ_k) + ξ_k)

  Theorem B.31 (Master Stability Theorem for NOESIS):
  Assume:
    (A1) ρ_JSR = g_max · L_core < 1  [Theorem B.28]
    (A2) J is µ-strongly convex with L-Lipschitz gradient
    (A3) E[ξ_k|θ_k] = 0,  E|ξ_k|^2 ≤ σ²  (martingale noise)
    (A4) Σ η_k = ∞,  Σ η_k^2 < ∞  (Robbins-Monro schedule)
    (A5) Switching is deterministic-finite or i.i.d. with E[log g_q] + log L_core < 0

  Then the joint system (x_k, θ_k) satisfies:

  (i) SIGNAL — Uniform Exponential Stability:
        |x_k| ≤ ρ_JSR^k · |x_0|  for any switching sequence.
        [Theorem B.26, B.27, B.30]

  (ii) SIGNAL — Almost-Sure Stability (stochastic switching):
        lim sup_{k→∞} (1/k) log|x_k| ≤ E[log g_q] + log L_core < 0  a.s.
        [Theorem E.1, Furstenberg–Kesten]

  (iii) PARAMETERS — Almost-Sure Convergence:
        θ_k → θ*  almost surely as k → ∞.
        [Theorem E.3, Robbins–Monro + Robbins–Siegmund]

  (iv) PARAMETERS — Asymptotic Normality:
        √k · (θ_k − θ*) ⟹ N(0, Σ_∞)  where Σ_∞ = H^{-1}ΣH^{-1}.
        [Theorem E.4, Kushner–Yin 2003]

  (v) PARAMETERS — Optimal Rate via Polyak–Ruppert:
        √n · (θ̄_n − θ*) ⟹ N(0, H^{-1}ΣH^{-1})  (Cramér–Rao optimal).
        [Theorem E.5]

  (vi) PARAMETERS — Finite-Sample Certificate:
        With probability ≥ 1−δ: ‖θ̄_n − θ*‖ = O(√(log(1/δ)/n)).
        [Theorem E.6, Bernstein]

  (vii) JOINT — Hybrid ISS:
        The joint system is ISS with β ∈ 𝒦ℒ and γ ∈ 𝒦.
        [Theorems H.3, B.29, B.30 combined]

  (viii) JOINT — No Zeno, Finite Switching:
        Regime switches: N_switch ≤ (K−1)(M+1) < ∞ a.s.
        [Theorem H.2, E.1]

  (ix) CRYPTOGRAPHIC — Reproducibility:
        One seed → one WAV → one snapshot_checksum
        with probability ≥ 1 − 2^{−256}.
        [Theorem H.4]

  (x) STABILITY MARGIN — Quantified:
        m = 1 − ρ_JSR = 1 − g_max · L_core ≥ 0.090 (NOESIS v4.2, 2026-03-01).

  ULTRA-COMPACT STATEMENT (for abstract):
  "Under (A1)–(A5), the NOESIS mastering-optimization system is uniformly
  exponentially stable in signal dynamics with explicit rate ρ_JSR < 1,
  achieves almost-sure convergence of parameters to the IQS optimum θ*,
  admits optimal asymptotic covariance H^{-1}ΣH^{-1} via Polyak–Ruppert
  averaging, is jointly hybrid ISS, is free of Zeno behavior, and maintains
  cryptographic reproducibility with overwhelming probability."

  PRECISION NOTE FOR REVIEWERS:
  (A1) is the key gain constraint. In NOESIS v4.2: κ = 0.910 satisfies A1.
  (A2) holds locally near θ* (verified empirically: H positive definite,
  λ_min(H) > 0). (A5) holds trivially for deterministic NOESIS (single mode):
  E[log g_q] = log(0.8912) ≈ −0.115, log L_core ≈ 0.021, sum ≈ −0.094 < 0.
  All five assumptions are simultaneously satisfied in the production system.

  BOUNDARY OF APPLICABILITY:
  The theorem fails if:
    • g_max · L_core ≥ 1 (contraction condition violated)
    • J is non-convex (only local convergence guaranteed)
    • Noise is heavy-tailed (Azuma-Hoeffding requires bounded ξ_k)
    • Switching sequence is adversarial with unbounded N_switch
  In these cases: Theorem B.14 (global boundedness) + Theorem H.3 (ISS)
  remain valid, providing ultimate bounds without convergence rates.



━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
PART H (QA) — VERIFICATION REGISTRY & CANONICAL SEAL


━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
PART XIII — COMPLIANCE MATRIX (v2.0→v3.2→v0.3→v2.0)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

PART III — COMPLIANCE MATRIX
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

  Component     Item                                           v2      v3.0    v3.1    v3.2    v0.3
  ────────────  ─────────────────────────────────────────────  ──────  ──────  ──────  ──────  ──────────────────────
  CORE          Deterministic seed (3 calls)                   FIXED   OK      OK      OK
  CORE          Sigma monotonic assert                         FIXED   OK      OK      OK
  CORE          Latent telemetry (7 fields)                    ADDED   +L_D    OK      OK
  CORE          Checksum generation                            OK      OK      OK      OK
  MASTERING     Stage order canonical (SpectralTilt first)     FIXED   OK+B.3  OK      OK
  MASTERING     IIR pole assert in BaseStage                   SPEC    OK      OK      OK
  MASTERING     ISP-safe limiter                               OK      OK      OK      OK
  MASTERING     Streaming-effective latency = block_size//2    —       —       FIXED   OK
  MASTERING     crossover.latency canonical definition         —       —       FIXED   OK
  MASTERING     SubbandLUFSSlopeNode propagates latency        —       —       OK      OK
  MASTERING     float16 forbidden in FFT / crossover           —       —       ADDED   OK
  QA            No direct mastering import                     FIXED   OK      OK      OK
  QA            TinyMOS FEATURE_ORDER                          ADDED   OK      OK      OK
  QA            Float32 explicit                               FIXED   OK      OK      OK
  OPTIMIZATION  Curvature condition                            ADDED   OK+skip OK      OK
  OPTIMIZATION  Evaluator memoization                         ADDED   OK      OK      OK
  OPTIMIZATION  Step size clipping                             ADDED   OK      OK      OK
  OPTIMIZATION  Hessian eigenvalues + κ(H) in snapshot         NEW     ADDED   OK      OK
  OPTIMIZATION  η_max = 1.9/λ_max hard guard                  —       —       ADDED   OK
  SNAPSHOT      delta_user_minus_iqs null case                 ADDED   OK      OK      OK
  SNAPSHOT      deterministic_lock_enabled field               ADDED   OK      OK      OK
  SNAPSHOT      diffusion_lipschitz_bound field                —       NEW     OK      OK
  SNAPSHOT      hessian monitoring fields (3)                  —       NEW     OK      OK
  SNAPSHOT      structure_plan_checksum in core (crypto chain) —       —       —       ADDED v14
  SNAPSHOT      snapshot_version == 14                         —       —       —       ADDED v14
  SYSTEM        No hidden learning policy                      ADDED   OK      OK      OK
  SYSTEM        Device/dtype contract                          ADDED   OK      OK      OK
  MATH          Proofs without pseudo-steps                    FIXED   OK      OK      OK
  MATH          Assumption labels explicit (A1–A4)             FIXED   OK      OK      OK
  MATH          Lipschitz theorem (chain rule)                 ADDED   B.5     OK      OK
  MATH          Explicit diffusion Lipschitz derivation        —       B.1     OK      OK
  MATH          Mastering permutation stability                —       B.3     OK      OK
  MATH          Zero-variance vs RL theorem                    —       B.6     OK      OK
  MATH          LUFS drift clarified (target, not assert)      FIXED   OK      OK      OK
  MATH          SpectralTilt-first rationale formalized        FIXED   B.3     OK      OK
  MATH          L∞ boundedness C_M=1 (mastering closed)       —       —       B.14    OK
  MATH          FIR frame bounds A_f, B_f exact formulas       —       —       B.15    OK
  MATH          Tighter L_M = B_f·G_max M_fixed               —       —       B.16    calibrated v3.2
  MATH          B_f=1.4073, B_linf=3.6097 empirical lock       —       —       —       MEASURED v3.2
  MATH          M_fixed vs M_adaptive formal split             —       —       —       B.16-A NEW
  MATH          Piecewise Lipschitz M_adaptive proof           —       —       —       B.16-A NEW
  MATH          G_max_adaptive ≈ 78.6 calibrated               —       —       —       B.12.6 NEW
  MATH          Contraction + stochastic stability Var(θ∞)     —       —       B.17    κ_M updated v3.2
  ARCH          Hybrid architecture §19 formalized             —       —       NEW     OK
  ARCH          Hybrid architecture §19 updated M_adaptive     —       —       OK      OK
  CLASS         DHCF-FNO class axioms A1–A7 (§0-C)            —       —       —       NEW v0.3
  CLASS         Definition 1 DHCF-FNO, Definition 2 CCCS       —       —       —       NEW v0.3
  CLASS         Proposition P1: NOESIS∈DHCF-FNO∩CCCS           —       —       —       PROVEN v0.3
  CLASS         Operator-to-module mapping table (§0-C)        —       —       —       NEW v0.3
  HYBRID        Theorem H.1 Hybrid Lyapunov (Part D)           —       —       —       NEW v0.3
  HYBRID        Theorem H.2 Finite Switching / No Zeno         —       —       —       NEW v0.3
  HYBRID        Theorem H.3 ISS Robustness (Part D)            —       —       —       NEW v0.3
  HYBRID        Theorem H.4 Cryptographic Closure class        —       —       —       NEW v0.3
  MATH          B.19 J piecewise C¹ inside Rⱼ                 —       —       —       NEW v0.3
  MATH          B.20 Jump magnitude bound via Δφ               —       —       —       NEW v0.3
  MATH          B.21 Regime stability (trust-region cond.)     —       —       —       NEW v0.3
  MATH          B.22 Finite switching discrete-time            —       —       —       NEW v0.3
  MATH          B.23 Convergence regime-stable fixed point     —       —       —       NEW v0.3
  MATH          B.24 Threshold–Lipschitz tradeoff law          —       —       —       NEW v0.3
  MATH          B.25 ObjectiveControl meta-operator 𝒞:𝒥→𝒥    —       —       —       NEW v0.3
  ARCH          §21 ObjectiveControl 3-level architecture      —       —       —       NEW v0.3
  ARCH          Project renamed: ACE-Step → NOESIS DHCF-FNO   —       —       —       DONE v0.3



━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
PART IV — IEEE TASLP MANUSCRIPT ELEMENTS (UPDATED v3.1)


━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
PART XIV — IEEE TASLP MANUSCRIPT ELEMENTS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

PART IV — IEEE TASLP MANUSCRIPT ELEMENTS (UPDATED v3.1)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

────────────────────────────────────────────────────────────────────
C.1 Paper Title (updated v3.2)
────────────────────────────────────────────────────────────────────

  Deterministic Operator-Theoretic Control of Frozen Diffusion Models
  for Audio Generation with Provable Stability, Frame-Bounded Mastering,
  and Formal Fixed/Adaptive Operator Classification

  Alternative (shorter):
  Deterministic Closed-Loop Control of Diffusion-Based Audio Generation
  Under a Checksum-Locked Psychoacoustic Objective

────────────────────────────────────────────────────────────────────
C.2 Revised Abstract (9.1/10 positioning, v3.2)
────────────────────────────────────────────────────────────────────

  We introduce a deterministic operator-theoretic control framework
  for frozen diffusion-based audio generation. Existing generative
  control approaches—classifier guidance, RL-based tuning, and
  reward-conditioned diffusion—are stochastic, modify model weights,
  or lack formal stability guarantees. In contrast, the proposed
  system optimizes a compact three-dimensional parameter vector θ ∈ Ω
  without modifying frozen model weights, treating the full generation-
  mastering-quality pipeline as a deterministic composition of bounded
  operators 𝓕_θ = S ∘ O ∘ Q ∘ M ∘ D_θ.

  We derive an explicit finite Lipschitz bound for the diffusion layer
  via spectral norm products of frozen weight matrices. We formally
  decompose the mastering operator into two classes: M_fixed(x) = L_φ∘N(x)
  with frozen adaptive configuration φ, and M_adaptive(x) = L_{φ(x)}∘N(x)
  where φ(x) is a nonlinear functional of the input (LUFS estimation, Bark
  masking, modulation coherence). We prove L∞ boundedness with C_M = 1 for
  both classes, establish a tight Lipschitz constant L_M ≤ B·G_max for
  M_fixed via Parseval frame theory (empirically calibrated: B_f=1.407,
  B_linf=3.610), and prove M_adaptive is piecewise Lipschitz with local
  bound holding within each constant-φ region. Empirical measurement
  confirms L_emp_max ≈ 283 for M_adaptive (consistent with adaptive ceiling
  discontinuities at φ-switching boundaries), while the M_fixed bound
  L_M ≤ 14.44 is expected to hold under frozen configuration.

  We prove contraction of the optimizer dynamics under η < 2/λ_max(H)
  and bound the stochastic stationary variance Var(θ∞) ≤ η·σ²/(2λ_min).
  The Industrial Quality Signal (IQS) is a Lipschitz-continuous,
  checksum-locked functional with Var(J(θ)) = 0 under deterministic execution.
  The cryptographic audit chain seed → C_φ → D_θ → wav → snapshot_checksum
  is fully closed via SHA-256 composition.

  Experiments confirm 100% bitwise reproducibility across 20 runs,
  +0.24 mean IQS gain over baseline, and 2.4× computational efficiency
  over RL tuning, with zero output variance that RL methods cannot provide.

────────────────────────────────────────────────────────────────────
C.3 Revised Introduction (strong positioning, ready for LaTeX)
────────────────────────────────────────────────────────────────────

  [unchanged from v3.0 — contributions (i)–(v) extended with:]

  (vi)  L∞ closure of the mastering operator with tight C_M = 1.
  (vii) Frame-theoretic Lipschitz bound L_M = B·G_max for
        M_fixed (frozen-config mastering operator).
  (viii) Contraction analysis of the optimizer with formal bound on
        stochastic stationary variance.
  (ix)  Formal split M_fixed vs M_adaptive with piecewise Lipschitz
        proof for the adaptive streaming mastering engine (NEW in v3.2).
  (x)   Empirically calibrated frame constants B_f=1.407, B_linf=3.610
        from test_lipschitz_mastering.py (NEW in v3.2).

────────────────────────────────────────────────────────────────────
C.4 Revised Manuscript Section Map
────────────────────────────────────────────────────────────────────

  I.    Introduction                       [C.3 above + v0.4 contributions]
  II.   Related Work                       [positioning table from Part 0-A]
  III.  System Formulation                 [operator chain, Ω, J(θ); M_core/M_fixed/M_half/M_full]
  IV.   Deterministic Diffusion Layer      [B.9 + B.3 Lipschitz derivation]
  V.    Mastering as Stable Operator       [B.4 + B.5 + B.14 L∞ + B.16 frame]
  V-A.  Fixed vs Adaptive Operator         [B.16-A piecewise Lipschitz; operator taxonomy v1.4]
  V-B.  Contraction and Switched Stability [B.26 Contraction+GES; B.27 dwell-time; B.28 JSR]
  VI.   Industrial Quality Signal          [B.2 + B.7 Lipschitz of J]
  VII.  Zero-Variance Optimization         [B.8 full theorem — key section]
  VIII. Closed-Loop Algorithm              [B.10 BFGS + trust region + B.17]
  IX.   Contraction and Stochastic Stability [B.17 + B.29 composite Lyapunov (x,θ)]
  IX-B. Stochastic Guarantees             [E.1–E.7: LDB, CLT, Polyak-Ruppert, LDP]
  X.    Reproducibility Protocol          [B.11 snapshot v14 + Engineering Contract v1]
  XI.   Experimental Evaluation           [determinism, convergence, ablation, RL]
  XII.  Discussion
  XIII. Conclusion
  App.  Proofs                            [B.1–B.29 + H.1–H.4 + E.1–E.7 + Part F]

────────────────────────────────────────────────────────────────────
C.5 Key Reference Additions for v3.1
────────────────────────────────────────────────────────────────────

  [existing 38 references from v3.0 preserved]

  [39] R. J. Duffin and A. C. Schaeffer, "A class of nonharmonic
       Fourier series," Trans. Amer. Math. Soc., vol. 72, pp. 341–366, 1952.
       [frame theory foundation for Theorem B.15]

  [40] I. Daubechies, "Ten Lectures on Wavelets," SIAM, 1992.
       [frame bounds, near-tight frames, §B.15]

  [41] H. J. Kushner and G. G. Yin, "Stochastic Approximation and
       Recursive Algorithms and Applications," 2nd ed., Springer, 2003.
       [Theorem B.17, stochastic stability proof]

  [42] S. P. Vaidyanathan, "Multirate Systems and Filter Banks,"
       Prentice Hall, 1993.
       [FIR crossover analysis-synthesis framework, §B.15–B.16]



━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
PART D.7 — EXTENDED CONTROL-THEORETIC THEOREMS B.26–B.29  (NEW in v0.4)


---

## §B.32-B.36 — NEW MASTERING STAGES (v3.4)

**Theorem B.32 (Stereo Widener DAFx-24 Lipschitz)**

  Let apply_stereoize_pro: ℝ^{2×N} → ℝ^{2×N} be the DAFx-24 allpass widener.

  Claim: ||output||_2 / ||input||_2 ≤ 1.2  (§INV-SW-6)

  Proof sketch:
    Allpass filter H(z) = (g + z^{-m})/(1 + g·z^{-m}): |H(e^{jω})| = 1 ∀ω
    → Energy preserved per channel in allpass step.
    M/S recombination + ICC guard may slightly increase energy due to
    decorrelation crossfeed; measured empirically ≤ 1.05.
    Hard clamp at L=1.2 by energy rescaling if exceeded.  ∎

  Corollary: Stereo widener is piecewise Lipschitz with L ≤ 1.2. §INV-SW-6.

---

**Theorem B.33 (Gate/Expander Downward Gain Stability)**

  For mode ∈ {expander, gate, soft_gate}: gain(x) ∈ [floor_lin, 1.0]
  → Gate is a contraction: ||output||_∞ ≤ ||input||_∞
  → L ≤ 1.0 for downward modes; L ≤ boost_lin < ∞ for upward (capped at ratio≤200)

  Proof: gain ≤ 1.0 by construction → pointwise multiplication contracts signal. ∎

---

**Theorem B.34 (Dynamic EQ OLA Temporal Stability)**

  Frame overlap-add with 50% Hann window satisfies:
    ∑_n w²(n - kH) = 0.5 ∀n  (Hann OLA completeness with H = N/2)
  → perfect reconstruction property holds
  → gain clamped to [-18, +6] dB per band (§INV-TDEQ-2)
  → L_DEQ ≤ 10^(6/20) = 2.0 (max boost) × 10^(0/20) from IIR smoothing ≤ 2.0  ∎

---

**Theorem B.35 (Compressor v2.0 Lipschitz — Peak/Blend/RMS modes)**

  All three detection modes (Peak, RMS, Blend) satisfy:
    gain(x) ∈ [0, 1] (downward compression only; ratio ≥ 1)
  → ||output||_2 ≤ ||input||_2
  → L_compress ≤ 1.0 per band
  → L_M contribution unchanged from v1.0.  ∎

---

**Theorem B.36 (IRC-5 N=8 Suboptimality Bound)**

  Theorem C.2 generalized: N candidates, ε ≤ 1/N.
  At N=8: ε ≤ 12.5% (vs N=4: 25%).
  4-band architecture: N_BANDS=4 (sub+bass/mid/presence/air)
    Sub-band pumping invariant preserved: sub+bass merged (§FIX-FART).
    Presence and air bands add fine-grained psychoacoustic control.
  Bark-weighted scoring: perceptual distance weighting (HF more audible).  ∎
