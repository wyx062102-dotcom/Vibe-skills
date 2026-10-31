[← 返回 Vibe Skills 合集 / Back to Vibe Skills](../README.md)

# Claude ⇄ GPT 项目交接

**Hand projects back and forth between Claude and GPT (ChatGPT, Codex) through one shared local handoff note.**

在 Claude 和 GPT 之间轮流做同一个项目：一边额度用完或想换个工具，另一边说一句“开始”就能接着做。

> 当前版本 / Current version: **v0.1.0** · 规范语言 / Skill language: 中文 (Chinese)
> 需要先部署 [project-handoff-setup](../project-handoff-setup/) / Requires project-handoff-setup first.

---

## 思路 / How it works

- 本地项目文件夹（默认 `D:\Projects`）是唯一的真相来源，每个项目的进度写在 `notes\交接记录.md`。
- Claude 和 GPT 用同一份交接记录、同一套口令：**开始**、**交接**、**新建项目：xxx**。
- 交接记录顶部有“交接状态”：开始时标记“进行中（工具名）”，交接时改回“空闲”。如果上一个 AI 没交接就结束了，下一个会提醒你。

| 工具 | 怎么读到交接记录 |
|---|---|
| Claude 聊天 / Claude Code | 已由 project-handoff-setup 部署 |
| Codex | 直接读写项目文件夹，规则在 `~\.codex\AGENTS.md` |
| ChatGPT 聊天 | 读不到本地文件，用复制粘贴“交接块”中转 |

## 部署会做什么 / What it sets up

- 给每份交接记录加上“交接状态”一节（原内容不动）
- 在 `CLAUDE.md` 里加上与 GPT 交接的规则
- 写入 Codex 的 `AGENTS.md`，并把 Basic Memory 接入 Codex（先备份 `config.toml`）
- 生成 ChatGPT 自定义指令 `GPT指令.txt`

开头只需选择 GPT 这边用什么、允许文件夹权限；最后把 ChatGPT 指令粘贴进设置、重开 Codex。

## 日常用法 / Daily use

| 场景 | 怎么做 |
|---|---|
| Claude → Codex | 在 Claude 说“交接”，到 Codex 说“开始” |
| Codex → Claude | 在 Codex 说“交接”，到 Claude 说“开始” |
| Claude → ChatGPT | 在 Claude 说“交给 GPT”，复制代码块，到 ChatGPT 说“开始”并粘贴 |
| ChatGPT → Claude | 在 ChatGPT 说“交接”，复制代码块，到 Claude 粘贴并说“接回 GPT 的进度” |

## 安装 / Install

```bash
git clone https://github.com/wyx062102-dotcom/vibe-skills.git
cp -r vibe-skills/claude-gpt-handoff ~/.claude/skills/
```

在 Claude 应用里，也可以把文件夹打包成 zip，在 设置 → Skills 里上传。然后对 Claude 说：“部署 Claude 和 GPT 交接”。

## 已知限制 / Known limitations

- ChatGPT 聊天默认读不到电脑上的文件，只能复制粘贴中转；需要直接改文件时，用 Codex。
- 自动部署目前只适配 Windows。
- 两边同时做同一个项目会互相覆盖进度，请轮流使用。

## 许可证 / License

[CC BY 4.0](https://creativecommons.org/licenses/by/4.0/)，见 [LICENSE](../LICENSE)。
