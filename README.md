# memobot-gemini-relay (memU bot 专用版)

这是一个高效、轻量的 Go 语言中继服务器，旨在让 **memU bot** 能够无缝使用 **Google Gemini API**。它通过将标准的 OpenAI 或 Anthropic (Claude) 请求协议转译为 Gemini 原生格式，解决了协议不兼容的问题。

## ✨ 特性

- **memU bot 深度适配**: 自动处理 memU bot 发出的 `/v1/messages` (Anthropic) 或 `/v1/chat/completions` (OpenAI) 请求。
- **协议转换**: 将各种 API 格式的消息流完整映射至 Gemini `generateContent` 接口。
- **内置代理**: 支持 `--proxy` 参数，方便在中国大陆等网络环境下通过本地代理访问 Google 服务。
- **极简运行**: 无需配置复杂的环境变量，启动即用。

## ⚙️ memU bot 配置指南

在 memU bot 的设置界面中，请按下图进行配置：

| 配置项 | 内容 |
| :--- | :--- |
| **LLM 提供商** | `Custom Provider` |
| **API 地址** | `http://127.0.0.1:6300/v1` |
| **API 密钥** | `你的 Google Gemini API Key` |
| **模型名称** | `gemini-3-flash-preview` (或其它 Gemini 模型) |

## 🚀 快速开始

### 运行
**基本运行**:
```bash
./memobot-gemini-relay
```

windows 直接运行 memubot-gemini-relay-windows.exe

**使用代理运行**:
```bash
./memobot-gemini-relay --proxy http://127.0.0.1:7890
```

**调试模式 (查看详细数据包)**:
```bash
./memobot-gemini-relay --debug
```

### go环境运行
```bash
go run memubot-gemini-relay.go
```

### 编译
```bash
go mod init memubot-gemini-relay && go build -o memubot-gemini-relay . && rm go.mod
GOOS=windows GOARCH=amd64 go build -o memubot-gemini-relay-windows.exe memubot-gemini-relay.go
```

## 🖥️ 运行效果
启动后，你会看到如下提示：
```text
用于 memU bot 的 Gemini API 中继工具
memU bot 设置如下：
----------------------------------
 LLM 提供商：Custom Provider
 API 地址：http://127.0.0.1:6300/
 API 密钥：【Gemini api key】
 模型名称：gemini-3-flash-preview
----------------------------------
使用 --proxy 让请求通过代理转发
如 --proxy http://127.0.0.1:7890
当前正在中继Gemini api
```

## 许可证
MIT License
