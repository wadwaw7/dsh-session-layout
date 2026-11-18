# dsh-session-layout

[English](README.md) · **中文** · 配套插件：[dsh-session-delete](https://github.com/wadwaw7/dsh-session-delete)

一套让多个 AI（DSH / Codex / Antigravity）在同一个工作区里互不串味的**会话目录排版规范**，
以 DSH 技能形态发布。

它同时是 [dsh-session-delete](https://github.com/wadwaw7/dsh-session-delete) 的配套规范：
那个插件删除会话时会一并清理项目里的会话文件夹，而它**只认**这套排版写下的元数据。

## 它解决什么

一个工作区里同时跑 DSH、Codex、Antigravity，会话产物很快就乱：目录名撞车、证据散在各处、
删会话时没人敢下手。这套规范给三条硬约束：

1. **一个对话 = 一个目录**，目录名 `YYYY-MM-DD-HHMM-<中文短主题>`。
2. **每个目录都有 `.session.json`**，写着 `provider` 与 `thread_id`（即 DSH 的 `DSH_SESSION_ID`）。
3. **两个固定文件**：会话说明、归档索引；证据、备份、一次性脚本都放在目录内。

## 一眼看懂的结构

```text
<工作区>/
├─ AGENTS.md                        多 AI 共用的工作区规则
├─ 会话/                            一个对话一个目录
│  ├─ 2026-11-18-1945-网站验收/
│  │  ├─ .session.json              身份：provider + thread_id
│  │  ├─ 会话说明.md                 目标 / 决定 / 结果
│  │  ├─ 归档索引.md                 每份产物的登记表
│  │  ├─ probe.mjs                  本对话用过的一次性脚本
│  │  └─ 证据/
│  │     └─ 备份/                    改动前的备份
│  └─ 2026-11-18-2010-另一个任务/
│     └─ …
└─ 智能限频工具/                     产品源码、构建产物、正式文档
```

`.session.json` 是整套东西的关键：

```json
{
  "provider": "dsh",
  "thread_id": "session-59f06ece-8970-4c3e-b961-c764ba27bdae",
  "topic": "网站验收",
  "created_at": "2026-11-18T19:45:00+08:00"
}
```

| 字段 | 含义 | 取值 |
| --- | --- | --- |
| `provider` | 产出该目录的宿主 | `dsh` / `codex` / `antigravity` |
| `thread_id` | 宿主会话 ID，与 DSH `DSH_SESSION_ID` 完全一致 | DSH 为 `session-<uuid>` |
| `topic` | 人类可读标签，仅用于展示，**从不参与匹配** | 自由文本 |
| `created_at` | 创建时间，ISO 8601 带时区 | 字符串 |

写它的时机与目录创建同一刻；`topic` 之后可以改，`thread_id` 一经写入不得改动。

## 删除时怎么定位到目录

```text
session-59f0…  ──thread_id 匹配──▶  会话/2026-11-18-1945-网站验收/     .session.json ✅ 删除
                                     会话/2026-11-18-2010-另一个任务/    thread_id 77ab… ✅ 保留
                                     会话/2026-11-19-手写笔记/          无 .session.json ✅ 保留
```

## 与 dsh-session-delete 的配合

删除插件一次清理三处：DSH 会话存储、工作区记录、项目里的会话目录。它判断「这个目录属于哪个会话」
**只看** `.session.json`：`provider === "dsh"` **且** `thread_id === <会话ID>`。

| 规范做法 | 删除时的效果 |
| --- | --- |
| 每个目录写 `.session.json`，`thread_id` 用真实会话 ID | 精确命中所删会话的那一个目录 |
| 目录名不含会话 ID | 名字撞车也不会误删 |
| 不同 AI 写各自 `provider` | 别的 AI 的同名目录原样保留 |
| 工程源码放工程目录，过程文件才进会话目录 | 删会话不波及源码 |
| 一次性脚本放会话目录，不放工作区根 | 脚本随会话一起清理 |

没有 `.session.json` 的目录一律保留，插件从不猜测。删除前 30 秒自查：

```bash
cat "会话/<目录>/.session.json"                            # 这个目录属于哪个会话
grep -rl '"thread_id": "<会话ID>"' 会话/*/.session.json     # 有没有重复声明
find "会话/<目录>" -type l                                  # 有没有联接（有则拒绝删除）
```

## 安装

### 插件市场

设置 → 插件市场 → 搜索 `dsh-session-layout` → 安装（市场会复制到 `~/.dsh/skills/`）。

### 手动

```powershell
New-Item -ItemType Directory -Force "$env:USERPROFILE\.dsh\skills\dsh-session-layout" | Out-Null
Copy-Item -Recurse -Force ".\*" "$env:USERPROFILE\.dsh\skills\dsh-session-layout\"
```

## 仓库里有什么

```text
dsh-session-layout/
├─ SKILL.md            技能本体：结构、命名、.session.json 契约、删除规则
├─ 参考/
│  └─ AGENTS.md        可直接放进工作区根的多 AI 共用规范
├─ 模板/
│  └─ 会话骨架/         新建会话目录时直接复制
│     ├─ .session.json
│     ├─ 会话说明.md
│     ├─ 归档索引.md
│     └─ 证据/备份/
├─ README.md           英文说明
└─ README.zh.md        本文件
```

## 披露

- 无云端依赖、无网络请求、不收集数据。
- 只读写工作区内的会话目录与 `AGENTS.md`。

## 许可证

MIT
