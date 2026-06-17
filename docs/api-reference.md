# API Reference

The NOESIS API (by amaimedia) is job-based: **submit → poll → retrieve**. Long-running
jobs (dubbing, video edit) return a `job_id`; poll `job_status` until `succeeded` and read
the `result_url`.

> Engine and model details are intentionally omitted. This documents the public surface only.

## Authentication

API key in the request header. Request access at **info@amaimedia.com**.

## Endpoints

### `dub_video` — flagship
Dub a video into a target language. Multi-speaker, voice-preserving, lip-synced.

| Param | Type | Notes |
|---|---|---|
| `file_url` | string | source video (required) |
| `target_lang` | string | e.g. `es`, `ru`, `zh` (required) |
| `quality` | string | `balanced` (default) · other tiers available |

→ Returns `{ job_id }`. Poll `job_status`.

### `clone_voice`
Clone a voice from a short reference and synthesize speech.

| Param | Type | Notes |
|---|---|---|
| `reference_audio_url` | string | 5–30 s reference (required) |
| `text` | string | text to speak (required) |
| `lang` | string | target language (required) |

→ Returns a URL to a **48 kHz** WAV.

### `edit_video`
Auto-cut a long video into short vertical clips.

| Param | Type | Notes |
|---|---|---|
| `file_url` | string | source video (required) |
| `mode` | string | `shorts` (default) |
| `count` | number | clips to produce (default 10) |

→ Returns `{ project_id }`. Poll `job_status`.

### `generate_music`
| Param | Type | Notes |
|---|---|---|
| `prompt` | string | description (required) |
| `duration` | number | seconds (default 60) |

### `generate_image`
| Param | Type | Notes |
|---|---|---|
| `prompt` | string | description (required) |
| `width` / `height` | number | default 1024 |
| `steps` | number | default 8 |
| `seed` | number | default −1 (random) |

### `job_status`
| Param | Type | Notes |
|---|---|---|
| `job_id` | string | required |

→ Returns `status` (`queued` · `running` · `succeeded` · `failed`), `progress` %,
and `result_url` when done. Suggested polling: ~5 s for dubbing, ~2 s for video edit.

### `health`
Liveness check. No parameters.

## Typical flow

```
job = dub_video(file_url=…, target_lang="es")
while True:
    s = job_status(job.job_id)
    if s.status == "succeeded": break
    sleep(5)
download(s.result_url)
```
