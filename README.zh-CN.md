# Codex Skill Admin

[English](README.md)

Codex Skill Admin 是一个 Codex 专用的 agent skill，用来通过 Codex 本地 `app-server` API 审计、关闭、恢复和验证本机 Codex skills。

适合在你想节省 prompt token 时使用：它会帮你找出近期没有使用证据的 skill，并把它们关闭，而不是卸载。

## 安装

```bash
npx skills add zjp1997720/codex-skill-admin -g -a codex --skill codex-skill-admin -y
```

安装前查看仓库里有哪些 skill：

```bash
npx skills add zjp1997720/codex-skill-admin --list
```

这个仓库使用 `npx skills` 支持的 Agent Skills 结构：可安装的 skill 位于 `skills/codex-skill-admin/SKILL.md`，辅助脚本和协议参考文件放在同一个 skill 目录里。

## 环境要求

- 支持 `codex app-server` 的 Codex CLI
- Python 3.10+
- 本地 skill 安装目录。`npx skills` 通常会把 Codex 兼容 skill 安装到 `$HOME/.agents/skills`；Codex 原生安装通常使用 `${CODEX_HOME:-$HOME/.codex}/skills`。

## 功能

- 列出当前启用和关闭的 Codex skills。
- 根据本地 Codex session 证据审计近期使用过的 skills。
- 先 dry-run，再关闭近期未使用的已启用 skills。
- 从备份恢复上一次关闭操作。
- 统计下一次 Codex prompt 中实际可见的 skill 数量。
- 区分桌面 UI 总数和真正影响 token 的启用数 / prompt 可见数。

## 安全默认值

- `disable-unused` 默认只 dry-run，必须传 `--apply` 才会真正关闭。
- `set` 默认只 dry-run，必须传 `--apply` 才会真正写入配置。
- system skills 默认保留。只有明确传 `--include-system` 时才会把 system skills 纳入候选。
- apply 模式的备份写入 `${CODEX_HOME:-$HOME/.codex}/backup/`。
- 备份文件包含本机 skill 路径和使用证据，按私有本机诊断数据处理。

## 使用

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

恢复上一次操作：

```bash
python3 "$SKILL_DIR/scripts/codex_skill_admin.py" restore --backup-dir "${CODEX_HOME:-$HOME/.codex}/backup/skill-disable-unused-YYYYMMDD-HHMMSS"
```

关闭或启用某一个明确的 skill：

```bash
python3 "$SKILL_DIR/scripts/codex_skill_admin.py" set --name codex-skill-admin --no-enabled
python3 "$SKILL_DIR/scripts/codex_skill_admin.py" set --name codex-skill-admin --no-enabled --apply
```

## 验证

应用修改后运行：

```bash
python3 "$SKILL_DIR/scripts/codex_skill_admin.py" verify --cwd "$PWD"
python3 "$SKILL_DIR/scripts/codex_skill_admin.py" list --cwd "$PWD" --force-reload --disabled
python3 "$SKILL_DIR/scripts/codex_skill_admin.py" prompt-count
```

成功标准：

- `--disabled` 输出里能看到目标 skills。
- `enabledCount` 相比操作前下降。
- 如果被关闭的 skills 原本会进入 prompt，`availableSkillCount` 相比操作前下降。

Codex 桌面 UI 顶部的「技能」数量统计的是已发现的 skill 总数，不是启用数量。关闭 skill 后，这个 UI 数字可以保持不变；真正影响 token 的指标是 `enabledCount` 和 `availableSkillCount`。

## 仓库结构

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

## 许可证

MIT
