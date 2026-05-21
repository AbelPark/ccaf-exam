## skill priority

1. enterprise: managed-settings.json
2. personal: ~/.claude/skills
3. project: project/.claude/skills
4. plugin: project/.claude-plugins /skills

---

## configuration multi-file skills

`name` and `description` are required — `allowed-tools` and `model` are optional but powerful additions

`allowed-tools` restricts which tools Claude can use when the skill is active — useful for read-only or security-sensitive workflows

keep SKILL.md under **500 lines** and link to supporting files (references, scripts, assets) that Claude reads only when needed

allowed-tools restricts which tools Claude can use when the skill is active — useful for read-only or security-sensitive workflows

The open standard suggests organizing your skill directory with:
scripts/ — Executable code (eg. make-template.js)
references/ — Additional documentation (eg. tools-codex.md)
assets/ — Images, templates, or other data files (eg. app-icon.svg)

---

## Skills vs. other Claude Code features

**CLAUDE.md** loads into _every conversation_ and is best for always-on project standards.
**Skills** load on demand and are best for _task-specific_ expertise
**Subagents** run in _isolated execution contexts_ — use them for delegated work. Skills add knowledge to your current conversation
**Hooks** are _event-driven_ (fire on file saves, tool calls). Skills are request-driven (activate based on what you're asking)
**MCP servers** provide _external tools and integrations_ — a different category entirely from skills

---

## Sharing skills

**Project skills** in .claude/skills are _shared automatically through Git_ — anyone who clones the repo gets them
**Plugins** let you distribute skills _across repositories via marketplaces for broader community use_
**Enterprise managed settings** deploy skills _organization-wide with the highest priority_, ideal for mandatory standards and compliance
**Subagents** _don't automatically see your skills_ — you must explicitly list skills in a custom agent's frontmatter _skills field_
**Built-in agents** (Explorer, Plan, Verify) _can't access skills at all_ — _only custom subagents_ defined in .claude/agents can

### Committing Skills to Your Repository

Place them in `.claude/skills`
Team coding standards
Project-specific workflows
Skills that reference your codebase structure

### Distributing Skills Through Plugins

In your plugin project, create a `skills` directory that follows a similar file structure to the `.claude` directory — each skill gets its own folder with a `SKILL.md` file inside.
This approach is best when your skills aren't too project-specific and can be useful to community members beyond your immediate team.

### Enterprise Deployment Through Managed Settings

The managed settings file supports features like `strictKnownMarketplaces` to control where plugins can be installed from:

```
"strictKnownMarketplaces": [
  {
    "source": "github",
    "repo": "acme-corp/approved-plugins"
  },
  {
    "source": "npm",
    "package": "@acme-corp/compliance-plugins"
  }
]
```

This is the right choice for mandatory standards, security requirements, compliance workflows, and coding practices that must be consistent across the organization. The keyword here is "must."

### Skills and Subagents

subagents don't automatically see your skills.
**Built-in agents (like Explorer, Plan, and Verify)** can't access skills at all
**Custom subagents** you define can use skills, but only when you explicitly list them
The generated agent file includes a skills field that lists which skills to load. Here's what the frontmatter looks like:

```
---
name: frontend-security-accessibility-reviewer
description: "Use this agent when you need to review frontend code for accessibility..."
tools: Bash, Glob, Grep, Read, WebFetch, WebSearch, Skill...
model: sonnet
color: blue
skills: accessibility-audit, performance-check
---
```

## Troubleshooting skills

**Start with the skills validator tool** — it catches structural problems before you spend time debugging other things
**If a skill doesn't trigger**, the cause is almost always the description — add trigger phrases that match how you actually phrase requests
**If a skill doesn't load**, check that SKILL.md is inside a named directory (not at the skills root) and the file name is exactly SKILL.md
**If the wrong skill gets used**, your descriptions are too similar — make them more distinct
For runtime errors, check dependencies, file permissions (chmod +x), and path separators (use forward slashes everywhere)
