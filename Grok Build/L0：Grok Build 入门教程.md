---
tags:
  - grok-build
  - harness
  - tutorial
  - agent
source: crates/codegen/xai-grok-pager/docs/tutorial/ (01→09)
created: 2026-08-09
---

# Grok Build 入门教程精炼

> 原文：`docs/tutorial/` 共 9 篇，按 01→09 顺序。本文是白话浓缩版，并突出 harness 核心概念。

## 一句话定位

Grok Build = **终端里的对话式 Agent**：能读代码、跑命令、改文件；你通过提示词、快捷键、斜杠命令控制它。

---

## 学习路径（01→09）

| #   | 主题                           | 你要记住的                                                   |
| --- | ---------------------------- | ------------------------------------------------------- |
| 01  | 从 Claude / Cursor / Codex 迁移 | 规则、技能、MCP、Hooks 大多自动兼容；`/import-claude` 一键导入            |
| 02  | 第一次提问                        | 边跑边打字会入队；`Esc` 取消；底部快捷键栏是实时提示                           |
| 03  | 附件与粘贴                        | `@` 贴文件，粘贴图片，`!` 直接跑 shell                              |
| 04  | 界面导航                         | 滚动区 / 提示区 / 快捷栏；`Tab` 切焦点；`Ctrl+T` todos、`Ctrl+G` tasks |
| 05  | 斜杠命令                         | `/` 与 `Ctrl+P` 是万能入口；重点：`/compact`、`/rewind`、`/btw`     |
| 06  | Worktree 并行                  | 多会话隔离改代码；`/fork` 分叉对话；`/dashboard` 看全貌                  |
| 07  | Plan 与权限                     | 有风险才问你；Plan 模式先方案后写码                                    |
| 08  | 个性化                          | 最高杠杆：`AGENTS.md`；其次 memory / skills / MCP / hooks       |
| 09  | 下一步                          | 自动存会话、`/docs` 深入、无头模式跑 CI                               |

---

## 界面三件套

```
┌─────────────────────────────┐
│  scrollback（对话滚动区）     │  ← Tab 聚焦后可选择、折叠、全屏看
├─────────────────────────────┤
│  prompt（输入区）            │
├─────────────────────────────┤
│  shortcuts bar（当前相关键位）│
└─────────────────────────────┘
```

| 键 | 作用 |
|----|------|
| `Tab` | 提示区 ↔ 滚动区 |
| `↑`/`↓` | 选上/下一条 |
| `Shift+←`/`→` | 按「你的一轮提问」跳转 |
| `PageUp`/`Down` | 翻页（提示区也能直接用） |
| `←`/`→` | 折叠/展开选中条目 |
| `Enter`（在滚动区） | 全屏查看该条目 |
| `Ctrl+T` | Todos 面板 |
| `Ctrl+G` | 后台任务面板 |
| `Ctrl+B` | 把卡住的长命令丢后台 |
| `/vim-mode` | 滚动区改用 vim 键位 |

### 控制节奏

| 操作 | 行为 |
|------|------|
| 运行中按 `Enter` | **排队**下一条，不打断 |
| 空提示再 `Enter` | 停当前轮，立刻发队列消息 |
| `Esc` | 立刻取消当前轮（草稿保留） |
| `Esc Esc`（空闲） | 清空提示；提示已空则打开 rewind |
| `Ctrl+Z` | 撤销误清 |
| `Ctrl+Q` ×2 | 退出（VS Code 系终端用 `Ctrl+D`） |

---

## 把上下文喂给 Agent

| 方式        | 怎么用                                                   |
| --------- | ----------------------------------------------------- |
| `@` 文件    | `@src/main.rs`；行范围 `@src/main.rs:10-50`；隐藏文件 `@!.env` |
| 粘贴图片      | macOS `Cmd+V` / Linux `Ctrl+V` / Windows `Alt+V`      |
| `!` shell | 空提示打 `!` 自己跑命令，输出进滚动区，Agent 也能看见                      |

原则：**指得越准，结果越好**。

---

## 核心概念（白话）

### 1. Permission Mode（权限模式）

> Agent **不会随便乱动**；读自动允许，写/危险命令要你点头。

- **永远自动允许**：读文件、搜索、安全只读命令（`ls`、`git status`、`grep`…）
- **链式命令逐段审**：`ls && rm -rf tmp` → 仍会为 `rm` 弹确认
- 弹窗选项：允许一次 / 始终允许这类 / 拒绝
- 信任本会话：`/always-approve` 或 `Ctrl+O` 跳过确认

**模式循环**（焦点在提示区时 `Shift+Tab`）：

```
Normal → Plan → Always-approve → …
```

| 模式 | 适合 |
|------|------|
| Normal | 「直接干」 |
| Plan | 「先想想怎么做」——只读探索 → 出方案 → 你批准后再写码 |
| Always-approve | 高度信任的会话，少打断 |

**Plan 审批键**：`a` 批准 · `c` 对某行评论 · `s` 要求修改（迭代到满意再实现）

直接进 Plan：`/plan` 或 `/plan <任务>`

---

### 2. Compaction（压缩上下文）

> 对话越长，上下文窗口越满。**Compaction = 把长对话压成摘要，腾出空间。**

| 手段 | 说明 |
|------|------|
| `/compact` | 手动压缩；可带提示：`/compact keep the auth details` |
| 自动 compact | 窗口快满时 Grok 会自己压 |
| `/context` | 随时看上下文花在哪 |

**习惯**：会话变慢 / 快满 → 先 `/context` 再 `/compact`。

> 相关但不同：`/rewind`（别名 `/undo`）是**回退对话到某轮**（之后的轮次丢弃；**磁盘上的文件改动不会自动回滚**）。

---

### 3. Subagent（子代理）

> 主 Agent 可派出**独立干活的子 Agent**，自己继续主线。教程 09 指向完整 How-to；日常相关能力：

| 能力 | 白话 |
|------|------|
| Worktree 会话 | 每个会话独立 checkout，互不踩文件 |
| `/fork` | 复制当前对话开并行分支，可加指令：`/fork try the async approach` |
| Dashboard | `/dashboard` 或 `Ctrl+\`：看谁在等你、谁在跑、谁完成了 |
| 后台任务 | `Ctrl+B` 丢后台；`Ctrl+G` 看状态 |

**Worktree 开法**：

- `Ctrl+N`（确认两次）→ 选 worktree
- 欢迎屏 `Ctrl+W`（在 git 仓库内）
- 命令行：`grok --worktree=my-feature "refactor the auth module"`（注意 `=`，否则名字会被当成 prompt）

深入：TUI 里 `/docs`，搜 subagents / session management。

---

### 4. Hooks（钩子）

> 在关键生命周期点**自动跑你的脚本**（和 Claude 生态类似）。

- 从 Claude 迁过来：`.claude/settings.json` 里的 hooks 会读；matcher 别名如 `Bash` 大多可原样用
- 管理入口：`/hooks`、`/plugins`
- `/import-claude` 可预览并勾选导入 hooks 等配置

用途直觉：改文件后跑格式化、危险命令前再拦一层、记录审计日志……

---

### 5. Skills（技能）

> **可复用的提示词包**。可用户调用的 skill 会自动变成斜杠命令。

| 来源 | 路径示例 |
|------|----------|
| Claude / Cursor 兼容 | `~/.claude/skills/`、`~/.cursor/skills/` 及项目级同构目录 |
| 扁平 command `.md` | 也会变成 slash command |
| 内置 resume | `/resume-claude`、`/resume-codex`、`/resume-cursor` |

管理：`/skills` · 命令面板 `Ctrl+P` · 斜杠 `/`

---

### 6. MCP（Model Context Protocol）

> 给 Agent 接**外部工具/数据源**的标准协议（数据库、GitHub、浏览器……）。

| 项 | 说明 |
|----|------|
| 自动发现 | `~/.claude.json`、`.cursor/mcp.json`、项目 `.mcp.json` |
| 管理 | `/mcps` |
| 最省事 | 直接说：*「add the Postgres MCP server for our staging db」* |

`grok inspect` 可列出本仓库发现的规则、技能、MCP，并标注来源；`[compat.claude]` / `[compat.cursor]` 可开关兼容。

---

## 从别的工具搬过来（01）

| 类别 | 自动读取 |
|------|----------|
| 规则 | `AGENTS.md`、`CLAUDE.md`（含嵌套）、`.claude/rules/`、`.cursor/rules/` |
| Skills / commands | Claude & Cursor 用户级 + 项目级 |
| MCP | Claude / Cursor / 项目 `.mcp.json` |
| Hooks | `.claude/settings.json` |

| 命令 | 作用 |
|------|------|
| `/import-claude` | 扫描 `~/.claude`，勾选写入 `.grok` |
| `/resume-claude` 等 | 续跑那边最近的会话 |
| `grok inspect` | 看「发现了什么、从哪来」 |

独有小功能：`/btw` 旁路提问不打断主任务；`/rewind` 回退对话轮次。

---

## 个性化优先级（08）

从高杠杆到低：

1. **`AGENTS.md`（仓库根）** — 构建命令、约定、雷区；每会话自动读  
   ```markdown
   # My Project
   - Run tests with `pnpm test`
   - Never edit files under generated/
   ```
2. **Memory** — 提示以 `#` 开头，或 `/remember`：`# the staging deploy uses eu-west`
3. **外观与习惯** — `/theme`、`/settings`（`F2`）、`/vim-mode`
4. **扩展** — Skills / MCP / Plugins / Hooks（需要时再加）

也可以**直接让 Grok 配自己**：「换浅色主题」「给这仓库写 AGENTS.md」……

---

## 常用斜杠命令速查

| 命令 | 干啥 |
|------|------|
| `/help` | 全部命令与快捷键 |
| `/model` | 换模型 / 推理力度 |
| `/resume` / `Ctrl+S` | 恢复旧会话 |
| `/new` | 新会话 |
| `/compact` | 压缩上下文 |
| `/context` | 看上下文占用 |
| `/btw` | 旁路问题，不打断当前任务 |
| `/rewind` / `/undo` | 回退对话轮次 |
| `/plan` | 进入规划模式 |
| `/always-approve` | 本会话总是批准 |
| `/docs` | 内置 How-to（`/docs web` 在线） |
| `/tutorial` | 重开这篇入门教程 |
| `/feedback` | 反馈给团队 |
| `/theme` `/settings` `/skills` `/mcps` `/hooks` `/plugins` | 个性化与扩展 |
| `/fork` | 分叉并行会话 |
| `/dashboard` | 多会话总览 |
| `/import-claude` | 导入 Claude 配置 |

**命令面板**：`Ctrl+P`（滚动区也可 `?`）  
**快捷键大全**：`Ctrl+.`（被终端吞键时用 `Ctrl+X`）

---

## 好习惯清单

- [ ] 新仓库先写 / 生成 `AGENTS.md`
- [ ] 大改先 `Shift+Tab` 进 Plan，小改 Normal
- [ ] 会话变长 → `/context` → `/compact`
- [ ] 并行实验用 worktree 或 `/fork`，别在同一工作区互踩
- [ ] 会话自动保存：`grok -c` 续最新，或 `/resume` 点选
- [ ] CI / 脚本：`grok -p "…" --output-format json`（无头模式）
- [ ] 保持更新：`grok update`；变更看 `/release-notes`
- [ ] 迷路：`/`、`Ctrl+P`、`/docs`、或直接问 Grok

---

## 心智模型一张图

```mermaid
flowchart TB
  You[你] -->|自然语言 / @文件 / 图片| Prompt
  Prompt --> Agent[Grok Agent]
  Agent --> Rules[AGENTS.md / 兼容规则]
  Agent --> Skills[Skills → 斜杠命令]
  Agent --> MCP[MCP 外部工具]
  Agent --> Hooks[Hooks 生命周期脚本]
  Agent -->|只读免费| Read[读文件 / 搜索]
  Agent -->|危险操作| Perm{Permission Mode}
  Perm -->|询问 / Always-approve| Act[改文件 / 跑命令]
  Agent -->|任务大或模糊| Plan[Plan Mode 先方案]
  Plan -->|你批准| Act
  Agent -->|上下文满| Compact[/compact 压缩]
  Agent -->|可分派| Sub[Subagent / Worktree / Fork]
```

---

## 原文索引

| 文件 | 标题 |
|------|------|
| `01-coming-from-another-tool.md` | Coming from Claude, Cursor, or Codex? |
| `02-first-prompt.md` | Your First Prompt |
| `03-attach-and-paste.md` | Attach Files, Images & Paste |
| `04-navigation.md` | Finding Your Way Around |
| `05-slash-commands.md` | Slash Commands |
| `06-worktrees.md` | Parallel Work: Worktrees |
| `07-plan-and-permissions.md` | Plan Mode & Permissions |
| `08-make-it-yours.md` | Make It Yours |
| `09-where-next.md` | Where to Go Next |

仓库路径：`grok-build/crates/codegen/xai-grok-pager/docs/tutorial/`

TUI 内重开：`/tutorial`

