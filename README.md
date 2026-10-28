# 脚本—视频—平台 全工作流

**Script → Video → Platform: an end-to-end workflow skill for short knowledge-explainer animations.**

一个可复用的 AI Skill：把「知识科普 / 资讯速报」短视频从**选风格、写脚本**，到**贴图动画与音效绑定、原声处理、双语字幕**，再到**多平台封面与发布文案**整理成一套工作规范。

A reusable AI skill that turns a short knowledge or news-explainer video into one consistent workflow — from picking a visual style and writing the script, through texture-based animation with bound sound effects, voice cleanup and bilingual subtitles, to per-platform covers and post copy.

> 当前版本 / Current version: **v0.4.0** · 规范语言 / Skill language: 中文 (Chinese)

---

## 包含什么 / What's inside

```
vibe-knowledge-video/
├── SKILL.md                     # 入口：范围、工作方式、输出前检查
├── agents/openai.yaml           # 显示名称与默认提示（可选）
└── references/
    ├── series-standard.md       # 系列制作规范：叙事、视觉、节奏、质量
    ├── planning-handoff.md      # 先定风格再写脚本；先展示再定稿；交接包
    ├── texture-animation.md     # 贴图动画：完整姿态切换，不做骨骼绑定
    ├── animation-sound.md       # 每个主要动画必须绑定音效（共享 event_id）
    ├── audio-sync.md            # 用户原声为时间基础；三层时间线；剪辑映射
    ├── publishing.md            # 中英文字幕、五平台封面与发布文案
    ├── templates.md             # 单期策划、分镜、音效事件表、复盘模板
    └── evolution.md             # 如何把审片反馈变成规范更新
```

| 文件 / File | 作用 / Purpose |
|---|---|
| `series-standard.md` | Narrative grammar, visual direction, pacing, quality bar |
| `planning-handoff.md` | Style first, then script; show in chat before finalizing; handoff package |
| `texture-animation.md` | AI-generated texture animation with whole-pose swaps (no rigging) |
| `animation-sound.md` | Every meaningful animation gets a matching sound, sharing one event timeline |
| `audio-sync.md` | Your own voice recording drives the timeline; edit-map rules |
| `publishing.md` | Chinese/English subtitles; covers and copy for Douyin, Xiaohongshu, Bilibili, YouTube, X |
| `templates.md` | Fill-in templates for planning, storyboard, SFX events, review |
| `evolution.md` | Turning review feedback into versioned rule changes |

## 核心原则 / Key principles

1. **先定风格，再写脚本** — Pick the visual style before writing the script.
2. **先在聊天展示，确认后再定稿** — Show drafts in chat; finalize only after you confirm.
3. **动画与音效绑定** — Each information-carrying animation has a matching sound (or a written reason for silence).
4. **保留本人原声** — Keep the creator's real voice; improve sibilance/dryness with A/B samples first; no silent swap to synthetic voice.
5. **配乐自动避让人声** — Licensed background music is ducked under the voice; music never replaces SFX.
6. **默认中英双语字幕** — Chinese on top, English below; separate `zh.srt`, `en.srt`, and a bilingual project.
7. **封面分平台构图** — Re-layout covers per platform; never stretch one image.
8. **只承诺实际核验过的内容** — Report only checks actually performed; platform UI beats old guesses.

## 安装 / Install

把 `vibe-knowledge-video/` 整个文件夹复制到你所用 AI 工具的 skills 目录，例如 Claude Code 的 `~/.claude/skills/`。

Copy the whole `vibe-knowledge-video/` folder into your AI tool's skills directory — for example `~/.claude/skills/` for Claude Code.

```bash
git clone https://github.com/wyx062102-dotcom/vibe-video.git
cp -r vibe-video/vibe-knowledge-video ~/.claude/skills/
```

然后对 AI 说："使用 vibe-knowledge-video，帮我策划下一期关于 X 的知识视频。"

Then ask: *"Use vibe-knowledge-video to plan my next explainer about X."*

## 已知限制 / Known limitations (v0.4.0)

- `publishing.md` 中的封面尺寸表是**设计起点，不是平台强制规格**。作者在实际发布界面中看到：抖音为 **3:4 竖封面 + 4:3 横封面**（表中仍写 9:16）；小红书封面比例尚待核对。发布前请以你自己账号的实际界面为准，后续版本会更新。
  The cover-size table in `publishing.md` is a *starting point, not a platform rule*. In the author's own upload screen, Douyin asked for a **3:4 portrait + 4:3 landscape** cover (the table still says 9:16); Xiaohongshu is not yet verified. Always check your own upload screen.
- 规范以中文撰写，面向作者个人系列；主题、时长、画幅、风格每期独立决定，不是硬性常量。
  Written in Chinese for the author's own series; topic, length, aspect ratio and style are decided per episode.
- 本仓库只包含 Skill 规范，**不包含**任何单期的录音、视频素材或第三方内容。
  This repo contains only the skill — no episode recordings, footage, or third-party assets.

## 分享方式 / Sharing

免费公开，**关注自愿**——不需要关注、点赞、评论或转发来领取。

Free and public. Following is optional — no follow/like/comment/repost required.

## 许可证 / License

[CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) — 可自由使用、修改、再分发，请署名并注明修改。
Free to use, adapt and share with attribution and a note of changes. See [LICENSE](LICENSE).
