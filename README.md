# FactReel — 中文知识短视频 Skill

**给一个选题，还你一条有出处的 40 秒知识短视频。**

An Agent Skill that turns a topic into a **fact-checked** 30–45s Chinese knowledge video:
verified facts → script beats → voiceover-timed scenes → karaoke captions → Remotion render → publish pack.

![License: MIT](https://img.shields.io/badge/License-MIT-yellow)
![Format: Agent Skill](https://img.shields.io/badge/Format-Agent%20Skill-blue)
![Video: 1080×1920](https://img.shields.io/badge/Video-1080%C3%971920-brightgreen)
![API key: none required](https://img.shields.io/badge/API%20key-none%20required-success)

<p align="center">
  <img src="docs/demo.gif" width="320" alt="FactReel demo：中国大模型混战史（时间线卡片逐张点亮，字幕逐字高亮）">
</p>

<p align="center"><sub>实拍 demo：选题「中国大模型混战史」→ 46.9 秒成片节选（原片 1080×1920，此 GIF 为 360×640 / 8 秒）</sub></p>

| 钩子帧 | 中段信息帧 | 结尾帧（结论 + 站队问题 + 来源行） |
|---|---|---|
| ![钩子帧](docs/still-hook.jpg) | ![中段帧](docs/still-mid.jpg) | ![结尾帧](docs/still-end.jpg) |

---

## 为什么做这个（而不是又一个“AI 一键成片”）

AI 生成知识视频最大的问题不是画质，是**瞎编**：假日期、假数字、假判例，配上自信的旁白。

FactReel 把顺序反过来：

1. **先查证，后写稿** — 每个日期/数字/结论必须先进入事实表，带来源。查不到的，要么删，要么在画面上明标「估算 / 示意复刻」。
2. **时长由配音实测决定** — 每句旁白单独生成、单独用 ffprobe 量时长，分镜跟着声音走，不靠“大概 4 秒一句”猜。
3. **字幕诚实分级** — 句级时间是实测的；句内逐字高亮若是按字数估算的，就明说估算，不冒充词级对齐。
4. **为完播而剪** — 默认 30–45 秒，0–3 秒钩子，每 3–4 秒一个信息点，结尾留一个能评论的问题。
5. **交的不是一个 mp4，是一套发布包** — 成片 + 封面帧 + 三平台标题/正文/置顶评论 + AI 生成内容标识提醒。

## 你会得到什么

| 产物 | 说明 |
|---|---|
| `fact-table.md` | 事实表：每条事实、数值、来源、核查日期 |
| `storyboard.md` | 分镜表：Beat、画面、旁白、实测时长 |
| `out/video.mp4` | 1080×1920 / 30fps / h264 + aac，烧录字幕 |
| `out/cover.png` | 封面帧（从成片关键帧选） |
| `publish-pack.md` | 抖音 / 视频号 / 小红书的标题、正文、置顶评论 |

## 快速开始

把本仓库的 skill 文件夹复制到你的 Agent skills 目录：

```text
skills/fact-reel/   →   <your-agent-skills-dir>/fact-reel/
```

然后对你的 Agent 说：

```text
用 fact-reel 做一条短视频，选题是：为什么越不懂的人越自信？
时长 40 秒左右，竖屏。
```

Agent 会先交事实表给你确认，**不会**跳过查证直接写稿。

环境要求：Node.js + 一个 Remotion 项目（skill 内含最小工程说明）、FFmpeg（含 ffprobe）。不需要任何付费 API key；配音可替换为你自己的 TTS。

## 工作流（6 阶段，3 道门）

```text
选题
 └─ 1 查证 ──→ 事实表 ──[门1：事实确认]──┐
 └─ 2 写稿 ──→ Beat 脚本（钩子/信息点/结尾问句）
 └─ 3 配音 ──→ 逐句 TTS + ffprobe 实测时长
 └─ 4 分镜 ──→ 时长驱动画面 ──[门2：静帧评审 ≥3 帧]
 └─ 5 渲染 ──→ 字幕烧录 + 混音 ──[门3：成片 ffprobe 校验]
 └─ 6 发布包 → 封面 + 三平台文案 + 来源片尾
```

完整规范见 `skills/fact-reel/SKILL.md`，细节在 `skills/fact-reel/references/`。

## 已验证的示例

| 示例 | 时长 | 查证要点 |
|---|---|---|
| 中国大模型混战史 | 46.9s | 8 个时间节点逐个联网核对日期与数字后才写稿 |
| AI 翻车名场面盘点 | 45.9s | 3 个真实案件，案号/判决/金额均对一手或强来源 |
| AI 助手月活竞速 | 29.3s | 单一数据口径锚点表，锚点间插值在画面明示 |

示例分镜见 `examples/`。

## 仓库结构

```text
fact-reel-skill/
├── README.md
├── LICENSE
├── docs/                    # demo.gif + 展示静帧
├── skills/
│   └── fact-reel/
│       ├── SKILL.md
│       ├── agents/openai.yaml
│       ├── references/      # 查证 / 写稿 / 视觉 / 字幕配音 / 发布 / 质检
│       └── templates/       # 事实表 / 分镜表 / 发布包 模板
└── examples/
    └── model-war/           # 完整走通的示例（分镜 + 旁白稿）
```

## 路线图

- [x] v0.1 单一视觉模板（时间线卡片）、3 道质量门、发布包
- [ ] v0.2 数据竞速模板（CSV → 条形竞速，含口径锚点表）
- [ ] v0.3 真词级字幕对齐（Whisper）、BGM 与响度规范
- [ ] v0.4 小红书卡片联动（同事实表一键出卡组）

## 许可

MIT。示例中引用的事实来源归原出处所有；成片请按发布平台规则标注 AI 生成内容。

---

*FactReel 的纪律只有一条：**没有出处的话，不配上自信的旁白。***
