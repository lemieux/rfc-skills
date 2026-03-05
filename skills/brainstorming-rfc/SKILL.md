---
name: brainstorming-rfc
description: Use before writing any RFC, technical proposal, or design doc. Also use when exploring a technical idea, evaluating a potential change, or when the user says "I want to propose...", "I'm thinking about changing...", "we should consider...", or "how should we approach...". Use when given a PRD that needs technical approach. Triggers before writing-technical-docs skill.
---

# RFC Ideation

## Overview

Transform rough ideas into RFC-ready thinking through collaborative dialogue. This skill is for *before* you open the RFC template—discovering what you're actually proposing and why.

Start by understanding the problem space, then ask questions one at a time to uncover constraints, stakeholders, and alternatives. Once the shape is clear, capture a structured ideation note that feeds into RFC drafting.

## When to Use

- Starting a new RFC or technical proposal
- Unclear on what exactly to propose
- Need to explore trade-offs before committing
- Want to document the "why" behind a proposal
- Exploring a technical idea before formalizing it
- Given a PRD that needs a technical approach

## The Process

**Understanding the problem:**
- Ask what triggered this—what pain, opportunity, or request started this?
- Ask questions one at a time to clarify the problem
- Prefer multiple choice questions when possible
- Focus on: what's broken, who's affected, why now

**Mapping the landscape:**
- Who are the stakeholders? (teams, services, users)
- What are the constraints? (timeline, compatibility, dependencies)
- What's been tried before? (prior art, abandoned approaches)
- What are the risks of doing nothing?

**Exploring approaches:**
- Propose 2-3 different approaches with trade-offs
- Lead with your recommended option and explain why
- For each approach: what it solves, what it costs, what it risks
- Ask which direction resonates before going deeper

**Validating the shape:**
- Present the emerging proposal in sections (200-300 words each)
- Check after each section: "Does this capture it?"
- Cover: problem summary, proposed approach, key trade-offs, open questions
- Be ready to backtrack if something doesn't fit

## Starting from PRD

If a PRD is provided, skip problem discovery and extract what's already defined.

**From the PRD:**
- Goal/problem statement → use directly in Problem Summary
- Stakeholders → verify completeness, add technical reviewers
- Scope/constraints → note as given constraints
- Success criteria → inform approach evaluation

**Still need to discover:**
- Technical constraints (APIs, data formats, compatibility)
- Prior art in the codebase
- Alternative technical approaches
- Implementation risks
- Dependencies on other teams/services

**Adjusted question flow:**

1. "I've read the PRD. Before we explore approaches, are there technical constraints not mentioned here? (Existing APIs we must use, compatibility requirements, performance targets?)"

2. "Has anything similar been attempted before? Any existing patterns we should follow or avoid?"

3. "I see three ways we could implement this: [A], [B], [C]. Which direction feels right, or should I explain the trade-offs?"

4. Present approach in sections, validate each

**PRD output adds to frontmatter:**
```yaml
source-prd: "PRD Name or Link"
```

And the note references the PRD in the Trigger section.

## Discovery Questions

Use these to guide the conversation. One at a time, not as a checklist dump.

**Problem discovery (skip if PRD provided):**
- "What's the trigger for this? What happened or what's about to happen?"
- "Who's feeling the pain most directly?"
- "What happens if we do nothing for 6 months?"

**Constraint discovery:**
- "Are there hard deadlines or dependencies?"
- "What can't change? (APIs, data formats, existing contracts)"
- "What teams or services would this touch?"

**Prior art:**
- "Has this been attempted before? What happened?"
- "Are there existing patterns in the codebase we should follow or avoid?"
- "What would the team expect to see proposed here?"

**Stakeholder mapping (verify if PRD provided):**
- "Who needs to approve this?"
- "Who will implement it?"
- "Who will be affected but isn't in the room?"

**Alternative exploration:**
- "If you couldn't do X, what would you do instead?"
- "What's the smallest version of this that would help?"
- "What's the version you'd build with unlimited time?"

## Output Format

**File path:** `docs/rfc-brainstorm-<topic>.md`

**Frontmatter:**
```yaml
---
project: Project Name
source-prd: PRD Name  # if starting from PRD
status: ready-for-rfc  # or needs-exploration
created: YYYY-MM-DD
tags:
  - rfc
  - ideation
---
```

**Note structure:**
```markdown
# RFC Ideation - <Topic>

## Problem Summary
<2-3 sentences: what's broken and why it matters>
<If from PRD: summarize, don't copy verbatim>

## Trigger
<What prompted this? Incident, request, opportunity?>
<If from PRD: Reference PRD - "See PRD Name for full requirements">

## Stakeholders
- **Approvers**: Person 1, Person 2
- **Implementers**: Team or Person
- **Affected**: <who else is impacted>

## Constraints
- <Hard constraint 1>
- <Hard constraint 2>
- <Timeline if relevant>
- <If from PRD: note which constraints came from PRD vs discovered>

## Approaches Considered

### Option A: <Name> (Recommended)
<What it is, why it's recommended>
- **Solves**: <key benefits>
- **Costs**: <effort, complexity, trade-offs>
- **Risks**: <what could go wrong>

### Option B: <Name>
<What it is, why it's not recommended>
- **Solves**: ...
- **Costs**: ...
- **Risks**: ...

### Option C: <Name> (if applicable)
...

## Open Questions
- [ ] <Question that needs answering before or during RFC>
- [ ] <Question that needs answering before or during RFC>

## Prior Art
- Previous RFC or Decision
- Related Project
- <External links if relevant>

## Next Step
Draft RFC using writing-technical-docs skill.
```

## Content Quality

Notes must be useful when read months later with no memory of this session.

- **Capture the WHY** - Facts without reasoning are useless
- **Assume amnesia** - Write for someone with zero context
- **Reference related documents** - Mention by name for traceability
- **Be specific** - "API latency" not "performance issues"

**Avoid AI voice patterns:**

| Don't write | Write instead |
|-------------|---------------|
| It's worth noting that this approach has risks. | This approach risks X and Y. |
| Let's dive into the constraints. | The constraints are: |
| This is important because it helps facilitate... | This matters because... |
| It could potentially cause issues... | This risks causing... |
| In this section, we will explore... | *(Just start the section)* |

Never use:
- Em dashes — use commas, periods, or parentheses
- "On one hand X, but on the other hand Y"
- Excessive hedging ("might possibly", "could potentially")
- Dramatic rhetorical questions ("But what happens when...?")
- Preamble before sections ("In this section we will...")

### Avoid LLM Voice

| Don't write | Write instead |
|-------------|---------------|
| It's worth noting that this approach has trade-offs. | This approach has trade-offs. |
| Let's dive into the constraints. | Constraints: |
| This is important because it helps facilitate... | This matters because... |
| It could potentially cause issues... | This risks causing... |
| In this section, we will explore... | *(Just start the section)* |

**Never use:**
- Em dashes — use commas, periods, or parentheses
- "Delve", "leverage", "facilitate", "comprehensive", "robust"
- Dramatic rhetorical questions ("But what happens when...?")
- Excessive hedging ("might possibly", "could potentially")
- Preamble before getting to the point

See `writing-technical-docs` skill for full anti-pattern list.

## After Ideation

**If ready:**
- Save the ideation note to `docs/`
- Offer: "Ready to start the RFC draft? I'll use the writing-technical-docs skill."

**If not ready:**
- Note which questions remain open in the Open Questions section
- Set status to `needs-exploration`
- Suggest: "We should answer X before drafting. Want to explore that now or pause here?"

## Key Principles

- **One question at a time** - Don't overwhelm
- **Multiple choice preferred** - Easier to react to options than generate from scratch
- **YAGNI ruthlessly** - If a feature isn't solving the core problem, cut it
- **Explore alternatives** - Always have 2-3 approaches before committing
- **Incremental validation** - Check understanding section by section
- **Capture uncertainty** - Open questions are valuable; don't pretend to know
