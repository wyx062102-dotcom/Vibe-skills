---
name: project-handoff-setup
description: 在新电脑或新 Claude 账号上一键部署“项目文件夹 + Basic Memory + 交接记录”的跨账号项目交接系统；用户说部署交接系统、设置项目交接、换电脑/换账号要接上项目时使用。
---

# 项目交接系统：一键部署

目标：用户在多个 Claude 账号、聊天界面和 Claude Code 之间，靠电脑本地的项目文件夹和交接记录无缝接续工作。

原则：**只在开头集中问一次选择、申请一次权限，之后全部自动完成**，中途不再打扰用户。用户不是专业开发者，说明要简单。不覆盖、不删除用户已有的文件和配置。

适用：Windows 电脑，且本会话能通过设备工具操作用户电脑（`mcp__remote-devices__*`）。没有这些工具时，告诉用户需要在 Claude 桌面版里打开这个对话并关联电脑，然后停止。

---

## 阶段 0：一次性收集选择（唯一一次提问）

先调用 `get_device_info`，拿到用户目录名、已有文件夹、本机 MCP 服务器列表，用来预填默认值和判断哪些已经装好。

然后用**一次** AskUserQuestion 问完（界面不支持就用编号列表）：

1. **语言**：中文 / English / 其他（让用户输入）。之后所有说明、生成的文件、账号指令都用这个语言。
2. **项目根目录位置**：`D:\Projects`（推荐，有 D 盘时）/ `C:\Users\<用户名>\Projects` / 其他（让用户输入完整路径）。建议路径只用英文和数字，避免工具对中文路径出错。
3. **已有项目**：现在就把已有的项目登记进来（让用户写项目名，逗号分隔）/ 先不建，之后再说。

记下：LANG、ROOT、项目列表。项目文件夹统一命名为“两位编号_项目名”（01_、02_……）。

## 阶段 1：一次性申请全部权限

告诉用户：“接下来会弹出几个权限请求，全部允许后我会自动完成安装，不再需要你操作。”然后连续申请，中间不做其他工作：

1. **命令执行能力**（二选一）：
   - 本会话有可用的 `device_bash` → 直接用它，跳过下一条。
   - 否则用电脑操作（computer use）：先加载 `computer-use` 技能，用 `computer_request_access` **一次**申请 Windows PowerShell、记事本、文件资源管理器。
2. **根目录**：ROOT 不存在时，用上一步得到的能力创建它（`New-Item -ItemType Directory -Force "<ROOT>"`）。然后用 `device_request_folder_access` 申请 ROOT 的访问权限，reason 写“在这里建立项目文件夹和交接规则文件”。

任何一项被拒绝：告诉用户缺了它哪些步骤无法自动完成，把那些步骤改为在最后给用户手动操作清单，其余照常自动进行。

## 阶段 2：全自动部署（不再提问）

用任务列表跟踪。每一步完成后自行检查结果，失败就按“排错”处理，处理不了的记入最后的报告。

### 2.1 准备安装脚本

把脚本写进 `<ROOT>\_setup\`（用 `device_commit_files` 写入；或在 PowerShell 里生成）。**.ps1 文件必须是带 BOM 的 UTF-8**，否则 Windows PowerShell 5.1 会把中文读成乱码。

`<ROOT>\_setup\setup.ps1` 内容（把 `<ROOT>` 和项目列表替换成实际值）：

```powershell
$ErrorActionPreference = 'Stop'
$log = "<ROOT>\_setup\setup.log"
Start-Transcript -Path $log -Force | Out-Null
$bin = "$HOME\.local\bin"
$env:Path = "$bin;$env:Path"

# 1. uv
if (-not (Get-Command uv -ErrorAction SilentlyContinue)) {
  powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
  $env:Path = "$bin;$env:Path"
}
uv --version

# 2. basic-memory
uv tool install basic-memory
basic-memory --version

# 3. 合并 Claude 桌面版配置（先备份，只增加 basic-memory，不动其他设置）
$cfg = "$env:APPDATA\Claude\claude_desktop_config.json"
if (Test-Path $cfg) {
  Copy-Item $cfg "$cfg.bak" -Force
  $j = Get-Content $cfg -Raw -Encoding UTF8 | ConvertFrom-Json
} else {
  New-Item -ItemType Directory -Force (Split-Path $cfg) | Out-Null
  $j = [pscustomobject]@{}
}
if (-not $j.PSObject.Properties['mcpServers']) {
  $j | Add-Member -NotePropertyName mcpServers -NotePropertyValue ([pscustomobject]@{})
}
$j.mcpServers | Add-Member -Force -NotePropertyName 'basic-memory' -NotePropertyValue ([pscustomobject]@{ command = "$bin\uvx.exe"; args = @('basic-memory','mcp') })
[IO.File]::WriteAllText($cfg, ($j | ConvertTo-Json -Depth 100), (New-Object Text.UTF8Encoding $false))
Get-Content $cfg -Raw | ConvertFrom-Json | Out-Null   # 校验 JSON 可解析

# 4. 项目文件夹 + 登记
$projects = @('01_项目A','02_项目B')   # 按用户给的列表替换；没有就留空 @()
foreach ($p in $projects) {
  foreach ($d in 'src','assets','docs','notes') { New-Item -ItemType Directory -Force "<ROOT>\$p\$d" | Out-Null }
  basic-memory project add $p "<ROOT>\$p\notes"
}
basic-memory project list
Stop-Transcript | Out-Null
```

### 2.2 运行并检查

- 有 `device_bash`：直接运行 `powershell -ExecutionPolicy Bypass -File "<ROOT>\_setup\setup.ps1"`。
- 用 computer use：打开 PowerShell，只输入上面这一行命令（纯英文路径，避免输入法问题），回车，等待完成。
- 运行后读取 `<ROOT>\_setup\setup.log`，确认：uv 和 basic-memory 都输出了版本号、配置 JSON 校验通过、每个项目都 `added successfully`。

### 2.3 写入其余文件（用 `device_commit_files`）

- `<ROOT>\CLAUDE.md`：见下方模板，按 LANG 翻译，替换 ROOT。Claude Code 打开 ROOT 或其中任一项目时都会读取它，换账号也有效。
- 每个项目的 `notes\交接记录.md`：用下方模板生成初始版本（若已存在则不动）。
- `<ROOT>\Claude指令.txt`：账号指令全文（见下方模板，按 LANG 翻译，替换 ROOT），方便以后修改后同步到各账号。

### 2.4 把账号指令放进剪贴板

有 computer use 时，用 `computer_write_clipboard` 把账号指令全文放进剪贴板，用户最后只需粘贴。

## 阶段 3：收尾（只剩用户必须亲手做的两件事）

Claude 无法修改账号设置，也不能在会话中途重启 Claude 应用（会断开与电脑的连接），所以最后用一条消息告诉用户：

1. **粘贴账号指令**：左下角头像 → Settings → General → **Instructions for Claude**（找不到就用设置里的搜索），粘贴（剪贴板里已有；或打开 `<ROOT>\Claude指令.txt` 复制），保存。**每个 Claude 账号各做一次。**
2. **重启 Claude**：从任务栏托盘右键 Claude → 退出，再打开。设置 → Developer 里 basic-memory 应显示 Running。

然后附上：部署结果（装了什么、建了哪些项目、配置备份在哪里）、任何没能自动完成的步骤，以及验证方法——重启后新开对话随便说一句，应列出项目并让你选择；Claude Code 新建会话时选 ROOT 文件夹，说“开始”，同样会列出项目。

日常用法一句话说明：“开始”、“交接”、“新建项目：xxx”。完成一个阶段就说“交接”，别等额度快用完；两个账号不要同时处理同一个项目。

## 排错（自动处理，处理不了才记入报告）

- `uv` 不是命令：PATH 未刷新，脚本里已补 PATH；仍失败就用完整路径 `$HOME\.local\bin\uv.exe`。
- 下载被拦截/网络错误：重试一次；仍失败记入报告，让用户检查网络或代理。
- 找不到配置文件目录：可能是商店版 Claude，路径在 `%LOCALAPPDATA%\Packages\Claude_*\LocalCache\Roaming\Claude\`，在那里查找 `claude_desktop_config.json` 后按同样方法合并。
- JSON 校验失败：用 `.bak` 恢复原文件，记入报告，给用户手动插入的那一段配置。
- 项目已登记：跳过，不算错误。

## 模板

### `<ROOT>\CLAUDE.md`

```markdown
# 项目交接规则（适用于 <ROOT> 下的所有项目）

始终用<LANG>回复我。

我的项目都在 <ROOT>，每个项目一个文件夹，命名为“两位编号_项目名”（如 01_毕业设计）。
每个项目的交接记录在项目文件夹的 notes\交接记录.md。

## 项目结构
- src\：代码
- assets\：素材（说明见 assets\README.md）
- docs\：文档、需求、参考资料
- notes\：交接记录

## 开始工作时
- 当前文件夹下有 notes\交接记录.md：先读取，用几句话告诉我上次进度、未解决的问题、下一步，再问我这次按“下一步”继续还是做别的。
- 没有：问我是否要查看现有文件，建立第一份交接记录。
- 当前打开的是 <ROOT> 本身：列出所有项目（按编号排序），问我继续哪一个或新建项目。

## 我说“交接”时
更新 notes\交接记录.md：项目概况、当前进度、重要决定及原因、关键文件及作用、未解决的问题、下一步（写到能直接动手）、更新日志末尾加一条（日期 + 做了什么）。写到另一个完全不了解情况的 Claude 能直接接手的程度。

## 我说“新建项目：xxx”时
1. 在 <ROOT> 下用下一个编号建文件夹，如 02_xxx。
2. 建 src、assets、docs、notes 四个子文件夹。
3. 生成第一份 notes\交接记录.md。
4. 如果 basic-memory 工具可用，登记为同名项目，路径指向该项目的 notes 文件夹。
```

### 账号指令（Instructions for Claude）

```
始终用<LANG>回复我。
我的所有项目在 <ROOT>，每个项目一个文件夹，命名为“两位编号_项目名”（如 01_毕业设计）。
每个项目在 basic-memory 里登记为同名项目，交接记录在项目的 notes\交接记录.md。

【开场】每次新对话的第一条回复都要做：
1. 用 basic-memory 列出所有项目，按编号排序。
2. 用可点击的选项问我：继续某个项目 / 新建项目 / 只是普通聊天。界面不支持可点击选项时，改用编号列表，我回复数字即可。
3. 选了已有项目：读取它的交接记录，用几句话告诉我上次进度、未解决问题、下一步；再按记录里的路径读取相关代码和素材说明后开始。
4. 提醒我：如果这次对话不在对应的 Claude 项目里，可以把它移过去，方便归类。

【新建】选了“新建项目”或我说“新建项目：xxx”时：
1. 问我项目名（如果还没说）。
2. 用下一个编号在 <ROOT> 下建好文件夹：src、assets、docs、notes。
3. 用 basic-memory 的工具直接登记为同名项目，并生成第一份交接记录。
4. 提醒我在 Claude 里新建同名的项目，并把这次对话移进去。

【交接】我说“交接”时：按交接记录模板更新该项目的交接记录，写到另一个完全不了解情况的 Claude 能直接接手的程度，并在更新日志加一条。
```

### `notes\交接记录.md`

```markdown
# 交接记录：<项目名>

## 项目概况
- 目标：（待补充）
- 项目文件夹：<ROOT>\<NN_项目名>
- 技术/工具：

## 当前进度
- 已完成：
- 进行中：

## 重要决定（及原因）
- 

## 关键文件
- 

## 未解决的问题
- 

## 下一步
1. 补充项目目标，整理现有代码和素材

## 更新日志
- <今天日期>：建立项目和交接记录
```

## 不要做的事

- 不覆盖已有的配置、CLAUDE.md、交接记录；配置改动前必须先备份。
- 不删除用户的任何文件或文件夹。
- 不在阶段 2 中途向用户提问；遇到问题先自行排错，处理不了的放进最后的报告。
- 不声称已修改账号设置或已重启应用——这两件事只能由用户完成。
