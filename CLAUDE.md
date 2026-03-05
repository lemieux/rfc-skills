# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Purpose

Claude Code skills for RFC and technical proposal writing. Skills follow the [Anthropic Agent Skills specification](./spec/agent-skills-spec.md) and work with Claude Code (plugin system), Claude.ai (direct upload), and the Claude API.

## Repository Structure

```
rfc-skills/
├── CLAUDE.md
├── README.md
├── .claude-plugin/
│   └── marketplace.json        # Plugin marketplace registration
├── skills/
│   ├── brainstorming-rfc/
│   │   └── SKILL.md
│   ├── writing-technical-docs/
│   │   ├── SKILL.md
│   │   └── references/
│   │       ├── rfc-example.md
│   │       ├── rfc-style-guide.md
│   │       ├── rfc-reviewer-prompt.md
│   │       └── rfc-template.md
│   ├── finalizing-technical-docs/
│   │   └── SKILL.md
│   └── incorporating-rfc-feedback/
│       └── SKILL.md
├── agents/                     # Subagent definitions invoked by skills
│   ├── rfc-writer.md
│   └── rfc-reviewer.md
├── spec/                       # Agent Skills specification docs
└── template/
    └── SKILL.md                # Starter template for new skills
```

## Skills Overview

| Skill | Purpose |
|-------|---------|
| `brainstorming-rfc` | Pre-RFC exploration and ideation dialogue |
| `writing-technical-docs` | RFC/technical doc writing and review workflow |
| `finalizing-technical-docs` | Mechanical cleanup before publishing |
| `incorporating-rfc-feedback` | Process and apply reviewer feedback |

## Agents

Agents are subagent definitions invoked by skills. They are not user-facing.

- **rfc-writer** - Writes and updates RFC drafts following the style guide. Handles initial drafting from source material and revisions based on feedback. Runs an internal review loop to catch quality issues before returning.
- **rfc-reviewer** - Reviews RFC drafts for style and quality issues. Outputs a structured list of issues (does not fix them). Checks for choppy writing, undefined references, missing diagrams, and other quality problems.

## Creating a New Skill

1. Create folder: `skills/<skill-name>/` (hyphen-case, lowercase)
2. Create `SKILL.md` with required frontmatter:

```yaml
---
name: skill-name          # Must match folder name
description: |            # When/why to use this skill
  Description here
---

# Skill Instructions
[Markdown body...]
```

3. Optional frontmatter: `license`, `allowed-tools`, `metadata`
4. Add to `.claude-plugin/marketplace.json` plugins array to register

## Skill Design Patterns

**Quality Guidelines:**
- Avoid AI-sounding patterns: em dashes, hedging ("It's worth noting"), rhetorical questions, bullet-heavy structure
- Write direct, professional prose with active voice
- Capture the WHY, not just what
- Assume amnesia: write for someone with zero context

**Skill Types:**
- **Workflow skills** (brainstorming-rfc): Guide discovery dialogues, produce structured outputs
- **Style skills** (writing-technical-docs): Enforce patterns and anti-patterns via checklists

## No Build System

This is a documentation-only repository. No compilation, testing, or linting infrastructure exists. Skills are pure markdown instruction files.
