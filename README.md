# claude-workflow-skills

Common workflow skills for [Claude Code](https://claude.ai/code) sessions. Install once via the
[ali5ter plugin marketplace](https://github.com/ali5ter/claude-plugins) and invoke with `/` in any
Claude Code session.

## Skills

| Skill | Invoke | Description |
|---|---|---|
| `promote` | `/promote` | Full release workflow: commit, push, PR, semver tag, GitHub release, branch cleanup |
| `audit-plugin` | `/audit-plugin` | Deep review of a Claude Code plugin/skill/agent against official best practices |
| `audit-standards` | `/audit-standards` | Audit project against personal dev standards in `~/.claude/CLAUDE.md` |
| `improve` | `/improve` | Analyse project for bugs, feature gaps, docs, security, competitive opportunities, and monetisation — file as GitHub issues |

## Installation

### Add the marketplace (once)

```text
/plugin marketplace add ali5ter/claude-plugins
```

### Install this plugin

```text
/plugin install claude-workflow-skills@ali5ter
```

### Update

```text
/plugin update claude-workflow-skills@ali5ter
```

## Usage

Each skill is invoked with its `/` prefix from any Claude Code session:

```text
/promote
/audit-plugin
/audit-standards
/improve
```

Skills can also trigger automatically when Claude Code detects a matching intent in the
conversation — for example, saying "ship this" or "get back to main" may invoke `/promote`
without an explicit slash command.

## Skill details

### `/promote`

Runs the full release workflow for the current project:

1. Commits any uncommitted changes with a conventional commit message
2. Pushes to remote
3. Creates and merges a PR if on a feature branch (with auto-merge)
4. Determines the next semver version from conventional commits since the last tag
5. Creates an annotated git tag and GitHub release with generated release notes
6. Cleans up merged feature branches
7. Returns to a clean `main`

Requires `gh` (GitHub CLI) to be authenticated.

### `/audit-plugin`

Performs a deep review of the Claude Code addon defined in the current project:

- Reads `plugin.json`, `SKILL.md`, and agent `.md` files
- Fetches current best-practice documentation
- Checks frontmatter fields, tool declarations, description format, and body quality
- Creates GitHub issues for each finding
- Writes a prioritised fix plan to the project `CLAUDE.md`

### `/audit-standards`

Audits the current project against the seven development principles in `~/.claude/CLAUDE.md`:

- Codify, Don't Document
- Bash Script UX with pfb
- Markdown Standards
- Professional Documentation Tone
- Version Control Everything

Creates GitHub issues for non-compliance and writes a prioritised fix plan to `CLAUDE.md`.

### `/improve`

Analyses the current project from multiple angles and files GitHub issues for every finding:

- Code quality and bugs — logic errors, anti-patterns, fragile assumptions
- Feature completeness — gaps relative to the project's stated purpose
- Documentation — missing, outdated, or misleading content
- Security — injection risks, exposed secrets, over-permissioned scopes
- Competitive landscape — what similar tools offer that this project doesn't
- Monetisation — sponsorship, premium tiers, marketplace listings, companion products

Produces a prioritised summary table with top issues and highest-leverage opportunities.

## Architecture

Skills are defined in `skills/<name>/SKILL.md` with YAML frontmatter. The `promote` skill uses
shell command injection (`` !`command` ``) to inject live git state before Claude processes the
instructions, giving it accurate context about the current branch, last tag, and recent commits.

## License

[MIT](LICENSE) © [Alister Lewis-Bowen](https://github.com/ali5ter)
