[← 返回 Vibe Skills 合集 / Back to Vibe Skills](../README.md)

# 跨账号项目交接系统

**Project handoff across Claude accounts: one local folder, one handoff note per project.**

用一套本地项目文件夹加交接记录，让多个 Claude 账号、Claude 聊天界面和 Claude Code 能接续同一个项目。一个账号的额度用完，换另一个账号说一句“开始”，就能从上次的进度继续。

> 当前版本 / Current version: **v0.1.0** · 规范语言 / Skill language: 中文 (Chinese)

---

## 它会自动完成什么 / What it sets up

- 安装 uv 和 [Basic Memory](https://github.com/basicmachines-co/basic-memory)，并接入 Claude 桌面版（先备份原配置）
- 建立项目根目录，每个项目一个“两位编号_项目名”文件夹（`src`、`assets`、`docs`、`notes`）
- 在 Basic Memory 里登记每个项目
- 写入 Claude Code 读取的 `CLAUDE.md` 交接规则，以及每个项目的 `notes\交接记录.md`
- 生成账号指令（Instructions for Claude）文本

**只有开头需要你操作**：选择语言和根目录，允许几项权限。最后再手动做两件事：把账号指令粘贴进每个账号的设置，然后重启 Claude。

## 日常用法 / Daily use

| 说什么 | 做什么 |
| --- | --- |
| 开始 | 列出项目，选一个后读取交接记录并汇报进度 |
| 交接 | 把进度、决定、下一步写进交接记录 |
| 新建项目：xxx | 建文件夹、登记项目、生成第一份交接记录 |

## 环境要求 / Requirements

- Windows
- Claude 桌面版已打开，并且对话已关联这台电脑（Claude 才能在你电脑上运行安装命令）

## 安装 / Install

**Claude 应用**：把 `project-handoff-setup/` 文件夹打包成 zip，在 设置 → Skills 里上传。

**Claude Code**：复制到 skills 目录：

```bash
git clone https://github.com/wyx062102-dotcom/vibe-skills.git
cp -r vibe-skills/project-handoff-setup ~/.claude/skills/
```

然后对 Claude 说：“部署交接系统”。

Then ask: *"Set up the project handoff system."*

## 已知限制 / Known limitations

- 自动安装目前只适配 Windows（PowerShell 脚本）。
- Claude 不能修改账号设置，也不能在会话中途重启 Claude 应用，这两步需要你亲手完成。
- 两个账号同时处理同一个项目时，交接记录可能互相覆盖，请轮流使用。

## 许可证 / License

[CC BY 4.0](https://creativecommons.org/licenses/by/4.0/)，见 [LICENSE](../LICENSE)。
