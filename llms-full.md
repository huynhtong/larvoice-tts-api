# Larvoice TTS API LLM Guide

This guide is optimized for AI agents and coding assistants integrating with Larvoice.

Larvoice creates audio from text using a user-selected or user-uploaded voice. The API supports quick previews, async jobs, batch jobs, SRT output, usage tracking, and voice management.

Base URL:

```text
https://api.larvoice.com
```

## Authentication

Use a user API key in every authenticated request:

```http
x-api-key: <your-api-key>
```

Users can get an API key from:

```text
https://t.me/larvoice_api_bot
```

User integrations only need `x-api-key`.

## Recommended Flow

1. Check account with `GET /v1/me`.
2. Fetch supported languages with `GET /v1/languages`.
3. Upload a voice with `POST /v1/voices`, or use a known HTTPS `ref_audio_url`.
4. Prefer `voice_id` after upload.
5. Optionally edit voice `name` and `ref_text` with `PATCH /v1/voices/:id`.
6. Preview with `POST /v1/tts/preview` or `POST /v1/voices/:id/preview`.
7. Create async job with `POST /v1/tts`, or many jobs with `POST /v1/tts/batch`.
8. Poll job with `GET /v1/tts/jobs/:job_id` until `completed` or `failed`.
9. Use `audio_url` and optional `srt_url` from completed job.
10. Check usage with `GET /v1/usage`.

## Endpoint Summary

```http
GET    /health
GET    /v1/me
GET    /v1/usage
GET    /v1/languages

POST   /v1/tts
POST   /v1/tts/preview
POST   /v1/tts/batch
GET    /v1/tts/jobs
GET    /v1/tts/jobs/:job_id
POST   /v1/tts/jobs/:job_id/cancel

POST   /v1/voices
GET    /v1/voices
GET    /v1/voices/:id
POST   /v1/voices/:id/preview
PATCH  /v1/voices/:id
PUT    /v1/voices/:id
DELETE /v1/voices/:id
```

## Account

```bash
curl 'https://api.larvoice.com/v1/me' \
  -H 'x-api-key: <your-api-key>'
```

Response includes quota:

```json
{
  "data": {
    "id": "...",
    "name": "customer-a",
    "key_prefix": "lv_xxxxxxxx",
    "character_limit": 1000000,
    "characters_used": 12000,
    "remaining_characters": 988000,
    "status": "active",
    "created_at": "2026-06-20T12:00:00.000Z",
    "last_used_at": "2026-06-21T12:00:00.000Z",
    "expires_at": null
  }
}
```

## Languages

```bash
curl 'https://api.larvoice.com/v1/languages'
```

Response:

```json
{
  "data": [
    {
      "code": "vi",
      "name": "Vietnamese",
      "native_name": "Tiếng Việt",
      "default": true,
      "supports_srt": true,
      "supports_voice_clone": true
    },
    {
      "code": "en",
      "name": "English",
      "native_name": "English",
      "default": false,
      "supports_srt": true,
      "supports_voice_clone": true
    }
  ],
  "meta": {
    "default_language": "vi"
  }
}
```

Use `language: "vi"` unless the user asks for English.

## Upload Voice

Use multipart form data:

```bash
curl -X POST 'https://api.larvoice.com/v1/voices' \
  -H 'x-api-key: <your-api-key>' \
  -F 'name=Giọng nữ bán hàng' \
  -F 'file=@voice-sample.mp3'
```

Response:

```json
{
  "data": {
    "voice_id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
    "name": "Giọng nữ bán hàng",
    "ref_text": "Xin chào, đây là đoạn giọng mẫu của tôi.",
    "ref_audio_url": "https://api.larvoice.com/ref-audio/<file>.mp3",
    "file_name": "<file>.mp3",
    "content_type": "audio/mpeg",
    "size_bytes": 123456,
    "max_reference_seconds": 2.5,
    "voice_count": 1,
    "max_voices": 50
  }
}
```

After upload, prefer `voice_id` for TTS requests.

Limits:

- File max: 25 MB.
- Formats: mp3, wav, m4a, aac, ogg, webm.

## Edit Voice

Use this when the user wants a better display name or wants to correct the transcript/reference text.

```bash
curl -X PATCH 'https://api.larvoice.com/v1/voices/<voice_id>' \
  -H 'Content-Type: application/json' \
  -H 'x-api-key: <your-api-key>' \
  -d '{
    "name": "Giọng nữ bán hàng",
    "ref_text": "Xin chào, đây là đoạn giọng mẫu của tôi."
  }'
```

Response:

```json
{
  "data": {
    "id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
    "name": "Giọng nữ bán hàng",
    "ref_text": "Xin chào, đây là đoạn giọng mẫu của tôi.",
    "updated_at": "2026-06-21T12:00:00.000Z"
  }
}
```

`PUT /v1/voices/:id` accepts the same body.

## List Voices

```bash
curl 'https://api.larvoice.com/v1/voices' \
  -H 'x-api-key: <your-api-key>'
```

## Get Voice

```bash
curl 'https://api.larvoice.com/v1/voices/<voice_id>' \
  -H 'x-api-key: <your-api-key>'
```

## Preview TTS

Use preview before long jobs. Preview returns `audio_url` directly.

```bash
curl -X POST 'https://api.larvoice.com/v1/tts/preview' \
  -H 'Content-Type: application/json' \
  -H 'x-api-key: <your-api-key>' \
  -d '{
    "voice_id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
    "text": "Xin chào, đây là bản nghe thử.",
    "language": "vi",
    "output_format": "wav"
  }'
```

Response:

```json
{
  "data": {
    "audio_url": "https://api.larvoice.com/files/tts_20260621_120010_123456.wav?expires=...&signature=...",
    "content_type": "audio/wav",
    "duration_seconds": 2.18,
    "expires_at": "2026-06-21T12:00:10.000000",
    "return_srt": false,
    "character_count": 34
  }
}
```

Preview max text: 500 characters.

## Preview Voice

Use this when the user is editing or selecting one saved voice.

```bash
curl -X POST 'https://api.larvoice.com/v1/voices/<voice_id>/preview' \
  -H 'Content-Type: application/json' \
  -H 'x-api-key: <your-api-key>' \
  -d '{
    "text": "Đây là bản nghe thử của voice này.",
    "language": "vi",
    "output_format": "wav"
  }'
```

Response shape is the same as `POST /v1/tts/preview`.

## Create TTS Job

Use this for normal or long text. The response is async: poll the `status_url`.

```bash
curl -X POST 'https://api.larvoice.com/v1/tts' \
  -H 'Content-Type: application/json' \
  -H 'x-api-key: <your-api-key>' \
  -d '{
    "gen_text": "Nội dung cần tạo giọng nói.",
    "voice_id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
    "language": "vi",
    "return_srt": true,
    "output_format": "wav"
  }'
```

Response:

```json
{
  "data": {
    "job_id": "9f6c2131-0f7c-41e4-90f2-ae3c7f336d00",
    "status": "queued",
    "character_count": 34,
    "status_url": "https://api.larvoice.com/v1/tts/jobs/9f6c2131-0f7c-41e4-90f2-ae3c7f336d00"
  }
}
```

## Batch TTS

Use for video chapters, course lessons, IVR prompts, and multi-scene content. Each item becomes one job.

```bash
curl -X POST 'https://api.larvoice.com/v1/tts/batch' \
  -H 'Content-Type: application/json' \
  -H 'x-api-key: <your-api-key>' \
  -d '{
    "voice_id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
    "language": "vi",
    "return_srt": true,
    "output_format": "wav",
    "items": [
      { "id": "intro", "gen_text": "Xin chào, đây là phần mở đầu." },
      { "id": "chapter_1", "gen_text": "Bây giờ chúng ta bắt đầu chương một." },
      { "id": "outro", "gen_text": "Cảm ơn bạn đã lắng nghe." }
    ]
  }'
```

Response:

```json
{
  "data": {
    "batch_size": 3,
    "total_character_count": 96,
    "jobs": [
      {
        "id": "intro",
        "job_id": "11111111-1111-1111-1111-111111111111",
        "status": "queued",
        "character_count": 34,
        "status_url": "https://api.larvoice.com/v1/tts/jobs/11111111-1111-1111-1111-111111111111"
      }
    ]
  }
}
```

Batch max: 50 items.

## Poll Job

```bash
curl 'https://api.larvoice.com/v1/tts/jobs/<job_id>' \
  -H 'x-api-key: <your-api-key>'
```

Statuses:

| Status | Meaning |
| --- | --- |
| `queued` | Job is waiting. |
| `processing` | Audio is being created. |
| `completed` | Audio is ready. Use `audio_url`. |
| `failed` | Job failed or was cancelled. Check `error`. |

Completed response includes `audio_url`. If `return_srt` was true, it also includes `srt_url` and `segments`.

## Cancel Job

Only queued jobs can be cancelled.

```bash
curl -X POST 'https://api.larvoice.com/v1/tts/jobs/<job_id>/cancel' \
  -H 'x-api-key: <your-api-key>'
```

Response:

```json
{
  "data": {
    "job_id": "9f6c2131-0f7c-41e4-90f2-ae3c7f336d00",
    "status": "failed",
    "character_count": 34,
    "error": {
      "code": "cancelled",
      "message": "Job cancelled by user"
    },
    "cancelled": true
  }
}
```

## Usage

```bash
curl 'https://api.larvoice.com/v1/usage?from=2026-06-01&to=2026-06-21' \
  -H 'x-api-key: <your-api-key>'
```

Use this for dashboards. It returns quota, voice counts, job counts, characters, and daily usage. Range max: 370 days.

## Request Fields

Common TTS fields:

| Field | Type | Notes |
| --- | --- | --- |
| `gen_text` | string | Text for async jobs. Required for `/v1/tts`. |
| `text` | string | Alias for preview endpoints. |
| `voice_id` | string | Preferred after uploading a voice. |
| `ref_audio_url` | string | HTTPS voice sample URL. Use when not using `voice_id`. |
| `language` | string | `vi` or `en`. Default: `vi`. |
| `output_format` | string | `wav` or `mp3`. Default: `wav`. |
| `return_srt` | boolean | Return SRT and segments for async jobs. |
| `post_speed` | number | 0.5 to 2. |
| `post_pitch` | number | -6 to 6. |
| `post_volume` | number | -20 to 20. |

Avoid sending unknown fields. Unknown fields are rejected with `400 validation_error`.

## Limits

| Resource | Limit |
| --- | --- |
| Preview text | 500 characters |
| Job text | 10000 characters per job by default |
| Batch items | 50 items |
| Voice upload file | 25 MB |
| Rate limit | 50 requests/minute/API key |

## Error Shape

```json
{
  "error": {
    "code": "validation_error",
    "message": "Invalid request",
    "details": [
      {
        "field": "gen_text",
        "message": "Parameter must be a non-empty string"
      }
    ]
  },
  "request_id": "01J..."
}
```

Common error codes:

| HTTP | Code | Meaning |
| --- | --- | --- |
| 400 | `validation_error` | Invalid body or parameter. |
| 401 | `unauthorized` | Missing or invalid API key. |
| 402 | `quota_exceeded` | Not enough character quota. |
| 403 | `voice_limit_exceeded` | Voice limit reached. |
| 404 | `not_found` | Job, voice, or file not found. |
| 409 | `job_already_processing` | Job cannot be cancelled anymore. |
| 413 | `payload_too_large` | File or JSON body too large. |
| 429 | `rate_limited` | Too many requests. |
| 502 | `tts_failed` | Audio could not be created. |
| 504 | `tts_timeout` | Audio creation timed out. |

## Agent Rules

- Prefer `voice_id` over `ref_audio_url` after a voice is uploaded.
- Use preview before long generation.
- Use async `/v1/tts` for production audio.
- Use `/v1/tts/batch` for multiple clips.
- Poll jobs until `completed` or `failed`.
- Never expose or log full API keys.
- Do not send unknown request fields.

## MCP Integration Guide

Larvoice includes a local MCP server in this repo. Use the user's API key as an environment variable in the MCP server, then call Larvoice through MCP tools.

Recommended MCP tools:

| Tool | Purpose | Endpoint |
| --- | --- | --- |
| `larvoice_get_account` | Read quota and account status. | `GET /v1/me` |
| `larvoice_get_usage` | Read usage for dashboards or summaries. | `GET /v1/usage` |
| `larvoice_list_languages` | Read supported languages. | `GET /v1/languages` |
| `larvoice_upload_voice` | Upload a voice file and create `voice_id`. | `POST /v1/voices` |
| `larvoice_list_voices` | List saved voices. | `GET /v1/voices` |
| `larvoice_get_voice` | Read one voice. | `GET /v1/voices/:id` |
| `larvoice_update_voice` | Edit voice name or ref text. | `PATCH /v1/voices/:id` |
| `larvoice_preview_tts` | Create a short preview audio. | `POST /v1/tts/preview` |
| `larvoice_preview_voice` | Preview a saved voice. | `POST /v1/voices/:id/preview` |
| `larvoice_create_tts_job` | Create one async TTS job. | `POST /v1/tts` |
| `larvoice_create_tts_batch` | Create many async TTS jobs. | `POST /v1/tts/batch` |
| `larvoice_get_job` | Poll one job. | `GET /v1/tts/jobs/:job_id` |
| `larvoice_list_jobs` | List jobs. | `GET /v1/tts/jobs` |
| `larvoice_cancel_job` | Cancel a queued job. | `POST /v1/tts/jobs/:job_id/cancel` |

MCP config shape:

```json
{
  "mcpServers": {
    "larvoice": {
      "command": "node",
      "args": ["<repo>/mcp/larvoice-mcp.mjs"],
      "env": {
        "LARVOICE_API_KEY": "lv_your_key",
        "LARVOICE_BASE_URL": "https://api.larvoice.com"
      }
    }
  }
}
```

Local run command:

```bash
LARVOICE_API_KEY=lv_your_key npm run mcp
```

Suggested tool inputs and outputs:

### larvoice_preview_tts

Input:

```json
{
  "voice_id": "string",
  "text": "string",
  "language": "vi",
  "output_format": "wav"
}
```

Output:

```json
{
  "audio_url": "string",
  "content_type": "audio/wav",
  "duration_seconds": 2.18,
  "character_count": 34
}
```

### larvoice_create_tts_job

Input:

```json
{
  "voice_id": "string",
  "gen_text": "string",
  "language": "vi",
  "return_srt": true,
  "output_format": "wav"
}
```

Output:

```json
{
  "job_id": "string",
  "status": "queued",
  "character_count": 34,
  "status_url": "string"
}
```

### larvoice_create_tts_batch

Input:

```json
{
  "voice_id": "string",
  "language": "vi",
  "return_srt": true,
  "output_format": "wav",
  "items": [
    { "id": "intro", "gen_text": "Xin chào." }
  ]
}
```

Output:

```json
{
  "batch_size": 1,
  "total_character_count": 9,
  "jobs": [
    {
      "id": "intro",
      "job_id": "string",
      "status": "queued",
      "character_count": 9,
      "status_url": "string"
    }
  ]
}
```

MCP wrapper behavior:

- Store the API key in environment config, not in prompts.
- Return `audio_url`, `srt_url`, `job_id`, and `status_url` as plain strings.
- For async jobs, do not wait forever. Poll with a reasonable timeout or return `status_url` to the caller.
- For file uploads, accept a local file path or file bytes, then send multipart form data to `POST /v1/voices`.
- Surface Larvoice error `code`, `message`, and `details` directly to the user.
