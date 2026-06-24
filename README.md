<h1 align="center">🎙️ onepod-Skill</h1>

<p align="center">
  <strong>播客处理全家桶 — 6 个 Claude Code Skill，覆盖发现、转录、提炼、写作全链路</strong>
</p>

<p align="center">
  <a href="LICENSE"><img src="https://img.shields.io/badge/License-MIT-blue.svg?style=for-the-badge" alt="MIT License"></a>
</p>

---

## 它能做什么

把 YouTube / 小宇宙播客变成可读的文章，一句话搞定。

```
"获取播客更新"          → 自动检查 30 个 YouTube 频道的新视频
"处理这个播客 [链接]"    → 提取字幕 → 提炼要点 → 输出文章
"小宇宙转文章 [链接]"    → 音频转录 → 输出文稿+文章
```

## 包含的 Skills

| Skill | 功能 | 触发词 |
|-------|------|--------|
| [**youtube-feed**](youtube-feed/) | 监控 30 个 YouTube 频道，获取最新更新 | "获取播客更新"、"YouTube 更新" |
| [**youtube-transcript-cn**](youtube-transcript-cn/) | 提取 YouTube 视频字幕，输出中文文字稿 | "YouTube 字幕"、"视频转文字" |
| [**content-digest**](content-digest/) | 长内容 → 短摘要 + 叙事文章（4 阶段提炼） | 提供视频/播客/文章内容，要求总结 |
| [**podcast-workflow**](podcast-workflow/) | 端到端编排：发现 → 字幕 → 提炼 → 保存 | "处理这个播客"、"播客工作流" |
| [**podcast-script-generator**](podcast-script-generator/) | 文稿 → 视频口播脚本 | "口播脚本"、"改成口播" |
| [**xiaoyuzhou-to-article**](xiaoyuzhou-to-article/) | 小宇宙播客音频 → 文稿 + 文章（Groq Whisper） | "小宇宙转文字"、发来 xiaoyuzhoufm.com 链接 |

## 处理链路

```
┌─────────────────────────────────────────────────────────┐
│  发现层                                                  │
│                                                         │
│  youtube-feed ──────────┐                               │
│  (30 个 YouTube 频道)     │                               │
│                          ▼                               │
│                  podcast-workflow（编排器）                │
│                       │                                  │
│  xiaoyuzhou-to-article─┘  (小宇宙独立路径)                │
├────────────────────────┤─────────────────────────────────┤
│  提取层                 │                                 │
│                        │                                 │
│  youtube-transcript-cn ◄┘  (YouTube 字幕 API)            │
│  transcribe.sh             (小宇宙 Groq Whisper)         │
├──────────────────────────────────────────────────────────┤
│  处理层                                                  │
│                                                         │
│  content-digest (4 阶段分析 → 短摘要 + 长文章)            │
├──────────────────────────────────────────────────────────┤
│  输出层                                                  │
│                                                         │
│  ├─► Obsidian                                           │
│  ├─► 飞书知识库                                          │
│  └─► podcast-script-generator (→ 视频口播脚本)           │
└──────────────────────────────────────────────────────────┘
```

## 安装

把每个 skill 目录软链接（或复制）到 `~/.claude/skills/`：

```bash
git clone https://github.com/zephyrwang6/onepod-Skill.git ~/.onepod-Skill

# 逐个链接到 Claude Code skills 目录
for skill in youtube-feed youtube-transcript-cn content-digest podcast-workflow podcast-script-generator xiaoyuzhou-to-article; do
  ln -sfn ~/.onepod-Skill/$skill ~/.claude/skills/$skill
done
```

### 前置依赖

| 依赖 | 用途 | 安装 |
|------|------|------|
| Python 3.10+ | 字幕提取、频道监控 | 系统自带或 `brew install python` |
| `youtube-transcript-api` | YouTube 字幕 | `pip install youtube-transcript-api` |
| `ffmpeg` + `ffprobe` | 小宇宙音频处理 | `brew install ffmpeg` |
| `GROQ_API_KEY` | 小宇宙 Whisper 转录（免费） | [console.groq.com](https://console.groq.com) 获取 |

## 致谢

- 小宇宙转写方案改编自 [agent-reach](https://github.com/Panniantong/agent-reach)
- 转录：[Groq](https://groq.com) Whisper large-v3 与 Llama 3.3 70B

## License

MIT
