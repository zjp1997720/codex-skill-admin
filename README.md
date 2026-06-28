# Codex Skill Admin

[![skills.sh](https://skills.sh/b/zjp1997720/codex-skill-admin)](https://skills.sh/zjp1997720/codex-skill-admin)

[中文文档](README.zh-CN.md)

Codex Skill Admin is a Codex-only agent skill for auditing, disabling, restoring, and verifying local Codex skills through Codex's local app-server API.

Use it when you want to reduce prompt token load by disabling unused skills without uninstalling them.

## Install

```bash
npx skills add zjp1997720/codex-skill-admin -g -a codex --skill codex-skill-admin -y
```

List the skill before installing:

```bash
npx skills add zjp1997720/codex-skill-admin --list
```

The repository uses the Agent Skills layout supported by `npx skills`: the installable skill lives at `skills/codex-skill-admin/SKILL.md`, with helper scripts and references beside it.

## Requirements

- Codex CLI with `codex app-server`
- Python 3.10+
- A local skill installation directory. `npx skills` commonly installs Codex-compatible skills under `$HOME/.agents/skills`; direct Codex installs commonly use `${CODEX_HOME:-$HOME/.codex}/skills`.

## What It Does

- Lists enabled and disabled Codex skills.
- Audits recently used skills from local Codex session evidence.
- Disables unused enabled skills with a dry-run first.
- Supports low-frequency cleanup, such as disabling skills used at most 2 times in the last 10 days.
- Restores a previous disable run from backup files.
- Counts skills visible in the next Codex prompt.
- Explains the difference between the desktop UI total count and effective enabled/prompt-visible counts.

## How It Works

Codex loads skill metadata into the prompt so it can decide which skill to use. When many skills stay enabled, the prompt gets heavier even if most of them are not useful for the current task.

This tool looks at local Codex session files and searches for real `SKILL.md` reads. If a skill was read recently, it is treated as used. If it was not read, or if it was read no more than your `--max-uses` threshold, it becomes a disable candidate.

Disabling a skill does not delete it. The script calls Codex's local `skills/config/write` API to mark the skill as disabled, writes a local backup, and lets you restore it later. The desktop UI may still count the skill because the file still exists; the important numbers are enabled skills and prompt-visible skills.

## Safety Defaults

- `disable-unused` is a dry run unless `--apply` is passed.
- `set` is a dry run unless `--apply` is passed.
- System skills are preserved by default. Use `--include-system` only when you explicitly want to consider system skills.
- Apply-mode backups are written under `${CODEX_HOME:-$HOME/.codex}/backup/`.
- Backup files contain local skill paths and usage evidence. Treat them as private machine-local diagnostics.

## Usage

```bash
SKILL_DIR="${CODEX_SKILL_ADMIN_DIR:-$HOME/.agents/skills/codex-skill-admin}"
if [ ! -d "$SKILL_DIR" ]; then
  SKILL_DIR="${CODEX_HOME:-$HOME/.codex}/skills/codex-skill-admin"
fi

python3 "$SKILL_DIR/scripts/codex_skill_admin.py" list --cwd "$PWD"
python3 "$SKILL_DIR/scripts/codex_skill_admin.py" audit-unused --cwd "$PWD" --days 30
python3 "$SKILL_DIR/scripts/codex_skill_admin.py" disable-unused --cwd "$PWD" --days 30
python3 "$SKILL_DIR/scripts/codex_skill_admin.py" disable-unused --cwd "$PWD" --days 30 --apply
python3 "$SKILL_DIR/scripts/codex_skill_admin.py" verify --cwd "$PWD"
```

Disable skills used at most 2 times in the last 10 days:

```bash
python3 "$SKILL_DIR/scripts/codex_skill_admin.py" disable-unused --cwd "$PWD" --days 10 --max-uses 2
python3 "$SKILL_DIR/scripts/codex_skill_admin.py" disable-unused --cwd "$PWD" --days 10 --max-uses 2 --apply
```

`usageCount` is based on distinct local session/source files, not repeated reads inside one session.

Restore a previous run:

```bash
python3 "$SKILL_DIR/scripts/codex_skill_admin.py" restore --backup-dir "${CODEX_HOME:-$HOME/.codex}/backup/skill-disable-unused-YYYYMMDD-HHMMSS"
```

Toggle one explicit skill:

```bash
python3 "$SKILL_DIR/scripts/codex_skill_admin.py" set --name codex-skill-admin --no-enabled
python3 "$SKILL_DIR/scripts/codex_skill_admin.py" set --name codex-skill-admin --no-enabled --apply
```

## Verification

After applying changes:

```bash
python3 "$SKILL_DIR/scripts/codex_skill_admin.py" verify --cwd "$PWD"
python3 "$SKILL_DIR/scripts/codex_skill_admin.py" list --cwd "$PWD" --force-reload --disabled
python3 "$SKILL_DIR/scripts/codex_skill_admin.py" prompt-count
```

The disabled list should include the target skills. `enabledCount` should drop, and `availableSkillCount` should drop when disabled skills were previously visible in the prompt.

The Codex desktop Skills tab count is a total discovered skill count. It may stay unchanged after disabling skills because disabled skills are still installed and visible in the management UI.

## Repository Layout

```text
.
├── README.md
├── README.zh-CN.md
├── LICENSE
├── skills.sh.json
└── skills/
    └── codex-skill-admin/
        ├── SKILL.md
        ├── agents/openai.yaml
        ├── scripts/codex_skill_admin.py
        └── references/app-server-protocol.md
```

## License

MIT
