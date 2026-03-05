---
name: writing-technical-docs
description: Guides RFC, proposal, and architecture document writing. Use when drafting technical documents, writing design docs, reviewing for style consistency, choosing between PRD/RFC/Implementation Plan, or avoiding AI-obvious patterns in technical writing.
---

# Technical Writing Style

## Overview

Guide for writing and reviewing technical documents (RFCs, proposals, architecture docs) that match a specific style: clear over clever, professional but accessible, audience-focused.

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

### RFC Altitude

The RFC must hit the right level of detail:

- **High enough:** A PM can understand complexity and integration points with other teams
- **Detailed enough:** Engineers understand how the design maps to business requirements
- **Not so detailed:** It reads like a recipe

An RFC documents decisions. An implementation plan documents execution.

### RFC vs Specification

An RFC proposes and persuades. It is not an implementation manual or operational runbook.

**What belongs in an RFC:**
- Architecture decisions and rationale
- API contracts (request/response shapes)
- Component responsibilities
- Key flows (happy path, error recovery)
- Design principles with reasoning

**What belongs in implementation docs (not the RFC):**
- Full client/server code implementations
- Retry logic, backoff algorithms, error handling code
- Configuration files (Redis YAML, Kubernetes manifests)
- Monitoring dashboards and alert thresholds
- Operational runbooks

If you find yourself writing copy-pasteable code or config files, you've drifted into implementation territory.

**Warning signs the RFC is too thin:**
- Proposal section is shorter than Background + Problem Statement combined
- API contracts are described in prose without showing actual request/response shapes
- Abandoned Ideas lists alternatives without explaining why they were attractive or the specific reasons they were rejected
- Design principles are stated without explaining why they matter

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
subagent_type: personal-skills:rfc-writer
prompt: |
  Source material: [source path]
  Output to: [output path]

  Audience: [who will read this - teams, familiarity level]
  Detail level: [high-level / standard / detailed - derived from audience]

  Write an RFC draft from this source material.
```

The writer reads the style guide, writes the draft, runs its internal review loop, and returns.

### Phase 3-5: Review Loop

The `rfc-writer` agent handles the review loop internally. It spawns `rfc-reviewer` to check its work and fixes MAJOR/MODERATE issues before returning.

If you need to run a review pass manually (e.g., on an existing draft):
```
subagent_type: personal-skills:rfc-reviewer
prompt: |
  Review the RFC at [draft path]
```

**Do NOT use `superpowers:code-reviewer`** - that's for code, not RFC style review.

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

## Patterns to Avoid

These patterns signal AI-generated text to readers and undermine credibility:

| Don't write | Write instead | Why it matters |
|-------------|---------------|----------------|
| It's worth noting that the API has rate limits. | The API has rate limits of 100 requests/minute. | Filler phrases add no information and delay the point. |
| Let's dive into the authentication flow. | The authentication flow works as follows. | Casual phrasing feels out of place in technical documents. |
| This is important because it helps facilitate... | This matters because... | "Facilitate" and similar words are corporate jargon. |
| On one hand X, but on the other hand Y. | X. However, Y. | Overly balanced hedging avoids taking a position. |
| It could potentially cause issues... | This risks causing... | Excessive hedging weakens the message. |
| In this section, we will explore... | *(Just start the section. No preamble.)* | Preambles waste the reader's time. |
| Em dashes for asides | Commas, periods, or parentheses | Em dashes create a rushed, fragmented tone and signal AI text. |
| "The X Trap" or "The X Problem" | Descriptive headers | Dramatic headers feel clickbaity in technical writing. |
| Dramatic rhetorical questions | Direct statements | Questions can feel condescending or manipulative. |
| Lists starting with same word pattern | Varied list openers | Repetitive structure feels mechanical. |
| "You need to..." or "You should..." | "We need to..." | RFCs are the team thinking together, not instruction manuals. |

## Writing Style Guidelines

### Vocabulary

- Use straightforward words. Avoid fancy or sophisticated vocabulary.
- Prefer "use" over "utilize", "help" over "facilitate", "show" over "demonstrate"
- Technical terms are fine when domain-appropriate, but explain them in the Glossary
- No buzzwords or corporate jargon

### Tone

- Direct and professional, not casual
- Confident but not arrogant. Present reasoning, not just conclusions.
- Explain the "why" behind decisions
- Acknowledge tradeoffs and limitations honestly
- Write as the team speaking to the team. Use "we" not "you".

### Sentence Structure

- Keep sentences manageable. Break long compound sentences.
- Lead with the main point, then provide supporting detail
- Use active voice when possible
- Vary sentence length. Short declarative sentences are useful for emphasis, but stacking them creates a choppy, authoritative tone that feels like commands rather than explanation.
- Connect related ideas with reasoning words (because, so, which means, since). The reader should understand *why*, not just *what*.

**Choppy (avoid):**
> Redis pub/sub is fire-and-forget. If no subscriber is listening, the message is lost. The buffer is authoritative. Pub/sub is an optimization.

**Connected (better):**
> Redis pub/sub is fire-and-forget, meaning if no subscriber is listening when a message is published, it's gone. Because of this, we treat the buffer as the authoritative source. Pub/sub pushes chunks for low-latency delivery, but the buffer is what makes the system reliable.

### Paragraphs

- One idea per paragraph
- First sentence should convey the paragraph's purpose
- 3-5 sentences typical, never walls of text

### Technical Explanations

- Break complex topics into digestible pieces
- Use concrete examples to illustrate abstract concepts
- Include code snippets inline when they clarify
- Reference diagrams and link to external resources

### Callouts and Formatting

- Use callout boxes for important caveats or scope limitations
- Use bullet lists for enumerating options, features, or steps
- Use tables for comparisons or structured data
- Bold key terms on first introduction
- No manual table of contents. Tooling (Confluence, Notion, etc.) generates navigation from headers automatically.
- No numbered headers (like "1. Overview", "2. Background"). Let the hierarchy speak for itself.

### Content Structure Rules

**Code blocks** are only for:
- Actual code (implementations, pseudocode)
- Configuration files (YAML, JSON)
- Commands (CLI, API calls)
- Data contracts (request/response shapes)

Never use code blocks for prose, section organization, or emphasis. If you're putting plain English sentences in a code block, something is wrong.

**Tables vs prose:**
- **Use tables** for dense reference data: API fields, status codes, configuration options, feature comparisons
- **Use prose with subheaders** for items that need explanation: risks, tradeoffs, design decisions, alternatives

Tables compress information. When readers need to understand *why*, not just *what*, prose is better.

**Component references:**
Define components before referencing them. If you mention "WebSocket pods" or "the buffer service," explain what they are first. The reader shouldn't encounter a term and wonder "what is that?" until later in the document.

## RFC Structure

For the full RFC template with section details, see `references/rfc-template.md`.

For an example RFC demonstrating the style and flow, see `references/rfc-example.md`. Use it to validate tone, section weight, and prose connectivity.

**Length guidance:**
- Target 500-1000 lines for most RFCs
- If the RFC grows significantly beyond this, pause and ask the user whether to trim content or continue. Review what might be extraneous (full implementations, operational detail, excessive edge cases) before proceeding.
- Diagrams and API contracts are dense information and don't count against length
- Full code implementations DO count against length (and probably shouldn't be there)

**Required Sections:**

| Section | Length | Purpose |
|---------|--------|---------|
| Abstract | 2-3 sentences | 10-second understanding for reader with no context |
| Background | As needed | All context needed to follow the rest. Done when a newcomer stops asking clarifying questions. |
| Proposal | 40-60% of total RFC length | The actual solution. Present ONE approach as the plan, not options. The tone should be "I thought this through, here's the design" not "maybe this could work." If you evaluated multiple approaches, pick the best one and move alternatives to Abandoned Ideas. Reviewers may change your mind, but don't leave things open for the sake of seeming collaborative. Break into sub-sections: architecture overview, component responsibilities, key flows (happy path, error cases, recovery), data model at schema level, integration points. If shorter than Background + Problem Statement, it's probably too thin. |
| Abandoned Ideas | 150-300 words per alternative | Write each alternative as a short narrative. Start with why someone might suggest it, then explain what we learned when we evaluated it. The reader should feel like they're hearing your thought process, not reading a form. Avoid templated formats like "What it was: ... Why attractive: ... Why rejected: ..." |

**Optional Sections:**

| Section | Length | When to Include |
|---------|--------|-----------------|
| Glossary | 1 line per term | Domain-specific jargon or acronyms exist |
| Problem Statement | As needed | Background alone doesn't make the problem obvious |
| Bill of Work | 50-100 words per component | Multiple teams or components involved |
| Rollout | 200-400 words | Deployment strategy, flags, or phased approach needed. Never include time estimates (weeks, sprints, dates). An RFC proposes what to build, not when. Timeline depends on which team picks it up, their capacity, and competing priorities. Describe phases and dependencies only. |
| Risks | 50-100 words per risk | Non-trivial risks exist |
| Future Steps | 100-300 words | Change enables longer-term strategy |

## Handling Feedback

### Responding to Reviews

- Address every comment, even if the response is "no change needed because X"
- Group related feedback into coherent revisions rather than point-by-point patches
- If a reviewer misunderstood, the Background section probably needs expansion
- Track substantive changes between versions (not typo fixes)

### When to Push Back

Push back on feedback when:
- The suggestion contradicts a constraint documented in Background
- The alternative was already covered in Abandoned Ideas (point them there)
- The change would break consistency with existing systems

Push back respectfully: "We considered this in Abandoned Ideas (link). The main blocker was X. Is there something we missed?"

### When to Fold

Fold on feedback when:
- Multiple reviewers raise the same concern independently
- We can't articulate why our approach is better, just different
- The cost of the change is low and it makes the reviewer more comfortable

## Code Examples and Diagrams

### Code Examples

- Prefer pseudocode over full implementations. Code in an RFC should illustrate a concept, not be copy-pasteable.
- Keep examples to 10-30 lines max. If you need more, it belongs in implementation docs.
- Use the language of the service being documented
- Default to TypeScript only for hypothetical or illustrative examples
- Include comments to explain non-obvious fields
- For API contracts, show actual request/response shapes as JSON (or the relevant format). Prose explains context and behavior; the data contract itself should be visible, not buried in sentences.

**Prose only (avoid):**
> Called by the backend when ready to stream. Returns streamId and a short-lived token. If streaming is unavailable, returns streamingSupported false.

**With data contract (better):**
> Called by the backend when ready to stream.
>
> **Request:**
> ```json
> {
>   "messageId": "msg_abc123",
>   "metadata": { "model": "claude-3-sonnet" }
> }
> ```
>
> **Response (200):**
> ```json
> {
>   "streamId": "stream_xyz789",
>   "streamToken": "eyJhbG...",
>   "streamingSupported": true
> }
> ```
>
> **Response (streaming unavailable):**
> ```json
> {
>   "streamingSupported": false,
>   "fallbackReason": "channel_unsupported"
> }
> ```

- Show both request and response when documenting APIs
- Format endpoint specs clearly:
  ```
  POST /conversations/:id/stream/start
  Authorization: Bearer <token>
  ```

### Diagrams

Every diagram reference MUST include an implementation. Placeholder descriptions alone are not acceptable.

**Draft phase: Use ASCII.** Mermaid inside markdown tables doesn't render well, making review difficult. ASCII is easier to visualize and edit during drafting.

**Finalize phase: Convert to mermaid.** After draft approval, convert ASCII to mermaid for easier export to Lucidchart.

**Simple linear flow:**
```
Client → Redis Buffer → Delivery Pod → Widget
```

**Multi-step with actors:**
```
┌────────┐    ┌────────┐    ┌────────┐
│ Client │───▶│ Buffer │───▶│Delivery│
└────────┘    └────────┘    └────────┘
```

Format diagrams in a table with caption:
```
| |
|:---:|
| `Client → Redis Buffer → Delivery Pod → Widget` |
| *Caption: High-level data flow* |
```

### Callout Boxes

- Use for scope limitations or important caveats
- Format:
  ```
  | info | Important note or caveat here |
  | :---- |
  ```
- Keep callout content concise. One key point.

## Review Checklist

When reviewing technical documents, check for:

### Structure

- [ ] Follows the RFC template sections
- [ ] Background sufficient for someone with no context
- [ ] Abandoned Ideas documented with full reasoning (what, why attractive, why rejected, when to reconsider)
- [ ] Section lengths appropriate (Abstract ≤3 sentences, Proposal is 40-60% of RFC)
- [ ] Proposal is meatier than Background + Problem Statement combined
- [ ] No horizontal lines (`---`) between sections

### Clarity

- [ ] Complex topics broken into manageable pieces
- [ ] Each paragraph focuses on one idea
- [ ] Vocabulary straightforward (no fancy words)
- [ ] No AI-obvious patterns (em dashes, "Let's dive in", etc.)
- [ ] Sentences vary in length and connect with reasoning (not staccato declarations)
- [ ] Uses "we" not "you" throughout

### Completeness

- [ ] The "why" explained, not just the "what"
- [ ] Security implications addressed
- [ ] Backward compatibility considered
- [ ] Edge cases and error handling discussed

### Technical Accuracy

- [ ] Code examples match the service's language
- [ ] API contracts show actual JSON shapes, not just prose descriptions
- [ ] Diagrams use ASCII with table container and caption

### Audience Focus

- [ ] Reader unfamiliar with domain could follow this
- [ ] Domain terms defined in Glossary
- [ ] Callouts used for important caveats

## Warning Signs the RFC Became a Spec

If you notice these patterns, the RFC has drifted into implementation territory:

- Table of Contents needed (the doc is too long)
- Multiple appendices
- Config file examples (YAML, JSON configs, Kubernetes manifests)
- Full class implementations with methods
- Retry/backoff code
- Monitoring metric code
- Operational runbooks
- Heavy diagrams (lifelines, numbered arrows) for simple linear flows
- "Option A / Option B" in Proposal section (pick one, move alternatives to Abandoned Ideas)
- Week-by-week rollout schedules

## Draft Markers

RFCs go through drafting before circulation. Use markers to flag items needing human review before the document is ready.

### Marker Format

Use HTML comments so markers don't render in preview:

```
<!-- REVIEW: description of what needs human review -->
```

### When to Use Markers

**Make decisions, then mark for review.** Don't leave blanks. The goal is to help the user by making a first pass, even if the decision was a toss-up. The right answer might be the one you picked.

- `<!-- REVIEW: Chose 24h token expiration for security. Confirm this works for long sessions. -->`
- `<!-- REVIEW: Assumed Redis cluster of 3 nodes. Platform team should validate. -->`

**For truly ambiguous choices** that don't impact other parts of the RFC, produce both versions inline:

```markdown
<!-- REVIEW: Two valid approaches below. Pick one and delete the other. -->

**Option A: Eager loading**
[content for option A]

**Option B: Lazy loading**
[content for option B]

<!-- END OPTIONS -->
```

Use this sparingly. Most decisions should be made, not deferred.

### Draft Status Section

Every RFC draft starts with a status section at the top (after the title):

```markdown
## Draft Status

**State:** Draft

**Items for review:**
- [ ] <!-- REVIEW: item 1 description -->
- [ ] <!-- REVIEW: item 2 description -->

---
```

This section:
- Makes draft state explicit
- Lists all review markers in one place
- Gets removed when the RFC is finalized

### No Open Questions Section

Do NOT include an "Open Questions" section in the RFC. Questions should be:
1. Resolved before writing (do the research)
2. Answered with a decision, then marked for review
3. Listed in Draft Status for visibility

An "Open Questions" section signals the author didn't finish their job.
