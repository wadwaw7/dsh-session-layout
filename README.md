# dsh-session-layout

English | [中文](README.zh.md)

A **session directory layout convention** for workspaces shared by several AI agents
(DSH / Codex / Antigravity), packaged as a DSH Skill.

It is the companion spec for [dsh-session-delete](https://github.com/wadwaw7/dsh-session-delete):
that plugin deletes a DSH session together with its project folder, and it recognises
the folder **only** through this convention.

## What it fixes

Running DSH, Codex and Antigravity in one workspace quickly produces a mess: colliding
folder names, evidence files scattered across the root, and nobody dares to delete a
session because it is unclear what belongs to what. The convention is three hard rules:

1. **One conversation, one directory**, named `YYYY-MM-DD-HHMM-<topic>`.
2. **Every directory carries `.session.json`** with `provider` and `thread_id`
   (the DSH `DSH_SESSION_ID`).
3. **Two fixed files** — `会话说明.md` (session notes) and `归档索引.md` (archive index);
   evidence, backups and one-off scripts stay inside the directory.

## Working with dsh-session-delete

The delete plugin removes three things at once: the DSH session storage, the workspace
record, and the session folder inside the project. It decides "which session owns this
folder" from `.session.json` alone — `provider === "dsh"` **and**
`thread_id === <session id>`:

| Convention | Effect when deleting |
| --- | --- |
| Every directory has `.session.json` with the real session id | Exactly one folder is matched |
| Folder names never embed a session id | Name collisions cannot cause a wrong delete |
| Each AI writes its own `provider` | Another AI's same-named folder is left untouched |
| Product source lives in the project directory, not the session folder | Deleting a session never touches source code |

A directory without `.session.json` is always kept: the plugin never guesses.

## Install

### Plugin marketplace

Settings → Plugin Marketplace → search `dsh-session-layout` → Install. The marketplace
copies it to `~/.dsh/skills/`.

### Manual

```powershell
New-Item -ItemType Directory -Force "$env:USERPROFILE\.dsh\skills\dsh-session-layout" | Out-Null
Copy-Item -Recurse -Force ".\*" "$env:USERPROFILE\.dsh\skills\dsh-session-layout\"
```

```bash
mkdir -p ~/.dsh/skills/dsh-session-layout
cp -r ./* ~/.dsh/skills/dsh-session-layout/
```

## Contents

| File | Purpose |
| --- | --- |
| `SKILL.md` | The skill itself: structure, naming, the `.session.json` contract, deletion integration |
| `参考/AGENTS.md` | Drop-in workspace-wide rules shared by every agent |
| `模板/会话骨架/` | Skeleton to copy when starting a new session directory |

## Disclosure

- No cloud dependency, no network requests, no data collection.
- Reads and writes only `会话/` inside the workspace and `AGENTS.md`.

## License

MIT
