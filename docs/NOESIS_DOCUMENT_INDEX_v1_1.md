[NOESIS_DOCUMENT_INDEX_v1_1.md](https://github.com/user-attachments/files/25818755/NOESIS_DOCUMENT_INDEX_v1_1.md)
"""NOESIS — Deterministic Hybrid Control Framework for Frozen Neural Operators (DHCF-FNO)
Copyright (c) 2026 AMAImedia.com
All rights reserved.
docs/NOESIS_DOCUMENT_INDEX_v1_1.md"""

# NOESIS DOCUMENT INDEX v1.1

```
Version:    v1.1  (2026-03-08)
Author:     Ilia Bolotnikov / AMAImedia.com (2026)
Status:     ACTIVE — update when document set changes
Supersedes: NOESIS_DOCUMENT_INDEX_v1_0.md (2026-03-07)
Changes v1.1:
  Phase completions (2026-03-08):
    R.GPU CLOSED: 10/10 PASS, IQS mean=0.7027, drift_max=0.0078 dB, 10/10 PASS tier
    P1.1 CLOSED:  true_peak_filter.py v1.0, mastering_pre_limiter.py v4.0,
                  mastering_chain.py v1.4, limiter.py (apply_limiter_tp)
  Document versions updated:
    TABLE_I        v1.1 → v1.2 (Phase R.GPU data, IQS v0.9 header, TP column)
    BENCHMARK_RESULTS v1.1 → v1.2 (R.GPU table, A/B, R.3 FAIL analysis)
    CROSS_AUDIT    v1.0 → v1.1 (Inv.#19 DONE, R.GPU CLOSED, R.3 known gap)
    TZ             v1.1 → v1.2 (phase statuses, R.GPU/P1.1 marked DONE)
  OPEN ISSUES table updated:
    R.GPU: Pending → CLOSED
    P1.1:  Pending → CLOSED
    Added: QA.1 FAD(PANN) — R.3 FAIL fix (CRITICAL, next session)
  BENCHMARK STATUS section replaced with real R.GPU data
  CODE FILES section updated: mastering_chain v1.4, true_peak_filter v1.0
  SYSTEM CONSTANTS: mastering_chain_version, pre_limiter_version added
```

---

## DOCUMENT HIERARCHY

### Tier 0 — THEORY

The complete protocol is TWO files read together:

| File | Lines | Content |
|------|-------|---------|
| `NOESIS_DHCF_FNO_PROTOCOL_v0_7.md` | 4198 | **COMPLETE CANONICAL BASE (sealed).** All formal theory: Parts 0–M. Axioms A1–A7. Proposition P1 (NOESIS ∈ DHCF-FNO). Theorems B.1–B.31 (full proofs). H.1–H.4 (Hybrid: Filippov, Zeno, ISS, Crypto). E.1–E.7 (Stochastic: Robbins-Monro, CLT, LDP). Propositions A.1–A.4, Theorem A.5. Engineering Contract v1 (Part F). Master Theorem + UGAS (Part G). Hierarchical Optimization J_extended (Part K). Interface Contracts (Part L). §M.1–§M.9. Operator taxonomy v1.4. |
| `NOESIS_DHCF_FNO_PROTOCOL_v0_8_ADDENDUM.md` | 783 | **ADDENDUM (extends v0.7, does not replace it).** CORR-1: IQS v0.7→v0.9 correction. CORR-2: Invariants #4/#6/#16 corrections. Part C: Theorems C.1 (Near-Tight FIR), C.2 (IRC-5 12.5%), C.3 (Phase Alignment). Part F: Theorems F.1 (KAD/MMD, ICML 2025), F.2 (MAD/MAUVE, SRC+0.15), §F.3 metric taxonomy. Part G: Definition G.1 (FNO interface), generator agnosticism. §M.10: /sleep CPM, Algorithm M.10.2. Invariants #19–#23. Canonical Formula Base v0.9. Quick-Reference table. BibTeX. |

---

### Tier 1 — CONTRACTS & SPECIFICATIONS

| File | Lines | Content |
|------|-------|---------|
| `NOESIS_ENGINEERING_CONTRACT_v0_8.md` | 809 | SS1–SS28 (sealed from v0.7). **NEW v0.8:** SS29 (KAD Invariant F.1), SS30 (True-Peak BS.1770-4), SS31 (Phase-Aligned Bass + Crossover + Merkle), SS32 (Segment Control 14-tag), SS33 (Memory /sleep), SS34 (Generator Agnosticism), SS35 (Coding Architecture Rule: orchestrator ≤200 lines + helpers). Full Invariant Index #1–#23. |
| `NOESIS_MASTERING_SPEC_v0_9.md` | 520 | DSP pipeline. **FIXED v0.9:** MonoBass = Stage 0. Canonical stage order: [0]MonoBass→[1]SpectralTilt→[2]LUFSSlope→[3]Multiband→[4]GlueBus→[5]BarkMask→[6]ModCoherence→[7]Limiter→[8]LUFSTrim. Theorem C.1/C.3 integrated. Stage 8.5 v2 (two-pass, sparse_benefit +4 dB, 5ms lookahead). Gen.guard (ARTIFACT_FLOOR=−55, gain_cap=+60 dB). Lipschitz chain (B_linf, G_max, L_M_fixed). Three-tier drift contract. |
| `NOESIS_MASTERING_AND_STEM_SPEC_v0_9.md` | 810 | Consolidated mastering + stem + vocal spec. Full pipeline Stages 0–9 + optional VST Stage 10. 12-band STFT stem separation standard (immutable: kick/snare/hihats/perc/subbass/bass/lead/synth/pad/vocals/fx/other). Cosine crossover, energy-normalized, n_fft=4096. Vocal conditioning pipeline. |

---

### Tier 2 — VERIFICATION & PLANNING

| File | Lines | Content |
|------|-------|---------|
| `NOESIS_ROADMAP_v0_11.md` | 955 | R.GPU **(DONE 2026-03-08)**. P1.1 **(DONE 2026-03-08)**. P1.2–P1.3 (KAD, FIR 8192). P2.1–P2.3 (Phase-Aligned Bass, Segment 14-tag, Stem EQ). P3.1–P3.5 (Merkle+JSONL, EventBus, editFLAM, VST3, MIDI). P4.1–P4.4 (IRC-5, Multiband Width, Demucs, Caption LLM). P5 (/sleep CPM). Scientific papers: KAD (ICML 2025), MAD (2025), Muse (2026), Kader Survey (2025). Priority matrix + dependency graph. |
| `NOESIS_CROSS_AUDIT_v1_1.md` | ~560 | Protocol↔Code audit. Items A–K (from v0.8). Items L–O (IQS v0.9, MOS proxy, Genre HD, TinyMOS). Items P–V (stage order, range, TruePeak, KAD, Snapshot, generator, architecture). **NEW v1.1:** Inv.#19/SS9.7/SS30 → DONE (P1.1); R.GPU → CLOSED; Conflict 9 (snapshot assertion); R.3 known gap in §6. |
| `NOESIS_MODULE_AND_MIGRATION_v1_0.md` | 587 | Full file tree 150+ files, 15 packages. 7-layer import DAG (no cycles). Zero-legacy verification (no "acestep" imports in production code). Legacy/import migration map. Orchestrator refactor priority queue. New packages spec (generation/, personalization/, vst/, midi/). |

---

### Tier 3 — OPERATIONAL

| File | Lines | Content |
|------|-------|---------|
| `NOESIS_TZ_v1_2.md` | ~800 | **v1.2 (2026-03-08).** R.GPU DONE, P1.1 DONE. QA.1 added (FAD PANN + PAL v2 deploy). P1.2b added (AudioSR Stage 0.5). Priority matrix updated. Recommended next sessions: QA.1 → P1.2b+P1.2 → P1.3. All A/B results recorded. |
| `NOESIS_TABLE_I_v1_2.md` | ~180 | **v1.2 (2026-03-08).** Phase R.GPU real data: 10/10 PASS, IQS mean=0.7027, drift_max=0.0078 dB, all PASS tier. mastering_chain v1.4, IQS v0.9. R.3 FAIL analysis (proxy design failure). Stage 8.5 v4 improvement analysis. |
| `BENCHMARK_RESULTS.md` | ~160 | **v1.2 (2026-03-08).** Phase R.GPU table (10/10 PASS). R.GPU vs R.1 comparison (IQS +42%, drift ×206). A/B: R2 B_WIN, R4 A_WIN, R5/R6 NEUTRAL. R.3 correlation analysis. Snapshot v16 sample with true_peak_dbtp. Stability block. |
| `NOESIS_DOCUMENT_INDEX_v1_1.md` | — | This file. |

---

## RESPONSIBILITY MATRIX

| Topic | Document | Section |
|-------|----------|---------|
| **Axioms A1–A7 (full text)** | Protocol v0.7 | PART 0 |
| **Proposition P1 (NOESIS ∈ DHCF-FNO proof)** | Protocol v0.7 | PART 0-C |
| **Theorems B.1–B.31 (full proofs: Lipschitz, BFGS, sigma, frame)** | Protocol v0.7 | PART II |
| **Hybrid theory H.1–H.4 (Filippov, Zeno, ISS, Crypto)** | Protocol v0.7 | PART D |
| **Stochastic theory E.1–E.7 (Robbins-Monro, CLT, LDP)** | Protocol v0.7 | PART E |
| **Engineering Contract v1 (§EC1–§EC11)** | Protocol v0.7 | PART F |
| **Master Theorem + UGAS + Bilinear System** | Protocol v0.7 | PART G |
| **Hierarchical optimization (J_extended, CSO, Propositions A.1–A.4)** | Protocol v0.7 | PART K |
| **Interface contracts + Mode architecture (Part L)** | Protocol v0.7 | PART L |
| **§M.1–§M.9 Documentation consolidation** | Protocol v0.7 | PART M |
| **Operator taxonomy v1.4** | Protocol v0.7 | PART I |
| **IQS formula correction (v0.7 5-term → v0.9 6-term)** | ADDENDUM v0.8 | CORR-1 |
| **Invariant corrections #4/#6/#16** | ADDENDUM v0.8 | CORR-2 |
| **Theorem C.1 (Near-tight crossover N_tap=8192)** | ADDENDUM v0.8 | §C.1 |
| **Theorem C.2 (IRC-5 suboptimality ≤12.5%)** | ADDENDUM v0.8 | §C.2 |
| **Theorem C.3 (Phase alignment energy conservation)** | ADDENDUM v0.8 | §C.3 |
| **Theorem F.1 (KAD distribution-free MMD, ICML 2025)** | ADDENDUM v0.8 | §F.1 |
| **Theorem F.2 (MAD human alignment SRC+0.15)** | ADDENDUM v0.8 | §F.2 |
| **§F.3 Metric taxonomy table** | ADDENDUM v0.8 | §F.3 |
| **Generator Agnosticism (Definition G.1, backend registry)** | ADDENDUM v0.8 | Part G |
| **Memory Consolidation /sleep (Algorithm M.10.2)** | ADDENDUM v0.8 | §M.10 |
| **Invariants #19–#23 (True-Peak, FIR, JSONL, VST3, EQ)** | ADDENDUM v0.8 | §INV |
| **Canonical Formula Base (IQS v0.9, J, EBU R128, KAD)** | ADDENDUM v0.8 | §FORMULA |
| **Invariants #1–#18 (base list)** | Protocol v0.7 | PART M |
| **Complete invariant index #1–#23** | CONTRACT v0.8 | §Full-Invariant-Index |
| Core stability contract SS1–SS10 | CONTRACT v0.8 | Part A |
| Hierarchical optimization SS11–SS17 | CONTRACT v0.8 | Part B |
| Frozen operator boundary SS18–SS28 | CONTRACT v0.8 | Part C |
| Coding rule (thin orchestrator ≤200 lines) | CONTRACT v0.8 | SS35 |
| Frame bounds contract (N_tap=8192, P1.3) | CONTRACT v0.8 | SS30 |
| Phase-aligned bass + IRC-5 contracts | CONTRACT v0.8 | SS31 |
| CPM/sleep contract (P5) | CONTRACT v0.8 | SS33 |
| Canonical mastering stage order (MonoBass=Stage 0) | MASTERING_SPEC v0.9 | §2.1 |
| Stage 8.5 v4 pre-limiter (CF-budget, TP-aware) | MASTERING_SPEC v0.9 | §2.3 |
| True-Peak 4x FIR (P1.1 DONE) | MASTERING_SPEC v0.9 | §2.3 / SS30 |
| Generation Artifact Guard (ARTIFACT_FLOOR=−55 LUFS) | MASTERING_SPEC v0.9 | §2.0 |
| Three-tier drift contract (PASS/PASS_CF_LIMITED/FAIL) | MASTERING_SPEC v0.9 | §3 |
| 12-band STFT stem separation (immutable standard) | MASTERING_AND_STEM_SPEC v0.9 | §Stem |
| Vocal pipeline (VOCAL_PREFIXES, de-esser) | MASTERING_AND_STEM_SPEC v0.9 | §Vocal |
| **IQS v0.9 formula (6-term canonical, Σ=1.00)** | ADDENDUM v0.8 + ROADMAP v0.11 | §FORMULA / §IQS |
| **IQS weights: α=0.50 η=0.25 β=0.08 γ=0.07 δ=0.06 ζ=0.04** | ADDENDUM v0.8 | §FORMULA |
| **IQS_max=0.75 (Lemma B.5-A proof)** | ADDENDUM v0.8 | §FORMULA |
| Quality fusion J = 0.60·IQS + 0.40·QA | ADDENDUM v0.8 | §FORMULA |
| MOS proxy (calibrated multi-factor) | ROADMAP v0.11 | §MOS |
| Genre HD floors (33 genres, 144 aliases) | ROADMAP v0.11 | §HD |
| Phase roadmap (QA.1 → P1.2 → P1.3 → P2 → P3 → P4 → P5) | ROADMAP v0.11 + TZ v1.2 | §Phases |
| Snapshot v16 new fields (P2.2, P3.1–P3.5) | ROADMAP v0.11 | §Snapshot |
| **Distance metric hierarchy (KAD primary, ADDENDUM §F.0)** | ADDENDUM v0.8 | §F.0, Inv.#16 |
| Protocol↔Code audit items A–V | CROSS_AUDIT v1.1 | A–V |
| P1.1 True-Peak deployment audit | CROSS_AUDIT v1.1 | Item R |
| R.GPU benchmark closure audit | CROSS_AUDIT v1.1 | §1 R.GPU row |
| R.3 Calibration FAIL (known gap → QA.1) | CROSS_AUDIT v1.1 | §6 |
| File tree (150+ files) + import DAG | MODULE v1.0 | §1–§3 |
| Orchestrator refactor priority list | MODULE v1.0 | §8 |
| Zero-legacy policy (no "acestep" imports) | MODULE v1.0 | §6 |
| Phase overview + open gaps | TZ v1.2 | §0–§4 |
| Per-genre empirical measurements (R.GPU) | TABLE_I v1.2 | Full |
| Abort conditions | CONTRACT v0.8 SS9 | + TZ v1.2 §3 |
| Snapshot v16 schema | CONTRACT v0.8 SS6 | + TZ v1.2 §6 |
| Modes (Studio/Fast) | CONTRACT v0.8 SS13 | + TZ v1.2 §3 |

---

## CHANGED CODE FILES (v1.3–v1.4, Phase R.GPU + P1.1)

| File | Version | Change |
|------|---------|--------|
| `mastering/true_peak_filter.py` | v1.0 | **NEW (P1.1).** Kaiser-sinc polyphase FIR L=4, N_per_phase=32, β=12.0, 128 taps. Stopband ≥80 dB. `upsample_4x_polyphase()`, `true_peak_dbtp()`, `true_peak_per_channel_dbtp()`. Pure numpy, deterministic. |
| `mastering/limiter.py` | v1.1 | **P1.1.** Added `apply_limiter_tp()` — TP-aware limiter with Step 6 final safety rescale. Output TP=-1.000 dBTP for ceiling=-1.0. |
| `mastering/mastering_post_limit.py` | v1.1 | **P1.1.** Calls `apply_limiter_tp()`, `limiter_type="tp_aware_v1"`. |
| `mastering/mastering_pre_limiter.py` | v4.0 | **P1.1 / Stage 8.5 v4.** ratio=200:1, threshold=ceiling−0.1 dBFS, release 60/30/15ms, cold-start envelope pre-init. Mode label `"cf_budget_v4"`. Output peaks ≤−1.015 dBFS. |
| `mastering/mastering_chain.py` | v1.4 | **P1.1.** Measures `output_true_peak_dbtp`, logs TP warning if >ceiling. Snapshot field `true_peak_dbtp` added. |
| `metrics/iqs.py` | v0.9 | (from v0.9 consolidation) Weights: α=0.50, η=0.25. IQS_max=0.75. 6-term canonical. IQS_WEIGHTS_CHECKSUM. 12 self-tests. |
| `tools/phase_r_enrichment.py` | v0.8 | MOS proxy rebuilt (calibrated multi-factor). Genre-aware HD normalization (33 genres). pass_gate 0.50→0.55. |
| `qa_external/tiny_mos_predictor.py` | v2.1 | PRODUCTION_FORCE_V1=True. calibrated_predict() semantic linear predictor. |
| `qa_external/industrial_quality.py` | v2.2 | DEFAULT_OBJECTIVE_WEIGHTS synced to v0.9. genre param in compute(). |

---

## SUPERSEDED DOCUMENTS REGISTRY

| Old file | Replaced by |
|----------|-------------|
| NOESIS_DHCF_FNO_PROTOCOL_v0_3/v0_4/v0_5/v0_6.md | Protocol v0.7 (base) |
| NOESIS_DHCF_FNO_PROTOCOL_v0_8 [archived] | NOESIS_DHCF_FNO_PROTOCOL_v0_8_ADDENDUM.md |
| NOESIS_ENGINEERING_CONTRACT_v0_3..v0_7.md | CONTRACT v0.8 |
| NOESIS_MASTERING_SPEC_v0_7/v0_8.md | MASTERING_SPEC v0.9 |
| NOESIS_STEM_AND_VOCAL_SPEC_v0_7.md | MASTERING_AND_STEM_SPEC v0.9 |
| NOESIS_ROADMAP_v0_5..v0_10.md | ROADMAP v0.11 |
| NOESIS_CROSS_AUDIT_v0_7..v1_0.md | **CROSS_AUDIT v1.1** |
| NOESIS_MODULE_AND_MIGRATION_v0_7..v0_9.md | MODULE_AND_MIGRATION v1.0 |
| NOESIS_TZ_v0_1..v1_1.md | **TZ v1.2** |
| NOESIS_TABLE_I_v1_0/v1_1.md | **TABLE_I v1.2** |
| BENCHMARK_RESULTS.md v1.0/v1.1 | **BENCHMARK_RESULTS.md v1.2** |
| NOESIS_DOCUMENT_INDEX_v0_5..v1_0.md | **DOCUMENT_INDEX v1.1 (this file)** |

---

## VERSIONING HISTORY

| Version | Files | Changes |
|---------|-------|---------|
| v0.3–v0.6 | 5–20 | Incremental build-up |
| v0.7 | 9 | Protocol Part M. CONTRACT unified. Phase S complete. |
| v0.8 | 10 | Phase R refactoring: 3 critical fixes, Stage 8.5 v2, Gen.guard |
| v0.9 | 11 | IQS v0.8→v0.9. MOS proxy rebuilt. Genre HD. TinyMOS v2 locked. |
| **v1.0** | **10** | Full consolidation. PROTOCOL sealed v0.7+ADDENDUM v0.8. CONTRACT v0.8 SS29–SS35. MASTERING_SPEC v0.9. ROADMAP v0.11. CROSS_AUDIT v1.0 A–V. MODULE v1.0. TZ v1.1. TABLE_I v1.0. |
| **v1.1** | **10** | **Phase completions 2026-03-08: R.GPU CLOSED (10/10 PASS, IQS mean=0.7027). P1.1 CLOSED (true_peak_filter v1.0, mastering_chain v1.4). CROSS_AUDIT v1.1. TABLE_I v1.2. BENCHMARK_RESULTS v1.2. TZ v1.2. Known gap: R.3 Calibration FAIL → QA.1.** |

---

## OPEN ISSUES (v1.1)

| Issue | Priority | Status |
|-------|----------|--------|
| **QA.1: FAD(PANN) + PAL v2 deploy (R.3 FAIL fix)** | **CRITICAL** | **Next session** |
| P1.2b: AudioSR Stage 0.5 (post-gen enhancement) | CRITICAL | After QA.1 |
| P1.2: KAD(PANN) primary metric upgrade | CRITICAL | After QA.1 |
| P1.3: FIR crossover N_tap=8192 (Theorem C.1) | CRITICAL | After QA.1 |
| P2.1: Phase-Aligned Bass (Theorem C.3) | HIGH | After P1.3 |
| P2.2: Segment Control 14 tags | HIGH | Unblocked (P1.1 DONE) |
| P2.3: Per-Stem Adaptive EQ (Invariant #23) | HIGH | After P1.2 |
| P3.1: Merkle + Chained JSONL (Invariant #21) | MEDIUM | After P2.2 |
| P3.4: VST3 Pedalboard (Invariant #22) | MEDIUM | Unblocked (P1.1 DONE) |
| R.5: ISO226 revalidation --online | MEDIUM | After QA.1 |
| R.6: GlueBus revalidation --online | MEDIUM | After QA.1 |
| modeling_noesis_v15_turbo.py refactor (2246 lines) | HIGH | Pending |
| TinyMOS supervised training (>=500 MOS labels) | LOW | After benchmark accumulation |
| P4.1: IRC-5 limiter (Theorem C.2) | LOW | After P1.1+P2.3 |
| P4.3: Demucs neural stem separation | FUTURE | Phase 4 |
| P4.4: Caption Optimizer LLM | FUTURE | Phase 4 |
| V.1: VocalExpressiveness pybind11 | FUTURE DEFERRED | After P4.3 |
| P5: /sleep Memory Consolidation (§M.10) | FUTURE | Phase 5 |
| ~~R.GPU: GPU validation 10/10 PASS~~ | ~~CRITICAL~~ | **CLOSED 2026-03-08** |
| ~~P1.1: True-Peak 4× FIR + EBU R128~~ | ~~CRITICAL~~ | **CLOSED 2026-03-08** |

---

## SYSTEM CONSTANTS (SEALED 2026-03-01, extended P1.1 2026-03-08)

```
B_f                     = 1.40730381    (upper frame bound, N_tap=2048)
A_f                     = 0.51969725    (lower frame bound, N_tap=2048)
B_linf                  = 3.60972762    (L∞ Young's bound, until P1.3)
L_core_empirical        = 1.0214        (mastering Lipschitz, 50 trials)
L_N_empirical           = 17.89         (DiT+LM chain Lipschitz)
kappa_empirical         = 0.910         (contraction factor, Theorem B.26)
stability_margin        = 0.090 (9.0%)  (GREEN zone)
operator_graph_checksum = f1d8a82c7d6e859667cddb7732b869ba3b82926501719e6da78b947edc3dd2ba
IQS_VERSION             = "0.9"
IQS weights             = alpha=0.50  eta=0.25  beta=0.08  gamma=0.07  delta=0.06  zeta=0.04
IQS_WEIGHTS_CHECKSUM    = (sealed, see metrics/iqs.py)
IQS_max                 = 0.75   IQS_min = -0.25
ARTIFACT_FLOOR_LUFS     = -55.0
PRE_LIFT_TARGET_LUFS    = -35.0
gain_cap_db             = +60 dB
silence_floor_lufs      = -69 LUFS
fix_nfe                 = 8       (Turbo model — SHIFT_TIMESTEPS only for 8 steps)
dtype_dit               = bfloat16 (NOT float16 — NaN overflow risk in DiT)
--- P1.1 additions (2026-03-08, sealed) ---
mastering_chain_version = "v1.4"
pre_limiter_mode        = "cf_budget_v4"
limiter_type            = "tp_aware_v1"
true_peak_filter_taps   = 128  (L=4 phases × N_per_phase=32)
true_peak_kaiser_beta   = 12.0
true_peak_stopband_db   = 80.0
true_peak_ceiling_dbtp  = -1.0  (EBU R128 / Spotify standard)
```

---

## BENCHMARK STATUS — Phase R.GPU (real GPU, 2026-03-08)

**ACTIVE — supersedes synthetic estimates from v1.0.**

| Genre     | Seed | IQS    | LUFS  | Drift (dB) | Tier | Pass |
|-----------|------|--------|-------|------------|------|------|
| ambient   | 42   | 0.7301 | -13.8 | 0.0042     | PASS | ✅   |
| edm       | 42   | 0.7103 |  -9.2 | 0.0031     | PASS | ✅   |
| hip-hop   | 42   | 0.6520 | -10.6 | 0.0034     | PASS | ✅   |
| jazz      | 42   | 0.7330 | -13.6 | 0.0076     | PASS | ✅   |
| classical | 42   | 0.7472 | -16.0 | 0.0040     | PASS | ✅   |
| neurofunk | 42   | 0.6236 |  -8.2 | 0.0013     | PASS | ✅   |
| lofi      | 42   | 0.7554 | -13.4 | 0.0037     | PASS | ✅   |
| metal     | 42   | 0.5792 |  -8.6 | 0.0055     | PASS | ✅   |
| pop       | 42   | 0.7441 | -12.2 | 0.0078     | PASS | ✅   |
| ambient   | 99   | 0.7515 | -13.8 | 0.0013     | PASS | ✅   |

```
IQS: mean=0.7027  std=0.0589  min=0.5792 (metal)  max=0.7554 (lofi)
10/10 PASS  |  drift_max=0.0078 dB  |  all PASS tier
Framework: mastering_chain v1.4, IQS v0.9, Snapshot v16
```

**R.3 Calibration:** FAIL (Pearson r=+0.1177, MAE=0.4405)
Root cause: proxy design failure. Fix: QA.1 FAD(PANN). Do NOT recalibrate IQS weights.

---

*Document: NOESIS_DOCUMENT_INDEX_v1_1.md*
*Version: v1.1 (2026-03-08)*
*Author: Ilia Bolotnikov / AMAImedia.com (2026)*
*Supersedes: NOESIS_DOCUMENT_INDEX_v1_0.md (2026-03-07)*
*Status: ACTIVE — next update after QA.1 + P1.2 (session 1)*
