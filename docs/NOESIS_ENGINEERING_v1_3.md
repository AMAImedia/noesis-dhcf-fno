[NOESIS_ENGINEERING_v1_3.md](https://github.com/user-attachments/files/26134365/NOESIS_ENGINEERING_v1_3.md)
"""NOESIS — Deterministic Hybrid Control Framework for Frozen Neural Operators (DHCF-FNO)
Copyright (c) 2026 AMAImedia.com
All rights reserved.
B:/Downloads/Portable/NOESIS_DHCF-FNO/docs/NOESIS_ENGINEERING_v1_3.md"""

# NOESIS ENGINEERING v1.3
## Mastering · LM Pipeline · Edit Suite · SVC · Modules · UI

```
Version:    1.3  (2026-03-21, CPU Day 2)
Author:     Ilia Bolotnikov / AMAImedia.com
Status:     ACTIVE
Supersedes: NOESIS_ENGINEERING_v1_1.md (2026-03-14)

Changes v1.3b (2026-03-21 — CPU Day 2):
  + sc_text_controller.py: 36→84 rules (§13.2 updated)
  + genre_mastering_master_profiles.py: 29→38 profiles (§2.x updated)
  + entries_rare.py: 55 entries, 11 rare genres registered in orchestrator
  + generate_paper_figures.py: 22→31 stages, R.REF2 J values
  + UI: DriftBadge TIER_CF_LIMIT 2.0→6.0, DisCoder toggle, taxonomy preview

Changes v1.3 (2026-03-20):
  + MasteringChain v3.3→v3.4 (29→31 stages) — Ozone 12 10/10 parity
  + §1.2: Stage map updated (Stage 2.8/7.5/8.1/9.5/9.6/12/14/16.5)
  + §1.5: Lipschitz bounds updated for new modules
  + §1.6: Industry equivalences updated (DAFx-24, Pro-Q4, Pro-G)
  + §LLM: Новый раздел — статус Qwen-моделей в проекте
  + compressor v2.0, irc5 v2.1 N=8/4-band, reference_eq ERB
  + n4_macro_planner.py BUG: local_files_only=True добавлен

Changes v1.2 (2026-03-18):
  + Stage 2.5 HF artifact scrubber (bfloat16 quantization noise)
  + Stage 6 StemAwareNode.process_audio() FIXED (was missing)
  + Stage 6 vocals tilt: 0.5->3.5 dB + peak +2.5 dB @ 3kHz
  + Stage 14.5 Adaptive Vocal Prominence Boost
  + Stage 18 pre-limiter soft-knee: 1.0->3.0 dB
  + Stage 20 CL-loop max_passes: 4->2 (pumping eliminated)
  + Stage 15/17b unified _apply_peak_ceiling() helper
  + §1.9 DisCoder post-generation re-vocoding (MUSHRA 88.14)
  + Prompt taxonomy: 1049 entries, TF-IDF (cli/prompt_taxonomy.py)
  + SDK: enable_discoder, self._device, DisCoder hook
  + REST API v1.1: enable_discoder, /api/quality/* endpoints
            NOESIS_MASTERING_AND_STEM_SPEC_v1_1.md  (merged)
            NOESIS_LM_PIPELINE_ADDENDUM_v1_2.md     (merged)
            NOESIS_EDIT_SUITE_v1_1.md               (merged)
            NOESIS_PLUGIN_REFERENCE.md              (merged)
            NOESIS_MODULE_AND_MIGRATION_v1_3.md     (merged)
            NOESIS_UI_FROZEN_SPEC_v1_0.md           (merged)
            NOESIS_INTEGRATION_ARCHITECTURE_v1_3.md (merged)
```

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
§1. MASTERING PIPELINE — MasteringChain v3.4 (31 Stages)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

§1.1 — Two-Phase Architecture

  M(x) = T ∘ L_φ ∘ N(x)

  M_fixed(x)   = L_φ ∘ N(x)          [Theorem B.16(ii)]  L_M ≤ B_linf·G_max = 14.44
  M_adaptive(x) = L_{φ(x)} ∘ N(x)   [Theorem B.16-A]    piecewise-Lipschitz
  T            = TPDF Dither          optional post-processing

  Signal Plane (IMMUTABLE):
    DiT 2.4B    frozen, bfloat16, fix_nfe=8, guidance_scale=7.0
    LM 662M     frozen, Qwen3-embedding conditioned
    VAE         frozen, Oobleck decoder
  Stage 11 (hard clamp): float32 ceiling clamp for WAV export ONLY

§1.2 — Canonical Stage Order (MasteringChain v3.4)

  REORDER RATIONALE v3.0:
    DC first (Stage 1 before RMS): DC offset affects RMS measurement → fix first
    CF before Loudness (Stage 16→17): CF changes dynamics → normalize after
    Multiband Width before Global (Stage 12→13): band-level → global = stable
    AudioSR removed from chain (external standalone tool)

  Stage 1    DC offset removal
             mastering/dc_removal.apply_dc_removal()
             L ≤ 1.0; zero-phase FIR notch, deterministic
             REASON v3.0: DC affects RMS measurement → must be first

  Stage 2    RMS pre-lift (Artifact Guard)
             generation_artifact_guard.rms_pre_lift()
             ARTIFACT_FLOOR = −55 LUFS; PRE_LIFT_TARGET = −35 LUFS; gain_cap = +60 dB
             L ≤ 1000 (bounded by gain_cap)

  Stage 2.5  HF artifact scrubber                        [NEW 2026-03-18]
             mastering/hf_artifact_scrubber.py
             Removes bfloat16 quantization noise in 12-20kHz band
             Method: spectral subtraction, noise floor from first 200ms
             §INV-HF-3: bypass if HF noise < -70 dBFS
             Deterministic (§EC8): no randomness, pure numpy FFT

  Stage 3    Working Level Normalize
             Normalize to −24 LUFS working level (stable reference for subsequent stages)

  Stage 4    3-band FIR crossover
             mastering/linear_phase_fir_crossover.py
             kernel_size=2048, Kaiser β=8.6, stopband ≥ 80 dB
             Frame bounds: A_f=0.51969725, B_f=1.40730381, B_linf=3.60972762
             L_crossover ≤ B_f = 1.4073 (Theorem B.15)

  Stage 5    Per-band multiband compression
             mastering/compressor.apply_multiband_compression()
             §MC-ACOUSTIC bypass: ratio=1.0 for acoustic genres (keep crossover split)

  Stage 6    Stem-aware Bus EQ + Vocal Prominence               [FIX 2026-03-18]
             mastering/stem_aware_node.py (P2.1 + P2.3)
             12 genre overrides; 8-band biquad cascade per band; L_EQ ≤ 2.0
             All IIR poles < 1.0 (assert at __init__)
             process_audio(audio, sr, genre) — ADDED (was missing)
             _STEM_TILT_TARGETS["vocals"]:
               high_gain_db: 0.5 -> 3.5 dB                    [FIX]
               peak_hz: 3000.0 Hz, peak_gain_db: +2.5 dB     [NEW]

  Stage 7    Mono bass alignment
             mastering/mono_bass.apply_mono_bass()
             FFT-phase, f < 120 Hz; energy-conserving (Theorem C.3)

  Stage 7.5  Vintage Tape Saturator                  [NEW 2026-03-20 v1.1]
             mastering/tape_saturator.py
             tanh(x·drive)/tanh(drive) + 1-pole HF loss + h2=x²·sign(x)
             +wow/flutter: sinusoidal time-warp (wow=0.5Hz, flutter=6Hz)
             +tape noise floor: -72 dBFS pink-tilt (-3dB/oct above 200Hz)
             §INV-TAPE-2: RMS delta≤1.5dB; §INV-TAPE-6: acoustic bypass

  Stage 8    Adaptive spectral tilt
             mastering/spectral_tools.apply_spectral_tilt()
             genre-aware; slope ≈ −0.4 dB/oct max; L_tilt ≈ 17.89 (dominant)
             NOTE: enable_spectral_transfer=False in diagnose_mastering selftests (§EC17)

  Stage 8.1  Air Exciter                              [NEW 2026-03-20 v1.1]
             mastering/exciter.py
             3-band multiband: body(200-2kHz)/presence(2k-8kHz)/air(8k-16kHz)
             h2=x²·sign(x) (even/warm), h3=x³ (odd/bright), blend 60%/40% (both/tube)
             §INV-EX-2: air-band boost ≤+3dB; §INV-EX-5: acoustic bypass

  Stage 9    ReferenceEQ (Ozone Match equivalent)         [UPG 2026-03-20 v1.1]
             mastering/reference_eq.py
             31-band ISO 226 + ERB smoothing for genre profiles
             WAV reference: ReferenceEQHighRes (96-band, 1/12-oct, ERB smoothed)

  Stage 9.5  Neural EQ dynamic                       [UPG 2026-03-20 v1.1]
             mastering/neural_eq.py
             FxNorm-style taxonomy-driven; prompt_taxonomy TF-IDF (1049 entries)
             OLA frames 50ms; IIR adaptive strength att=20ms/rel=100ms
             §INV-NEQ-D2: ±6dB; §INV-NEQ-D3: passthrough

  Stage 9.6  Dynamic EQ temporal                          [NEW 2026-03-20]
             mastering/dynamic_eq.py
             5 genre-aware bands + Spectral Dynamics air (per-bin trigger, Pro-Q4)
             OLA 30ms frames; IIR att=10ms/rel=100ms
             §INV-TDEQ-2: [-18,+6]dB; §INV-TDEQ-3: passthrough

  Stage 10   SpectralTransfer
             mastering/adaptive_spectral_tilt_stage.py
             DeepAFx-ST DSP; disabled in selftests (§EC17)

  Stage 11   Glue bus compression                        [UPG 2026-03-20 v2.0]
             mastering/compressor.apply_glue_compression()
             v2.0: Blend detection + 2ms lookahead + CF auto-release
             GlueBus v2.0; 1176-style; L ≤ 2.0
             §MC-ACOUSTIC bypass: FULL bypass for acoustic genres

  Stage 12   Stereo Widener (DAFx-24)                    [UPG 2026-03-20 v2]
             mastering/stereo.apply_stereoize_pro()
             Algorithm: Das DAFx-24, allpass cascade (Schroeder 1961)
             LF (≤120Hz) always mono; MF+HF: 4-stage allpass decorrelation
             ICC (ANSI S1.11); transient guard; mono compat ICC≥0.08
             §INV-SW-5: deterministic (seed=0); §INV-SW-6: L≤1.2

  Stage 13   Global stereo enhance
             mastering/stereo.apply_stereo_enhance()

  Stage 14   De-esser v2.1                                [UPG 2026-03-20]
             mastering/de_esser_v2.apply_de_esser_v2_fast()
             Adaptive sibilance band: auto-detects peak in 3–12kHz (1/3-oct search)
             5ms lookahead; compound mode (two-pass: mild 6dB + harsh 12dB)
             L≤1.0 (energy only decreases); §INV: passthrough

  Stage 14.5 Adaptive Vocal Prominence Boost            [NEW 2026-03-18]
             mastering/vocal_ratio_analyzer.py + mastering/vocal_prominence_boost.py
             Spectral band energy ratio analysis (250Hz-4kHz vs bass+HF)
             Boost: conservative +0..+6dB if vocal is buried
             §INV-VPB-3: bypass for ambient/edm/house/techno/jazz/classical

  Stage 15   Inter-stage Peak Ceiling (unified _apply_peak_ceiling) [FIX 2026-03-18]
             Soft-limit to −3 dBFS before loudness norm (safety headroom)

  Stage 16   CF-Reduction Pre-Limiter
             mastering/mastering_pre_limiter.apply_cf_reduction()
             Enabled: cf_reduction_enabled=true per genre in mastering_config.yaml
             §MC-ACOUSTIC bypass: FULL bypass (CF=17 is natural dynamics)

             Tier assignment:
               AGGRESSIVE (metal/neurofunk/hardstyle/dubstep): ratio 8:1, attack 0.5ms
               HOT (edm/trap/drill/phonk/dnb/house/techno):    ratio 6:1, attack 1ms
               STANDARD (hip-hop/pop/rock/soul/rnb/funk):      ratio 4:1, attack 2ms
               SPARSE (ambient/jazz/classical/lofi/folk/...):  cf_reduction_enabled=false

             REASON v3.0 before Stage 17: CF changes dynamics → normalize after

  Stage 17   BS.1770-4 LUFS Normalize
             mastering/mastering_normalize.normalize_stage()
             Closed-loop ≤ 8 passes; target from §6 GENRE_LUFS_TARGETS

  Stage 18   Two-pass adaptive pre-limiter                     [FIX 2026-03-18]
             mastering/mastering_pre_limiter.pre_limit_compress_v2()
             sparse_benefit +4 dB; 5ms lookahead; O(N) deque; L ≤ 2.0
             soft-knee: 1.0 -> 3.0 dB (reduces audible clamping click)
             ratio=200:1 preserved (LUFS drift bounded)
             §MC-ACOUSTIC reduced: ratio ≤ 1.5:1, threshold = −6 dBFS,
             density detection disabled

  Stage 19   IRC-5 Multiband Limiter                      [UPG 2026-03-20 v2.1]
             mastering/irc5_limiter.py (P4.1, Theorem C.2)
             N_CANDIDATES=8 (ε≤12.5%); N_BANDS=4 (sub+bass/mid/presence/air)
             BAND_FREQS=(500, 8000, 16000 Hz)
             +bark-weighted distortion psychoacoustic scoring
             ceiling = −1.0 dBFS; §MC-ACOUSTIC: UNCHANGED

  Stage 20   Post-limit Closed Loop LUFS Trim                  [FIX 2026-03-18]
             mastering/mastering_post_limit.post_limit_trim()
             Drift classification applied here (PROTOCOL §4)
             _CL_MAX_PASSES: 4 -> 2 (eliminates pumping artifacts)

  Stage 21   FrozenVST (optional)
             vst/frozen_vst_operator.py (P3.4)
             plugin.reset() before every process() call (Invariant #22)
             Enabled: vst_enabled=true in mastering_config.yaml

  Stage 22   TPDF Dither (conditional — see §1.4)
             mastering/finalizer.apply_dither()
             _dither_seed = seed ^ 0xD17HEEED (determinism, Invariant)
             MUST be AFTER LUFS measurement and drift classification

  [Post-chain]: Safety Hard-Clamp ±C_M = ±0.8912 in mastering_chain.py
               Applied AFTER LUFS measurement; drift uses pre-clamp value.

§1.3 — §MC-ACOUSTIC Bypass (v3.0)

  _ACOUSTIC_GENRES = frozenset({
    "ambient", "ambient2", "classical", "jazz", "folk", "lofi", "lo-fi",
    "acoustic", "new age", "drone", "meditation"
  })

  Detection: _is_acoustic = genre.lower().strip() in _ACOUSTIC_GENRES

  Bypassed stages for acoustic:
    Stage  5 Multiband comp  → ratio=1.0 (no gain reduction; keep crossover)
    Stage 11 Glue bus        → FULL bypass (return audio unchanged)
    Stage 16 CF Reduction    → FULL bypass (CF=17 is natural dynamics)
  Reduced:
    Stage 18 Pre-limiter     → ratio ≤ 1.5:1, threshold = −6 dBFS,
                               density detection disabled
  Unchanged:
    Stage 19 IRC-5 Limiter   → ceiling=−1.0 dBFS (NEVER modify, all genres)

  Rationale: Sub-dominant material (sub_energy/total > 0.7) incompatible
  with density-based compression. 4× compression stages: kurtosis 407 → <20,
  SNR −4.32 dB → >15 dB after bypass. Tests: kurtosis < 20 (PASS),
  TP < −0.5 dBFS (PASS).

§1.4 — Dither Gate (5 conditions — see PROTOCOL §8)

§1.5 — Frame Bounds and Lipschitz Chain

  Lipschitz signal flow:
    Stage 2  RMS pre-lift:   L ≤ 60 dB gain_cap (bounded)
    Stage 4  FIR crossover:  L ≤ B_f = 1.4073
    Stage 8  SpectralTilt:   L ≈ 17.89 (DOMINANT TERM — DC bin gain ×9.47)
    Stage 11 GlueBus:        L ≤ 2.0
    Stage 12 MidSide:        L ≤ 1.5
    Stage 14 De-esser:       L ≤ 1.0 (downward gain only)
    N(x) total:              L_N ≈ 17.89 (measured, not product of per-stage)
    Stage 12 Stereo:         L ≤ 1.2 (§INV-SW-6, energy Lipschitz)
    Stage 19 IRC-5:          L_core = 1.0214 empirical, C_M ≤ 1.0, G_max = 4.0
    L_M_fixed_theory = B_linf × G_max = 3.60972762 × 4.0 = 14.44
    After P1.3: L_M_fixed ≤ 1.05 × 4.0 = 4.2

§1.6 — Industry Plugin Equivalence (Reference Only)

  Stage  1 DC removal:          VPS Scope DC filter
  Stage  4 FIR crossover:       FabFilter Pro-MB / iZotope Ozone 12 Dynamics
  Stage  6 Stem EQ:             Ableton stem bus + 8-band EQ
  Stage  7 Mono bass:           Nugen Monofilter 4
  Stage  8 SpectralTilt:        Eiosis AirEQ (tilt shelf)
  Stage  9 ReferenceEQ:         iZotope Ozone Match EQ
  Stage 11 GlueBus:             UA 1176 / iZotope Ozone Dynamics
  Stage 12 Stereo widener:       Das DAFx-24 / iZotope Ozone Imager v2 / Softube Wider
  Stage  2.8 Gate/Expander:      FabFilter Pro-G / iZotope Ozone Gate / SSL X-Gate
  Stage  7.5 Tape Saturator:      iZotope Ozone Vintage Tape
  Stage  8.1 Air Exciter:         iZotope Ozone Exciter (Warm/Retro/Tube modes)
  Stage  9.5 Neural EQ:           iZotope Ozone Neural EQ (dynamic mode)
  Stage  9.6 Dynamic EQ:          FabFilter Pro-Q 4 Dynamic + Spectral Dynamics
  Stage 12 Stereo widener:        Das DAFx-24 / iZotope Ozone Imager v2 / Softube Wider
  Stage 14 De-esser v2.1:         FabFilter Pro-DS
  Stage 17 LUFS normalize:      Ozone RX Loudness (target mode)
  Stage 18 Pre-limiter:         FabFilter Pro-L2 (5ms lookahead)
  Stage 19 IRC-5 Limiter:       iZotope IRC5 (Ozone 12) / AOM Invisible Limiter G2
  Stage 22 TPDF Dither:         iZotope RX Dither

  FUNDAMENTAL INVARIANT: no stage raises peak level above its input.
  Formalization: Theorem B.16: L_M ≤ B_linf · G_max.

§1.7 — OVERRIDES_DIR Gate (Invariant #16, sealed 2026-03-13)

  All AGMS lookups gated behind:
    if self._overrides_dir and genre:
        # load YAML overrides
    else:
        # use constructor defaults: LUFS=ctor, char=0.5, upward=0.0, dr_select=off

  Without overrides_dir: FileNotFoundError prevented.
  With overrides_dir: LUFS/char/dr_select from YAML → profiles → ctor.

§1.8 — True-Peak Detection (P1.1, BS.1770-4)

  Implementation: polyphase Kaiser FIR
    L=4 phases, N_taps=128, β=12.0
    x_up = upfirdn(h_kaiser, x, up=4, down=1)
    TP_dBTP = 20·log10(max|x_up|)
    Ceiling: TP < −0.1 dBTP (Invariant #6)

§1.9 — Post-Generation Re-Vocoding (DisCoder)           [NEW 2026-03-18]

  Optional quality enhancement stage applied BEFORE mastering chain.
  Activated via: sdk = NoesisSDK(enable_discoder=True)
              or: generate_audio(..., enable_discoder=True)

  Pipeline:
    AceStepBackend.generate() -> WAV (HiFiGAN vocoder)
    -> DisCoderBackend.enhance(audio, sr) -> HD WAV (DisCoder)
    -> mastering chain

  DisCoder (ETH Zurich, ICASSP 2025):
    Architecture: ConvNeXt encoder (DAC-aligned) + fine-tuned DAC decoder
    Quality:      MUSHRA 88.14 vs HiFiGAN 78.97 (statistically significant)
    Size:         ~430M params, ~1.7GB
    VRAM:         ~1.2GB fp16 GPU | ~0 CPU (CPU mode: ~60s/30s track)

  §INV-DISC-4: passthrough on any error
  §INV-DISC-2: eval mode, no gradients, deterministic

  Key files:
    qa_external/discoder_backend.py     DisCoderBackend class
    qa_external/bigvgan_v2_backend.py   alternative BigVGAN v2
    cli/sdk.py                          enable_discoder hook

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
§2. LM AUDIO PIPELINE
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

§2.1 — GGUF-Verified Pipeline (Source of Truth)

  Phase 1 (LM): CoT → BPM, Key, Lyrics
    Model:  acestep-5Hz-lm-0.6B (Qwen3ForCausalLM, vocab=217,204)
    Output: structural metadata (text tokens)

  Phase 2 (LM): Audio Codes (5Hz FSQ tokens — MUSICAL BLUEPRINT)
    Model:  same acestep-5Hz-lm-0.6B
    Output: FSQ indices, shape [1, T_5hz] where T_5hz = duration × 5
    Codebook: ResidualFSQ 8×8×8×5×5×5 = 64,000 entries
    Token offset: 151,669 (<|audio_code_0|> in vocab)

  FSQ detokenization:
    get_output_from_indices(audio_codes) → lm_hints_5Hz
    detokenize(lm_hints_5Hz) → lm_hints_25Hz  (5Hz → 25Hz upscale)

  is_covers gate:
    is_covers=1 → DiT uses LM musical blueprint as conditioning
    is_covers=0 → DiT generates without structural guidance (CURRENT DEFAULT)

  DiT diffusion:
    guidance_scale=7.0 (CFG dual-pass: vt = vt_uncond + 7.0*(vt_cond - vt_uncond))
    fix_nfe=8, bfloat16, offload_dit_to_cpu=True

  VAE decode:
    AutoencoderOobleck → raw audio float32 48kHz
    Known error: AutoencoderOobleck.__init__() unexpected kwarg 'dtype' → offline fallback

§2.2 — Current Implementation Status

  Implemented:
    cli/lm_encoder.py   NoesisLM_HF.generate_codes()
                        temp=1.0, top_p=0.95; tokens → FSQ indices (offset 151,669)
    cli/sdk.py          lm_tokenizer loaded at init (AutoTokenizer from lm dir)
    modeling_noesis_v15_turbo.py  refactored 2285→110 lines orchestrator
                        Sub-modules: modeling_dit_v15.py, modeling_encoders_v15.py,
                        modeling_attention_v15.py, modeling_shims_v15.py,
                        modeling_generation_v15.py

  Disabled (pending Phase L):
    LM warm-up: is_covers=0; structural tokens not yet injected
    Caption optimizer: rule-based captions only (not LoRA-guided)
    is_covers=1 path: available in code, not used in production

§2.3 — Adapter Layer

  adapter/adapter_types.py — GenerationRequest dataclass:
    seed: int
    duration: float
    caption: str
    genre: str = ""
    lyrics: str = "[instrumental]"
    guidance_scale: float = 7.0   # SEALED
    inference_steps: int = 8      # SEALED
    shift: float = 3.0            # SEALED, SEPARATE from guidance_scale
    infer_method: str = "ode"     # SEALED
    audio_prompt: Optional[np.ndarray] = None     # Edit Suite E1
    audio_prompt_sr: int = 44100                  # Edit Suite E1
    audio_prompt_strength: float = 0.7            # [0.0,1.0], INV-E1-1

  adapter/adapter_call.py:
    def call_handler(handler, req):
        torch.manual_seed(req.seed)   # 3x RNG lock (mandatory)
        np.random.seed(req.seed)
        random.seed(req.seed)
        result = handler.generate_music(
            guidance_scale=req.guidance_scale,  # 7.0 — dual-pass CFG
            shift=req.shift,                    # 3.0 — SEPARATE param
            inference_steps=req.inference_steps,
            audio_prompt=req.audio_prompt,      # Edit Suite: None or np.ndarray
            ref_audio_strength=req.audio_prompt_strength,
            ...
        )
        return result["audios"][0]["tensor"].cpu().numpy().astype(np.float32)

§2.4 — Known Audio Quality Limitations

  Spectral profile (baseline, ACE-Step v1.5 Turbo):
    Spectral centroid: 4000–10000 Hz (actual) vs targets 1600–3000 Hz
    Flatness range: 0.033–0.214 (R.REF3 confirmed music, not noise)
    IQS centroid fix: linear score = 1 − |centroid_hz − target| / 10000
    (Gaussian sigma fix prevents circular r=0.899 synthetic correlation)

  VRAM budget (RTX 3060 Laptop 6GB):
    DiT 2.4B bfloat16: ~4.80 GB (offloaded after inference)
    LM 662M:           ~1.30 GB (always CPU)
    VAE Oobleck:       ~0.30 GB (CPU → GPU for decode)
    Qwen3-Emb 0.6B:    ~0.60 GB (always CPU)
    Total > 6GB → offload_dit_to_cpu=True REQUIRED

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
§3. EDIT SUITE (E1–E5) — edit_suite.py
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

  Status: ALL DONE (2026-03-14). 68 tests PASS.
  Implementation: edit_suite.py, voice_cloning.py (→ SVC §4), mashup.py

§3.1 — E1: Extend (Audio Continuation)

  edit_suite.py::extend(audio_np, sr, duration_add, genre, prompt, seed)
  Mechanism: ACE-Step audio_prompt conditioning
  Invariants:
    INV-E1-1: audio_prompt_strength ∈ [0.0, 1.0] (ValueError)
    INV-E1-2: output SR = 44100 Hz
    INV-E1-3: deterministic (3x RNG lock preserved)
    INV-E1-4: LUFS drift ≤ 1.0 dB from source
    INV-E1-5: max track 480s (ACE-Step limit)
  Default strength: 0.7 (strong continuation reference)

§3.2 — E2: Edit Instruments

  edit_suite.py::edit_instruments(audio_np, sr, genre, keep_stems, prompt, seed)
  Mechanism: HTDemucs 12-band → stem swap → re-mix
  keep_stems: list of stem names to preserve from original
  New stems generated via ACE-Step conditioned on remaining prompt

§3.3 — E3: Edit Vocals

  edit_suite.py::edit_vocals(audio_np, sr, genre, prompt, seed)
  Mechanism: HTDemucs vocal stem → N11 vocal backbone processing
  Note: For vocal TIMBRAL cloning, use §4 SVC (E6), not E3.
  E3 edits vocal style/quality; E6 converts singer identity.

§3.4 — E4: Cover and Remix

  edit_suite.py::cover(audio_np, sr, genre, prompt, seed, strength=0.85)
  edit_suite.py::remix(audio_np, sr, genre, prompt, seed, strength=0.50)
  Mechanism: ITO (style transfer reference) + ACE-Step audio conditioning
  Invariants:
    INV-E4-1: cover strength ∈ [0.70, 0.95]
    INV-E4-2: remix strength ∈ [0.30, 0.70]
  Difference: cover → strong ref (preserves melody); remix → moderate ref (genre shift)

§3.5 — E5: Speed / Tempo Change

  edit_suite.py::speed(audio_np, sr, ratio, seed=None)
  Mechanism: librosa time-stretch (TSM, phase vocoder)
  Invariants:
    INV-E5-1: tempo_ratio ∈ [0.6, 1.4] (ValueError)
    INV-E5-2: per-channel (no mono-mix before stretch)
    INV-E5-3: LUFS normalize after speed()
  Note: pitch is preserved (time-stretch, not pitch-shift)

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
§4. SINGING VOICE CONVERSION (E6 — SVC) — singing_voice_cloner.py
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

  CRITICAL DEFINITION: E6 is SVC (Singing Voice Conversion), NOT TTS.
    TTS: produces speech from text. Models: CosyVoice2, Qwen3-TTS, OpenVoice v2.
    SVC: converts AI-generated vocals to sound like singer X while preserving melody.
    CosyVoice2/OpenVoice v2 EXCLUDED from E6 — they cannot sing melodies.

  VERSION ROADMAP:
    v1.0 (CURRENT):  Seed-VC fine-tuned on 200GB studio vocals = NOESIS model.
                     Conditioning: CAM++ timbre vector only.
    v2.0 (FUTURE):   Qwen3-Omni Audio Encoder as "smart ear" + Qwen3.5-0.8B as "brain".
                     Richer conditioning: style/emotion/vibrato/energy, not just timbre.
                     DO NOT mix v1.0 and v2.0 components — separate phases.

§4.1 — NOESIS-SVC v1.0 Pipeline (5 Steps)

  Step 1: ACE-Step Turbo → song.wav (DiT 2.4B, ~4.8 GB VRAM, offload after)
  Step 2: HTDemucs → vocals.wav + instrumental.wav (CPU, stem_separator.py)
  Step 3: Seed-VC + F0 → cloned_vocals.wav (~0.8 GB VRAM fp16)
  Step 4: librosa mix → cloned_vocals + instrumental (CPU)
  Step 5: MasteringChain v3.0 → final.wav (CPU)

  VRAM strategy (RTX 3060 6GB — SEQUENTIAL REQUIRED):
    DiT and Seed-VC CANNOT coexist in VRAM simultaneously.
    1. generate(prompt) → ACE-Step in VRAM → song.wav → ACE-Step to CPU
    2. separate(song.wav) → HTDemucs CPU → vocals.wav + instr.wav
    3. convert(vocals, ref) → Seed-VC in VRAM → cloned.wav → Seed-VC to CPU
    4. mix + master → CPU only → final.wav

§4.2 — NOESIS-SVC v1.0 Architecture (CURRENT)

  Base: Seed-VC fine-tuned on 200GB studio vocals = NOESIS proprietary model.
  GitHub: github.com/Plachtaa/seed-vc (base architecture)
  Type: Zero-shot SVC with F0 conditioning
  Fine-tuned on: 200GB studio vocals + 8,764 DJ stems → our competitive advantage.

  5 internal blocks (v1.0):
    Block 1: CAM++ Speaker Encoder  (22 MB, CPU)
             ref.wav (5-30s) → timbre vector ℝ^512
             Understands: WHO sings (timbral identity)
             Does NOT understand: style, emotion, vibrato, energy contour

    Block 2: Whisper-base Encoder   (74 MB, GPU)
             Extracts WHAT is sung (phoneme content + melody), not WHO

    Block 3: RMVPE F0 Extractor     (~5 MB, CPU)
             Extracts note-by-note pitch curve (F0 trajectory)

    Block 4: DiT Flow Backbone      (~200 MB, GPU)  ← MAIN CONVERSION
             [timbre_vec ℝ^512] + [content] + [F0] → converted voice latents

    Block 5: BigVGAN Vocoder        (~100 MB, GPU)
             mel-spectrogram → WAV (400× faster than real-time)

  TOTAL VRAM (fp16 inference): ~0.8 GB.

  v1.0 LIMITATION: CAM++ encodes TIMBRE only.
    It does NOT capture: singing style, emotional delivery, vibrato pattern,
    diction clarity, energy contour, breath control dynamics.
    Result: "sounds like singer X, but in NOESIS style" rather than
    "sounds exactly like singer X performing this song."

§4.3 — NOESIS-SVC v2.0 Architecture (FUTURE — Phase SVC_v2)

  Core insight: replace CAM++ (22MB, timbre-only) with a two-component
  "smart ear + brain" system that understands STYLE, not just timbral identity.

  Component 1: Qwen3-Omni Audio Encoder (0.6B, FROZEN) — "Smart Ear"
    Role: understands SINGING STYLE from reference audio
    Captures: emotion (happy/melancholic/aggressive), energy contour (soft→loud),
              vibrato depth and rate, diction clarity, breath patterns,
              phrasing nuance — everything beyond raw timbre
    Size:     0.6B params, ~1.2 GB fp16
    Usage:    Qwen3-Omni audio encoder frozen; ONLY the encoder, NOT the full model
    Output:   768-dim rich audio embedding (per-segment, ~2-5s windows)

  Distillation step (offline, one-time):
    Qwen3-Omni audio embedding (768-dim) → W_distill (MLP 768→22×32=704) → 22MB embedding
    Result: 22MB "style descriptor" — same footprint as CAM++, but semantically richer
    Training: triplet loss on (same singer, different singer, same style/diff singer)
    Frozen after training: distilled_style_encoder.pt (~22MB)

  Component 2: Qwen3.5-0.8B — "Brain" (SEPARATE CONTROL COMPONENT)
    Role: NOT for voice conversion directly
    Role: interprets user text prompts about desired singing style
          e.g. "make it sound more melancholic" → style delta vector
          e.g. "increase vibrato in chorus" → targeted style conditioning
    Output: style_delta ∈ ℝ^512 (additive to style descriptor)
    Size:   0.8B, ~1.6 GB fp16 → CPU inference
    Usage:  Optional text-to-style-delta when user provides style prompt

  v2.0 Conditioning flow:
    ref.wav (5-30s)
      │
      ├─ CAM++ (22MB, CPU)         → timbre_vec ℝ^512    [WHO, preserved from v1.0]
      │
      └─ Distilled Style Encoder   → style_vec ℝ^512     [HOW they sing, NEW]
           (22MB, CPU, distilled from Qwen3-Omni)
                    ↑
               [optional] Qwen3.5-0.8B (CPU) + user style prompt → style_delta ℝ^512

    DiT Flow Backbone receives:
      [timbre_vec + style_vec + style_delta] + [content] + [F0]
      → richer, more faithful voice conversion

  VRAM budget v2.0 (inference, sequential):
    Distilled Style Encoder: ~22MB       (negligible, CPU)
    Qwen3.5-0.8B (optional): ~1.6 GB    (CPU only, no GPU)
    Seed-VC DiT + Vocoder:   ~0.8 GB    (GPU, same as v1.0)
    Qwen3-Omni encoder:      NOT loaded at inference (distilled offline)
    TOTAL GPU: ~0.8 GB (identical to v1.0)

  v2.0 INVARIANTS:
    INV-SVC-V2-1: Qwen3-Omni runs OFFLINE for distillation only — never at inference
    INV-SVC-V2-2: distilled_style_encoder.pt ≤ 30MB (target: 22MB)
    INV-SVC-V2-3: Qwen3.5-0.8B runs on CPU only — never loaded to GPU during SVC
    INV-SVC-V2-4: v2.0 is BACKWARD COMPATIBLE — style_vec defaults to zeros if absent
    INV-SVC-V2-5: DiT Flow Backbone fine-tune required after adding new conditioning dims

  DO NOT confuse roles:
    Qwen3-Omni in SVC:     audio encoder for style DISTILLATION (offline, one-time)
    Qwen3-Omni elsewhere:  audio QA/captioning (unrelated use case)
    Qwen3.5-0.8B in SVC:   text→style_delta CONTROL (optional, CPU)
    Qwen3.5-0.8B in N4:    macro section planner (completely separate module)

§4.4 — Sealed Constants (SVC v1.0)

  SEED_VC_CONFIG        = "config_dit_mel_seed_uvit_whisper_base_f0_44k.yml"
  SEED_VC_REPO          = "github.com/Plachtaa/seed-vc"
  SEED_VC_VRAM_GB       = 0.8      # fp16 inference
  SEED_VC_TRAIN_VRAM_GB = 2.5      # LoRA fine-tune
  SVC_F0_CONDITION      = True     # MANDATORY — disabling loses melody
  SVC_DIFFUSION_STEPS   = 50       # quality; 4-10 for fast test
  SVC_FP16              = True     # VRAM economy
  SVC_SR                = 44100    # 44k config native
  SVC_BATCH_SIZE        = 2        # RTX 3060 6GB fine-tune
  SVC_MAX_STEPS         = 10000    # overnight fine-tune
  SVC_CHECKPOINT        = "models/noesis_svc/noesis_svc_v1.pth"

  Invariants:
    INV-E6-1: f0_condition=True MANDATORY (melody preserving)
    INV-E6-2: ACE-Step and Seed-VC NEVER in VRAM simultaneously (sequential)
    INV-E6-3: VRAM inference ~0.8 GB fp16 | VRAM train ~2.5 GB
    INV-E6-4: sr=44100 Hz (Seed-VC 44k config native)
    INV-E6-5: ref audio 5-30 seconds (CAM++ speaker encoder requirement)

§4.5 — Installation (v1.0)

  pip install (via python_embedded\python.exe -m pip):
    git+https://github.com/Plachtaa/seed-vc.git
  Downloads: CAM++, Whisper-base, RMVPE, DiT backbone, BigVGAN (~500 MB total)

  For dataset preparation:
    python_embedded\python.exe -m pip install faster-whisper silero-vad

§4.6 — First Test Command (v1.0)

  python inference.py \
    --source vocals_from_track.wav \     # vocal stem from HTDemucs
    --target ref_singer.wav \            # 5-30s singer reference
    --output ./test_output/ \
    --diffusion-steps 50 \
    --f0-condition True \                # MANDATORY for singing
    --fp16 True

§4.7 — Dataset and Fine-tune Plan (v1.0)

  Data:
    200 GB studio vocals (clean, no music, professional recording) ← PRIMARY ADVANTAGE
    8,764 DJ tracks → HTDemucs → vocal stems (D:\DJ-ская)
  Fine-tuned Seed-VC on this data = NOESIS-SVC v1.0 = our proprietary model.
  NOT the base Seed-VC weights — our custom checkpoint with unique data advantage.

  Fine-tune config:
    base:       Seed-VC frozen weights
    config:     config_dit_mel_seed_uvit_whisper_base_f0_44k.yml
    batch_size: 2 (RTX 3060 6GB)
    max_steps:  10000 (~3-5 overnight runs)
    output:     checkpoints/noesis_svc_v1/model_10000.pth (~300 MB)

§4.8 — New Files Required

  v1.0 (immediate):
    singing_voice_cloner.py      Main SVC: fine-tuned Seed-VC load, inference, API
    noesis_svc_backend.py        DHCF-FNO backend wrapper
    prepare_svc_dataset.py       200GB vocals: silero-VAD + filter + JSONL
    stem_to_svc_pipeline.py      Full pipeline: ACE-Step → HTDemucs → Seed-VC → mix
    test_singing_voice_cloner.py 12 tests: load/inference/f0/speaker_sim/integration
    run_svc_finetune.bat         Overnight training script (batch=2, 10K steps)

  v2.0 (future, Phase SVC_v2):
    svc_style_distiller.py       Offline: Qwen3-Omni encoder → 22MB style descriptor
    distilled_style_encoder.pt   Trained output (~22MB)
    svc_text_style_controller.py Qwen3.5-0.8B → style_delta ℝ^512 (optional)
    singing_voice_cloner_v2.py   Updated SVC with [timbre + style + style_delta] conditioning

§4.9 — IQS Integration

  v1.0: speaker_similarity = CAM++_cosine(ref_speaker, cloned_output)
  v2.0: style_fidelity     = cosine(distilled_style_vec_ref, distilled_style_vec_output)
        Both logged in snapshot non-core (not part of IQS formula).

  Quality gates:
    v1.0: speaker_sim ≥ 0.80 → PASS
    v2.0: speaker_sim ≥ 0.80 AND style_fidelity ≥ 0.70 → PASS

  Checkpoint enters snapshot core:
    svc_checkpoint_v1 = SHA256(noesis_svc_v1.pth)   [v1.0]
    svc_checkpoint_v2 = SHA256(noesis_svc_v2.pth)   [v2.0, future]

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
§5. MASHUP (E7) — mashup.py
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

  mashup.py::mashup(track_a, track_b, sr, blend, group_blend, seed)
  Mechanism: BPM-align (librosa) + 12-band NOESIS stem group blending

  Stem groups (NOESIS 12-band → 4 groups):
    rhythm:  ["kick", "snare_clap", "hihats", "percussion"]
    bass:    ["subbass", "bass"]
    melody:  ["lead", "synth", "pad"]
    texture: ["vocals", "fx", "other"]

  group_blend: {"rhythm": 0.2, "bass": 0.3, "melody": 0.8, "texture": 0.5}
    Value -1 means: use global blend parameter
  Invariant INV-E7-1: group_blend values ∈ [0.0, 1.0] or -1

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
§6. REST API — api_routes.py
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

  Ports: FastAPI :8765 | Next.js :3000 (proxy /api/* → :8765) | Gradio :7860

  Edit Suite endpoints (9 REST):
    POST /ui/api/v1/edit/speed
    POST /ui/api/v1/edit/extend
    POST /ui/api/v1/edit/cover
    POST /ui/api/v1/edit/remix
    POST /ui/api/v1/edit/instruments
    POST /ui/api/v1/edit/vocals
    POST /ui/api/v1/edit/voice_clone     ← SVC (Seed-VC), NOT TTS
    POST /ui/api/v1/edit/mashup
    GET  /ui/api/v1/edit/schema

  All: audio_b64 (base64-WAV) + JSON params in; audio_b64 + JSON metrics out.

  Generation endpoints:
    POST /ui/api/v1/generate
    POST /ui/api/v1/master
    GET  /ui/api/v1/health

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
§7. STEM SEPARATION (12-Band Standard — IMMUTABLE)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

  NOESIS 12-band stem standard (EDM/Hip-Hop 2026, Invariant #13):
  Order is FIXED. n_fft=4096, cosine crossover, energy-normalized.

    1. kick         30–120 Hz
    2. snare_clap   120–350 Hz
    3. hihats       5k–16.1 kHz
    4. percussion   350 Hz–5 kHz
    5. subbass      30–75 Hz ONLY
    6. bass         75–250 Hz
    7. lead         1.5k–8 kHz
    8. synth        250 Hz–8 kHz
    9. pad          250 Hz–4 kHz
   10. vocals       120 Hz–8 kHz
   11. fx           4k–16.1 kHz
   12. other        remainder

  HTDemucs backend (P4.3, DONE):
    htdemucs_ft model (preferred) or htdemucs_6s fallback
    Axiom A6 fallback: stft/demucs pluggable backends
    Deterministic: fixed seed (Invariant INV-N-5)

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
§8. N-SERIES PLANNERS AND MODELS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

  N4 Macro Planner — n4_macro_planner.py   [8/8 PASS]
    Model:  Qwen3.5-0.8B LoRA r=8
    Input:  genre_id (33) + global_style_token (256-dim)
    Output: section_sequence [intro/verse/chorus/bridge/drop/outro/fill/break]
    Sequence tags (14, from structure_controller_v4.py):
      intro, verse, pre-chorus, chorus, post-chorus, bridge, drop, build,
      breakdown, outro, solo, fill, hook, ambient
    VRAM: ~1.0 GB fp16 training

  N5 Meso Planner — n5_meso_planner.py     [8/8 PASS]
    Input:  section_sequence + genre + tempo
    Output: chord_progressions [root, quality] per bar
    Chord vocab: 288 (12 roots × 24 qualities)

  N6 Micro Planner — n6_micro_planner.py   [8/8 PASS]
    Input:  chord_progressions + section + genre
    Output: performance tokens (timing, velocity, articulation)
    Lipschitz: W_micro K=0.80 < 0.95 (required for stability)

  N7 Music World Model — n7_mwm.py         [10/10 PASS]
    Architecture: 0.3B Transformer decoder
    State: S_t ∈ ℝ^512 (7 sub-vectors × 64 dims + 128 learned pad)
    Transition: S_{t+1} = f_θ(S_t, u_t)
    Lipschitz K=0.077 (< 0.95 target) — stable trajectories
    Latency: < 5ms/bar on i7-12700H (target for inference)
    TRAINING DATA: FMA-large + Jamendo + NOESIS-MOS v2 corpus

  N8 AI Band — n8_ai_band.py               [8/8 PASS]
    5 stateless agents: DrumAgent, BassAgent, HarmonyAgent, MelodyAgent, VocalAgent
    Total: ~1.25B params (all on CPU, GPU for DiT only)
    Conductor: stateless rule-based coordinator (no neural net)
    Invariants: INV-BAND-1 (no weight modification), INV-BAND-2 (bar sync),
                INV-BAND-3 (determinism), INV-BAND-4 (non-interference)

  N11 Vocal Backbone — n11_backbone.py     [8/8 PASS]
    Model: Qwen3-TTS-12Hz-1.7B (LoRA r=16)
    SR: 48kHz (Qwen3-TTS-Tokenizer-12Hz-48kHz)
    NOTE: N11 is for vocal GENERATION (TTS for singing).
          For vocal TIMBRAL CLONING, use N_SVC (Seed-VC §4) instead.

  test_n4_n8_integration.py                [5/5 PASS]
    Full N4→N5→N6→N7→N8→N11 stack integration test

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
§9. UI DESIGN SYSTEM (SEALED 2026-03-13)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

§9.1 — Stack

  React UI (primary): Next.js + TypeScript + Tailwind v4
    Port: 3000 (next dev)
    Proxy: /api/* → FastAPI :8765
    Build: next build (0 TS errors at seal 2026-03-13)

  Gradio UI (parallel): ui/app.py
    Port: 7860
    Role: inference + training access; Edit tab (8 nested tabs)

  FastAPI backend: main.py + api_routes.py
    Port: 8765

  WRONG: Vite 6 / port 5173 (outdated — Next.js migration DONE 2026-03-13)

§9.2 — Design Tokens (IMMUTABLE — NEVER change)

  #D1FE17  = acid neon yellow
             ALL action buttons, active tabs, sliders, checkboxes,
             focus rings, progress bars, IQS values
             NEVER replace with pink / purple / blue / green / white

  #000000  = background (NEVER change)
  #111111  = surface/cards (NEVER change)
  Orbitron = headings (NEVER change)
  Inter    = body text (NEVER change)
  JetBrains Mono = numbers, checksums, code (NEVER change)

§9.3 — Sealed Files (require explicit user confirmation to modify)

  ace-step-ui/package.json         "dev":"next dev" — NEVER "vite"
  ace-step-ui/next.config.ts       proxy :8765 — NEVER change port
  ace-step-ui/services/api.ts      API contract — NEVER touch
  ace-step-ui/context/             Auth+I18n state — NEVER touch
  ace-step-ui/server/              Node.js server — NEVER touch
  ace-step-ui/audiomass-editor/    Audio editor — NEVER touch
  ace-step-ui/i18n/translations.ts 508 keys × 11 languages — only add
  ace-step-ui/data/news.json       Multilingual news — only append
  ui/theme.py                      Gradio theme — only extend, never remove tokens
  ui/app.py                        Layout sealed; event wiring OK

§9.4 — 11 Sealed Languages (NEVER remove)

  en zh es hi ar pt ru fr de ja ko

§9.5 — UI Components

  Next.js components:
    Header, IqsBadge, DriftBadge, SongCard, StemExportModal (12 stems)
    GenerationPanel (33 genres + vocal + mastering)
    SessionPanel, AudioCard, MasteringPanel, ProgressTracker
    SettingsPanel, AboutPanel, MobileNav

  Gradio tabs (app.py):
    Generation, Mastering, Edit Suite (8 nested: Speed/Extend/Cover/Remix/
    Instruments/Vocals/Voice Clone/Mashup), Training, Settings

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
§10. AGMS — Adaptive Genre Mastering System (N2.1–N2.5)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

  Purpose: Per-genre mastering parameter learning from DJ collection.
  Dataset: models/NOESIS_MOS/dj_collection_aes_classified.jsonl (8,091 tracks)
           Format: {genre, lufs_int, true_peak, flatness, aes_score, ...}

  Components:
    genre_mastering_master_profiles.py  33-genre master profiles (N2.1)
    genre_profile_learner.py            EMA learning, MIN_HISTORY=10 (N2.2)
    mastering_profiles_panel.py         Gradio AGMS panel (N2.3)
    llm_mastering_advisor.py            Qwen3 mastering advisor (N2.4)
    mastering_chain.py                  IRC5 char/upward_compress_db (N2.5)

  Constants:
    AGMS_MIN_HISTORY = 10
    AGMS_EMA_ALPHA   = 0.15

  AGMS modifies ONLY (Control Plane):
    mastering_config.yaml, genre_profiles_overrides.yaml, pre_limiter_overrides.yaml
  AGMS NEVER touches: generation, DiT weights, IQS formula, source code.

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
§11. GENRE SYSTEM
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

  33 genres, 144 aliases. Module: genre/genre_vocab.py
  Genre routing: GenreDetector v2 (N2.1) normalizes aliases → canonical genre.

  DSP parameter YAML files:
    mastering_config.yaml           33 genres (LUFS, ceiling, stereo_width,
                                    high_shelf_gain, low_shelf_gain,
                                    de_esser_enabled, cf_reduction_enabled,
                                    dither_enabled, stem_aware_eq_enabled)
    pre_limiter_overrides.yaml      33 genres (sparse_benefit_db, lookahead_ms,
                                    attack_ms, sparse_ratio, dense_ratio)
    genre_profiles_overrides.yaml   33 genres (entropy_mean, energy_alpha,
                                    limiter_aggression, phase_mean, cfg_curve_mid)

  Mastering_chain priority (§1.7): YAML genres.* > master_profiles > ctor default.

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
§12. MODULE MAP (core source files)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

  adapter/adapter_types.py          GenerationRequest + GenerationResult
  adapter/adapter_call.py           3x RNG lock + generate_music() dispatch
  adapter/backend_acestep.py        AceStepBackend (audio_prompt wired)
  cli/sdk.py                        SDK entry: generate_music(), guidance_scale=7.0
  mastering/mastering_chain.py      MasteringChain v3.4 (31 stages)
  mastering/irc5_limiter.py         IRC-5 multiband limiter
  mastering/de_esser_v2.py          De-esser Stage 14
  mastering/pre_limiter.py          Two-pass adaptive pre-limiter
  mastering/mastering_normalize.py  BS.1770-4 LUFS normalize
  mastering/true_peak_filter.py     4x polyphase Kaiser FIR
  metrics/iqs.py                    IQS production (checksum 12c2f47c)
  metrics/iqs_weights.py            AutoEval proxy (checksum 9097e7)
  qa_external/noesis_mos_inference.py  NOESIS-MOS v1 backend
  qa_external/perceptual_analysis_layer.py  PAL v2
  stems/stem_separator.py           HTDemucs 12-band
  edit_suite.py                     E1-E5 (Extend/Cover/Remix/Instruments/Vocals/Speed)
  singing_voice_cloner.py           E6 SVC via Seed-VC (NEW — §4)
  mashup.py                         E7 BPM-align + stem group blend
  genre/genre_vocab.py              33 genres + 144 aliases
  genre/genre_mastering_master_profiles.py  AGMS master profiles
  n4_macro_planner.py               Section token generation
  n5_meso_planner.py                Chord progression
  n6_micro_planner.py               Performance tokens
  n7_mwm.py                         Music World Model (S_t ∈ ℝ^512)
  n8_ai_band.py                     5-agent Band system
  n11_backbone.py                   Qwen3-TTS vocal backbone
  operator_registry.py              Backend registry (checksum 7c0d5aba...)
  snapshot_utils.py                 v16+ snapshot with Merkle
  autoeval_agent.py                 Autonomous overnight optimization
  autoeval_param_schema.py          Schema v1.3.0, 29 params, 3073949b
  phase_r_direct.py                 Phase R benchmark (GPU, guidance_scale=7.0)
  run_selftest_all.py               N4-N11 65/65 self-tests

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
§13. SESSION 1 COMPONENTS (DONE 2026-03-12, 46/46 + 42/42 tests)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

  Phase ITO · Phase SC · Phase StemAware · Phase AudioSR · Phase VIZ · Phase B2B
  Total: 46/46 new tests PASS, 42/42 regression PASS

§13.1 — ITO: Reference-Based Mastering Style Transfer

  File: mastering/ito_style_transfer.py
  Source: ITO-Master (Koo et al., Sony AI, ISMIR 2025, arXiv 2506.16889)
          ST-ITO (arXiv 2410.21233)
  Tests: test_ito_style_transfer.py — 10/10 PASS

  Architecture:
    ReferenceProfile dataclass:
      lufs, crest_db, spectral_centroid_hz, high_energy_ratio,
      mid_energy_ratio, low_energy_ratio
    Extraction: bs1770 LUFS + true_peak_filter + FFT spectral analysis
    Override generation: reference profile → MUTABLE_PARAMS delta → YAML

  Invariants:
    §INV-ITO-1: reference audio NEVER enters signal path (analysis only)
    §INV-ITO-2: all overrides validated against MUTABLE_PARAMS bounds
    §INV-ITO-3: reference unavailable → passthrough + log WARNING (Axiom A6)
    §INV-ITO-4: deterministic given same reference + target + genre

  Usage:
    from mastering.ito_style_transfer import ITOStyleTransfer
    ito = ITOStyleTransfer()
    overrides = ito.extract_and_apply(reference_wav, target_wav, genre)

§13.2 — SC: Text-Controlled Mastering via Natural Language

  File: mastering/sc_text_controller.py
  Source: SonicMaster-inspired text → mastering parameter mapping
  Tests: test_sc_text_controller.py — 10/10 PASS
  Rules: 84 keyword rules (was 36 — updated CPU Day 2 2026-03-21)
    Categories: EDM structural (drop/buildup/breakdown/anthem/festival)
                Vocal types (spoken/whisper/choir/vocal)
                Emotional (tension/energy/calm/intense/chill/euphoric/hypnotic)
                Platform (spotify/apple music/youtube/tidal/dj mix/mastered)
                Genre shorthand (trap/phonk/lofi/orchestral/jazz/kpop/shoegaze)
                Dynamic (dynamic/maximized/gentle/air/presence/muddy/sub bass)

  Architecture:
    SC_SYSTEM_PROMPT: instructs Qwen3 to output JSON mastering parameters
    Available parameters (AutoEval schema v1.3.0):
      high_shelf_gain:    [-6.0, +6.0] dB
      low_shelf_gain:     [-6.0, +6.0] dB
      transient_emphasis: [0.0, 3.0] dB
      compression_ratio:  [1.0, 8.0]
      sparse_benefit_db:  [0.0, 6.0] dB
      stereo_width:       [0.8, 1.4]
      target_lufs:        [-18.0, -6.0] LUFS

  Invariants:
    §INV-SC-1: LLM output NEVER directly writes to any file
    §INV-SC-2: all params validated against schema bounds
    §INV-SC-3: rule-based fallback always available (no hard Qwen3 dep)
    §INV-SC-4: empty/unparseable LLM response → passthrough + log WARNING

  Usage:
    from mastering.sc_text_controller import SCTextController
    sc = SCTextController(llm_fn=qwen3_fn)  # or None for rule-based
    overrides = sc.process("make it punchier and brighter")

§13.3 — StemAware: Per-Stem Density Profiling

  File: mastering/stem_density_profiler.py
  Tests: test_stem_density_profiler.py — 8/8 PASS

  Role: profiles stem density for Stage 8.5 (pre-limiter) adaptive thresholds.
  Input: 12-stem HTDemucs separation output
  Output: per-stem density score → feeds sparse_benefit_db calculation

§13.4 — VIZ: Visual DSP Plugin Panel

  File: ui/viz_dsp_panel.py
  Tests: test_viz_dsp_panel.py — 5/5 PASS

  Architecture:
    build_dsp_panel() → Gradio Blocks instance
    Lazy import: `import gradio as gr` inside function (tests work without gradio)
    Backed by autoeval_param_schema.MUTABLE_PARAMS (non-genre-specific params)
    5–10 adjustable knobs per DSP stage
    Sliders map to MUTABLE_PARAMS bounds from AutoEval schema v1.3.0

  Philosophy: Sausage Fattener minimal-knobs approach.
  Three DSP plugin equivalents (Phase VIZ future extension):
    NOESIS-Saturator:  5 knobs (drive, tone, mix, bias, warmth)
    NOESIS-MasterMind: 8 knobs (comp, limit, EQ low/mid/hi, width, lufs, punch)
    NOESIS-StemMix:    6 knobs (drums, bass, melody, vocal, fx, other)

  Usage in Gradio:
    from ui.viz_dsp_panel import build_dsp_panel
    dsp_panel = build_dsp_panel()  # embed in app.py

§13.5 — AudioSR: Audio Super-Resolution Wiring

  File: mastering/audiosr_stage.py
  Tests: test_audiosr_wiring.py — 5/5 wiring tests PASS

  Status: wired as post-generation stage (external tool, not in chain by default)
  Role: 48kHz → high-quality upsampling via AudioSR (Liu et al.)
  Invariant: AudioSR runs AFTER MasteringChain, not inside (§EC20 compliant)

§13.6 — B2B API: REST Endpoints

  File: api_routes.py
  Tests: test_api_routes_b2b.py — 8/8 PASS

  B2B endpoints (FastAPI :8765):
    POST /ui/api/v1/generate      — full generation pipeline
    POST /ui/api/v1/master        — standalone mastering
    POST /ui/api/v1/stems         — stem separation (HTDemucs 12-band)
    GET  /ui/api/v1/health        — system health + test counts
    GET  /ui/api/v1/schema        — AutoEval schema v1.3.0 (29 params)

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
§14. AGMS DETAIL — Adaptive Genre Mastering System (N2.1–N2.5)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

  All subphases DONE 2026-03-12/13. Tests: 24 new + 16 regression PASS.

§14.1 — N2.1: Genre Master Profiles

  File: genre/genre_mastering_master_profiles.py
  Content: 33 genres × IRC5 character/upward tables + DR-adaptive params
  GenreDetector v2: 38 canonical genre profiles (+hyperpop/darkwave/shoegaze/bossa_nova/orchestral/soundtrack/world/afrobeat/kpop added CPU Day 2), 144 aliases — normalizes input

§14.2 — N2.2: LLM Mastering Advisor (llm_mastering_advisor.py)

  Model: Qwen3.5-0.8B (offline — no internet required at runtime)
  Pipeline: user natural-language description → suggestion → YAML override
  Integration: suggestion goes through SC_SYSTEM_PROMPT validation before write
  Invariant: LLM output NEVER directly writes to YAML — goes through validator

§14.3 — N2.3: Genre Profile Learner (genre_profile_learner.py)

  Algorithm: EMA (Exponential Moving Average) loop
  Constants:
    MIN_HISTORY_FOR_AUTO = 10   (cold-start guard §LEARN-5)
    NOESIS_LEARNER_ONLINE env var for online learning mode
  Input: per-track mastering results → EMA update to genre profile
  Protection: ≥10 tracks required before first automatic update

§14.4 — N2.4: Mastering Profiles Panel (mastering_profiles_panel.py)

  Gradio UI panel for AGMS profiles — embedded in app.py Edit tab
  Shows: current genre profile values + update history + drift statistics

§14.5 — N2.5: MasteringChain AGMS Integration

  IRC5 character/upward_compress_db wiring into mastering_chain.py
  overrides_dir gate (see §1.7): ALL 3 AGMS lookups behind gate:
    1. LUFS target lookup (genre_mastering_master_profiles)
    2. IRC5 character lookup
    3. upward_compress_db lookup
  §FIX-SAND: upward_compress noise gate _UPWARD_RMS_GATE=0.001
  §FIX-HIGHS: HF bands always neutral irc5_char=0.5 (prevents HF harshness)
  diagnose_mastering selftest: enable_spectral_transfer=False (§EC17)
  Test: 24/24 test_n25_mastering_wiring.py PASS

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
§15. AUDIOSR STANDALONE TOOL
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

  AudioSR (Liu et al., ICLR 2024) — post-generation super-resolution.
  Status: wired (Phase ASR.v1 DONE); ASR.v2 (inference-time scaling) PLANNED.

  Phase ASR.v1 (DONE):
    audiosr_stage.py: wiring into generation pipeline
    Input: 44.1kHz or 48kHz WAV → AudioSR → enhanced quality output
    NOT inside MasteringChain (runs as post-step, §EC20 compliant)

  Phase ASR.v2 (PLANNED):
    Inference-time scaling: N_candidates ≤ 4, select best by SNR+spectral
    arXiv 2508.02391 approach
    Priority: LOW (see ROADMAP §6)

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
§16. DOCS AUDIT P2.x (DONE 2026-03-12/13)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

  87 docs scanned, 18 files patched:
    drift thresholds 1.0/2.0 → 6.0 dB propagated everywhere
    guidance_scale 0.0 → 7.0 throughout
    Vite 5173 → Next.js 3000 in all UI references
  version.py updated to current roadmap version
  All incorrect B_linf = 4.32... values corrected to 3.60972762

---

## §N3. Implementation Plan 2026 — Pending Features

### §N3.K — Root Key Emotional Taxonomy

**File:** `key_emotional_profile.py`
**Status:** DONE (2026-03-17)

Affective dimensions (5D, normalized 0.0–1.0):
- `brightness` — spectral brightness, major/minor polarity
- `tension` — harmonic tension, dissonance level
- `melancholy` — sad/nostalgic affect
- `heroism` — triumphant, epic, powerful character
- `warmth` — emotional warmth, intimacy, comfort

24-key table (12 Major + 12 Minor). Integration:
- `prompt_utils.py::build_caption()` — inject `mood_tag` from key
- `n4_macro_planner.py::generate_plan()` — key_affect step before caption assembly
- Invariant: zero frozen-model changes; pure prompt engineering
- Integration complete: n4_macro_planner.py key_affect regex step added
- Tests: test_key_emotional_profile.py 12/12 PASS, test_prompt_utils.py T19/T20/T21 PASS

### §N3.J — NOESIS-MOS Jumbo: Dual Audio Encoder

**Files:** `noesis_mos_jumbo.py`, `noesis_mos_jumbo_trainer.py`
**Status:** CODE DONE — noesis_mos_jumbo.py written; weights PENDING GPU (~8h)
**Reference:** ALARM arXiv:2603.09556; OLMo Hybrid GatedDeltaNet (Apache 2.0)

Architecture:
```
MuQ-large-msd-iter [frozen] → proj_muq: [B, T_a, 1024] → [B, T_a, 512]
PANN-CNN14         [frozen] → proj_pann: [B, 2048] → [B, 1, 512]
concat + LayerNorm: [B, T_a+1, 512]
Qwen3-Embedding-0.6B [frozen] → text_proj: [B, 1024] → [B, 1, 512]
CrossAttention (8 heads): audio_tokens ← text_emb
HybridSequenceAggregator: 2× GatedDeltaNet + 1× MultiHeadAttention (2:1 ratio)
OrdinalRegressionHead: K=10 thresholds, CDF-based
Trainable params: ~3.1M
```

Fusion formula:
```
z_muq   = W_muq · h_muq       ∈ R^{T_a × 512}
z_pann  = W_pann · p_pann      ∈ R^{1 × 512}
z_audio = LayerNorm([z_muq; z_pann])  ∈ R^{(T_a+1) × 512}
z_text  = W_text · e_qwen3     ∈ R^{1 × 512}
z_fused = CrossAttn(Q=z_audio, K=z_text, V=z_text)
mos_raw = OrdinalHead(mean_pool(z_fused))
```

Promotion criterion: Pearson r ≥ 0.86 on held-out validation (vs v1 r=0.837).
PANN-CNN14 already present in project (`fad_pann_backend.py`) — zero new downloads.
Command when GPU free: run_mos_finetune_n15.bat (~8h)
Expected: r improvement from 0.837 → ≥ 0.86

### §N3.G — GGUF Reasoning LLM

**Files:** `planner_llm.py` (GGUFLLMBackend impl), `llm_mastering_advisor.py` (CoT)
**Status:** DEFERRED — облако / RTX 4090+
**Reason:** Qwen3.5-0.8B (~1.6GB fp16) достаточен для RTX 3060. N4 planner работает.
           GGUF 4B требует llama-cpp-python, не приоритет при текущем железе.
**Model (future):** Qwen3.5-4B Q4_K_M (~2.3 GB RAM)

VRAM budget (когда актуально — облако):
- DiT 2.4B (offloaded): ~0 VRAM during inference
- LM 662M: ~1.3 GB VRAM
- Qwen3-Embedding-0.6B: ~0.6 GB VRAM
- GGUF Qwen3.5-4B Q4_K_M: CPU RAM only
- TOTAL GPU VRAM: ~1.9 GB (within 6 GB limit)

Invariant: temperature=0.0 → deterministic (§INV-DET compatible).

### §N3.V — Voxtral Mini 4B Realtime STT

**Status:** REMOVED — не нужен в NOESIS DHCF-FNO пайплайне.
**Reason:** HeartTranscriptor-oss (Whisper fine-tune, 3.06GB) заменяет для lyric
           transcription. Voxtral оптимизирован для STT в реальном времени —
           use case отсутствует в генерации музыки.
**Future:** Рассмотреть для NOESIS-VC-ONE (голосовой ассистент) в облаке.
**Replaced by:** `qa_external/heart_transcriptor_backend.py` (DONE 2026-03-17)
  - HeartMuLa/HeartTranscriptor-oss, Apache 2.0, 3.06GB
  - PER=0.09 English (лучше Suno v5 и MiniMax)
  - Использование: N2.3 SingMOS датасет, QA vocal accuracy

### §N3.M — MSR Benchmark (dawn_chorus_en)

**File:** `benchmark_msr_eval.py`
**Status:** DONE (2026-03-17)
**File:** `benchmark_msr_eval.py`
**Dataset:** `ai-coustics/dawn_chorus_en` (~90 min foreground-background speech)

Metric: SI-SDR (Scale-Invariant Signal-to-Distortion Ratio):
```
SI-SDR = 10 · log10( ||α·s||² / ||α·s − ŝ||² )
         where α = ⟨ŝ, s⟩ / ||s||², s = reference, ŝ = estimated
```

Output: JSONL report compatible with `benchmark_runner.py`.
Also computes SDR, PESQ per segment.
Tests: test_benchmark_msr_eval.py 9/9 PASS

### §N3.FAD — FAD(CLAP-MA) Backend

**File:** `qa_external/fad_clap_backend.py`
**Status:** DONE (2026-03-17)
**Model:** `laion/larger_clap_music` (~400MB, Apache 2.0)

Research basis: ICASSP 2025 — FAD-CLAP-MA демонстрирует наилучшую корреляцию
с human perception качества музыки (Huang et al. 2025 confirm).

**Why better than FAD(PANN):**
- FAD(PANN) коррелирует плохо с human listening — исторический артефакт
- FAD(CLAP-MA): LAION-MA checkpoint, GTZAN accuracy 71% vs VGGish ~50%
- Лучше коррелирует с human Bradley-Terry parameters (PCC + SCC)
- Text-audio alignment score как бонус (cosine similarity)

**API:**
```python
from qa_external.fad_clap_backend import CLAPFADBackend, get_clap_fad_backend
backend = CLAPFADBackend(device="cpu")
backend.build_reference("ref_corpus/", cache_path="clap_ref.npz")
result = backend.compute("generated/")
# result.fad — float, lower=better
# также: backend.compute_text_alignment(audio, sr, "dark edm 128 bpm")
```

**§INV-CLAP-4:** длинные треки mean-pool (не bag-of-embeddings) — Huang 2025 bug fix.
**Tests:** test_fad_clap_backend.py 9 тестов (Fréchet math, chunking, save/load)

---

### §N3.TRANS — HeartTranscriptor Lyric Backend

**File:** `qa_external/heart_transcriptor_backend.py`
**Status:** DONE (2026-03-17)
**Model:** `HeartMuLa/HeartTranscriptor-oss` (3.06GB, Apache 2.0, Whisper fine-tune)
**Download:** `huggingface_hub.snapshot_download('HeartMuLa/HeartTranscriptor-oss', local_dir='models/HeartTranscriptor-oss')`

**Use cases:**
1. N2.3 SingMOS датасет — транскрипция вокала из 200GB studio collection
2. QA pipeline — Phoneme Error Rate (PER) генерированных треков
3. vocal_presence guard для voice_cloner.py

**PER benchmark (HeartMuLa paper 2026):**
- English: 0.09 (лучше Suno v5, MiniMax Music 2.0)
- Chinese: 0.12

**API:**
```python
from qa_external.heart_transcriptor_backend import HeartTranscriptorBackend
backend = HeartTranscriptorBackend(device="cpu")
result = backend.transcribe("vocal.wav")               # TranscriptResult
result = backend.transcribe_with_per("vocal.wav", ref) # + PER score
results = backend.transcribe_dir("vocals/", max_files=500)
```

**Fallback:** если HeartTranscriptor не скачан → openai/whisper-large-v3 автоматически.

---
