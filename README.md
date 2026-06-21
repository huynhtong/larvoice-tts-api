# Larvoice TTS API

Text-to-Speech API tiếng Việt dành cho production.

Tạo giọng nói AI chất lượng cao từ văn bản thông qua HTTP API đơn giản. Hỗ trợ voice cloning, xử lý bất đồng bộ và khả năng mở rộng cho ứng dụng, sản phẩm SaaS, AI Agent và hệ thống tự động hóa.

Khác với LarVoice.com dành cho người dùng cuối, LarVoice API được xây dựng dành riêng cho lập trình viên và tích hợp hệ thống.

### Tính năng

* Tạo giọng nói từ văn bản tiếng Việt và tiếng Anh và nhiều ngôn ngữ sẽ cập nhật trong tương lai
* Voice Cloning từ file audio mẫu
* HTTP REST API đơn giản
* Xử lý bất đồng bộ cho khối lượng lớn
* File audio và SRT tự động tạo
* Signed URL bảo mật
* Quản lý voice riêng cho từng tài khoản
* Quota và usage tracking theo API key

### Quy trình hoạt động

1. Upload hoặc chọn voice mẫu
2. Gửi nội dung văn bản tới API
3. Nhận `job_id`
4. Theo dõi trạng thái xử lý
5. Nhận file audio khi hoàn thành

```text
Text → Job → Audio Output
```

API chạy bất đồng bộ: gửi text để tạo job, poll trạng thái cho đến khi có audio.

```text
https://api.larvoice.com
```

## Mục lục

- [Xác thực](#xác-thực)
- [Nhận API key miễn phí](#nhận-api-key-miễn-phí)
- [Endpoint](#endpoint)
- [Quick start](#quick-start)
- [Tạo job TTS](#tạo-job-tts)
- [Nghe thử TTS](#nghe-thử-tts)
- [Tạo nhiều job TTS](#tạo-nhiều-job-tts)
- [Kiểm tra trạng thái job](#kiểm-tra-trạng-thái-job)
- [Danh sách job](#danh-sách-job)
- [Thông tin tài khoản](#thông-tin-tài-khoản)
- [Thống kê usage](#thống-kê-usage)
- [Danh sách ngôn ngữ](#danh-sách-ngôn-ngữ)
- [Upload voice mẫu](#upload-voice-mẫu)
- [Quản lý voice](#quản-lý-voice)
- [Tham số request](#tham-số-request)
- [File output](#file-output)
- [Rate limit và quota](#rate-limit-và-quota)
- [Lỗi thường gặp](#lỗi-thường-gặp)
- [Chính sách dữ liệu](#chính-sách-dữ-liệu)
- [LLM và MCP](#llm-và-mcp)
- [Playground](#playground)

## Xác thực

Gửi API key trong header `x-api-key`:

```http
x-api-key: <your-api-key>
```

API key chỉ xem được job do chính key đó tạo. Nếu mất key, hãy tạo key mới trong Larvoice Mini App.

## Nhận API key miễn phí

Người dùng có thể nhận API key miễn phí qua Telegram bot Larvoice:

```text
https://t.me/larvoice_api_bot
```

Cách lấy key:

1. Mở Telegram.
2. Tìm bot `@larvoice_api_bot` hoặc mở link trên.
3. Bấm `Start`.
4. Mở Larvoice Mini App từ bot.
5. Đăng nhập bằng Telegram trong Mini App.
6. Sao chép API key được tạo cho tài khoản của bạn.

API key đầy đủ chỉ hiển thị khi tạo key. Hãy lưu key ở nơi an toàn. Nếu mất key hoặc nghi ngờ key bị lộ, hãy tạo key mới trong Mini App; key cũ sẽ bị vô hiệu hóa.

## Endpoint

```http
GET    /health
GET    /playground
GET    /app
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
GET    /ref-audio/:file
HEAD   /ref-audio/:file
GET    /files/:file?expires=...&signature=...
HEAD   /files/:file?expires=...&signature=...
```

## Quick start

### 1. Kiểm tra service

```bash
curl 'https://api.larvoice.com/health'
```

```json
{"ok":true}
```

### 2. Tạo job TTS

```bash
curl -X POST 'https://api.larvoice.com/v1/tts' \
  -H 'Content-Type: application/json' \
  -H 'x-api-key: <your-api-key>' \
  -d '{
    "gen_text": "Xin chào, đây là Larvoice TTS API.",
    "ref_audio_url": "https://media.publit.io/file/larvoice/voice-custom/Ngoc-Huyen-v1.mp3",
    "language": "vi",
    "return_srt": true,
    "output_format": "wav"
  }'
```

Response `202 Accepted`:

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

### 3. Poll trạng thái

```bash
curl 'https://api.larvoice.com/v1/tts/jobs/9f6c2131-0f7c-41e4-90f2-ae3c7f336d00' \
  -H 'x-api-key: <your-api-key>'
```

Khi hoàn thành:

```json
{
  "data": {
    "job_id": "9f6c2131-0f7c-41e4-90f2-ae3c7f336d00",
    "status": "completed",
    "character_count": 34,
    "audio_url": "https://api.larvoice.com/files/tts_20260620_120010_123456.wav?expires=1782043210&signature=...",
    "content_type": "audio/wav",
    "duration_seconds": 2.18,
    "srt_url": "https://api.larvoice.com/files/tts_20260620_120010_123456_aligned.srt?expires=1782043210&signature=...",
    "segments": [],
    "expires_at": "2026-06-21T12:00:10.000000",
    "created_at": "2026-06-20T12:00:00.000Z",
    "updated_at": "2026-06-20T12:00:10.000Z",
    "started_at": "2026-06-20T12:00:05.000Z",
    "completed_at": "2026-06-20T12:00:10.000Z"
  }
}
```

Khi thất bại:

```json
{
  "data": {
    "job_id": "9f6c2131-0f7c-41e4-90f2-ae3c7f336d00",
    "status": "failed",
    "character_count": 34,
    "error": {
      "code": "tts_failed",
      "message": "Không tạo được audio"
    },
    "created_at": "2026-06-20T12:00:00.000Z",
    "updated_at": "2026-06-20T12:03:00.000Z",
    "started_at": "2026-06-20T12:02:55.000Z",
    "completed_at": "2026-06-20T12:03:00.000Z"
  }
}
```

## Tạo job TTS

```http
POST /v1/tts
```

Headers:

```http
Content-Type: application/json
x-api-key: <your-api-key>
```

Có hai cách chỉ định giọng mẫu: dùng `ref_audio_url` hoặc `voice_id`.

Dùng `ref_audio_url` (URL giọng mẫu HTTPS):

```json
{
  "gen_text": "Nội dung cần tạo giọng nói.",
  "ref_audio_url": "https://media.publit.io/file/larvoice/voice-custom/Ngoc-Huyen-v1.mp3",
  "language": "vi",
  "return_srt": true,
  "output_format": "wav"
}
```

Dùng `voice_id` (voice đã upload qua `POST /v1/voices`):

```json
{
  "gen_text": "Nội dung cần tạo giọng nói.",
  "voice_id": "<voice_id>",
  "language": "vi",
  "return_srt": true,
  "output_format": "wav"
}
```

Nếu gửi cả hai, `ref_audio_url` được ưu tiên.

## Nghe thử TTS

```http
POST /v1/tts/preview
```

Dùng để nghe thử nhanh một đoạn ngắn trước khi tạo job dài. Preview tối đa `500` ký tự và trả `audio_url` trực tiếp.

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
    "audio_url": "https://api.larvoice.com/files/tts_20260620_120010_123456.wav?expires=1782043210&signature=...",
    "content_type": "audio/wav",
    "duration_seconds": 2.18,
    "expires_at": "2026-06-21T12:00:10.000000",
    "return_srt": false,
    "character_count": 34
  }
}
```

## Tạo nhiều job TTS

```http
POST /v1/tts/batch
```

Dùng khi cần tạo nhiều đoạn audio cùng voice, ví dụ video nhiều cảnh, khóa học, IVR hoặc nội dung theo chương. Các cấu hình chung như `voice_id`, `language`, `output_format`, `return_srt` có thể đặt ở cấp ngoài; mỗi item chỉ cần `id` và `gen_text`.

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

Response `202 Accepted`:

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

Mỗi item tạo một job riêng và poll bằng `status_url` như job TTS bình thường. Tối đa `50` item mỗi batch. Quota trừ theo tổng ký tự của toàn bộ item.

## Kiểm tra trạng thái job

```http
GET /v1/tts/jobs/:job_id
```

```bash
curl 'https://api.larvoice.com/v1/tts/jobs/<job_id>' \
  -H 'x-api-key: <your-api-key>'
```

| Status | Ý nghĩa |
| --- | --- |
| `queued` | Job đang chờ xử lý. |
| `processing` | Job đang được tạo audio. |
| `completed` | Job hoàn thành, response có `audio_url`. |
| `failed` | Job thất bại, response có `error.code` và `error.message`. |

### Hủy job

```http
POST /v1/tts/jobs/:job_id/cancel
```

Hủy job đang `queued`. Job đã `processing` hoặc đã hoàn thành không thể hủy.

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

## Danh sách job

```http
GET /v1/tts/jobs
```

```bash
curl 'https://api.larvoice.com/v1/tts/jobs?limit=20' \
  -H 'x-api-key: <your-api-key>'
```

Query params:

| Param | Mô tả |
| --- | --- |
| `limit` | Số job mỗi trang. Mặc định `20`, tối đa `100`. |
| `cursor` | Cursor trang kế tiếp từ response trước. |
| `status` | Lọc theo `queued`, `processing`, `completed`, `failed`. |

Response có `pagination.next_cursor` nếu còn trang tiếp theo. Thứ tự: mới nhất trước.

## Thông tin tài khoản

```http
GET /v1/me
```

Kiểm tra quota, số ký tự đã dùng, số ký tự còn lại và hạn dùng API key.

```bash
curl 'https://api.larvoice.com/v1/me' \
  -H 'x-api-key: <your-api-key>'
```

Response:

```json
{
  "data": {
    "id": "6fd50e12-52ae-4b83-a9b1-0848f64cbe15",
    "name": "customer-a",
    "key_prefix": "lv_xxxxxxxx",
    "character_limit": 100000,
    "characters_used": 3400,
    "remaining_characters": 96600,
    "status": "active",
    "created_at": "2026-06-20T12:00:00.000Z",
    "last_used_at": "2026-06-20T12:10:00.000Z",
    "expires_at": null
  }
}
```

`expires_at: null` nghĩa là API key không có ngày hết hạn cố định.

## Thống kê usage

```http
GET /v1/usage
```

Xem quota, số job, ký tự đã gửi và thống kê theo ngày. Mặc định trả 30 ngày gần nhất. Có thể lọc bằng `from` và `to` theo định dạng `YYYY-MM-DD`.

```bash
curl 'https://api.larvoice.com/v1/usage?from=2026-06-01&to=2026-06-21' \
  -H 'x-api-key: <your-api-key>'
```

Response:

```json
{
  "data": {
    "range": {
      "from": "2026-06-01",
      "to": "2026-06-21",
      "days": 21
    },
    "quota": {
      "character_limit": 1000000,
      "characters_used": 12000,
      "remaining_characters": 988000
    },
    "voices": {
      "voice_count": 3,
      "max_voices": 50
    },
    "summary": {
      "total_jobs": 12,
      "queued_jobs": 1,
      "processing_jobs": 0,
      "completed_jobs": 10,
      "failed_jobs": 1,
      "submitted_characters": 14300,
      "completed_requests": 10,
      "completed_characters": 12000
    },
    "daily": [
      {
        "date": "2026-06-21",
        "jobs": 4,
        "submitted_characters": 3200,
        "completed_requests": 3,
        "completed_characters": 2400
      }
    ]
  }
}
```

Range tối đa `370` ngày.

## Danh sách ngôn ngữ

```http
GET /v1/languages
```

Endpoint public để frontend lấy danh sách ngôn ngữ hỗ trợ.

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

## Upload voice mẫu

```http
POST /v1/voices
```

Upload file audio để tạo voice riêng cho API key. Có thể đặt tên voice bằng field `name`.

Sau khi upload, API trả về `voice_id`, `name`, `ref_text` và `ref_audio_url`. Khi tạo TTS về sau, nên dùng `voice_id` để Larvoice tự dùng đúng voice mẫu và `ref_text` đã lưu.

```bash
curl -X POST 'https://api.larvoice.com/v1/voices' \
  -H 'x-api-key: <your-api-key>' \
  -F 'name=Giọng nữ bán hàng' \
  -F 'file=@voice-sample.mp3'
```

Response `201 Created`:

```json
{
  "data": {
    "voice_id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
    "name": "Giọng nữ bán hàng",
    "ref_text": "Xin chào, đây là đoạn giọng mẫu của tôi.",
    "ref_audio_url": "https://api.larvoice.com/ref-audio/<sha256>.mp3",
    "file_name": "<sha256>.mp3",
    "content_type": "audio/mpeg",
    "size_bytes": 123456,
    "max_reference_seconds": 2.5,
    "voice_count": 1,
    "max_voices": 20
  }
}
```

File tối đa `25 MB`. Định dạng hỗ trợ: `mp3`, `wav`, `m4a`, `aac`, `ogg`, `webm`.

Sau khi upload, dùng `voice_id` trong `POST /v1/tts`:

```json
{
  "gen_text": "Nội dung cần tạo giọng nói.",
  "voice_id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
  "language": "vi",
  "output_format": "wav"
}
```

## Quản lý voice

### Danh sách voice

```http
GET /v1/voices
```

```bash
curl 'https://api.larvoice.com/v1/voices' \
  -H 'x-api-key: <your-api-key>'
```

Response:

```json
{
  "data": [
    {
      "id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
      "ref_audio_id": "...",
      "name": "Giọng nữ bán hàng",
      "ref_text": "Xin chào, đây là đoạn giọng mẫu của tôi.",
      "ref_audio_url": "https://api.larvoice.com/ref-audio/<sha256>.mp3",
      "file_name": "<sha256>.mp3",
      "content_type": "audio/mpeg",
      "size_bytes": 123456,
      "created_at": "2026-06-20T12:00:00.000Z",
      "updated_at": "2026-06-20T12:05:00.000Z",
      "last_used_at": "2026-06-20T12:10:00.000Z",
      "use_count": 5
    }
  ],
  "meta": {
    "voice_count": 1,
    "max_voices": 20
  }
}
```

### Chi tiết voice

```http
GET /v1/voices/:id
```

```bash
curl 'https://api.larvoice.com/v1/voices/<voice_id>' \
  -H 'x-api-key: <your-api-key>'
```

Response:

```json
{
  "data": {
    "id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
    "ref_audio_id": "...",
    "name": "Giọng nữ bán hàng",
    "ref_text": "Xin chào, đây là đoạn giọng mẫu của tôi.",
    "ref_audio_url": "https://api.larvoice.com/ref-audio/<sha256>.mp3",
    "file_name": "<sha256>.mp3",
    "content_type": "audio/mpeg",
    "size_bytes": 123456,
    "created_at": "2026-06-20T12:00:00.000Z",
    "updated_at": "2026-06-20T12:05:00.000Z",
    "last_used_at": "2026-06-20T12:10:00.000Z",
    "use_count": 5
  }
}
```

### Nghe thử voice

```http
POST /v1/voices/:id/preview
```

Nghe thử voice đã upload với một đoạn text ngắn. Có thể dùng `text` hoặc `gen_text`.

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

Response giống `POST /v1/tts/preview`.

### Sửa tên voice và ref text

```http
PATCH /v1/voices/:id
```

User có thể sửa `name` và `ref_text` của voice đã upload. `ref_text` nên khớp với nội dung nghe được trong voice mẫu để kết quả cloning ổn định hơn.

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
    "updated_at": "2026-06-20T12:05:00.000Z"
  }
}
```

Có thể dùng `PUT /v1/voices/:id` với body giống `PATCH`.

### Xóa voice

```http
DELETE /v1/voices/:id
```

```bash
curl -X DELETE 'https://api.larvoice.com/v1/voices/<voice_id>' \
  -H 'x-api-key: <your-api-key>'
```

Response:

```json
{
  "data": {
    "id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
    "deleted": true
  }
}
```

## Tham số request

### Field bắt buộc

| Field | Kiểu | Mô tả |
| --- | --- | --- |
| `gen_text` | string | Nội dung cần tạo giọng nói. Tối đa `10000` ký tự mỗi job. Quota trừ theo số ký tự. |
| `ref_audio_url` | string | URL giọng mẫu HTTPS. Bắt buộc nếu không có `voice_id`. |
| `voice_id` | string | UUID voice đã upload qua `POST /v1/voices`. Bắt buộc nếu không có `ref_audio_url`. Khi dùng `voice_id`, Larvoice tự dùng `ref_text` đã lưu của voice. |

### Field tùy chọn

| Field | Kiểu | Mặc định | Giới hạn | Mô tả |
| --- | --- | --- | --- | --- |
| `language` | string | `vi` | `vi`, `en` | Ngôn ngữ. |
| `output_format` | string | `wav` | `wav`, `mp3` | Định dạng audio đầu ra. |
| `return_srt` | boolean | `false` | | Trả thêm file SRT và segments. |
| `ref_text` | string | Theo voice đã lưu | Tối đa `2000` ký tự | Nội dung tham chiếu cho voice mẫu. Thường không cần gửi nếu dùng `voice_id`. |
| `post_speed` | number | `1.1` | `0.5`..`2` | Tốc độ hậu xử lý. |
| `post_pitch` | number | `0` | `-6`..`6` | Chỉnh cao độ hậu xử lý. |
| `post_volume` | number | `0.1` | `-20`..`20` | Tăng/giảm âm lượng hậu xử lý. |
| `speed` | number | `0.8` | `0.5`..`2` | Tốc độ tạo giọng (khác `post_speed`). |

Các field lạ sẽ bị reject `400 validation_error`. `output_audio: true` không được hỗ trợ; dùng `audio_url` sau khi job hoàn thành.

## File output

Khi job `completed`, dùng `audio_url` để tải file audio. Nếu request có `return_srt: true`, response có thêm `srt_url` và `segments`.

URL output có dạng:

```text
https://api.larvoice.com/files/<file>?expires=<unix>&signature=<hmac>
```

- Link tải có thời hạn, tối đa `24` giờ.
- Nếu link hết hạn, API trả `403 Forbidden`.
- Có thể dùng `HEAD` để kiểm tra metadata file trước khi tải.

## Rate limit và quota

- Mỗi API key có quota ký tự riêng.
- Mỗi request TTS trừ quota theo số ký tự trong `gen_text`.
- Nếu job thất bại sau khi hết retry, quota được hoàn lại.
- Rate limit: `50` request/phút/API key.
- Job retry tối đa `3` lần. Job `processing` quá `5` phút sẽ được recover hoặc fail.

## Lỗi thường gặp

Error response chuẩn:

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

| HTTP | Code | Ý nghĩa |
| --- | --- | --- |
| `400` | `validation_error` | Body sai hoặc thiếu field. |
| `401` | `unauthorized` | Thiếu hoặc sai API key. |
| `402` | `quota_exceeded` | Không đủ quota ký tự. |
| `403` | `forbidden` | Không được phép gọi endpoint này. |
| `403` | `voice_limit_exceeded` | Vượt giới hạn số voice upload. |
| `404` | `not_found` | Không tìm thấy job, voice hoặc file. |
| `413` | `payload_too_large` | File upload hoặc body JSON quá lớn. |
| `429` | `rate_limited` | Vượt rate limit. |
| `502` | `tts_failed` | Không tạo được audio. |
| `504` | `tts_timeout` | Xử lý quá thời gian cho phép. |

## Chính sách dữ liệu

- Không lưu trữ dài hạn nội dung text khách hàng gửi lên.
- Dữ liệu job, audio output và SRT output chỉ tồn tại tối đa `24` giờ.
- Output chỉ truy cập được bằng signed URL; không có public file listing.
- API key chỉ hiển thị một lần khi được cấp; hệ thống chỉ lưu hash.
- Dữ liệu khách hàng không được bán, chia sẻ cho bên thứ ba, hoặc dùng để huấn luyện model.
- Khi cần xóa dữ liệu sớm, liên hệ Larvoice kèm `job_id` hoặc API key liên quan.

## LLM và MCP

Larvoice có tài liệu riêng cho AI agent, coding assistant và workflow automation:

```text
https://api.larvoice.com/llms.txt
https://api.larvoice.com/llms-full.md
```

`llms.txt` là bản index ngắn cho agent discovery. `llms-full.md` là hướng dẫn tích hợp đầy đủ, gồm flow khuyến nghị, endpoint, request mẫu, response mẫu, limits và error shape.

Repo này có MCP server local để user tích hợp Larvoice vào MCP client:

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

Chạy thử local:

```bash
LARVOICE_API_KEY=lv_your_key npm run mcp
```

Nếu xây MCP server hoặc custom tool wrapper cho AI agent, nên map các tool user-facing sau:

| MCP tool gợi ý | Endpoint |
| --- | --- |
| `larvoice_get_account` | `GET /v1/me` |
| `larvoice_get_usage` | `GET /v1/usage` |
| `larvoice_list_languages` | `GET /v1/languages` |
| `larvoice_upload_voice` | `POST /v1/voices` |
| `larvoice_list_voices` | `GET /v1/voices` |
| `larvoice_get_voice` | `GET /v1/voices/:id` |
| `larvoice_update_voice` | `PATCH /v1/voices/:id` |
| `larvoice_preview_tts` | `POST /v1/tts/preview` |
| `larvoice_preview_voice` | `POST /v1/voices/:id/preview` |
| `larvoice_create_tts_job` | `POST /v1/tts` |
| `larvoice_create_tts_batch` | `POST /v1/tts/batch` |
| `larvoice_get_job` | `GET /v1/tts/jobs/:job_id` |
| `larvoice_list_jobs` | `GET /v1/tts/jobs` |
| `larvoice_cancel_job` | `POST /v1/tts/jobs/:job_id/cancel` |

Nguyên tắc cho AI agent:

- Dùng `voice_id` sau khi upload voice.
- Dùng preview trước khi tạo job dài.
- Dùng batch khi cần nhiều đoạn audio.
- Poll job đến khi `completed` hoặc `failed`.
- Không log API key đầy đủ.
- Không gửi field lạ ngoài docs.

## Playground

Trải nghiệm API trực tiếp tại:

```text
https://api.larvoice.com/playground
```

Giọng demo có sẵn:

| Giọng | Ngôn ngữ | URL |
| --- | --- | --- |
| Ngọc Huyền | vi | `https://media.publit.io/file/larvoice/voice-custom/Ngoc-Huyen-v1.mp3` |
| Anh Quân | vi | `https://media.publit.io/file/larvoice/voice-custom/AnhQuan.mp3` |
| Adam | en | `https://media.publit.io/file/larvoice/voice-custom/Adam.mp3` |
| Vy Anh | vi | `https://media.publit.io/file/larvoice/voice-custom/VyAnh.mp3` |
