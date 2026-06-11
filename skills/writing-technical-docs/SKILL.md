---
name: writing-technical-docs
description: Guides RFC, proposal, and architecture document writing. Use when drafting technical documents, writing design docs, reviewing for style consistency, choosing between PRD/RFC/Implementation Plan, or avoiding AI-obvious patterns in technical writing.
---

# Technical Writing Style

## Overview

Guide for writing and reviewing technical documents (RFCs, proposals, architecture docs) that match a specific style: clear over clever, professional but accessible, audience-focused.

This file covers orchestration: choosing the document type, gathering inputs, and running the write→review→revise loop with subagents. All style and structure rules (anti-patterns, section requirements, diagrams, draft markers) live in `references/rfc-style-guide.md`, which is the single source of truth for the writer and reviewer subagents.

<rfc_constraints>
Critical constraints for RFC writing:
- Target 500-1000 lines. If growing beyond this, pause and ask the user whether to trim.
- Prefer pseudocode over full implementations. Code should illustrate, not be copy-pasteable.
- Present ONE solution as the plan in the Proposal section. Move alternatives to Abandoned Ideas.
- Never include time estimates. Describe phases and dependencies only.
- Write Abandoned Ideas as narrative prose, not templated formats.
</rfc_constraints>

## Document Hierarchy

Technical documents serve different purposes and audiences. Understanding where each document type fits prevents writing at the wrong level of detail.

| Document | Purpose | Audience |
|----------|---------|----------|
| PRD | **Why?** - Business justification, market fit, user needs | Stakeholders, product |
| RFC | **What?** - Architecture decisions, system design, integration points | Engineers, PMs, technical stakeholders |
| Implementation Plan | **How?** - Detailed execution steps, code-level decisions | Engineers doing the work |

**The Wedding Schedule Analogy:**
- **PRD** = "We're having a wedding" (why, vision, budget, guest expectations)
- **RFC** = Wedding schedule (ceremony flow, reception structure, vendor responsibilities, timing, contingencies for rain)
- **Implementation Plan** = DJ's playlist, caterer's prep timeline, photographer's shot list

The schedule tells the wedding party what happens and when, who's responsible for what, and how the pieces connect. It doesn't script the officiant's words or list every canapé. An RFC should feel like this: detailed enough that teams can coordinate, not so detailed that engineers just follow it mechanically.

An RFC documents decisions. An implementation plan documents execution. For the full altitude rules (what belongs in an RFC vs implementation docs, warning signs the RFC is too thin or became a spec), see `references/rfc-style-guide.md`.

## Workflow

### Traditional Flow

PRD → Research → RFC → Circulate for Review → Implementation Plan

### AI-Assisted Flow

With AI tools, iterating on implementation is fast. Use this to inform the RFC:

PRD → Implementation Plan (PoC) → RFC (informed by learnings) → Circulate → Revise Implementation Plan

**Why this works:** The PoC implementation reveals what's actually feasible, surfaces edge cases, and identifies integration points. The RFC then documents the approach with real knowledge, not speculation. After review, the implementation plan gets refined based on feedback.

The PoC implementation plan is disposable. Don't get attached to it.

## When to Use

- Deciding what type of document to write (PRD vs RFC vs Implementation Plan)
- Drafting RFC sections (Background, Problem Statement, Proposal, etc.)
- Writing architecture documentation
- Reviewing technical content for style consistency
- Generating code examples for documentation
- Revising drafts based on reviewer feedback

## Workflow: Write→Review→Revise

This skill uses subagents to keep the main context lean. The RFC draft (500-1000 lines) stays in a file, not in conversation context.

### Reference Documents

- `references/rfc-style-guide.md` - Writing style, structure rules, and quality standards. Used by both writer and reviewer.
- `references/rfc-reviewer-prompt.md` - Structured prompt for the reviewer subagent with output format.
- `references/rfc-template.md` - RFC section template with guidance.
- `references/rfc-example.md` - Example RFC demonstrating proper style (STYLE ONLY - do not use content).

### Phase 1: Gather Inputs

Main agent collects:
- Source material (implementation plan, PRD, etc.)
- Output file path
- Any user requirements or constraints
- **Audience context** (ask the user)

**Ask about audience:**
```
"Who is the primary audience for this RFC?

- Which team(s) will review this?
- Are they familiar with this domain/system, or is this new to them?
- Any external teams who need context on how your systems work?

This helps calibrate how much background and detail to include."
```

**Derive detail level from audience:**

| Audience | Detail Level | What to adjust |
|----------|--------------|----------------|
| Your own team, familiar domain | High-level | Skip basics everyone knows. Focus on what's new/different. Lighter background. |
| Cross-team, mixed familiarity | Standard | Explain integration points. Define your team's systems for others. |
| External teams, unfamiliar domain | Detailed | Fuller background. Define terms. Explain why things work the way they do. |

**Example:**
- "Payments feature for Payments engineers" → High-level. They know Payments.
- "Payments feature involving Platform integration" → Standard. Platform team needs Payments context.
- "New architecture for platform-wide adoption" → Detailed. Many teams, varying familiarity.

### Phase 2: Write (Subagent)

Spawn the `rfc-writer` agent:
```
subagent_type: rfc-skills:rfc-writer
prompt: |
  Source material: [source path]
  Output to: [output path]

  Audience: [who will read this - teams, familiarity level]
  Detail level: [high-level / standard / detailed - derived from audience]

  Write an RFC draft from this source material.
```

The writer reads the style guide, writes the draft to the output file, and returns.

### Phase 3-5: Review Loop

You (the main agent) orchestrate the review loop. Subagents cannot spawn other subagents, so the writer cannot review its own work. After the writer returns, spawn the reviewer:

```
subagent_type: rfc-skills:rfc-reviewer
prompt: |
  Review the RFC at [draft path]
```

The same works for a standalone review pass on an existing draft.

**Do NOT use a code-review agent or skill** - those are for code, not RFC style review.

The reviewer outputs issues only; it does not fix them.

**Loop logic (max 3 iterations):**
1. Parse reviewer output for MAJOR, MODERATE, and MINOR issues
2. If MAJOR or MODERATE issues exist:
   - Feed issue list back to WRITER subagent
   - Writer fixes MAJOR and MODERATE issues, updates draft file
   - Back to reviewer for next pass
3. If only MINOR issues remain:
   - Ask user: "The following MINOR issues were found: [list]. Would you like me to fix these?"
   - If yes → writer fixes MINOR issues, one final review pass
   - If no → proceed to output
4. If no issues → proceed to output
5. If 3 iterations pass with MAJOR/MODERATE issues still open, stop the loop. Present remaining issues to the user and ask how to proceed.

**Severity levels:**
- MAJOR: Structural issues (missing sections, undefined components, wrong abstraction level)
- MODERATE: Readability issues (choppy writing, missing transitions, unclear explanations)
- MINOR: Polish issues (formatting, minor wording)

The writer maintains context across the loop. The reviewer is always fresh (objective review).

### Phase 6: Output

- Return file path to final draft
- Draft contains `<!-- REVIEW: ... -->` markers for items needing human review
- Draft contains Draft Status section at top listing all markers

### Why This Approach?

- **Reviewer outputs issues, not fixes**: Cleaner separation of concerns (matches Anthropic's code-review pattern)
- **Writer gets concrete list**: Clear actionable items to address
- **Fresh reviewer context**: Objective assessment without bias from having written the draft
- **User checkpoint for MINOR issues**: User decides what's worth fixing

## Style and Structure Rules

All writing rules live in `references/rfc-style-guide.md`. Read it before writing or reviewing RFC content directly (without subagents). It covers:

- AI-obvious patterns to avoid (em dashes, hedging, filler phrases, "you" vs "we")
- Vocabulary, tone, sentence structure, and paragraph rules
- Code blocks, tables vs prose, component references, callouts
- Required and optional RFC sections with length targets
- Code examples, API contracts, and diagram formats (ASCII in draft, mermaid in finalize)
- Draft markers (`<!-- REVIEW: ... -->`) and the Draft Status section
- The full review checklist

## Handling Reviewer Feedback

Once the RFC is circulated and external feedback arrives, use the `incorporating-rfc-feedback` skill. It covers evaluating feedback against RFC constraints, when to push back vs fold, and delegating updates back to the `rfc-writer` agent.
