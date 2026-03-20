[NOESIS_MASTER_PLAN_v2_2.md](https://github.com/user-attachments/files/26134371/NOESIS_MASTER_PLAN_v2_2.md)
"""NOESIS — Deterministic Hybrid Control Framework for Frozen Neural Operators (DHCF-FNO)
Copyright (c) 2026 AMAImedia.com
All rights reserved.
B:/Downloads/Portable/NOESIS_DHCF-FNO/docs/NOESIS_MASTER_PLAN_v2_1.md"""

# NOESIS MASTER PLAN v2.1
## Полное задание для terminal Claude Code

```
Version:    v2.2  (2026-03-21, CPU Day 2 complete)
Author:     Ilia Bolotnikov / AMAImedia.com
Status:     ACTIVE
Hardware:   RTX 3060 Laptop 6GB | i7-12700H | 64GB DDR5
GPU status: FREE — SVC training завершён (~step 10000)
Context:    CPU задачи → GPU-последовательность по освобождению
```

---

## ОБЯЗАТЕЛЬНАЯ ИНСТРУКЦИЯ ДЛЯ CLAUDE CODE

```
Execute the following steps immediately. Do not describe — execute them.
Working directory: B:\Downloads\Portable\NOESIS_DHCF-FNO\
Python: python_embedded\python.exe
All new files must begin with the NOESIS header (full path in 4th line).
After each task: run verification command and report PASS/FAIL.
Never modify: iqs_weights.py, constants.py, iqs.py (FROZEN FILES).
```

---

## СОСТОЯНИЕ НА 2026-03-21 (после CPU Day 2)

```
MasteringChain:  v3.2  (27 stages: 22 + 0.5 + 2.5 + 9.5 + 14.5 + 16.5)
Tests PASS:      782+  (79 test files)
Roadmap:         v0.28
Engineering:     v1.2
TZ:              v2.1

НОВЫЕ STAGE в v3.2:
  Stage 0.5   UniverSR SR          qa_external/universr_backend.py
  Stage 2.5   HF artifact scrubber mastering/hf_artifact_scrubber.py
  Stage 9.5   NeuralEQ (taxonomy)  mastering/neural_eq.py
  Stage 14.5  VocalProminenceBoost mastering/vocal_prominence_boost.py
  Stage 16.5  TransientShaper      mastering/transient_shaper.py

НОВЫЕ ИНСТРУМЕНТЫ:
  cli/vram_orchestrator.py       5-phase sequential cascade, peak 4.8GB
  tools/dcae_decoder_finetune.py LoRA r=8 на FMA-large, скрипт готов
  qa_external/universr_backend.py UniverSR ICASSP 2026 MIT, 4 ODE шага
  mastering/neural_eq.py         taxonomy-driven FxNorm для 1049 entries
  mastering/transient_shaper.py  Ozone Stabilizer аналог, Bello 2005

SDK re-vocoding: DisCoder (MUSHRA 88.14) → BigVGAN v2 (83.42) → passthrough
VRAMOrchestrator: Generation→Revocoding→Vocal→Mastering, пик 4.8GB

API FIXES (2026-03-18):
  noesis_api/routes/generate.py: sdk.generate() → sdk.generate_audio() FIXED
  api/router_vocal.py:           /vocal/change endpoint ADDED (N3.4)
  noesis_api/main.py:            vocal/mix/msr/master routers WIRED
  start_noesis.bat:              Next.js/3000 → Vite/5173 FIXED

GPU SEQUENCES READY:
  run_gpu_day1_sequence.bat  Phase R + QA.1 FAD (~2.5ч)
  run_gpu_day2_sequence.bat  FAD(CLAP-MA) + N1.4 Jumbo (~10ч)
  run_gpu_day3_sequence.bat  N11 LoRA + 165-track benchmark (~8ч)
```

---

## ═══════════════════════════════════════════════════════
## ЧАСТЬ 1: CPU ЗАДАЧИ — ВСЕ ВЫПОЛНЕНЫ (CPU Day 2, 2026-03-21)
## ═══════════════════════════════════════════════════════
## Кроме: C4 UniverSR (ждёт скачки) + arXiv Overleaf (ручной)
## ═══════════════════════════════════════════════════════

### ЗАДАЧА C1: noesis_caption_trainer.py — Windows invariants fix [BLOCKER]

**Проблема:** `AutoTokenizer.from_pretrained()` и `AutoModelForCausalLM.from_pretrained()`
вызываются без `local_files_only=True`. DataLoader не проверен на `num_workers`.

**Файл:** `noesis_caption_trainer.py`

**Шаги:**

1. Найти все `from_pretrained(` — добавить `local_files_only=True` где отсутствует:
```python
# Найти:
tokenizer = AutoTokenizer.from_pretrained(
    str(self.BASE_MODEL),
# Заменить на:
tokenizer = AutoTokenizer.from_pretrained(
    str(self.BASE_MODEL),
    local_files_only=True,   # §INV-OFFLINE
```

2. Найти `DataLoader(` или создание датасета — добавить/проверить:
```python
# Если есть DataLoader:
loader = DataLoader(dataset, batch_size=..., num_workers=0, pin_memory=False)
# §INV-WIN-1: num_workers=0 (Windows spawn deadlock)
# §INV-WIN-2: pin_memory=False (pinning-thread deadlock)
```

3. Добавить в начало `train()`:
```python
import os
os.environ.setdefault("HF_HUB_OFFLINE", "1")
os.environ.setdefault("TRANSFORMERS_OFFLINE", "1")
```

**Verification:**
```
python_embedded\python.exe -c "
import re, pathlib
src = pathlib.Path('noesis_caption_trainer.py').read_text()
checks = {
    'local_files_only': 'local_files_only=True' in src,
    'HF_HUB_OFFLINE':  'HF_HUB_OFFLINE' in src,
    'num_workers=0':    'num_workers=0' in src or 'num_workers = 0' in src,
}
for k,v in checks.items(): print(f'  {\"OK\" if v else \"FAIL\"}: {k}')
all_ok = all(checks.values())
print('PASS' if all_ok else 'FAIL')
"
```

---

### ЗАДАЧА C2: test_fad_clap_backend.py — дорасширить до 17 тестов [4 теста]

**Файл:** `tests/test_fad_clap_backend.py`
**Текущее состояние:** 13 тестов. Нужно 17 (+4).

**Добавить 4 теста:**

```python
# В класс TestCLAPFADBackend (или вне классов если flat структура):

def test_text_alignment_method_exists():
    """compute_text_alignment метод должен существовать (GPU-pending)."""
    from qa_external.fad_clap_backend import CLAPFADBackend
    assert hasattr(CLAPFADBackend, 'compute_text_alignment'), \
        "compute_text_alignment missing — needed for CLAP text guidance"

def test_backend_model_id_constant():
    """CLAP model ID должен указывать на larger_clap_music."""
    from qa_external.fad_clap_backend import CLAPFADBackend
    b = CLAPFADBackend.__new__(CLAPFADBackend)
    # Check class has model_id attr or constant
    import qa_external.fad_clap_backend as m
    assert hasattr(m, '_MODEL_ID') or hasattr(m, 'CLAP_MODEL_ID') or \
           'larger_clap_music' in open('qa_external/fad_clap_backend.py').read(), \
        "laion/larger_clap_music model ID not found in fad_clap_backend"

def test_embed_dim_512():
    """CLAP embedding dim должен быть 512."""
    import qa_external.fad_clap_backend as m
    src = open('qa_external/fad_clap_backend.py').read()
    import re
    dims = re.findall(r'embed_dim.*?(\d+)', src) + re.findall(r'(\d+).*?embed_dim', src)
    assert any(d == '512' for d in dims), \
        "CLAP embed_dim=512 not found in fad_clap_backend"

def test_version_constant_in_version_py():
    """FAD CLAP backend version должен быть в version.py."""
    import version
    assert hasattr(version, 'FAD_CLAP_BACKEND_VERSION'), \
        "FAD_CLAP_BACKEND_VERSION missing from version.py"
    assert hasattr(version, 'CLAP_EMBED_DIM'), \
        "CLAP_EMBED_DIM missing from version.py"
```

**Verification:**
```
python_embedded\python.exe -m pytest tests\test_fad_clap_backend.py -q --tb=short
Ожидается: 17 passed
```

---

### ЗАДАЧА C3: Paper v14 — J-score и ablation update [TEXT EDIT]

**Файл:** TeX источник (если найден) или `NOESIS_PAPER_SUPPLEMENT_v1_0.md`

**Шаг 1:** Найти TeX:
```
dir /s /b *.tex
```
Если не найден — патч только в `docs/NOESIS_PAPER_SUPPLEMENT_v1_0.md`.

**Шаг 2 (если TeX найден):**

Найти и обновить таблицу J_mean:
```latex
% Найти строку с 0.3146 или J_mean
% Добавить footnote:
\footnote{Pre-Qwen3-inject baseline (2026-03-12). N1.2 fix applied;
Phase R re-run pending GPU (~2026-03-21). Expected J improvement.}
```

Обновить Abstract:
```latex
% Найти: J\geq 0.65
% Заменить на: J\geq 0.65 (target; Phase R re-run scheduled post-GPU)
```

Обновить §5.3 Ablation:
```latex
% Добавить строку в ablation table:
N1.2 Qwen3-inject & qwen3\_encode\_fn: None→injected & J improvement (Phase R pending) \\
```

**Шаг 3 (всегда):** Обновить `docs/NOESIS_PAPER_SUPPLEMENT_v1_0.md`:
```
Найти: J_mean = 0.3146
Добавить после: (pre-N1.2 baseline; N1.2 fix applied 2026-03-12; Phase R re-run pending)

Найти: MasteringChain v1.8
Заменить на: MasteringChain v3.2 (27 stages)

Добавить раздел:
## Quality Improvements (2026-03-18)
- Stage 0.5: UniverSR SR (ICASSP 2026, MIT, 4 ODE steps, 2GB VRAM)
- Stage 9.5: NeuralEQ taxonomy-driven FxNorm (1049 taxonomy entries)
- Stage 16.5: TransientShaper (Ozone Stabilizer analogue, Bello 2005)
- Re-vocoding: DisCoder (MUSHRA 88.14) → BigVGAN v2 fallback
- Expected quality lift: ~4.5/10 → ~8.0/10 (after GPU + DisCoder)
```

---

### ЗАДАЧА C4: UniverSR activation [1 СТРОКА — после скачки весов]

**После того как скачаются модели:**
```
dir models\UniverSR-audio
dir models\UniverSR-speech
```
**Если директории не пустые:**

```
python_embedded\python.exe -c "
from qa_external.universr_backend import get_universr_backend
b = get_universr_backend('audio', 'cuda')
print('available:', b.available)
if b.available:
    print('UniverSR-audio READY — enable_universr=True')
else:
    print('Still downloading or wrong path')
"
```

**Включить в генерацию:**
```python
# В start_noesis.bat или config:
sdk = NoesisSDK(enable_universr=True, enable_discoder=True, device="cuda")
```

---

## ═══════════════════════════════════════════════════════
## ЧАСТЬ 2: GPU ПОСЛЕДОВАТЕЛЬНОСТЬ (~2026-03-21)
## ═══════════════════════════════════════════════════════

```
ЗАПУСКАТЬ ТОЛЬКО ПОСЛЕ ОСВОБОЖДЕНИЯ GPU.
Проверить: tasklist | findstr python — SVC training завершён.
SVC checkpoint: artifacts/noesis_svc_v1/model_10000.pth
```

### GPU ДЕНЬ 1 (~2.5ч) — run_gpu_day1_sequence.bat

**G1.1: Phase R re-run с MasteringChain v3.2** (~2ч)
```
python_embedded\python.exe phase_r_direct.py
```
Ожидается: 10 WAV + 10 JSONL в `artifacts/`.
J_mean > 0.40 (v3.4: все 11 Ozone modules 10/10 активны, DAFx-24 stereo).
Записать в `BENCHMARK_RESULTS.md` новую строку:
```
| 2026-03-21 | X.XXXX | X.XXXX | MasteringChain v3.4 + DAFx-24 + Ozone10/10 |
```

**G1.2: QA.1 FAD(PANN)** (~30 мин)
```
python_embedded\python.exe run_qa1.py
```
Ожидается: `artifacts/qa1_results.jsonl` с FAD(PANN) числом.

**G1.3: UniverSR A/B тест** (~30 мин, если модели скачаны)
```
python_embedded\python.exe -c "
import numpy as np, soundfile as sf
from qa_external.universr_backend import get_universr_backend
import glob

backend = get_universr_backend('audio', 'cuda')
results = []
for wav in glob.glob('artifacts/reref2_*.wav')[:5]:
    audio, sr = sf.read(wav, dtype='float32')
    if audio.ndim == 1: audio = np.vstack([audio, audio])
    result = backend.enhance(audio, sr)
    results.append({'file': wav, 'applied': result.applied, 'snr': result.snr_db})
    print(f'{wav}: applied={result.applied} snr={result.snr_db:.1f}dB')

import json
with open('artifacts/universr_ab_test.jsonl', 'w') as f:
    for r in results: f.write(json.dumps(r) + '\n')
print('UniverSR A/B test done')
"
```

---

### GPU ДЕНЬ 2 (~10ч) — run_gpu_day2_sequence.bat

**G2.1: FAD(CLAP-MA) Reference Build** (~30 мин)
```
run_gpu_day2_sequence.bat
```
Ожидается: `artifacts/clap_ref_dj.npz` (референс 8764 треков).
Добавить в `BENCHMARK_RESULTS.md` строку FAD(CLAP-MA).

**G2.2: N1.4 MOS Jumbo finetune** (~8ч)
```
python_embedded\python.exe qa_external\noesis_mos_jumbo_trainer.py ^
    --dataset artifacts/noesis_mos/train_fma_small.jsonl ^
    --val_dataset artifacts/noesis_mos/val_fma_small.jsonl ^
    --ckpt_dir artifacts/noesis_mos_jumbo ^
    --epochs 20 --batch_size 16
```
Promotion criterion: r ≥ 0.86 → `artifacts/noesis_mos_jumbo/noesis_mos_jumbo.pt`.
Если r < 0.86 после 20 эпох: сохранить best, пересмотреть lr.

---

### GPU ДЕНЬ 3 (~8ч) — run_gpu_day3_sequence.bat

**G3.1: N11 LoRA train** (~4ч)
```
python_embedded\python.exe voice\n11_lora_trainer.py train
```
Ожидается: `artifacts/n11_lora/n11_lora_v1.pt`, MOS_vocal ≥ 3.5.

**После n11_lora_v1.pt готов — vocal endpoints живые:**
```
python_embedded\python.exe main.py &
curl -X POST http://localhost:8765/api/vocal/synth \
     -H "Content-Type: application/json" \
     -d "{\"lyrics\":\"test\",\"genre\":\"pop\",\"seed\":42}"
Ожидается: 200 OK с audio_b64
```

**G3.2: 165-track benchmark** (~4ч)
```
python_embedded\python.exe benchmark_165.py --generate
python_embedded\python.exe benchmark_msr_eval.py
```
Ожидается: 33×5=165 WAV в `gradio_outputs/benchmark_165/`.
SI-SDR/SDR/PESQ в JSONL. Добавить сводку в `BENCHMARK_RESULTS.md`.


---

## QWEN MODELS STATUS (2026-03-20)

```
BUG FIX ВЫПОЛНЕН:
  n4_macro_planner.py: local_files_only=True добавлен в _load_model()
  (раньше AutoTokenizer/AutoModelForCausalLM вызывались без этого флага)
  §INV-OFFLINE теперь соблюдён для обеих моделей.

Qwen3.5-0.8B (models/Qwen3.5-0.8B):
  ✓ N4MacroPlanner LLM mode (deterministic fallback если нет модели)
  ✓ Caption LoRA base (noesis_caption_trainer.py — ~2ч CPU)
  ✓ local_files_only=True FIXED 2026-03-20
  NEXT: запустить noesis_caption_trainer.py (CPU, сейчас)

Qwen3-0.6B (models/qwen3_lm):
  ✓ LLMMasteringAdvisor (advisory, не в main chain)
  ✓ local_files_only=True уже был

prompt_taxonomy: НЕ LLM — TF-IDF 1049 entries (CPU-only)

N3.G Qwen3.5-4B: DEFERRED (RTX 4090+)
```

---

### GPU НОЧИ 4-8 (опционально): DCAE decoder fine-tune

```
python_embedded\python.exe tools\dcae_decoder_finetune.py ^
    --steps 50000 --batch_size 4 --lr 1e-4 --lora_rank 8
```
Checkpoint каждые 5000 шагов: `artifacts/dcae_decoder_lora/step_XXXXXX.pt`.
Promotion: val_loss < 0.002 → `models/dcae_decoder_lora_v1/`.
Expected: +4..+6 MUSHRA, -6...-8dB HF noise.

---

### GPU (отдельная ночь): N2.4 SVC finetune

```
run_svc_finetune.bat
```
Ожидается: `artifacts/noesis_svc_v1/` — веса N2.4 + N3.4 voice_changer полный.

---

## ═══════════════════════════════════════════════════════
## ЧАСТЬ 3: PENDING FEATURES (после GPU)
## ═══════════════════════════════════════════════════════

### Phase ITO — Reference-guided mastering

**Файл:** `mastering/ito_style_transfer.py` (существует, DEFERRED)

**Activation condition:** пользователь предоставляет референс-трек.
```python
# Включить в MasteringChain:
chain = MasteringChain(
    ito_reference_path="path/to/reference.wav",
    enable_ito=True,
)
```
Нужен трек от пользователя для тестирования.

---

### SFT model A/B тест

**Создать:** `run_sft_ab_test.bat`
```batch
@echo off
:: ACE-Step SFT vs Turbo: 3 жанра, guidance_scale=4.0, steps=25
:: Сравнить IQS, MOS, субъективно

python_embedded\python.exe phase_r_direct.py ^
    --model acestep-sft ^
    --guidance_scale 4.0 ^
    --inference_steps 25 ^
    --output_dir artifacts/sft_ab_test
echo SFT run done

python_embedded\python.exe phase_r_direct.py ^
    --model acestep-v15-turbo ^
    --guidance_scale 7.0 ^
    --inference_steps 8 ^
    --output_dir artifacts/turbo_ab_test
echo Turbo run done

python_embedded\python.exe -c "
import json, glob, numpy as np
sft_results   = [json.loads(l) for f in glob.glob('artifacts/sft_ab_test/*.jsonl') for l in open(f)]
turbo_results = [json.loads(l) for f in glob.glob('artifacts/turbo_ab_test/*.jsonl') for l in open(f)]
if sft_results and turbo_results:
    print(f'SFT   IQS mean: {np.mean([r.get(\"IQS\",0) for r in sft_results]):.4f}')
    print(f'Turbo IQS mean: {np.mean([r.get(\"IQS\",0) for r in turbo_results]):.4f}')
"
```

---

## ═══════════════════════════════════════════════════════
## ЧАСТЬ 4: arXiv PDF ФИНАЛИЗАЦИЯ (после GPU Day 1)
## ═══════════════════════════════════════════════════════

После GPU Day 1 (J_mean известен):

**Вариант А — WSL pdflatex:**
```
wsl pdflatex NOESIS_DHCF_FNO_v14.tex
wsl bibtex NOESIS_DHCF_FNO_v14
wsl pdflatex NOESIS_DHCF_FNO_v14.tex
wsl pdflatex NOESIS_DHCF_FNO_v14.tex
```

**Вариант Б — Overleaf:**
```
Загрузить: noesis_v13_source.zip + references.bib + патч из Задачи C3
Compile → Download PDF
```

**Финал:** `NOESIS_DHCF_FNO_v14.pdf`
Submission: arXiv cs.SD + eess.AS

---

## КОНТРОЛЬНЫЕ ЧЕКЛИСТЫ

### CPU задачи (перед GPU):
```
[x] C1: noesis_caption_trainer.py — local_files_only + num_workers=0 + pin_memory=False  DONE 2026-03-18
[x] C2: test_fad_clap_backend.py — 17→21 tests PASS                                      DONE 2026-03-21
[x] C3: Paper SUPPLEMENT v1.2 §7 Quality, SVC=DONE, 84 rules, 38 profiles                DONE 2026-03-21
[ ] C4: UniverSR активирован — run tools\activate_universr.py после скачки
[ ] run_sft_ab_test.bat создан (script in plan, write when GPU ready)
[x] test_constants.py + test_autoeval_param_schema.py PASS (каждый раз)

# CPU DAY 2 (2026-03-21) — ALL DONE:
[x] expand_caption_dataset.py → 414→579 pairs (taxonomy_expand_v1)
[x] taxonomy/entries_rare.py — 55 entries, 11 genres, registered in orchestrator
[x] sc_text_controller.py — 36→84 rules (EDM/vocal/emotional/platform/genre)
[x] genre_mastering_master_profiles.py — 29→38 profiles
[x] DriftBadge.tsx — TIER_CF_LIMIT 2.0→6.0 (Drift v1.3 BUG FIX)
[x] CreatePanel.tsx — DisCoder toggle + taxonomy preview widget
[x] api.ts + noesis_rest_api.py — POST /api/taxonomy/search endpoint
[x] version.py — FAD_CLAP_BACKEND_VERSION + CLAP_EMBED_DIM + CLAP_MODEL_ID
[x] CLAUDE.md / AGENTS.md / BENCHMARK_RESULTS.md / ROADMAP v0.30 updated
[x] generate_paper_figures.py — 31 stages, R.REF2 J values
[~] Caption LoRA — RUNNING (artifacts/caption_lora/caption_lora_v1.pt, ~2h CPU)
[ ] arXiv Overleaf — paper v14 PDF (manual, Overleaf needed)
```

### После GPU Day 1 (~2026-03-21):
```
[ ] SVC training done: artifacts/noesis_svc_v1/model_10000.pth
[ ] Phase R re-run: J_mean > 0.3146, 10/10 WAV, все PASS/CF_LIM
[ ] QA.1 FAD(PANN): artifacts/qa1_results.jsonl
[ ] UniverSR A/B: artifacts/universr_ab_test.jsonl
[ ] BENCHMARK_RESULTS.md обновлён с новой строкой
```

### После GPU Day 2:
```
[ ] FAD(CLAP-MA): artifacts/clap_ref_dj.npz + FAD число
[ ] N1.4 Jumbo: artifacts/noesis_mos_jumbo/noesis_mos_jumbo.pt, r ≥ 0.86
[ ] BENCHMARK_RESULTS.md: FAD(CLAP-MA) строка добавлена
```

### После GPU Day 3:
```
[ ] N11 LoRA: artifacts/n11_lora/n11_lora_v1.pt
[ ] /api/vocal/synth: curl 200 OK с audio_b64
[ ] /api/vocal/change: curl 200 OK
[ ] 165-track benchmark: 165 WAV + JSONL
[ ] arXiv v14 PDF готов
```

### GPU ночи (DCAE fine-tune):
```
[ ] step_050000.pt сохранён
[ ] val_loss < 0.002 → promote → models/dcae_decoder_lora_v1/
[ ] MasteringChain v3.3: enable_dcae_lora=True
```

---

## МЕТРИКИ УСПЕХА

```
Метрика                    v1.0 план      Сейчас         Цель GPU
─────────────────────────────────────────────────────────────────
Tests PASS                 ~462           782+           850+
MasteringChain             v1.8 22st      v3.2 27st      v3.2 (stable)
Stage 0.5 UniverSR         MISSING        CODE DONE      ACTIVE (models ready)
Stage 9.5 NeuralEQ         MISSING        DONE taxonomy  VERIFIED
Stage 16.5 TransientShaper MISSING        DONE           VERIFIED
DisCoder cascade           MISSING        CODE DONE      ACTIVE
VRAMOrchestrator           MISSING        DONE           TESTED
DCAE fine-tune script      MISSING        DONE           RUN (5 nights)
API fixes                  BROKEN         ALL FIXED      STABLE
Phase R J_mean             0.3146         —              >0.50 (est)
FAD(CLAP-MA)               0              —              <50 (est)
N1.4 Jumbo r               0              trainer DONE   ≥0.86
n11_lora_v1.pt             MISSING        —              READY
/vocal/synth live          NO             routed/no wts  YES
165-track benchmark        PLAN           —              DONE
arXiv v14                  v13 ready      C3 patch ready SUBMITTED
Quality score              ~4.5/10        ~7.0/10        ~8.5/10
```

---

## БЫСТРЫЙ СТАРТ (для новой сессии Claude Code)

```
:: Проверить baseline перед любой работой:
python_embedded\python.exe -m pytest tests\test_constants.py tests\test_autoeval_param_schema.py -q

:: Версия и состояние:
python_embedded\python.exe -c "
import version as v
print(f'Chain: {v.MASTERING_CHAIN_VERSION}')
print(f'Tests: {v.TESTS_PASS_COUNT}')
print(f'Roadmap: {v.ROADMAP_VERSION}')
"

:: UniverSR статус:
python_embedded\python.exe -c "
from qa_external.universr_backend import get_universr_backend
b = get_universr_backend('audio', 'cpu')
print('UniverSR-audio available:', b.available)
"
```

---

## АРХИВ ИЗМЕНЕНИЙ v1.0 → v2.0

```
ДОБАВЛЕНО в v2.0:
  + MasteringChain v3.2 полный stage map (27 stages)
  + UniverSR Stage 0.5 (ICASSP 2026 MIT)
  + NeuralEQ Stage 9.5 taxonomy-driven
  + TransientShaper Stage 16.5 Ozone Stabilizer
  + VRAMOrchestrator 5-phase cascade
  + DCAE decoder fine-tune (GPU ночи 4-8)
  + Все API fixes (generate.py, router_vocal, main.py, start_noesis.bat)
  + SFT model A/B тест скрипт (C pending)
  + Phase ITO activation instructions
  + UniverSR A/B тест в GPU Day 1
  + Точные метрики: 782 tests, v3.2 chain, v0.28 roadmap

ОБНОВЛЕНО в v2.0:
  + Метрики успеха: 450→782 tests, v1.8→v3.2 chain
  + Чеклист CPU day 3: ROADMAP v0.26→v0.28 (уже создан)
  + GPU Day 1: добавлен UniverSR A/B тест
  + GPU НОЧИ: DCAE fine-tune (5 nights)

ВЫПОЛНЕНО из v1.0 (не повторяется):
  ✓ diagnose_silence_regression.py
  ✓ StemAwareNode.process_audio() FIXED
  ✓ noesis_mos_jumbo_trainer.py
  ✓ test_noesis_mos_jumbo.py (12 tests)
  ✓ test_n34_voice_changer.py (10 tests)
  ✓ test_heart_transcriptor_backend.py (8 tests)
  ✓ ROADMAP_v0_28.md
  ✓ run_gpu_day1/2/3_sequence.bat
  ✓ _CL_MAX_PASSES 4→2
  ✓ knee_db 1.0→3.0
  ✓ vocals high_gain_db +0.5→+3.5
  ✓ sdk.generate() → sdk.generate_audio()
```

---

*Document: яNOESIS_MASTER_PLAN_v2_0.md*
*Version:    v2.2  (2026-03-21, CPU Day 2 complete)*
*Author: Ilia Bolotnikov / AMAImedia.com*
*Status: ACTIVE — CPU Phase remaining + GPU Phase*
*Supersedes: яNOESIS_MASTER_PLAN_v1_0.md*
