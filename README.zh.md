# dsh-session-layout

[English](README.md) | 中文

一套让多个 AI（DSH / Codex / Antigravity）在同一个工作区里互不串味的**会话目录排版规范**，DSH Skill 形态。
配套插件 [dsh-session-delete](https://github.com/wadwaw7/dsh-session-delete) 依赖它做精确删除。

## 它解决什么

一个工作区里同时跑 DSH、Codex、Antigravity 时，会话产物很容易混在一起：目录名撞车、证据到处散、
删除会话时不敢下手。这套规范给出三条硬约束：

1. **一个对话 = 一个目录**，目录名 `YYYY-MM-DD-HHMM-<中文短主题>`。
2. **每个目录有 `.session.json`**，写着 `provider` 与 `thread_id`（= DSH 的 `DSH_SESSION_ID`）。
3. **两个固定文件**：`会话说明.md`、`归档索引.md`；证据、备份、一次性脚本都在目录内。

## 与 dsh-session-delete 的配合

插件删除会话时同时清理三处：DSH 会话存储、工作区记录、项目里的会话目录。
它判断「这个目录属于哪个会话」**只看** `.session.json` 里的 `provider === "dsh"` 与 `thread_id`：

| 规范做法 | 删除时的效果 |
| --- | --- |
| 每个目录写 `.session.json`，`thread_id` 用真实会话 ID | 精确命中所删会话的那一个目录 |
| 目录名不含会话 ID | 名字撞车也不会误删 |
| 不同 AI 写各自 `provider` | 别的 AI 的同名目录原样保留 |
| 工程源码放在宿主目录、不放进会话目录 | 删除会话不波及源码 |

没有 `.session.json` 的目录，插件一律保留、不猜、不乱删。

## 安装

### 插件市场

设置 → 插件市场 → 搜索 `dsh-session-layout` → 安装（市场会把它复制到 `~/.dsh/skills/`）。

### 手动

```powershell
New-Item -ItemType Directory -Force "$env:USERPROFILE\.dsh\skills\dsh-session-layout" | Out-Null
Copy-Item -Recurse -Force ".\*" "$env:USERPROFILE\.dsh\skills\dsh-session-layout\"
```

## 内容

| 文件 | 作用 |
| --- | --- |
| `SKILL.md` | 技能本体：结构、命名、`.session.json` 契约、与删除插件的配合 |
| `参考/AGENTS.md` | 可直接放进工作区根的多 AI 共用规范 |
| `模板/会话骨架/` | 新建会话目录时直接复制的骨架（含双核心文件与 `.session.json`） |

## 披露

- 无云端依赖，无网络请求，不收集数据。
- 只读写工作区内的 `会话/` 目录与 `AGENTS.md`。

## License

MIT
