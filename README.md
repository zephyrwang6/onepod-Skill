# xiaoyuzhou-to-article

把小宇宙（Xiaoyuzhou FM）播客链接转成**逐字文稿**和**文章**的 Claude Code Skill。

实现原理：下载小宇宙音频 → Groq Whisper 转录成文字 → 由 Claude 编写成可读文章。

> 转写部分改编自 [Panniantong/agent-reach](https://github.com/Panniantong/agent-reach) 的小宇宙处理方案。

## 它能做什么

输入一条小宇宙单集链接（`xiaoyuzhoufm.com/episode/xxx`），输出：

1. **逐字文稿** —— Whisper 转录 + 可选的中文标点润色
2. **文章** —— Claude 在文稿基础上改写成结构化、可读的文章

## 实现链路

```
小宇宙链接
  → 抓页面 HTML 里的音频 CDN 直链（无需登录/API）
  → curl 下载音频
  → ffmpeg 转码压缩（单声道 64k mp3）
  → 按 10 分钟/段切片（避免单段过长导致 Whisper 524 超时）
  → Groq Whisper large-v3 转录（language=zh，带 429/5xx 自动重试）
  → 清洗 Whisper 幻觉噪声（"请点赞订阅"、"字幕由 Amara.org 提供"等）
  → Groq Llama 3.3 70B 补中文标点+分段（可选 --polish）
  → 输出逐字文稿
  → Claude 把文稿编写成文章
```

关键点：小宇宙把音频 CDN 直链（`media.xyzcdn.net/*.m4a|mp3`）直接写在单集页面的 HTML 里，`curl` 即可拿到，**无需登录、无需官方 API**。

## 前置要求

**1. 命令行工具**：`curl`、`ffmpeg`、`ffprobe`、`perl`、`python3`

- macOS：`brew install ffmpeg`
- 若 brew 不可用（如 Xcode 许可证未接受），下载静态版二进制放到 `~/.local/bin/`（脚本会自动把该目录加入 PATH）。macOS arm64 静态版可从 [osxexperts.net](https://www.osxexperts.net/) 获取 `ffmpeg` 与 `ffprobe`。
- （可选）`pip3 install certifi`，避免润色步骤的 SSL 证书校验失败。

**2. Groq API Key（免费）** —— 转录必需：

1. 注册 <https://console.groq.com>，创建一个 `gsk_` 开头的 key
2. `export GROQ_API_KEY=gsk_xxxxx`（建议写进 `~/.zshrc`）

## 直接用脚本

```bash
export GROQ_API_KEY=gsk_xxxxx
bash scripts/transcribe.sh --polish "https://www.xiaoyuzhoufm.com/episode/xxxx" "/tmp/transcript.md"
```

- `--polish`：转录后用 Groq Llama 3.3 70B 补中文标点+分段（Whisper 对中文标点较弱，开启后更易读）。
- 第二个参数为输出路径，省略则默认 `/tmp/podcast_transcript.txt`。

## 作为 Claude Skill 使用

把整个目录放进 `~/.claude/skills/xiaoyuzhou-to-article/`，然后对 Claude 说：

> 把这个小宇宙链接转成文章 https://www.xiaoyuzhoufm.com/episode/xxxx

Claude 会运行转录脚本，再把文稿改写成文章。详见 [SKILL.md](SKILL.md)。

## 已知限制

- 仅支持小宇宙**单集页面**（`/episode/`），不支持播客主页。
- 音频直链靠正则从页面 HTML 提取；若小宇宙改版导致提取失败，需更新 `scripts/transcribe.sh` 里的正则。
- 转录用 Groq 免费额度，大量使用可能触发速率限制（脚本会自动等待重试）。
- 幻觉噪声清洗是启发式的，可能漏掉新的噪声模式，可按需在 `clean_transcript` 里补规则。

## 致谢

- 小宇宙转写方案改编自 [agent-reach](https://github.com/Panniantong/agent-reach)。
- 转录：[Groq](https://groq.com) 的 Whisper large-v3 与 Llama 3.3 70B。

## License

MIT
