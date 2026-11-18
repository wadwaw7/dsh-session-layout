# dsh-session-layout

**English** · [中文](README.zh.md) · Companion plugin: [dsh-session-delete](https://github.com/wadwaw7/dsh-session-delete)

A **session folder layout convention** for workspaces shared by several AI agents
(DSH / Codex / Antigravity), packaged as a DSH Skill.

It is the companion spec for [dsh-session-delete](https://github.com/wadwaw7/dsh-session-delete):
that plugin deletes a DSH session together with its project folder, and it recognises the
folder **only** through this convention.

## What it fixes

Running DSH, Codex and Antigravity in one workspace quickly produces a mess: colliding folder
names, evidence scattered across the root, and nobody dares to delete a session because it is
unclear what belongs to what. The convention is three hard rules:

1. **One conversation, one directory**, named `YYYY-MM-DD-HHMM-<topic>`.
2. **Every directory carries `.session.json`** with `provider` and `thread_id`
   (the DSH `DSH_SESSION_ID`).
3. **Two fixed files** — the session notes and the archive index; evidence, backups and
   one-off scripts stay inside the directory.

## Layout at a glance

```text
<workspace>/
├─ AGENTS.md                        rules shared by every agent
├─ 会话/                            one folder per conversation
│  ├─ 2026-11-18-1945-site-audit/
│  │  ├─ .session.json              identity: provider + thread_id
│  │  ├─ 会话说明.md                 notes: goal / decisions / results
│  │  ├─ 归档索引.md                 table of every artifact produced
│  │  ├─ probe.mjs                  one-off script used in this conversation
│  │  └─ 证据/
│  │     └─ 备份/                    backups taken before a change
│  └─ 2026-11-18-2010-other-task/
│     └─ …
└─ my-tool/                         product source, build output, real docs
```

`.session.json` is the whole point:

```json
{
  "provider": "dsh",
  "thread_id": "session-59f06ece-8970-4c3e-b961-c764ba27bdae",
  "topic": "site-audit",
  "created_at": "2026-11-18T19:45:00+08:00"
}
```

| Field | Meaning | Value |
| --- | --- | --- |
| `provider` | Which agent produced the folder | `dsh` / `codex` / `antigravity` |
| `thread_id` | Host session id, identical to DSH `DSH_SESSION_ID` | `session-<uuid>` for DSH |
| `topic` | Human label, display only — **never matched** | free text |
| `created_at` | Creation time, ISO 8601 with offset | string |

Write it together with the folder; `topic` may be edited later, `thread_id` never.

## How deletion resolves a folder

```text
session-59f0…  ──thread_id match──▶  会话/2026-11-18-1945-site-audit/   .session.json ✅ DELETE
                                      会话/2026-11-18-2010-other-task/  thread_id 77ab… ✅ KEEP
                                      会话/2026-11-19-manual-notes/     no .session.json ✅ KEEP
```

## Working with dsh-session-delete

The delete plugin removes three things at once: DSH session storage, the workspace record, and
the session folder inside the project. It decides "which session owns this folder" from
`.session.json` alone — `provider === "dsh"` **and** `thread_id === <session id>`:

| Convention | Effect when deleting |
| --- | --- |
| Every folder has `.session.json` with the real session id | Exactly one folder is matched |
| Folder names never embed a session id | Name collisions cannot cause a wrong delete |
| Each AI writes its own `provider` | Another AI's same-named folder is left untouched |
| Product source lives in the project directory, not the session folder | Deleting a session never touches source code |
| One-off scripts live in the session folder, not the workspace root | Scripts are cleaned up with the session |

A folder without `.session.json` is always kept: the plugin never guesses. Thirty seconds
before deleting:

```bash
cat "会话/<dir>/.session.json"                             # which session owns this folder
grep -rl '"thread_id": "<sessionId>"' 会话/*/.session.json # any duplicate claim
find "会话/<dir>" -type l                                 # symlinks (deletion is refused)
```

## Install

### Plugin marketplace

Settings → Plugin Marketplace → search `dsh-session-layout` → Install. The marketplace copies
it to `~/.dsh/skills/`.

### Manual

```powershell
New-Item -ItemType Directory -Force "$env:USERPROFILE\.dsh\skills\dsh-session-layout" | Out-Null
Copy-Item -Recurse -Force ".\*" "$env:USERPROFILE\.dsh\skills\dsh-session-layout\"
```

## What is in this repo

```text
dsh-session-layout/
├─ SKILL.md            the skill: structure, naming, .session.json contract, deletion rules
├─ 参考/
│  └─ AGENTS.md        drop-in workspace rules shared by every agent
├─ 模板/
│  └─ 会话骨架/         copy this to start a new session folder
│     ├─ .session.json
│     ├─ 会话说明.md
│     ├─ 归档索引.md
│     └─ 证据/备份/
├─ README.md           this file (English)
└─ README.zh.md        Chinese
```

## Disclosure

- No cloud dependency, no network requests, no data collection.
- Reads and writes only the session folder inside the workspace, plus `AGENTS.md`.

## License

MIT
