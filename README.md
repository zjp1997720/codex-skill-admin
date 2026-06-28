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
- Restores a previous disable run from backup files.
- Counts skills visible in the next Codex prompt.

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
```

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
python3 "$SKILL_DIR/scripts/codex_skill_admin.py" list --cwd "$PWD" --force-reload --disabled
python3 "$SKILL_DIR/scripts/codex_skill_admin.py" prompt-count
```

The disabled list should include the target skills, and `availableSkillCount` should drop versus the pre-run count.

## Repository Layout

```text
.
├── README.md
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
