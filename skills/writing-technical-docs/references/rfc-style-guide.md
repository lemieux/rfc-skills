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

If you find yourself writing copy-pasteable code or config files, you've drifted into implementation territory.

### Warning Signs the RFC Is Too Thin

- Proposal section is shorter than Background + Problem Statement combined
- API contracts are described in prose without showing actual request/response shapes
- Abandoned Ideas lists alternatives without explaining why they were attractive or the specific reasons they were rejected
- Design principles are stated without explaining why they matter

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

**Critical:** Vary sentence length. Connect related ideas with reasoning words (because, so, which means, since). The reader should understand *why*, not just *what*.

- Keep sentences manageable. Break long compound sentences.
- Lead with the main point, then provide supporting detail.
- Use active voice when possible.
- Short declarative sentences are useful for emphasis, but stacking them creates a choppy, authoritative tone that feels like commands rather than explanation.

**Choppy (avoid):**
> Redis pub/sub is fire-and-forget. If no subscriber is listening, the message is lost. The buffer is authoritative. Pub/sub is an optimization.

**Connected (better):**
> Redis pub/sub is fire-and-forget, meaning if no subscriber is listening when a message is published, it's gone. Because of this, we treat the buffer as the authoritative source. Pub/sub pushes chunks for low-latency delivery, but the buffer is what makes the system reliable.

### Paragraphs

- One idea per paragraph
- First sentence conveys the paragraph's purpose
- 3-5 sentences typical, never walls of text

### Technical Explanations

- Break complex topics into digestible pieces
- Use concrete examples to illustrate abstract concepts
- Include code snippets inline when they clarify
- Reference diagrams and link to external resources

### Formatting

- Bold key terms on first introduction
- No manual table of contents. Tooling (Confluence, Notion, etc.) generates navigation from headers automatically.
- No numbered headers (like "1. Overview", "2. Background"). Let the hierarchy speak for itself.

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

**Format:** Wrap in a table with caption. The caption is a row of the same table, not text below it:

```markdown
| |
|:---:|
| `Client → Redis Buffer → Delivery Pod → Widget` |
| *Caption: High-level data flow for stream delivery* |
```

Backtick-wrapped flow labels like the one above are the prescribed format for simple linear flows. They count as diagrams, not as code-block misuse.

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

## Code Examples

- Prefer pseudocode over full implementations. Code in an RFC should illustrate a concept, not be copy-pasteable.
- Keep examples to 10-30 lines max. If you need more, it belongs in implementation docs.
- Use the language of the service being documented
- Default to TypeScript only for hypothetical or illustrative examples
- Include comments to explain non-obvious fields

## API Contracts

Show actual request/response shapes, not just prose descriptions. Prose explains context and behavior; the data contract itself should be visible, not buried in sentences. Show both request and response when documenting APIs, and format endpoint specs clearly:

```
POST /conversations/:id/stream/start
Authorization: Bearer <token>
```

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

Use this sparingly. Most decisions should be made, not deferred.

### Draft Status Section

Every draft starts with:

```markdown
## Draft Status

**State:** Draft

**Items for review:**
- [ ] <!-- REVIEW: item 1 -->
- [ ] <!-- REVIEW: item 2 -->
```

No trailing `---` after the section. Horizontal lines are banned everywhere in the document, including here.

## RFC Sections

### Length Guidance

- Target 500-1000 lines for most RFCs
- Line targets assume prose wrapped at roughly 80-100 characters. If authoring one paragraph per line, judge by content volume, not raw line count.
- Diagrams and API contracts are dense information and don't count against length
- Full code implementations DO count against length (and probably shouldn't be there)

### Required Sections

| Section | Length | Purpose |
|---------|--------|---------|
| Abstract | 2-3 sentences | 10-second understanding for reader with no context |
| Background | As needed | All context needed to follow the rest. Done when a newcomer stops asking clarifying questions. |
| Proposal | 40-60% of total RFC length | The actual solution. Present ONE approach as the plan, not options. The tone should be "I thought this through, here's the design" not "maybe this could work." If you evaluated multiple approaches, pick the best one and move alternatives to Abandoned Ideas. Reviewers may change your mind, but don't leave things open for the sake of seeming collaborative. Break into sub-sections: architecture overview, component responsibilities, key flows (happy path, error cases, recovery), data model at schema level, integration points. If shorter than Background + Problem Statement, it's probably too thin. |
| Abandoned Ideas | 150-300 words per alternative | Write each alternative as a short narrative. Start with why someone might suggest it, then explain what we learned when we evaluated it. The reader should feel like they're hearing your thought process, not reading a form. Avoid templated formats like "What it was: ... Why attractive: ... Why rejected: ..." |

### Optional Sections

| Section | Length | When to Include |
|---------|--------|-----------------|
| Glossary | 1 line per term | Domain-specific jargon or acronyms exist |
| Problem Statement | As needed | Background alone doesn't make the problem obvious |
| Bill of Work | 50-100 words per component | Multiple teams or components involved |
| Rollout | 200-400 words | Deployment strategy, flags, or phased approach needed. Never include time estimates (weeks, sprints, dates). An RFC proposes what to build, not when. Timeline depends on which team picks it up, their capacity, and competing priorities. Describe phases and dependencies only. |
| Risks | 50-100 words per risk | Non-trivial risks exist (use prose with subheaders, not tables) |
| Future Steps | 100-300 words | Change enables longer-term strategy |

### No Open Questions Section

Do NOT include an "Open Questions" section in the RFC. Questions should be:
1. Resolved before writing (do the research)
2. Answered with a decision, then marked for review
3. Listed in Draft Status for visibility

An "Open Questions" section signals the author didn't finish their job.

## Review Checklist

When reviewing, check for:

### Structure
- [ ] Follows RFC template sections
- [ ] Background sufficient for someone with no context
- [ ] Section lengths appropriate (Abstract ≤3 sentences, Proposal is 40-60% of RFC)
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
- [ ] Backward compatibility considered
- [ ] Edge cases and error handling discussed
- [ ] REVIEW markers appear BOTH inline AND in Draft Status section

### Technical Accuracy
- [ ] API contracts show actual JSON shapes
- [ ] Code examples match the service's language
- [ ] Diagrams have ASCII implementation, not just descriptions
- [ ] Code blocks used only for actual code
- [ ] No content from example RFC leaked into this document

### Audience Focus
- [ ] Reader unfamiliar with domain could follow this
- [ ] Domain terms defined in Glossary
- [ ] Callouts used for important caveats

## Warning Signs: RFC Became a Spec

- Table of Contents needed (too long)
- Multiple appendices
- Config file examples (YAML, JSON configs, Kubernetes manifests)
- Full class implementations with methods
- Retry/backoff code
- Monitoring metric code
- Operational runbooks
- Heavy diagrams (lifelines, numbered arrows) for simple linear flows
- "Option A / Option B" in Proposal section (pick one, move alternatives to Abandoned Ideas)
- Week-by-week rollout schedules
