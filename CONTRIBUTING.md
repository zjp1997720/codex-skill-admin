# Contributing

Changes should keep the skill portable, conservative, and easy to verify.

## Guidelines

- Do not hard-code personal paths, usernames, or machine-specific directories.
- Keep `SKILL.md` focused on agent instructions.
- Put deterministic behavior in `scripts/`.
- Preserve dry-run defaults for write operations.
- Preserve system skills by default unless a user explicitly opts in.
- Update README examples when CLI flags change.

## Validate

```bash
python3 -m py_compile skills/codex-skill-admin/scripts/codex_skill_admin.py
python3 "$HOME/.codex/skills/.system/skill-creator/scripts/quick_validate.py" skills/codex-skill-admin
npx skills add . --list
```
