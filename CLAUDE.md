# CLAUDE.md — ali5ter/claude-workflow-skills

## Project Status

Live at <https://github.com/ali5ter/claude-workflow-skills>. Installable via the
`ali5ter` plugin marketplace.

## Purpose

`ali5ter/claude-workflow-skills` is a Claude Code plugin that provides three common workflow
skills for use in any Claude Code session:

| Skill | Invoke | Purpose |
|---|---|---|
| `promote` | `/promote` | Full release workflow: commit → push → PR → tag → release → clean up |
| `audit-plugin` | `/audit-plugin` | Deep review of a Claude Code plugin/skill/agent against best practices |
| `audit-standards` | `/audit-standards` | Audit project against dev standards in `~/.claude/CLAUDE.md` |

## Architecture

Each skill is a `skills/<name>/SKILL.md` file with YAML frontmatter. The plugin is registered
in `.claude-plugin/plugin.json` and distributed via the `ali5ter/claude-plugins` marketplace.

```text
claude-workflow-skills/
├── .claude-plugin/
│   └── plugin.json        # Plugin manifest (name, version, description)
├── skills/
│   ├── promote/
│   │   └── SKILL.md       # Release workflow skill
│   ├── audit-plugin/
│   │   └── SKILL.md       # Plugin/skill/agent audit skill
│   └── audit-standards/
│       └── SKILL.md       # Standards compliance audit skill
├── CLAUDE.md              # This file
├── README.md              # User-facing documentation
├── LICENSE                # MIT
├── .gitignore
└── .markdownlint.json     # MD013 line-length 120, front_matter: false
```

## Skill Development Notes

### Frontmatter constraints

- `description` must be a single unbroken line (no `\n`, no block scalars)
- `allowed-tools` follows least-privilege — only tools the skill actually needs
- `front_matter: false` in `.markdownlint.json` exempts frontmatter from MD013 line-length checks

### Shell injection

The `promote` skill uses `` !`command` `` blocks to inject live git state (branch, status,
last tag, recent commits) before Claude processes the instructions. This ensures the skill
operates on accurate current context without requiring an extra read step.

### Skill body guidelines

- Steps are numbered and actionable
- Code blocks always specify a language
- Prose lines ≤ 120 characters
- Large reference content goes in separate files, referenced from SKILL.md

## Install Commands

```text
/plugin marketplace add ali5ter/claude-plugins
/plugin install claude-workflow-skills@ali5ter
```

## Current Status

**v1.0.0 — Initial release (2026-04-19):**

- `promote` skill: full release workflow with shell injection for live git context
- `audit-plugin` skill: deep review of Claude Code addons with GitHub issue generation
- `audit-standards` skill: project audit against `~/.claude/CLAUDE.md` dev principles
- Added to `ali5ter/claude-plugins` marketplace
- Replaces the three legacy `.claude/commands/` files in `ali5ter/carrybag-lite`

## Next Steps

- Gather real-world usage feedback
- Watch for Claude Code plugin framework updates (especially `allowed-tools` scoping syntax)
- Consider additional workflow skills (e.g., `review-pr`, `triage-issues`)
