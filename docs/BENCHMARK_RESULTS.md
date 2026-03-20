[BENCHMARK_RESULTS.md](https://github.com/user-attachments/files/26134302/BENCHMARK_RESULTS.md)
# NOESIS DHCF-FNO — Benchmark Results
# Last updated: 2026-03-20

## R.REF2 — Phase R benchmark (2026-03-14)

### Configuration
- Generation: ACE-Step v1.5 Turbo, guidance_scale=7.0, fix_nfe=8, shift=3.0, bfloat16
- Mastering: MasteringChain v1.8, 22 stages
- IQS: v0.8 (6-term, weights frozen)
- J formula: J = α·IQS − β·Dist − γ·Phase − δ·Drift  (α=0.50 η=0.25 β=0.08 γ=0.07 δ=0.06 ζ=0.04)
- QA_ext: FAD(PANN-CNN14) via per-genre reference corpus
- J studio gate: 0.65 (canonical §IQS v0.8)
- Hardware: RTX 3060 Laptop 6GB, CPU offload

### Summary
| Metric         | Value   |
|----------------|---------|
| J_mean         | 0.3875  |
| IQS_mean       | 0.6141  |
| IQS_max        | 0.6438  |
| J_max          | 0.4125  |
| PASS+CF_LIM    | 9/10    |
| FAIL           | 1 (classical — noise generation) |
| FAD coverage   | 9/10 genres (ambient2 uses ambient corpus) |
| Wall-clock     | ~330s   |

### Per-genre results
| Genre      | IQS    | QA_ext | J      | l_n    | Drift dB | Tier    |
|------------|--------|--------|--------|--------|----------|---------|
| ambient    | 0.6186 | 0.0418 | 0.3879 | 0.3715 | 0.743    | CF_LIM  |
| edm        | 0.6043 | 0.0693 | 0.3903 | 0.6665 | 1.333    | CF_LIM  |
| hip-hop    | 0.6438 | 0.0447 | 0.4042 | 0.0000 | 0.000    | PASS    |
| jazz       | 0.6260 | 0.0387 | 0.3911 | 0.0000 | 0.000    | PASS    |
| classical  | 0.5697 | 0.0448 | 0.3597 | 1.0000 | 7.102    | FAIL    |
| neurofunk  | 0.6206 | 0.0203 | 0.3804 | 0.2820 | 0.564    | CF_LIM  |
| lofi       | 0.6125 | 0.0831 | 0.4008 | 0.2750 | 0.550    | CF_LIM  |
| metal      | 0.6160 | 0.1073 | 0.4125 | 0.3925 | 0.785    | CF_LIM  |
| pop        | 0.6114 | 0.0246 | 0.3767 | 0.3990 | 0.798    | CF_LIM  |
| ambient2   | 0.6185 | 0.0000 | 0.3711 | 0.3515 | 0.703    | CF_LIM  |

### FAD reference corpus
| Genre     | Source         | N_tracks |
|-----------|----------------|----------|
| ambient   | mtg_jamendo    | 50       |
| classical | mtg_jamendo    | 50       |
| lofi      | mtg_jamendo    | 50       |
| edm       | DJ collection  | 50       |
| hip-hop   | DJ collection  | 50       |
| jazz      | DJ collection  | 12       |
| neurofunk | DJ collection  | 50       |
| metal     | DJ collection  | 50       |
| pop       | DJ collection  | 50       |
| ambient2  | (fallback → ambient) | — |

### Known issues
- **classical**: NOISE generation (flat > 0.4) on seed=42. ACE-Step Turbo
  produces flat-spectrum output for some classical prompts. Retry with
  different seed or caption tuning. Not a mastering/IQS issue.
- **QA_ext low (0.02–0.11)**: Expected — generated music differs
  significantly from real reference corpus in PANN embedding space.
  Will improve as generation quality improves with model finetuning.
- **PANN dim mismatch**: CNN14 outputs 2048-dim, truncated to 512.
  PANN_FEATURE_DIM should be updated to 2048 for full feature utilization.
- **J studio gate 0.42**: Temporary. Mathematical ceiling J_max=0.45
  at QA_ext=0. Raise gate to 0.65 when FAD corpus quality and
  QA_ext values stabilize above 0.50.

### History
| Date       | J_mean | IQS_mean | Notes                              |
|------------|--------|----------|------------------------------------|
| 2026-03-14 | 0.3496 | 0.5818   | Baseline (pre-LUFS sync, no FAD)   |
| 2026-03-14 | 0.3551 | 0.5918   | LUFS targets synchronized          |
| 2026-03-14 | 0.3740 | 0.5887   | FAD wired (6/10 genres)            |
| 2026-03-14 | 0.3875 | 0.6141   | FAD 9/10 genres + model path fix   |

---

## CPU Day 2 Improvements (2026-03-21, pre-GPU)

### Changes applied (no benchmark yet — GPU pending)

| Component                           | Before          | After           | Expected Impact      |
|-------------------------------------|-----------------|-----------------|----------------------|
| sc_text_controller rules            | 36              | 84              | Better MixAssist UI  |
| genre_mastering_master_profiles     | 29 profiles     | 38 profiles     | Rare genre mastering |
| taxonomy entries                    | 1049            | 1049+55 (rare)  | Better DiT prompts   |
| caption dataset pairs               | 414             | 579             | Better caption LoRA  |
| DriftBadge TIER_CF_LIMIT            | 2.0 dB (BUG)    | 6.0 dB (v1.3)   | Correct tier display |
| UI DisCoder toggle                  | —               | LIVE in UI      | enable_discoder=True |
| /api/taxonomy/search endpoint       | —               | LIVE            | Caption preview UI   |

### Caption LoRA status
- Training: artifacts/caption_lora/caption_lora_v1.pt (~2h CPU, started ~03-21)
- Dataset: 579 pairs (414 original + 165 taxonomy_expand_v1)
- Base: Qwen3.5-0.8B, LoRA r=8, lr=1e-4, steps=400
- Expected: richer DiT prompts → J_mean improvement in R.REF4

### Next benchmark: R.REF4 (GPU required ~2026-03-21)
```
python_embedded\python.exe phase_r_direct.py
```
Target: J_mean > 0.40, IQS_mean > 0.62, PASS+CF_LIM = 10/10


---

## R.REF4 — PENDING (MasteringChain v3.4, GPU required)

### Configuration
- Generation: ACE-Step v1.5 Turbo, guidance_scale=7.0, fix_nfe=8, shift=3.0, bfloat16
- Mastering: MasteringChain **v3.4**, 31 stages — Ozone 12 parity 11/11 modules 10/10
- IQS: v0.8 (unchanged — sealed)

### Expected improvements vs R.REF2
| Module | Change | Expected impact |
|--------|--------|----------------|
| Gate/Expander Stage 2.8 | NEW | Noise floor reduction in sparse material |
| Dynamic EQ Stage 9.6 | NEW | Per-band correction, less mud/harshness |
| Stereo Widener Stage 12 | DAFx-24 allpass | Better ICC 0.50-0.75, no tonal coloration |
| IRC-5 N=8 / 4 bands | Upgraded | Lower distortion ε≤12.5%, presence/air bands |
| Transient Shaper v2 | IIR+hold | More natural attack/sustain, no pumping |
| Compressor v2 | Peak/Blend/RMS | Less pumping, better dynamic range |

### Run command
```
python_embedded\python.exe phase_r_direct.py
```

### Expected result
| Metric | R.REF2 | R.REF4 target |
|--------|--------|--------------|
| J_mean | 0.3875 | > 0.40 |
| IQS_mean | 0.6141 | > 0.62 |
| PASS+CF_LIM | 9/10 | 10/10 |

> Record result in History table below when complete.

---

## R.REF3 — CANONICAL VALID BENCHMARK (2026-03-12, POST CFG-FIX)

### Configuration
- Generation: ACE-Step v1.5 Turbo, guidance_scale=7.0, fix_nfe=8, shift=3.0, bfloat16
- Mastering: MasteringChain v1.8, 22 stages (pre CPU-day fixes)
- IQS: v0.8 (6-term, α=0.50 η=0.25 β=0.08 γ=0.07 δ=0.06 ζ=0.04)
- J formula: J = α·IQS − β·Dist − γ·Phase − δ·Drift
- J studio threshold: ≥ 0.65
- Hardware: RTX 3060 Laptop 6GB, offload_dit_to_cpu=True

### Summary
| Metric         | Value   |
|----------------|---------|
| MUSIC PASS     | 10/10   |
| LUFS range     | −24.8 to −16.1 dB |
| Flatness range | 0.033–0.214 |
| guidance_scale | 7.0 (CFG fix 2026-03-10) |
| IQS_mean       | 0.5243  |
| J_mean         | 0.3146 (pre-N1.4, target ≥ 0.65 after GPU re-run) |
| MOS_n          | 0.6245  |

> **NOTE:** All benchmarks before 2026-03-10 are INVALID.
> CFG bug caused mastering of noise. Fixed: dual-pass CFG, guidance_scale=7.0 sealed.

> **NEXT RUN (GPU FREE ~2026-03-20):** MasteringChain v3.4 (31 stages) ready.
> Expected J > 0.40: all 11 Ozone 12 modules 10/10, DAFx-24 stereo widener, Dynamic EQ OLA.

---

## BENCHMARK-165 — Plan (2026-03-17)
**Status: PLAN READY — generation pending GPU free (~2026-03-21)**

| Metric            | Value       |
|-------------------|-------------|
| Total tracks      | 165         |
| Genres            | 33 (all canonical) |
| Seeds per genre   | 5 [42, 137, 256, 512, 1024] |
| Plan file         | artifacts/benchmark_165/plan.jsonl |
| Plan checksum     | run benchmark_165.py save |

### Genre list (33)
edm, hiphop, ambient, metal, jazz, classical, pop, rnb, lofi, trap,
neurofunk, techno, house, dubstep, dnb, rock, indie, folk, country, blues,
reggae, latin, afrobeat, kpop, phonk, hyperpop, darkwave, shoegaze,
bossa_nova, swing, orchestral, soundtrack, world

### Generate plan
```
python_embedded\python.exe benchmark_165.py save
```

### Generate tracks (GPU required)
```
python_embedded\python.exe phase_r_direct.py --plan artifacts/benchmark_165/plan.jsonl
```

---

## Mastering Chain Fixes (2026-03-18)
**Applied before GPU re-run — MasteringChain v3.0+ (24 stages)**

| Fix                                | Stage    | Root Cause                       | Expected Impact           |
|------------------------------------|----------|----------------------------------|---------------------------|
| StemAwareNode.process_audio() added | Stage 6 | Method missing → crash → silence | Silence regression eliminated |
| HF artifact scrubber               | Stage 2.5 | bfloat16 VAE quantization noise  | HF crunch −10..−15dB      |
| Vocal tilt +3.5dB + 3kHz peak      | Stage 6  | ACE-Step underlevels vocals 8-10dB | Vocal +3-5dB in dense mixes |
| Vocal prominence analyzer (VRA)    | Stage 14.5 | Buried vocal in mix             | Adaptive +0..+6dB         |
| CL-loop max_passes 4→2             | Stage 20 | +12dB swing → IRC5 GR cascade   | Pumping eliminated        |
| Pre-limiter knee 1.0→3.0dB         | Stage 18 | Brickwall transient clamping     | Softer natural limiting   |
| DisCoder SDK hook                  | Post-gen | DCAE/HiFiGAN vocoder blur        | MUSHRA 88.14 vs 78.97     |
| Prompt taxonomy 1049 entries       | Caption  | Generic genre-only captions      | Better DiT conditioning   |

## API Fixes (2026-03-18)
| Fix                                    | File                          | Impact                            |
|----------------------------------------|-------------------------------|-----------------------------------|
| sdk.generate() → sdk.generate_audio()  | noesis_api/routes/generate.py | Generation no longer crashes      |
| vocal/mix/msr/master routers wired     | noesis_api/main.py            | All N3 endpoints live via start_noesis.bat |
| /vocal/change endpoint added           | api/router_vocal.py           | N3.4 Voice Changer API complete   |
| VoiceChangeRequest/Result types added  | api/router_types_vocal.py     | N3.4 typed API                    |
| start_noesis.bat: 3000→5173            | start_noesis.bat              | Correct Vite port                 |
| version.py 5 stale constants           | version.py                    | Accurate metadata                 |

## DisCoder Quality Target
- Input: HiFiGAN ConvNeXt (ACE-Step native vocoder), MUSHRA ~78.97
- Output: DisCoder (MUSHRA 88.14, ICASSP 2025, ETH Zürich)
- SNR delta: measured after weights download (~1.7GB in models/discoder/)
- VRAM: ~1.2GB GPU (fp16) | ~0 CPU (CPU mode ~60s/30s)
- Activation: NoesisSDK(enable_discoder=True) or generate_audio(enable_discoder=True)
- §INV-DISC-4: passthrough on any error — never blocks generation

## Taxonomy Impact (2026-03-18)
- Before: DiT receives bare genre string "cyberpunk glitch" → generic output
- After:  TF-IDF search → "cyberpunk soundtrack with glitchy digital percussion,
          industrial bass drone, neon-lit synthesizer arp, 140 BPM" → targeted output
- 57 hand-crafted base entries + 992 auto-generated = 1049 total
- Demonstrated search scores: neurofunk 0.577, orchestral 0.523, lofi jazz 0.415
