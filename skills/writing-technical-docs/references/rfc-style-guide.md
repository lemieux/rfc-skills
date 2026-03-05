# RFC Style Guide

Reference document for writing and reviewing RFCs. Used by both writer and reviewer subagents.

> **WARNING**: The example RFC (`rfc-example.md`) demonstrates STYLE and FORMAT only.
> DO NOT use any content from the example in your RFC. The example topic
> (transit route optimization, MetroLink) is fictional and unrelated to your task.
> Study the structure, tone, and formatting patterns - not the subject matter.

## RFC Constraints

- Target 500-1000 lines. If growing beyond this, trim content.
- Prefer pseudocode over full implementations. Code should illustrate, not be copy-pasteable.
- Present ONE solution in the Proposal section. Move alternatives to Abandoned Ideas.
- Never include time estimates. Describe phases and dependencies only.
- Write Abandoned Ideas as narrative prose, not templated formats.

## Document Hierarchy

| Document | Purpose | Audience |
|----------|---------|----------|
| PRD | **Why?** - Business justification, market fit, user needs | Stakeholders, product |
| RFC | **What?** - Architecture decisions, system design, integration points | Engineers, PMs, technical stakeholders |
| Implementation Plan | **How?** - Detailed execution steps, code-level decisions | Engineers doing the work |

An RFC documents decisions. An implementation plan documents execution.

### RFC Altitude

The RFC must hit the right level of detail:

- **High enough:** A PM can understand complexity and integration points
- **Detailed enough:** Engineers understand how the design maps to business requirements
- **Not so detailed:** It reads like a recipe

### What Belongs in an RFC

- Architecture decisions and rationale
- API contracts (request/response shapes)
- Component responsibilities
- Key flows (happy path, error recovery)
- Design principles with reasoning

### What Does NOT Belong

- Full client/server code implementations
- Retry logic, backoff algorithms, error handling code
- Configuration files (Redis YAML, Kubernetes manifests)
- Monitoring dashboards and alert thresholds
- Operational runbooks

## Patterns to Avoid

These patterns signal AI-generated text:

| Don't write | Write instead | Why |
|-------------|---------------|-----|
| It's worth noting that... | [Just state the fact] | Filler phrases add no information |
| Let's dive into... | The X works as follows. | Casual phrasing feels out of place |
| This helps facilitate... | This enables... | "Facilitate" is corporate jargon |
| On one hand X, but on the other hand Y | X. However, Y. | Overly balanced hedging |
| It could potentially cause... | This risks causing... | Excessive hedging weakens message |
| In this section, we will explore... | [Just start the section] | Preambles waste time |
| Em dashes for asides | Commas, periods, or parentheses | Em dashes signal AI text |
| "The X Trap" or "The X Problem" | Descriptive headers | Dramatic headers feel clickbaity |
| Dramatic rhetorical questions | Direct statements | Questions feel condescending |
| Lists starting with same word | Varied list openers | Repetitive structure feels mechanical |
| "You need to..." | "We need to..." | RFCs are the team thinking together |

## Writing Style

### Vocabulary

- Use straightforward words. Avoid fancy vocabulary.
- Prefer "use" over "utilize", "help" over "facilitate"
- Technical terms are fine when domain-appropriate
- No buzzwords or corporate jargon

### Tone

- Direct and professional, not casual
- Confident but not arrogant
- Explain the "why" behind decisions
- Acknowledge tradeoffs honestly
- Use "we" not "you"

### Sentence Structure

**Critical:** Vary sentence length. Connect related ideas with reasoning words (because, so, which means, since).

**Choppy (avoid):**
> Redis pub/sub is fire-and-forget. If no subscriber is listening, the message is lost. The buffer is authoritative. Pub/sub is an optimization.

**Connected (better):**
> Redis pub/sub is fire-and-forget, meaning if no subscriber is listening when a message is published, it's gone. Because of this, we treat the buffer as the authoritative source. Pub/sub pushes chunks for low-latency delivery, but the buffer is what makes the system reliable.

### Paragraphs

- One idea per paragraph
- First sentence conveys the paragraph's purpose
- 3-5 sentences typical, never walls of text

## Content Structure Rules

### Section Delimiters

Do NOT use horizontal lines (`---`) between sections. Headers already provide visual separation. Horizontal lines add clutter without value.

### Code Blocks

Code blocks are ONLY for:
- Actual code (implementations, pseudocode)
- Configuration files (YAML, JSON)
- Commands (CLI, API calls)
- Data contracts (request/response shapes)

Never use code blocks for prose, section organization, or emphasis.

### Tables vs Prose

- **Use tables** for dense reference data: API fields, status codes, configuration options
- **Use prose with subheaders** for items needing explanation: risks, tradeoffs, design decisions

Tables compress information. When readers need to understand *why*, prose is better.

### Component References

Define components before referencing them. If you mention "WebSocket pods" or "the buffer service," explain what they are first.

### Callouts

Use for scope limitations or important caveats:
```
| info | Important note or caveat here |
| :---- |
```

## Diagrams

Every diagram reference MUST include an implementation. Placeholder descriptions alone are not acceptable.

### Draft Phase: Use ASCII

During drafting, use ASCII diagrams. Mermaid inside markdown tables doesn't render well, making review difficult. ASCII is easier to visualize and edit during the draft cycle.

**Format:** Wrap in a table with caption:

```markdown
| |
|:---:|
| `Client → Redis Buffer → Delivery Pod → Widget` |
| *Caption: High-level data flow for stream delivery* |
```

**Simple linear flow:**
```
Client → Redis Buffer → Delivery Pod → Widget
```

**Multi-step with actors:**
```
┌────────┐    ┌────────┐    ┌────────┐
│ Client │───▶│ Buffer │───▶│Delivery│
└────────┘    └────────┘    └────────┘
     │                           │
     └───────────────────────────┘
              (ack)
```

**State transitions:**
```
[Start] → Idle → Connecting → Connected
                      ↓
                   Error
```

### Finalize Phase: Convert to Mermaid

After the draft is approved, convert ASCII diagrams to mermaid for easier export to Lucidchart. This happens in the finalize step, not during drafting.

Mermaid types to use:
- `sequenceDiagram` for request/response flows
- `flowchart` for architecture and data flow
- `stateDiagram-v2` for state machines

## API Contracts

Show actual request/response shapes, not just prose descriptions.

**Prose only (avoid):**
> Returns streamId and a short-lived token.

**With data contract (better):**
> **Response (200):**
> ```json
> {
>   "streamId": "stream_xyz789",
>   "streamToken": "eyJhbG...",
>   "streamingSupported": true
> }
> ```

## Draft Markers

Use HTML comments to flag items for human review:

```
<!-- REVIEW: description of what needs review -->
```

**Make decisions, then mark for review.** Don't leave blanks.

### Marker Placement (BOTH locations required)

REVIEW markers go in TWO places:

1. **Inline** - At the exact location in the document where the decision was made
2. **Draft Status** - Listed at the top for easy scanning

**Example inline:**
```markdown
We chose Redis for the buffer because it supports pub/sub natively.
<!-- REVIEW: Assumed Redis cluster of 3 nodes. Platform team should validate. -->
```

**Example in Draft Status:**
```markdown
## Draft Status

**Items for review:**
- [ ] <!-- REVIEW: Assumed Redis cluster of 3 nodes. Platform team should validate. -->
```

The inline marker shows WHERE in the document the decision lives. The Draft Status list makes it easy to see ALL decisions at a glance.

### Making Decisions

- `<!-- REVIEW: Chose 24h token expiration. Confirm this works for long sessions. -->`
- `<!-- REVIEW: Assumed Redis cluster of 3 nodes. Platform team should validate. -->`

**For truly ambiguous choices** that don't impact other parts, produce both versions:

```markdown
<!-- REVIEW: Two valid approaches. Pick one and delete the other. -->

**Option A: Eager loading**
[content]

**Option B: Lazy loading**
[content]

<!-- END OPTIONS -->
```

### Draft Status Section

Every draft starts with:

```markdown
## Draft Status

**State:** Draft

**Items for review:**
- [ ] <!-- REVIEW: item 1 -->
- [ ] <!-- REVIEW: item 2 -->

---
```

## RFC Sections

### Required Sections

| Section | Length | Purpose |
|---------|--------|---------|
| Abstract | 2-3 sentences | 10-second understanding |
| Background | As needed | All context needed to follow the rest |
| Proposal | 40-60% of RFC | The actual solution. ONE approach, not options. |
| Abandoned Ideas | 150-300 words per alternative | Narrative explaining what, why attractive, why rejected |

### Optional Sections

| Section | When to Include |
|---------|-----------------|
| Glossary | Domain-specific jargon exists |
| Problem Statement | Background alone doesn't make problem obvious |
| Bill of Work | Multiple teams/components involved |
| Rollout | Deployment strategy needed (no time estimates) |
| Risks | Non-trivial risks exist (use prose with subheaders, not tables) |
| Future Steps | Change enables longer-term strategy |

## Review Checklist

When reviewing, check for:

### Structure
- [ ] Follows RFC template sections
- [ ] Background sufficient for someone with no context
- [ ] Proposal is meatier than Background + Problem Statement combined
- [ ] Abandoned Ideas documented with full reasoning
- [ ] Diagrams use ASCII with table container and caption (mermaid in finalize phase)
- [ ] No horizontal lines (`---`) between sections

### Clarity (CRITICAL - Check Every Section)
- [ ] **CHOPPY WRITING CHECK**: Read each section aloud. If it sounds like a bulleted list read as sentences, it's choppy. Look for:
  - Three or more consecutive short sentences (under 10 words each)
  - Paragraphs where every sentence starts with the subject
  - Missing transition words (because, so, which means, since, however)
  - Sections that feel like facts dumped in a list rather than explained
- [ ] Sentences vary in length and connect with reasoning
- [ ] Each paragraph focuses on one idea
- [ ] No AI-obvious patterns (em dashes, "Let's dive in", etc.)
- [ ] Uses "we" not "you"

### Completeness
- [ ] The "why" explained, not just the "what"
- [ ] Components defined before referenced
- [ ] Security implications addressed
- [ ] Edge cases discussed
- [ ] REVIEW markers appear BOTH inline AND in Draft Status section

### Technical Accuracy
- [ ] API contracts show actual JSON shapes
- [ ] Diagrams have ASCII implementation, not just descriptions
- [ ] Code blocks used only for actual code
- [ ] No content from example RFC leaked into this document

## Warning Signs: RFC Became a Spec

- Table of Contents needed (too long)
- Multiple appendices
- Config file examples
- Full class implementations
- Retry/backoff code
- Week-by-week rollout schedules
