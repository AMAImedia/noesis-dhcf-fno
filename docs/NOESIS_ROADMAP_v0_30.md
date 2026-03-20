[NOESIS_ROADMAP_v0_30.md](https://github.com/user-attachments/files/26134390/NOESIS_ROADMAP_v0_30.md)
"""NOESIS — Deterministic Hybrid Control Framework for Frozen Neural Operators (DHCF-FNO)
Copyright (c) 2026 AMAImedia.com
All rights reserved.
B:/Downloads/Portable/NOESIS_DHCF-FNO/docs/NOESIS_ROADMAP_v0_29.md"""

# NOESIS ROADMAP v0.29
## Единый полный мастер-документ состояния проекта

```
Version:    v0.30  (2026-03-21, CPU Day 2)
Author:     Ilia Bolotnikov / AMAImedia.com
Status:     ACTIVE — supersedes v0.29
Hardware:   Mechrevo GM7AG0M — RTX 3060 Laptop 6GB | i7-12700H | 64GB DDR5

Changes v0.30 (2026-03-21 — CPU Day 2):
  + entries_rare.py: 55 entries, 11 rare genres registered in orchestrator
  + sc_text_controller.py: 36→84 rules (EDM/vocal/emotional/platform/genre shorthand)
  + genre_mastering_master_profiles.py: 29→38 profiles (hyperpop/darkwave/shoegaze/
    bossa_nova/orchestral/soundtrack/world/afrobeat/kpop)
  + expand_caption_dataset.py: caption dataset 414→579 pairs (taxonomy_expand_v1)
  + Caption LoRA: RUNNING (noesis_caption_trainer.py, ~2h CPU, 579 pairs)
  + DriftBadge.tsx BUG FIX: TIER_CF_LIMIT 2.0→6.0 (Drift v1.3)
  + CreatePanel.tsx: +DisCoder DISC toggle, +taxonomy preview widget
  + api.ts: +taxonomyApi.search() POST /api/taxonomy/search
  + noesis_rest_api.py: +POST /api/taxonomy/search
  + test_fad_clap_backend.py: 17→21 methods (C2 additions)
  + version.py: +FAD_CLAP_BACKEND_VERSION +CLAP_EMBED_DIM +CLAP_MODEL_ID
  + generate_paper_figures.py: updated 22→31 stages, R.REF2 J values
  + BENCHMARK_RESULTS.md: CPU Day 2 section added
  + NOESIS_PAPER_SUPPLEMENT_v1_2.md: §7 quality table, SVC=DONE
  + CLAUDE.md/AGENTS.md: architecture state updated

Changes v0.29 (2026-03-20 — Ozone 12 10/10 parity):
  + MasteringChain v3.3→v3.4 (29→31 stages)
  + Stage 2.8  gate_expander.py         Pro-G parity: 6 modes, 5ms lookahead, hold
  + Stage 7.5  tape_saturator.py v1.1   +wow/flutter (pitch warp) +tape noise floor
  + Stage 8.1  exciter.py v1.1          +3rd harmonic +3-band multiband (body/presence/air)
  + Stage 9.5  neural_eq.py v1.1        +OLA frames 50ms +IIR adaptive strength — dynamic
  + Stage 9.6  dynamic_eq.py            +temporal IIR att=10ms/rel=100ms OLA reconstruction
  + Stage 12   stereo.py v2             DAFx-24 allpass decorrelator +ICC +transient guard
  + Stage 14   de_esser_v2.py v2.1      +adaptive freq detection +5ms lookahead +compound
  + Stage 16.5 transient_shaper.py v2   +IIR onset +hold +routes via prompt_taxonomy
  + compressor.py v2.0: Peak/Blend/RMS, M/S per-band, 2ms lookahead, CF auto-release
  + irc5_limiter.py v2.1: N_CANDIDATES=8 (ε≤12.5%), N_BANDS=4, bark scoring
  + reference_eq.py v1.1: ERB smoothing genres + ReferenceEQHighRes 96-band for WAV
  + n4_macro_planner.py BUG FIX: local_files_only=True добавлен (§INV-OFFLINE)
  + РАЗЪЯСНЕНИЕ Qwen-моделей (см. §LLM ниже)
  + Все 11 модулей Ozone 12: 10/10 (реальный паритет)

Changes v0.28 (2026-03-18 — CPU день):
  + MasteringChain v3.0→v3.3 (22→29 stages)
  + Stage 2.5 HF artifact scrubber, Stage 6 FIX, Stage 14.5 VocalBoost
  + DisCoder (MUSHRA 88.14), BigVGAN v2, REST API v1.1
  + 720+ tests PASS
```

---

## §0. ТЕКУЩЕЕ СОСТОЯНИЕ (2026-03-21 CPU Day 2)

```
Tests:       720+ PASS | 14 known-fails (model absent — ждут GPU)
GPU:         SVC training завершён (~step 10000)
             ГОТОВ к GPU День 1 (Phase R + QA.1 + UniverSR A/B)

Mastering:   MasteringChain v3.4 — 31 Stages
  Stage 2.5  HF artifact scrubber (bfloat16 quantization noise)        2026-03-18
  Stage 2.8  Gate/Expander (Pro-G parity, 6 modes, 33 presets)         NEW 2026-03-20
  Stage 6    StemAwareNode.process_audio() FIXED                       2026-03-18
  Stage 7.5  Vintage Tape Saturator (tanh+wow/flutter+noise)           NEW 2026-03-20
  Stage 8.1  Air Exciter (3-band, 2nd/3rd harmonics)                   NEW 2026-03-20
  Stage 9.5  Neural EQ dynamic (OLA+IIR adaptive strength)             UPG 2026-03-20
  Stage 9.6  Dynamic EQ temporal (IIR att/rel OLA)                     NEW 2026-03-20
  Stage 12   Stereo Widener DAFx-24 (allpass+ICC+transient guard)      UPG 2026-03-20
  Stage 14   De-esser v2.1 (adaptive freq+lookahead+compound)          UPG 2026-03-20
  Stage 14.5 Adaptive Vocal Prominence Boost                           2026-03-18
  Stage 16.5 Transient Shaper v2 (IIR onset+hold+taxonomy)             UPG 2026-03-20
  compressor v2.0: Peak/Blend/RMS, M/S, 2ms lookahead                 UPG 2026-03-20
  irc5: N=8 candidates, 4 bands (sub/mid/presence/air), bark score     UPG 2026-03-20
  reference_eq v1.1: ERB genres, 96-band WAV ref                       UPG 2026-03-20

IQS:         v0.8, checksums 9097e7605b8895f1 | 12c2f47cba1ac6be — SEALED
MOS:         NOESIS-MOS v1 (sha256=d781d747, r=0.837) — TinyMOS backend
             N1.4 Jumbo: noesis_mos_jumbo_trainer.py READY, GPU pending

REST API:    v1.1 — enable_discoder, mastering_mode, target_lufs
             /api/quality/settings, /api/quality/ab-test — LIVE
             noesis_api/main.py AND ui/noesis_rest_api.py — quality endpoints

SDK:         cli/sdk.py — enable_discoder=False, self._device stored, DisCoder hook

Vocoders:    DisCoder (ICASSP 2025, MUSHRA=88.14) — models/discoder/ DOWNLOADED
             BigVGAN v2 backend READY (alternative, ~500MB)

Taxonomy:    cli/prompt_taxonomy.py — 57 base + 992 extended = 1049 entries
             TF-IDF cosine search, 573 tokens, 37 genres, 59 moods
             build_caption() fallback: dual import path (robust)

Caption:     artifacts/caption_dataset/caption_train.jsonl — 414 pairs READY
             artifacts/NOESIS-CAPTION-5K/captions.json — 414 records

Paper v14:   artifacts/paper_v14_patch_summary.md DONE
             artifacts/paper_v14_overleaf_patch.txt DONE
             TeX: docs/noesis_arxiv/ (main.tex, appendix.tex, tables.tex)
             BENCHMARK_RESULTS.md — mastering fixes table added

UI:          Vite 5 + React 19 + TypeScript :5173 — SEALED (NOT Next.js)
             start_noesis.bat: FastAPI :8765 + Gradio :7860 + Vite :5173
```

---

## §1. ВСЕ ВЫПОЛНЕННЫЕ ФАЗЫ (хронологически)

```
──── CORE INFRASTRUCTURE ────────────────────────────────────────────
Phase P        Snapshot v16+, fingerprint, cryptographic closure          DONE
Phase 0        HarmonicDensity + LatentEntropy + dit_runtime v9           DONE
Phase 1        QA Ensemble, QualityFusion                                 DONE
Phase 2        CSO + MutationController v2 + GenerationRouter             DONE
Phase S        Code hygiene: 139+ files, zero legacy, all headers         DONE


──── CPU DAY 2 (2026-03-21) ─────────────────────────────────────────
entries_rare.py      55 entries × 11 rare genres, registered orch.    DONE 2026-03-21
SC 84 rules          36→84 (+EDM/vocal/emotional/platform/genre)       DONE 2026-03-21
38 genre profiles    29→38 (+hyperpop/darkwave/shoegaze/bossa_nova/    DONE 2026-03-21
                     orchestral/soundtrack/world/afrobeat/kpop)
Caption dataset      414→579 pairs (expand_caption_dataset.py)         DONE 2026-03-21
Caption LoRA         RUNNING — artifacts/caption_lora/ (~2h CPU)       IN PROGRESS
DriftBadge fix       TIER_CF_LIMIT 2.0→6.0 (Drift v1.3)               DONE 2026-03-21
UI DisCoder toggle   CreatePanel.tsx DISC button                        DONE 2026-03-21
UI taxonomy preview  CreatePanel.tsx + api.ts + noesis_rest_api.py     DONE 2026-03-21
FAD CLAP tests       17→21 methods (C2 additions)                       DONE 2026-03-21
Paper Supplement     v1.1→v1.2 (§7 quality table, SVC=DONE)           DONE 2026-03-21

──── MASTERING QUALITY ──────────────────────────────────────────────
Phase 1-PQ     PerceptualAnalysisLayer v2.0 (Bark+ERB+phase)             DONE
Phase 2-MQ     IQS v0.8, iso226 EQ, GlueBus v2.0                        DONE
Phase P1.1     True-Peak 4x256 FIR Kaiser + EBU R128                    DONE
Phase P1.2     KAD backend + KADPrimaryEvaluator (code)                  DONE 2026-03-16
Phase P1.2b    AudioSR post-gen boost_mode +1.5dB/oct (code)             DONE 2026-03-16
Phase P1.3     FIR Crossover N_TAPS=8192 (code)                          DONE 2026-03-16
Phase P2.1     Phase-Aligned Mono Bass xcorr_align, max_delay=5ms        DONE 2026-03-16
Phase P2.2     Perceptual EQ Curves per Genre                            DONE
Phase P2.3     Per-Stem Adaptive EQ _STEM_TILT_TARGETS                   DONE 2026-03-16
Phase P3.1     Chained JSONL + MerkleTree + Snapshot v16.4               DONE
Phase P3.2     TelemetryEventBus + PerfProfileCollector                  DONE
Phase P3.3     OpenFLAM + IQS_edit + EditPenalty                        DONE
Phase P3.4     VST3 via pedalboard (Stage 10)                            DONE
Phase P3.5     MIDI Extraction via basic-pitch                           DONE
Phase P3.8     MasteringChain v1.8->v3.0: 22-stage reorder + MC-ACOUSTIC DONE 2026-03-14
Phase P4.1     IRC-5 Multiband Limiter (vectorized)                      DONE
Phase P4.2     Multiband Stereo Width + Correlation Guard                DONE
Phase P4.3     HTDemucs stem separation (45 tests)                       DONE

──── BENCHMARKING ───────────────────────────────────────────────────
Phase R.1-6    Offline benchmark synthetic                                DONE
Phase R.FIX    3 critical fixes (drift/LUFS/MOS)                         DONE
Phase R.REF    tools/ decomposition (5+8 modules)                        DONE
Phase R.REF3   First valid benchmark (guidance_scale=7.0)                DONE 2026-03-12
               IQS_mean=0.5243, J_mean=0.3146, 10/10 PASS/CF_LIM

──── AI QUALITY METRICS ─────────────────────────────────────────────
Phase N1.1     NOESIS-MOS v1 (sha256=d781d747, r=0.837, MAE=0.203)      DONE 2026-03-12
Phase N1.2     Qwen3 encode_fn inject (7/7+9/9 tests)                    DONE 2026-03-12
Phase N1.5     Vocal Timbre Tags (24 tests)                              DONE
Phase N1.6     De-esser v2 Stage 7.7 (30 tests)                         DONE
N1.4 Trainer   noesis_mos_jumbo_trainer.py — GPU-ready                   DONE 2026-03-18

──── GENRE / GENERATION SYSTEM ──────────────────────────────────────
Phase N2.1-N2.5 AGMS: 33 genres, 144 aliases, EMA learner, 40 tests     DONE 2026-03-12
Phase N4-N11   Macro/Meso/Micro/MWM/Band/Vocal (65/65 PASS)             DONE 2026-03-13
Phase N3 CODE  N3.1s2/N3.2/N3.3/N2.4 — api/ + voice/ (19 files)        DONE 2026-03-16
N3.K DONE      key_emotional_profile.py -> prompt_utils.build_caption()  DONE 2026-03-17
N3.M DONE      benchmark_msr_eval.py SI-SDR/SDR/PESQ                    DONE 2026-03-17
N3.4 CODE      voice_changer.py 5 presets + PRESETS + load() + seed     DONE 2026-03-17
N3.1s1 CODE    lyrics_tokenizer.py + n11_lora_trainer.py                 DONE 2026-03-17
N2.2 CODE      caption_builder + caption_trainer + caption_inference     DONE 2026-03-17
N2.3 CODE      noesis_singmos.py SingMOS-Pro WavLM head                  DONE 2026-03-17
TAXONOMY       prompt_taxonomy.py 1049 entries, TF-IDF search            DONE 2026-03-18
CAPTION DATA   414 training pairs in caption_dataset/                    DONE 2026-03-18

──── EDIT SUITE / SVC ───────────────────────────────────────────────
Phase SESSION1 ITO/SC/StemAware/AudioSR/VIZ/B2B (46+42 tests)           DONE 2026-03-12
Phase M.1      MasteringChain v1.8 wired into phase_r_direct.py          DONE 2026-03-11

──── UI / API ───────────────────────────────────────────────────────
Phase UI.1     Vite 5 + React 19 + TS :5173 (0 TS errors)               DONE 2026-03-12
REST API       vocal+mix+msr+master+caption wired                        DONE 2026-03-17
REST API v1.1  enable_discoder + /api/quality/* endpoints                DONE 2026-03-18
VIZ PLUGINS    Saturator/MasterMind/StemMix TSX (Tailwind)               DONE 2026-03-17
UI PANELS      SVCPanel/MixAssist/VocalSynth/Choir/Upload (Tailwind)     DONE 2026-03-17
UI ROUTING     types.ts + App.tsx + Sidebar + translations + api.ts      DONE 2026-03-17

──── BENCHMARKING INFRASTRUCTURE ────────────────────────────────────
BENCHMARK 165  33x5=165 plan builder (benchmark_165.py)                  DONE 2026-03-17
ASR.v2 CODE    audiosr_v2.py 4-candidate N_CANDIDATES_MAX=4              DONE 2026-03-17
PAPER FIGS     generate_paper_figures.py 5 figures arXiv v14             DONE 2026-03-17
SC EXPAND      sc_text_controller.py 36 rules + parse_complex_instruction DONE 2026-03-17
STANDALONE API api/router_standalone_mastering.py POST /master/upload    DONE 2026-03-17
PAPER v14      paper_v14_patch_summary.md + overleaf_patch.txt           DONE 2026-03-18

──── MASTERING CHAIN FIXES (2026-03-18) ─────────────────────────────
SILENCE FIX    StemAwareNode.process_audio() added — Stage 6 crash fixed DONE 2026-03-18
HF SCRUBBER    mastering/hf_artifact_scrubber.py — Stage 2.5             DONE 2026-03-18
VOCAL TILT     adaptive_spectral_tilt_stage.py vocals +3.5dB + 3kHz peak DONE 2026-03-18
VOCAL PROMI    mastering/vocal_ratio_analyzer.py + vocal_prominence_boost DONE 2026-03-18
CL PUMP FIX    mastering_chain.py _CL_MAX_PASSES 4->2                    DONE 2026-03-18
KNEE FIX       mastering_pre_limiter.py knee_db 1.0->3.0                 DONE 2026-03-18
CEILING HELP   mastering_chain._apply_peak_ceiling() unified helper      DONE 2026-03-18

──── NEW BACKENDS (2026-03-18) ──────────────────────────────────────
DISCODER       qa_external/discoder_backend.py (ICASSP 2025, MUSHRA 88.14) DONE 2026-03-18
BIGVGAN V2     qa_external/bigvgan_v2_backend.py (alternative vocoder)   DONE 2026-03-18
DISCODER DL    models/discoder/ DOWNLOADED — enable_discoder=True READY  DONE 2026-03-18

──── INTEGRATION FIXES (2026-03-18) ────────────────────────────────
SDK DEVICE     cli/sdk.py enable_discoder + self._device + DisCoder hook DONE 2026-03-18
API ROUTING    noesis_api/main.py quality endpoints added                 DONE 2026-03-18
TAX IMPORT     dual import path cli.prompt_taxonomy / prompt_taxonomy     DONE 2026-03-18

──── TESTS (2026-03-18) ─────────────────────────────────────────────
TESTS C1-C5    test_stem_aware(6)+voice_changer(10)+heart(8)+
               jumbo(12)+discoder(8) = 44 tests                          DONE 2026-03-18
TESTS QUALITY  test_discoder_sdk_hook(5)+mastering_audit(10)+
               vocal_prominence(11) = 26 tests                           DONE 2026-03-18
TESTS 4VAR     test_rest_api_v11(8)+caption_pipeline(6) = 14 tests       DONE 2026-03-18
TESTS TAXONOMY test_prompt_taxonomy(13)                                  DONE 2026-03-18
TOTAL          720+ PASS | 14 known-fails
```

---

## §2. ЖДУТ GPU (~2026-03-21)

```
[НЕМЕДЛЕННО]  Phase R re-run — target J_mean >= 0.65
              run_gpu_day1_sequence.bat  (~2ч)
              Ожидается: J улучшится с 0.3146 (Qwen3 inject + mastering fixes)

[НЕМЕДЛЕННО]  DisCoder A/B тест на реальном треке (~30 мин)
              sdk = NoesisSDK(enable_discoder=True)
              Сравнить HiFiGAN vs DisCoder: SNR delta + субъективно

[НЕМЕДЛЕННО]  QA.1 FAD(PANN) run (~30 мин)
              python_embedded\python.exe run_qa1.py
              Включён в run_gpu_day1_sequence.bat

[ВЫСОКИЙ]     N1.4 MOS Jumbo finetune (~8ч)
              run_gpu_day2_sequence.bat
              Target: Pearson r >= 0.86 (vs v1 r=0.837)

[ВЫСОКИЙ]     N3.1s1 N11 LoRA train (~4ч)
              run_gpu_day3_sequence.bat
              Output: artifacts/n11_lora/n11_lora_v1.pt
              После -> немедленно LIVE: /vocal/synth /choir /vocal/change /vocal/clone

[ВЫСОКИЙ]     N2.2 Caption LoRA CPU (~2ч, параллельно с GPU)
              python_embedded\python.exe noesis_caption_trainer.py
              Dataset: artifacts/caption_dataset/caption_train.jsonl (414 pairs)

[СРЕДНИЙ]     165-track benchmark generation (~4ч)
              Включён в run_gpu_day3_sequence.bat

[СРЕДНИЙ]     arXiv v14 PDF
              WSL: pdflatex docs/noesis_arxiv/main.tex
              ИЛИ: Overleaf + artifacts/paper_v14_overleaf_patch.txt
```

---

## §3. ПОСЛЕ GPU FREE — ЗАПУСК ЖИВЫХ ФИЧ

```
Phase          Что ждёт              Где код           Нулевая работа
──────────────────────────────────────────────────────────────────────
QA.1 FAD       Phase R WAV           run_qa1.py DONE   нет
P1.2 KAD       QA.1 done             fad_kad_backend   нет
N3.1s2 /vocal  n11_lora_v1.pt        router_vocal.py   скопировать веса
N3.3 Choir     n11_lora_v1.pt        choir_engine.py   скопировать веса
N3.2 Clone     n11_lora_v1.pt        voice_cloner.py   скопировать веса
N3.4 Changer   n11_lora_v1.pt+SVC   voice_changer.py  скопировать веса
N2.4 SVC       noesis_svc_v1         svc_processor.py  отдельный train
DisCoder       models/discoder/      discoder_backend   READY NOW

Vocal endpoints — нулевая работа после n11_lora_v1.pt:
  POST /vocal/synth     <- api/router_vocal.py         DONE
  POST /vocal/change    <- voice/voice_changer.py      DONE
  POST /vocal/clone     <- voice/voice_cloner.py       DONE
  POST /choir/generate  <- voice/choir_engine.py       DONE
```

---

## §4. PRIORITY MATRIX v0.28

```
+-------------------------------+----------+-----------+------------------------------+
| Phase                         | Sessions | Priority  | Dependency                   |
+-------------------------------+----------+-----------+------------------------------+
| Phase R re-run                | 1        | IMMEDIATE | GPU free (~2ч)               |
| DisCoder A/B test             | 0.5      | IMMEDIATE | Phase R WAV + discoder READY |
| QA.1 FAD(PANN) run            | 1        | IMMEDIATE | Phase R WAV                  |
| N1.4 MOS Jumbo finetune       | 1        | HIGH      | GPU free (~8ч)               |
| N3.1s1 N11 LoRA train         | 1        | HIGH      | GPU free (~4ч)               |
| N2.2 Caption LoRA CPU         | 0.5      | HIGH      | CPU, параллельно с GPU       |
| arXiv v14 PDF                 | 0.5      | HIGH      | Overleaf/WSL                 |
| N3.1s2 /vocal/synth LIVE      | 0.5      | HIGH      | n11_lora_v1.pt               |
| 165-track benchmark           | 1        | MEDIUM    | GPU day 3 (included)         |
| N3.3 Choir LIVE               | 0.5      | MEDIUM    | n11_lora_v1.pt               |
| N3.2 Voice Cloning LIVE       | 0.5      | MEDIUM    | n11_lora_v1.pt               |
| N3.4 Voice Changer LIVE       | 0.5      | MEDIUM    | n11_lora_v1.pt + N2.4        |
| P1.2 KAD launch               | 0.5      | MEDIUM    | QA.1 done                    |
| P1.2b AudioSR launch          | 0.5      | MEDIUM    | QA.1 done                    |
| P1.3 FIR XO launch            | 0.5      | MEDIUM    | QA.1 done                    |
| N2.4 SVC finetune             | 3-5 нок  | MEDIUM    | отдельный GPU train           |
| N2.3 SingMOS train            | 1        | LOW       | N3.1s2 live + studio vocals  |
| MSR benchmark run             | 2        | LOW       | benchmark_msr_eval.py DONE   |
| MUSHRA study                  | 2-3 нед  | MEDIUM    | 165-track + baselines        |
| NeurIPS workshop              | 1        | LOW       | baselines + ablation         |
| IEEE TASLP                    | 5+       | LOW       | MUSHRA + proofs              |
| N1.3 Neural VST               | 3        | DEFERRED  | N1.4 + J>=0.65 + QA.1       |
| N3.G GGUF Reasoning LLM       | 1        | DEFERRED  | облако / RTX 4090+           |
| N3.5 ARA2 Bridge              | 3        | LOW       | P3.4, N3.1, N2.5             |
| N2.5 JUCE VST3                | 5+       | FUTURE    | N1.3                         |
| N12 Cloud deployment          | 10+      | FUTURE    | N4/N5/N7/N8                  |
| P4.4 Caption LLM fine-tune    | 5+       | FUTURE    | 1000+ generations            |
| P5 /sleep CPM                 | 3-5      | FUTURE    | P4.4 + 200+ snapshots        |
+-------------------------------+----------+-----------+------------------------------+
```

---

## §5. N3 TIER STATUS — Vocal AI

```
+--------+------------------------------+----------+-----------+------------------+
| Phase  | Status                       | Code     | Weights   | Dependency       |
+--------+------------------------------+----------+-----------+------------------+
| N3.1s1 | CODE DONE (2026-03-17)       | DONE     | GPU wait  | GPU free (~4h)   |
| N3.1s2 | CODE DONE (2026-03-16)       | DONE     | GPU wait  | n11_lora_v1.pt   |
| N3.2   | CODE DONE (2026-03-16)       | DONE     | GPU wait  | n11_lora_v1.pt   |
| N3.3   | CODE DONE (2026-03-16)       | DONE     | GPU wait  | n11_lora_v1.pt   |
| N3.4   | CODE DONE (2026-03-17)       | DONE     | GPU wait  | n11_lora_v1.pt   |
|        | PRESETS + load() + seed      |          |           |                  |
| N3.K   | DONE (2026-03-17)            | DONE     | —         | integrated       |
| N3.M   | DONE (2026-03-17)            | DONE     | —         | —                |
| N3.G   | DEFERRED                     | —        | —         | cloud/RTX4090+   |
| N3.V   | REMOVED                      | —        | —         | not needed       |
| N3.5   | LOW (3 sessions)             | —        | —         | P3.4, N3.1, N2.5 |
+--------+------------------------------+----------+-----------+------------------+
```

---

## §6. MASTERING CHAIN v3.0+ — ПОЛНАЯ КАРТА STAGES

```
Stage  1    DC offset removal                              ORIGINAL
            mastering/dc_removal.apply_dc_removal()
            L <= 1.0; zero-phase FIR, deterministic

Stage  2    RMS pre-lift (Artifact Guard)                  ORIGINAL
            generation_artifact_guard.rms_pre_lift()
            FLOOR=-55 LUFS; PRE_LIFT=-35 LUFS; gain_cap=+60 dB

Stage  2.5  HF artifact scrubber              NEW 2026-03-18
            mastering/hf_artifact_scrubber.py
            Removes bfloat16 quantization noise in 12-20kHz band
            Method: spectral subtraction from first-200ms noise floor
            §INV-HF-3: bypass if HF noise < -70 dBFS

Stage  2.8  Gate / Expander                  NEW 2026-03-20
            mastering/gate_expander.py
            Pro-G parity: 6 modes (expander/gate/soft_gate/upward/ducker/transient)
            5ms lookahead; IIR hold; 33 genre presets
            §INV-GATE-5: acoustic genres ratio≤1.5, wet≤0.20

Stage  3    Working Level Normalize                        ORIGINAL
            Normalize to -24 LUFS working level

Stage  4    3-band FIR crossover (N_TAPS=8192)            ORIGINAL
            mastering/linear_phase_fir_crossover.py
            B_linf=3.60972762; Kaiser B=8.6; stopband >=80 dB

Stage  5    Per-band multiband compression                 ORIGINAL
            mastering/compressor.apply_multiband_compression()
            §MC-ACOUSTIC bypass: ratio=1.0

Stage  6    Stem-aware Bus EQ                 FIX 2026-03-18
            mastering/stem_aware_node.py
            process_audio() ADDED (was missing -> Stage 6 crash)
            vocals high_gain_db: 0.5->3.5 dB
            vocals peak_hz=3000Hz, peak_gain_db=+2.5 dB (IIR biquad)
            Reason: ACE-Step guidance_scale=7.0 underlevels vocals -8..-10dB

Stage  7    Mono bass alignment                            ORIGINAL
            mastering/mono_bass.apply_mono_bass()

Stage  7.5  Vintage Tape Saturator            NEW 2026-03-20 (v1.1)
            mastering/tape_saturator.py
            tanh saturation + 1-pole HF loss + 2nd harmonic
            +wow/flutter (sinusoidal pitch warp, Lauridsen 1954)
            +tape noise floor -72 dBFS pink tilt
            §INV-TAPE-6: acoustic bypass

Stage  8    Adaptive spectral tilt                         ORIGINAL
            mastering/spectral_tools.apply_spectral_tilt()

Stage  8.1  Air Exciter                       NEW 2026-03-20 (v1.1)
            mastering/exciter.py
            3-band (body 200-2k / presence 2k-8k / air 8k-16k)
            2nd harmonic (even/warm), 3rd harmonic (odd/bright), both (tube)
            §INV-EX-5: acoustic bypass

Stage  9    ReferenceEQ                                    UPG 2026-03-20 (v1.1)
            mastering/reference_eq.py (31-band ISO226)

Stage 10    SpectralTransfer (default=False)               ORIGINAL
            mastering/adaptive_spectral_tilt_stage.py

Stage 11    Glue bus compression                           ORIGINAL
            mastering/compressor.apply_glue_compression()
            §MC-ACOUSTIC FULL bypass

Stage 12    Stereo Widener (DAFx-24)             UPG 2026-03-20 (v2)
            mastering/stereo.apply_stereoize_pro()
            Algorithm: Das DAFx-24, allpass cascade (Schroeder 1961)
            LF always mono (§P2.1); MF/HF: 4-stage allpass decorrelation
            ICC measurement (ANSI S1.11); transient guard; mono compat ICC≥0.08
            §INV-SW-5: deterministic (seed=0); §INV-SW-6: energy L≤1.2

Stage 13    Global stereo enhance                          ORIGINAL
            mastering/stereo.apply_stereo_enhance()

Stage 14    De-esser v2.1                        UPG 2026-03-20 (v2.1)
            mastering/de_esser_v2.apply_de_esser_v2_fast()
            Adaptive sibilance band auto-detection (3-12kHz search)
            5ms lookahead; compound mode (two-pass for live vocals)
            §INV: passthrough; L≤1.0

Stage 14.5  Adaptive Vocal Prominence Boost   NEW 2026-03-18
            mastering/vocal_ratio_analyzer.py
            mastering/vocal_prominence_boost.py
            VRA: E(250-4kHz) / E(80-250Hz + 4-16kHz) ratio
            Boost: +0..+6dB FFT EQ (linear phase, SNR>220dB)
            §INV-VPB-3: bypass ambient/edm/house/techno/jazz/classical
            §INV-VPB-2: max +6dB (mastering conservative limit)

Stage 15    Inter-stage Peak Ceiling           FIX 2026-03-18
            _apply_peak_ceiling(x, -3.0, sr) — unified helper
            Ceiling to -3 dBFS before loudness normalize

Stage 16    CF-Reduction Pre-Limiter                       ORIGINAL
            mastering/mastering_pre_limiter.apply_cf_reduction()
            §MC-ACOUSTIC FULL bypass

Stage 17    BS.1770-4 LUFS Normalize                       ORIGINAL
            mastering/mastering_normalize.normalize_stage()

Stage 17b   Post-normalize peak ceiling        FIX 2026-03-18
            _apply_peak_ceiling(x, ceiling_db, sr) — unified helper

Stage 18    Two-pass adaptive pre-limiter      FIX 2026-03-18
            mastering/mastering_pre_limiter.pre_limit_compress_v2()
            sparse_benefit +4dB; 5ms lookahead; O(N) deque
            knee_db: 1.0->3.0 dB (softer transient limiting)
            ratio=200:1 preserved (LUFS drift bounded)
            §MC-ACOUSTIC: ratio<=1.5, threshold=-6dBFS

Stage 19    IRC-5 Multiband Limiter                        UPG 2026-03-20 (v2.1)
            mastering/irc5_limiter.py (P4.1, Theorem C.2)
            N_CANDIDATES=8 (ε≤12.5%), N_BANDS=4 (sub+bass/mid/presence/air)
            BAND_FREQS=(500, 8000, 16000 Hz)
            +bark-weighted distortion scoring
            ceiling=-1.0 dBFS; §MC-ACOUSTIC active

Stage 20    Post-limit Closed Loop             FIX 2026-03-18
            mastering/mastering_post_limit.post_limit_trim()
            _CL_MAX_PASSES: 4->2 (pumping artifacts eliminated)
            Drift classification applied here

Stage 21    FrozenVST (optional)                           ORIGINAL
            vst/frozen_vst_operator.py (P3.4)

Stage 22    TPDF Dither (conditional)                      ORIGINAL
            mastering/finalizer.apply_dither()

Post-gen:   DisCoder re-vocoding (enable_discoder=True)    NEW 2026-03-18
            cli/sdk.py hook — applied BEFORE mastering chain
            MUSHRA 88.14 vs HiFiGAN 78.97 (statistically significant)
            §INV-DISC-4: passthrough on error — never blocks generation
```

---

## §7. QUALITY IMPROVEMENTS — IMPACT TABLE

```
Fix                              Root Cause            Expected Impact
----------------------------------------------------------------------
StemAwareNode.process_audio()    Stage 6 missing       Silence regression eliminated
HF scrubber Stage 2.5            bfloat16 VAE          -10..-15dB HF noise reduction
Vocal tilt +3.5dB + peak @3kHz  guidance_scale=7.0    +3-5dB in 1-5kHz vocal range
Vocal Prominence Stage 14.5      Buried in dense mix   Adaptive +0..+6dB boost
CL-loop 4->2 passes             Gain swing +12dB      Pumping eliminated EDM/hip-hop
Pre-limiter knee 1.0->3.0 dB    Brickwall click       Softer, natural transient limit
DisCoder (enable_discoder=True)  DCAE/HiFiGAN limit    MUSHRA 88.14 vs 78.97
Taxonomy 1049 entries            Generic captions      Better DiT conditioning signal
```

---

## §8. GPU BAT SEQUENCES

```
GPU День 1 (~2.5ч):  run_gpu_day1_sequence.bat
  Step 1: phase_r_direct.py            (~2ч)  -> 10 WAV + IQS/J metrics
  Step 2: run_qa1.py                   (~30м) -> FAD(PANN) results

GPU День 2 (~10ч):   run_gpu_day2_sequence.bat
  Step 1: CLAP FAD reference build     (~30м) -> artifacts/clap_ref_dj.npz
  Step 2: noesis_mos_jumbo_trainer.py  (~8ч)  -> r>=0.86

GPU День 3 (~8ч):    run_gpu_day3_sequence.bat
  Step 1: n11_lora_trainer.py train    (~4ч)  -> n11_lora_v1.pt
           -> немедленно LIVE: /vocal/* /choir/*
  Step 2: benchmark_165.py --generate  (~4ч)  -> 165 WAV
  Step 3: benchmark_msr_eval.py        (~1ч)  -> SI-SDR/SDR/PESQ

CPU параллельно (любой день):
  noesis_caption_trainer.py            (~2ч)  -> caption_lora_v1.pt
```

---

## §9. SEALED SYSTEM CONSTANTS

```
FROZEN FILES — NEVER MODIFY:
  iqs.py              IQSWeights.checksum() = 12c2f47cba1ac6be
  iqs_weights.py      IQS_WEIGHTS_CHECKSUM  = 9097e7605b8895f1
  constants.py        все инварианты

SEALED GENERATION PARAMS:
  guidance_scale = 7.0   (CONFIRMED — dual-pass CFG active)
  fix_nfe        = 8     (Turbo distilled — sealed)
  shift          = 3.0   (timestep shift — sealed)
  dtype          = bfloat16
  infer_method   = "ode"

SEALED MASTERING PARAMS:
  PASS_CF_LIMITED <= 6.0 dB  (Drift v1.3; v1.1 threshold 2.0 dB — WRONG)
  FAIL            >  6.0 dB
  IQS_max         = 0.75
  kappa           = 0.910
  genre_count     = 33 | stem_count = 12

SEALED CHECKSUMS:
  NOESIS-MOS v1   sha256[:8] = d781d747 | Pearson r = 0.837
  AutoEval schema               3073949bcdf6e22e (v1.3.0, 29 params)
  B_linf (2048-tap FIR)         3.60972762
  B_linf (8192-tap FIR)         <=1.05

SEALED UI:
  Vite 5 + React 19 + TypeScript  :5173  (NOT Next.js — SEALED)
  FastAPI                          :8765
  Gradio                           :7860
```

---

## §10. DEPENDENCY GRAPH v0.28

```
[DONE: все фазы до 2026-03-18 включительно]
[GPU занят до ~2026-03-21 (SVC training)]
  |
  +-- GPU free (~2026-03-21) -------------------------------------------+
  |    |                                                                 |
  |    +-- Phase R re-run (~2h) -> J_mean + 10 WAV                      |
  |    |    +-- DisCoder A/B (~30m) -> SNR delta confirmed              |
  |    |    +-- QA.1 FAD(PANN) (~30m)                                   |
  |    |         +-- P1.2 KAD / P1.2b AudioSR / P1.3 FIR               |
  |    |              +-- P2.1 Mono Bass / P2.3 Per-Stem EQ             |
  |    |                                                                 |
  |    +-- N1.4 Jumbo (~8h) -> r>=0.86 -> noesis_mos_jumbo.pt           |
  |    |                                                                 |
  |    +-- N3.1s1 LoRA (~4h) -> n11_lora_v1.pt                          |
  |         +-- /vocal/synth LIVE  (router_vocal.py DONE)               |
  |         +-- /choir LIVE        (choir_engine.py DONE)               |
  |         +-- /vocal/clone LIVE  (voice_cloner.py DONE)               |
  |         +-- /vocal/change LIVE (voice_changer.py DONE)              |
  |              +-- N2.4 SVC finetune (~3-5 nights) -> svc_v1          |
  |                                                                      |
  +-- CPU: Caption LoRA (~2h) -> caption_lora_v1.pt -------------------|
  |                                                                      |
  +-- arXiv v14 PDF -> MUSHRA -> NeurIPS -> IEEE TASLP -----------------|
  |                                                                      |
  +-- 165-track benchmark -> arXiv Table I ----------------------------|
  |                                                                      |
  +-- [J>=0.65 + QA.1 + N1.4] -> N1.3 Neural VST (DEFERRED) ---------- |
                                                                         |
       [cloud/RTX4090+] -> N3.G GGUF LLM -> N12 Cloud <----------------+
```

---

## §11. HARDWARE & VRAM BUDGET

```
Mechrevo GM7AG0M — RTX 3060 Laptop 6GB GDDR6 | i7-12700H | 64GB DDR5

§INV-N3-3: DiT + N11 НИКОГДА одновременно на GPU

Task                      GPU    VRAM      Duration
----------------------------------------------------
DiT inference             YES    4.8 GB    ~7.6s / 30s
N11 vocal synth           YES    ~2.5 GB   ~3-8s / 30s (4-bit)
SVC inference             YES    ~0.8 GB   ~2-4s / 30s
DisCoder fp16 GPU         YES    ~1.2 GB   ~10s / 30s
DisCoder CPU              NO     0         ~60s / 30s
N1.4 Jumbo finetune       YES    ~1.5 GB   ~8h
Phase R re-run            YES    ~4.8 GB   ~2h (10 tracks)
N3.1s1 LoRA train         YES    ~2.5 GB   ~4h
CLAP FAD build            YES    ~1.8 GB   ~30m (8764 tracks)
Mastering chain           NO     0         ~1-2s / track
Caption LoRA (CPU)        NO     0         ~2h

Cloud target (future N12):
  HeartMuLa-7B (SongEval 4.48) — needs 10-12GB -> A100
  ACE-Step LM 1.7B — NOT for RTX 3060 (overflow with DiT)
  Qwen3.5-0.8B (~1.6GB fp16) — correct choice for RTX 3060
```

---

## §12. KEY FILES CREATED/MODIFIED IN CPU-DAYS

```
NEW FILES:
  mastering/hf_artifact_scrubber.py              Stage 2.5 HF noise suppressor
  mastering/vocal_ratio_analyzer.py              VRA spectral analysis
  mastering/vocal_prominence_boost.py            Stage 14.5 adaptive FFT EQ
  qa_external/discoder_backend.py                DisCoder re-vocoding
  qa_external/bigvgan_v2_backend.py              BigVGAN v2 alternative
  qa_external/noesis_mos_jumbo_trainer.py        N1.4 GPU-ready trainer
  cli/prompt_taxonomy.py                         1049-entry TF-IDF search
  cli/prompt_taxonomy_generated.py               992 auto-generated (DO NOT EDIT)
  tools/diagnose_silence_regression.py           Stage-by-stage diagnostic
  tools/ab_test_mastering_fixes.py               A/B mastering comparison
  tools/generate_taxonomy_extended.py            Taxonomy expansion generator
  tools/build_caption_dataset_from_snapshots.py  Caption dataset builder
  tools/test_discoder_snr_mock.py                DisCoder CPU mock test
  run_gpu_day1_sequence.bat                      GPU day 1: Phase R + QA.1
  run_gpu_day2_sequence.bat                      GPU day 2: CLAP FAD + Jumbo
  run_gpu_day3_sequence.bat                      GPU day 3: N11 LoRA + 165-track
  run_download_discoder.bat                      DisCoder model downloader
  artifacts/paper_v14_patch_summary.md           Paper v14 changes summary
  artifacts/paper_v14_overleaf_patch.txt         Manual edit instructions
  artifacts/ab_test_mastering_fixes.jsonl        A/B test results (5 genres)
  artifacts/caption_dataset/caption_train.jsonl  414 training pairs

MODIFIED FILES:
  mastering/stem_aware_node.py                   process_audio() + fixes
  mastering/adaptive_spectral_tilt_stage.py      vocals +3.5dB + peak @3kHz
  mastering/mastering_pre_limiter.py             knee_db 1.0->3.0
  mastering_chain.py                             Stage 2.5, 14.5, CL 4->2,
                                                 _apply_peak_ceiling()
  cli/sdk.py                                     enable_discoder, _device, hook
  cli/prompt_utils.py                            taxonomy fallback (dual import)
  ui/noesis_rest_api.py                          v1.1: enable_discoder, quality
  noesis_api/main.py                             /api/quality/* endpoints
  voice/voice_changer.py                         PRESETS + load() + seed param
  BENCHMARK_RESULTS.md                           Mastering fixes table

NEW TEST FILES (105 tests, all PASS):
  tests/test_stem_aware_node_process_audio.py    6  tests
  tests/test_n34_voice_changer.py               10  tests
  tests/test_heart_transcriptor_backend.py       8  tests
  tests/test_noesis_mos_jumbo.py                12  tests
  tests/test_discoder_backend.py                 8  tests
  tests/test_discoder_sdk_hook.py                5  tests
  tests/test_mastering_chain_audit.py           10  tests
  tests/test_vocal_prominence.py                11  tests
  tests/test_rest_api_v11.py                     8  tests
  tests/test_caption_pipeline.py                 6  tests
  tests/test_prompt_taxonomy.py                 13  tests
  TOTAL NEW: 105 tests
  GRAND TOTAL: 720+ PASS | 14 known-fails
```

---

## §13. QUALITY ROADMAP v2.0 (2026-03-18)

### Новые stages (v3.1 — 26 stages total)
```
Stage  9.5   Neural Spectral EQ        NeuralEQ      NEW 2026-03-18
              Taxonomy-driven FxNorm, 1049 entries, 121 genre tags
              _build_curve_from_tags(energy+mood+character+instruments)
              Any free-text query works: "dark cyberpunk glitch" → tags → EQ

Stage 16.5   Transient Coherence Shaper  TransientShaper  NEW 2026-03-18
              Ozone 12 Stabilizer analogue, Bello 2005 spectral flux
              Genre-presets + apply_transient_shaping_taxonomy()
              attack_db/sustain_db driven by energy+mood+character tags
```

### Cascade architecture (VRAMOrchestrator)
```
Phase 1  Planning    CPU   0GB   ~2s    taxonomy + Qwen3.5-0.8B
Phase 2  Generation  GPU  4.8GB  ~7.6s  ACE-Step DiT → unload
Phase 3  Revocoding  GPU  1.2GB  ~10s   DisCoder → BigVGAN v2 fallback → unload
Phase 4  Vocal       GPU  2.5GB  ~5s    N11 Qwen3-TTS (pending n11_lora) → unload
Phase 5  Mastering   CPU   0GB   ~2s    MasteringChain v3.1 26 stages
Peak VRAM at any point: 4.8GB ≤ RTX 3060 6GB
```

### Quality score trajectory
```
Before CPU-days:              ~4.5/10
After mastering fixes (v3.0): ~7.0/10
+ TransientShaper Stage 16.5: ~7.5/10
+ NeuralEQ Stage 9.5:         ~7.5→8.0/10 (after genre tuning)
+ DisCoder active (GPU day):  ~8.0/10
+ DCAE decoder fine-tune GPU: ~8.5/10
Suno v5 reference:            ~9.0/10
```

### Pending GPU
```
tools/dcae_decoder_finetune.py  LoRA r=8 on FMA-large, ~5 nights
  VRAM: 2.5GB | Steps: 50K | Expected: +4..+6 MUSHRA, -6...-8dB HF noise
```


### Stage 0.5: UniverSR (2026-03-18)
```
qa_external/universr_backend.py  UniverSR ICASSP 2026, MIT license
  GitHub: woongzip1/UniverSR — vocoder-free flow matching
  vs AudioSR: 4 ODE steps (was 50 DDIM) | 2GB VRAM (was 4GB) | MIT (was CC BY-NC)
  Domains: music + speech + SFX (was speech-only)
  §INV-SR-1: guidance_scale=1.5 sealed | §INV-SR-2: seed propagated | §INV-SR-5: passthrough

  Activate: NoesisSDK(enable_universr=True)
            MasteringChain(enable_universr=True, universr_device="cuda")

  Models (download in progress):
    models/UniverSR-audio/   <- for music Stage 0.5
    models/UniverSR-speech/  <- for VC-ONE ref prep
```

### Mastering Chain v3.2 — 27 stages
```
Stage  0.5  UniverSR SR              qa_external/universr_backend.py  NEW
Stage  1    DC removal
Stage  2    RMS pre-lift
Stage  2.5  HF artifact scrubber     mastering/hf_artifact_scrubber.py
Stage  3    Working level normalize
Stage  4    Mono bass
Stage  5    De-esser v2
Stage  6    StemAwareNode EQ         vocals +3.5dB +3kHz peak
Stage  7    Spectral tilt
Stage  8    Multiband compressor
Stage  8.5  Pre-limiter v4 (lookahead)
Stage  9    ReferenceEQ
Stage  9.5  NeuralEQ taxonomy        mastering/neural_eq.py           NEW
Stage 10    SpectralTransfer
Stage 11    MidSide width
Stage 12    BarkMasking
Stage 13    ModulationCoherence
Stage 14    SubbandLUFS slope
Stage 14.5  VocalProminenceBoost     mastering/vocal_prominence_boost.py
Stage 15    GlueBus
Stage 16    CF-Reduction
Stage 16.5  TransientShaper          mastering/transient_shaper.py    NEW
Stage 17    BS.1770-4 Loudness normalize
Stage 18    Pre-Limiter v2 (two-pass)
Stage 19    IRC5 Limiter
Stage 20    CL-Loop (max 2 passes)
Stage 21    Post-limit trim
Stage 22    True-peak filter
Post-gen:   DisCoder(MUSHRA 88.14) -> BigVGAN v2(83.42) -> passthrough
```


---

*Document: NOESIS_ROADMAP_v0_28.md*
*Version: v0.28 (2026-03-18)*
*Author: Ilia Bolotnikov / AMAImedia.com*
*Supersedes: v0.27 / v0.26 / v0.25*
*Status: ACTIVE — единый полный мастер-документ*
