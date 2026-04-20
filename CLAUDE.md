# CLAUDE.md — ali5ter/claude-workflow-skills

## Project Status

Live at <https://github.com/ali5ter/claude-workflow-skills>. Installable via the
`ali5ter` plugin marketplace.

## Purpose

`ali5ter/claude-workflow-skills` is a Claude Code plugin that provides common workflow
skills for use in any Claude Code session:

| Skill | Invoke | Purpose |
|---|---|---|
| `promote` | `/promote` | Full release workflow: commit → push → PR → tag → release → clean up |
| `audit-plugin` | `/audit-plugin` | Deep review of a Claude Code plugin/skill/agent against best practices |
| `audit-standards` | `/audit-standards` | Audit project against dev standards in `~/.claude/CLAUDE.md` |
| `improve` | `/improve` | Analyse project for bugs, features, docs, security, competitive landscape, and monetisation |
| `triage` | `/triage` | Validate all open issues, close invalid ones, flag complex ones for planning, fix and promote the rest |

## Architecture

Each skill is a `skills/<name>/SKILL.md` file with YAML frontmatter. The plugin is registered
in `.claude-plugin/plugin.json` and distributed via the `ali5ter/claude-plugins` marketplace.

```text
claude-workflow-skills/
├── .claude-plugin/
│   └── plugin.json        # Plugin manifest (name, version, description)
├── .github/
│   └── workflows/
│       └── weekly-audit.yml  # GitHub Actions weekly standards audit (Sundays 5:55am ET)
├── skills/
│   ├── promote/
│   │   └── SKILL.md       # Release workflow skill
│   ├── audit-plugin/
│   │   └── SKILL.md       # Plugin/skill/agent audit skill
│   ├── audit-standards/
│   │   └── SKILL.md       # Standards compliance audit skill
│   ├── improve/
│   │   └── SKILL.md       # Multi-angle project improvement analysis skill
│   └── triage/
│       └── SKILL.md       # Issue triage: validate, close invalid, flag complex, fix and promote
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

### Interactive vs non-interactive skills

Skills may be invoked interactively (human at a terminal) or non-interactively (GitHub Actions,
CCR scheduled triggers). Skills that prompt for input must handle both:

```bash
if [ -t 0 ]; then
  read -p "Question? " answer   # interactive — prompt the user
else
  answer="safe-default"          # non-interactive — apply a safe default or hard stop
fi
```

Rules of thumb:

- **Hard stop** if proceeding without input would be dangerous (e.g. staging secrets)
- **Skip gracefully** if the prompt is optional context (e.g. issue numbers to close)
- Never leave a skill blocked waiting for input that will never arrive

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

**v1.3.0 — Current (2026-04-20):**

- `triage` skill: validate all open issues, close invalid ones with documentation, surface
  complex ones for planning discussion, fix actionable ones, and run `/promote`

**v1.2.3 — (2026-04-20):**

- `improve` skill: multi-angle analysis across 6 dimensions (code, features, docs, security, competitive, monetisation)
- `audit-standards`: deduplication check prevents re-filing existing open issues on repeated runs
- `promote`: auto-closes GitHub issues via `Closes #N`; `git add -u` instead of `git add -A` to avoid staging secrets
- GitHub Actions `weekly-audit.yml`: runs every Sunday 5:55am ET, filed 59 issues across 6 repos on first run
- `gh auth` pre-flight check (Step 0) added to all four skills
- Non-interactive `[ -t 0 ]` pattern applied throughout; documented in this file

**v1.0.0 — Initial release (2026-04-19):**

- `promote` skill: full release workflow with shell injection for live git context
- `audit-plugin` skill: deep review of Claude Code addons with GitHub issue generation
- `audit-standards` skill: project audit against `~/.claude/CLAUDE.md` dev principles
- Added to `ali5ter/claude-plugins` marketplace
- Replaces the three legacy `.claude/commands/` files in `ali5ter/carrybag-lite`

## Next Steps

- Add `contents:read` to `GH_AUDIT_TOKEN` for private repos to enable file-level audit
- Work through the 59 issues filed across 6 audited repos
- Consider `/review-pr` skill (closes the release lifecycle loop) — issue #4
- `/setup-schedules` skill deferred — issue #7
- GitHub Sponsors + ccpi packaging — issue #6
- Watch for Claude Code plugin framework updates (especially `allowed-tools` scoping syntax)
