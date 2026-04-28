# 豆包 Seed ASR 2.0 流式语音识别

> 将本地音频文件（m4a / wav / mp3 / aac / ogg）转写为文字，基于豆包语音识别 2.0 API（WebSocket 流式协议）。

## 功能特点

- **流式识别**：实时通过 WebSocket 发送音频并接收转写结果
- **自动格式转换**：自动将输入音频转换为 16kHz mono 16bit PCM
- **代理支持**：自动识别 `http_proxy` / `https_proxy` 环境变量
- **Claude Code Skill**：可直接用 `/doubao-asr` 在 Claude Code 中调用

## 安装

### 依赖

```bash
pip install websocket-client python-socks
```

音频格式转换（m4a / mp3 / ogg）需要 ffmpeg：
```bash
# macOS
brew install ffmpeg

# Ubuntu/Debian
sudo apt install ffmpeg
```

### Claude Code Skill 模式（推荐）

```bash
# Clone 到 Claude Code skills 目录
git clone https://github.com/Chauncy-Guo/doubao-asr.git ~/.claude/skills/doubao-asr

# 然后直接使用
/doubao-asr /path/to/audio.m4a --out result.md
```

### 独立 CLI 使用

```bash
# Clone
git clone https://github.com/Chauncy-Guo/doubao-asr.git
cd doubao-asr

# 配置凭证
export DOUBAO_APP_ID="<YOUR_APP_ID>"
export DOUBAO_ACCESS_TOKEN="<YOUR_ACCESS_TOKEN>"

# 运行
python3 doubao_asr.py /path/to/audio.m4a --out result.md
```

## 凭证配置

| 方式 | 设置方法 |
|------|----------|
| **环境变量**（推荐） | `DOUBAO_APP_ID`, `DOUBAO_ACCESS_TOKEN`, `DOUBAO_SECRET_KEY` |
| **命令行参数** | `--app-id`, `--access-token` |
| **代码默认值** | 直接修改 `doubao_asr.py` 顶部的 `APP_ID` / `ACCESS_TOKEN` / `SECRET_KEY` |

豆包 API 凭证申请：https://console.volcengine.com/ark

## CLI 参数

```bash
python3 doubao_asr.py <audio_path> [选项]

位置参数:
  audio_path              音频文件路径（支持 m4a/wav/mp3/aac/ogg/pcm）

选项:
  --out PATH              输出 Markdown 文件路径（默认打印到 stdout）
  --app-id TEXT           豆包 App ID
  --access-token TEXT      豆包 Access Token
  --resource-id TEXT       API Resource ID（默认: volc.seedasr.sauc.duration）
```

## 使用示例

```bash
# 基本转写
python3 doubao_asr.py recording.m4a

# 输出到文件
python3 doubao_asr.py recording.m4a --out transcript.md

# 指定凭证
python3 doubao_asr.py recording.m4a \
  --app-id YOUR_APP_ID \
  --access-token YOUR_TOKEN \
  --out transcript.md

# Claude Code 中使用
/doubao-asr /Users/me/recordings/meeting.m4a --out meeting.md
```

## 项目结构

```
doubao-asr/
├── doubao_asr.py   # 主程序（CLI + WebSocket 客户端）
├── skill.json      # Claude Code skill 元数据
└── README.md      # 本文档
```

## 技术细节

- **协议**：WebSocket + 二进制自定义帧（4字节头 + payload）
- **端点**：`wss://openspeech.bytedance.com/api/v3/sauc/bigmodel_async`
- **音频**：16kHz / mono / 16bit PCM，每帧 640 bytes（20ms）
- **响应**：累计文本通过 `result.text` 返回

## License

MIT
