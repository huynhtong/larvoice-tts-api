# Larvoice MCP

Larvoice MCP package giúp kết nối Larvoice Text-to-Speech API với Cursor, Codex, Claude Desktop, Kiro, Antigravity, OpenClaw, Hermes và các MCP client hỗ trợ command server.

Package npm:

```bash
npx -y @larvoice/mcp
```

## Auth

Cách dễ nhất là chạy lệnh login:

```bash
npx -y @larvoice/mcp login
```

Lệnh này mở một trang local trong browser để user dán API key. Key được kiểm tra và lưu cục bộ tại:

```text
~/.larvoice/mcp.json
```

Sau đó MCP client có thể chạy:

```json
{
  "mcpServers": {
    "larvoice": {
      "command": "npx",
      "args": ["-y", "@larvoice/mcp"]
    }
  }
}
```

Cách nâng cao: điền API key trong phần `env` của MCP config:

```text
LARVOICE_API_KEY=lv_your_key
```

`LARVOICE_BASE_URL` có thể bỏ qua nếu dùng Larvoice cloud mặc định.

## Cursor

Thêm vào MCP config của Cursor:

```json
{
  "mcpServers": {
    "larvoice": {
      "command": "npx",
      "args": ["-y", "@larvoice/mcp"],
      "env": {
        "LARVOICE_API_KEY": "lv_your_key",
        "LARVOICE_BASE_URL": "https://api.larvoice.com"
      }
    }
  }
}
```

## Codex

Thêm vào MCP config của Codex:

```toml
[mcp_servers.larvoice]
command = "npx"
args = ["-y", "@larvoice/mcp"]

[mcp_servers.larvoice.env]
LARVOICE_API_KEY = "lv_your_key"
LARVOICE_BASE_URL = "https://api.larvoice.com"
```

## Claude Desktop

Thêm vào MCP config của Claude Desktop:

```json
{
  "mcpServers": {
    "larvoice": {
      "command": "npx",
      "args": ["-y", "@larvoice/mcp"],
      "env": {
        "LARVOICE_API_KEY": "lv_your_key",
        "LARVOICE_BASE_URL": "https://api.larvoice.com"
      }
    }
  }
}
```

## Kiro, Antigravity, OpenClaw, Hermes

Nếu client hỗ trợ MCP qua command, dùng:

```text
command: npx
args: -y @larvoice/mcp
env:
  LARVOICE_API_KEY=lv_your_key
  LARVOICE_BASE_URL=https://api.larvoice.com
```

## Chạy thử

```bash
LARVOICE_API_KEY=lv_your_key npx -y @larvoice/mcp
```

Nếu terminal hiện dòng sau là server đã khởi động đúng:

```text
Larvoice MCP server running with base URL https://api.larvoice.com
```

Khi chạy trực tiếp trong terminal, process sẽ đứng chờ. Đây là hành vi bình thường vì MCP server chờ MCP client giao tiếp qua `stdio`.

## Tools

| Tool | Mục đích |
| --- | --- |
| `larvoice_get_account` | Xem quota và trạng thái tài khoản. |
| `larvoice_get_usage` | Xem thống kê sử dụng. |
| `larvoice_list_languages` | Xem ngôn ngữ hỗ trợ. |
| `larvoice_upload_voice` | Upload voice mẫu và tạo `voice_id`. |
| `larvoice_list_voices` | Xem danh sách voice đã lưu. |
| `larvoice_get_voice` | Xem chi tiết một voice. |
| `larvoice_update_voice` | Sửa tên voice hoặc ref text. |
| `larvoice_delete_voice` | Xóa voice. |
| `larvoice_preview_tts` | Nghe thử một đoạn TTS ngắn. |
| `larvoice_preview_voice` | Nghe thử voice đã lưu. |
| `larvoice_create_tts_job` | Tạo một job TTS. |
| `larvoice_create_tts_batch` | Tạo nhiều job TTS. |
| `larvoice_get_job` | Kiểm tra trạng thái một job. |
| `larvoice_list_jobs` | Xem danh sách job. |
| `larvoice_cancel_job` | Hủy job đang chờ. |

## Gợi ý dùng với AI agent

- Gọi `larvoice_get_account` trước để kiểm tra API key và quota.
- Upload voice một lần, sau đó dùng `voice_id` cho các job sau.
- Dùng `larvoice_preview_tts` hoặc `larvoice_preview_voice` trước khi tạo nội dung dài.
- Dùng `larvoice_create_tts_batch` cho nhiều đoạn audio như video nhiều cảnh, khóa học, IVR hoặc nội dung theo chương.
- Poll job bằng `larvoice_get_job` đến khi `completed` hoặc `failed`.
- Không đưa API key vào prompt, log hoặc nội dung chat.
