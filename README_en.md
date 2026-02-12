# memubot-openai-relay (memU bot Special Edition)

This is a high-efficiency, lightweight Go language relay server designed to enable **memU bot** to seamlessly use the **OpenAI-Compatible Model API**. It solves protocol incompatibility issues by translating standard OpenAI or Anthropic (Claude) request protocols into the OpenAI-Compatible Model native format.

## ✨ Features

- **memU bot Deep Adaptation**: Automatically handles `/v1/messages` (Anthropic) or `/v1/chat/completions` (OpenAI) requests sent by memU bot.
- **Protocol Conversion**: Maps message streams in various API formats completely to the OpenAI-Compatible Model native format.
- **🔧 Function Call Support**: Fully supports Anthropic/MiniMax style tool calls (`tool_use`/`tool_result`).

- **Built-in Proxy**: Supports the `--proxy` parameter, facilitating access to models through a local proxy in network environments like mainland China.
- **TPM Rate Limiting**: Supports `--tpm` parameter (e.g., `0.9M`), using a token bucket algorithm to smooth request rates and prevent API rate limits.
- **Minimalist Operation**: No complex environment variable configuration required, ready to use upon startup.

## ⚙️ memU bot Configuration Guide

In the settings interface of memU bot, please configure as shown below:

| Configuration Item | Content |
| :--- | :--- |
| **LLM Provider** | `Custom Provider` |
| **API Address** | `http://127.0.0.1:6300/` |
| **API Key** | `OpenAI-Compatible Model API Key` |
| **Model Name** | `Pro/deepseek-ai/DeepSeek-V3.2` (or other OpenAI compatible models) |

## 🚀 Quick Start

### Running
**Basic Run**:
```bash
chmod +x ./memubot-openai-relay
./memubot-openai-relay --url https://api.siliconflow.cn/v1/chat/completions
```

The `--url` is followed by the actual model address.

For Windows, directly run `memubot-openai-relay-windows.exe`.

**Run with Proxy**:
```bash
./memubot-openai-relay --proxy http://127.0.0.1:7890
```

**Enable TPM Rate Limiting (Prevent 429 Errors)**:
```bash
./memubot-openai-relay --tpm 0.9M  # Limit to 900k tokens/minute
./memubot-openai-relay --tpm 1000000 # Limit to 1M tokens/minute
```

**Debug Mode (View detailed packets)**:
```bash
./memubot-openai-relay --debug
```

### Run with Go Environment
```bash
go run memubot-openai-relay.go
```

### Compilation
```bash
go mod init memubot-openai-relay && go build -o memubot-openai-relay . && rm go.mod
GOOS=windows GOARCH=amd64 go build -o memubot-openai-relay-windows.exe memubot-openai-relay.go
```

## 🔧 Function Call Support

Supports Anthropic/MiniMax style tool definitions:
```json
{"name": "bash", "description": "...", "input_schema": {...}}
```

### Available Tools List (19 items)

| # | Tool Name | Description |
|---|---------|------|
| 1 | `bash` | Execute bash commands |
| 2 | `str_replace_editor` | File view/edit (view/create/str_replace/insert) |
| 3 | `download_file` | Download file from URL |
| 4 | `web_search` | Web search (Tavily AI) |
| 5 | `macos_launch_app` | Launch macOS application |
| 6 | `macos_mail` | Apple Mail operations |
| 7 | `macos_calendar` | Apple Calendar operations |
| 8 | `macos_contacts` | Apple Contacts query |
| 9 | `feishu_send_text` | Send Feishu text message |
| 10 | `feishu_send_image` | Send Feishu image |
| 11 | `feishu_send_file` | Send Feishu file |
| 12 | `feishu_send_card` | Send Feishu message card |
| 13 | `feishu_delete_chat_history` | Delete Feishu chat history |
| 14 | `service_create` | Create background service |
| 15 | `service_list` | List all services |
| 16 | `service_start` | Start service |
| 17 | `service_stop` | Stop service |
| 18 | `service_delete` | Delete service |
| 19 | `service_get_info` | Get service information |

### Test Prompt

| Test Tool | Prompt |
| :--- | :--- |
| `bash` | `Check what files are on my desktop` |
| `str_replace_editor` | `Help me create a test.txt file on the desktop with content "Hello!"` |
| `web_search` | `Search for today's weather` |
| `download_file` | `Download this image and save to desktop: https://example.com/image.png` |
| `macos_launch_app` | `Open Calendar app` |
| `macos_contacts` | `Search contacts for someone named "Zhang San"` |
| `macos_mail` | `Check how many unread emails I have` |
| `feishu_send_text` | `Send me a message saying "Test successful!"` |
| `feishu_send_card` | `Send a green message card with title "Test Report"` |
| `service_list` | `List what background services I have running now` |
| Combined Test | `Check the content of test.txt on the desktop, then send it to me via Feishu` |

### Notes

1. **New Conversation Start Test**: It is recommended to clear the conversation history and restart to ensure `thought_signature` is passed correctly.
2. **Thinking Mode**: Ignores returned `thinking` content.
3. **Debug Mode**: Use `--debug` to view complete request/response data.

## 📦 Context Caching
 
OpenAI/DeepSeek has its own caching logic (e.g. automatic disk caching for prompts > 1024 tokens), which cannot be manually configured via this relay.


## 🚦 TPM Rate Limiting

To address TPM (Tokens Per Minute) limits on certain models, this tool includes a built-in **token bucket algorithm** for smooth rate limiting.

### How to Enable
Use the `--tpm` parameter to specify the rate limit, supporting `K/M` suffixes or raw numbers:
```bash
./memubot-openai-relay --tpm 0.9M     # 900,000 tokens/min
./memubot-openai-relay --tpm 2000000  # 2,000,000 tokens/min
```

### Mechanism

1. **Estimated Deduction**: Before sending a request, tokens are roughly estimated and deducted based on the JSON Body size (bytes/3).
2. **Smooth Waiting**: If tokens are insufficient, the program calculates the wait time and automatically blocks (Sleeps) before sending the request.
3. **Safe Correction**: Only deducts extra tokens if estimated too low; over-estimations are not refunded, serving as a safety buffer.
4. **429 Smart Throttling**:
   - On standard 429 error: Additional tokens are deducted, and a 61-second cooldown is enforced.
   - On `"Resource has been exhausted"` error: Triggers a 30-minute throttling mode, forcing a 61-second interval between requests.
5. **Output Control**: When TPM is enabled, a 1-second forced wait precedes each request, and `maxOutputTokens` is capped at 4000.

> [!TIP]
> It is recommended to set this to 90% of the model's TPM limit (e.g., set `0.9M` for a 1M limit) to provide a safety buffer.
## 🖥️ Running Effect
After startup, you will see the following prompt:
```text
     用于 memU bot 的 OpenAI-Compatible API 中继工具
               memU bot 中配置如下：
--------------------------------------------------------
        LLM 提供商：Custom Provider
        API 地址：http://127.0.0.1:6300/
        API 密钥：【OpenAI-Compatible api key】
        模型名称：【OpenAI-Compatible-reasoner】
--------------------------------------------------------
[ ] --debug 显示处理状态
[ ] --proxy 代理，如 --proxy http://127.0.0.1:7890
[ ] --tpm 速率限制，如 --tpm 0.9M
[✓] --url https://api.siliconflow.cn/v1/chat/completions
--------------------------------------------------------
当前正在中继 OpenAI-Compatible API
```

## License
MIT License
