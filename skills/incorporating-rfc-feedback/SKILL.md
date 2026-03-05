---
name: incorporating-rfc-feedback
description: Process external reviewer feedback on circulated RFCs. Use when you have reviewer comments to address, need to evaluate feedback against RFC constraints, or want to systematically update an RFC based on review.
---

# Incorporating RFC Feedback

## Overview

RFC feedback requires evaluation, not blind acceptance.

**Core principle:** Evaluate before accepting. Verify against RFC constraints. Technical reasoning over social comfort.

RFC feedback is suggestions to evaluate, not orders to follow. The RFC already documents constraints (Background), rejected alternatives (Abandoned Ideas), and design rationale. Feedback that ignores these needs pushback, not acceptance.

## The Response Pattern

```
WHEN receiving RFC feedback:

1. READ: Complete feedback without reacting
2. PARSE: Break into discrete items
3. VERIFY: Check each against RFC and source material
4. EVALUATE: Does it improve the RFC or ignore constraints?
5. RESPOND: Accept, push back, or request clarification
6. IMPLEMENT: Update RFC with accepted changes
```

## Gather Inputs

**At the start, request:**
- RFC file path
- Source materials (PRD, POC, spec, implementation plan)
- Feedback to process (paste, link, or file)

The RFC is an abstraction layer. If feedback questions something fundamental, verify against the source material, not just the RFC. The RFC might have abstracted incorrectly.

**Example:** Reviewer says "Why not use WebSockets?" The RFC's Abandoned Ideas says "polling was simpler." But the implementation plan might reveal WebSockets were rejected due to a platform constraint not mentioned in the RFC. The source material has the full context.

## Forbidden Responses

**NEVER:**
- "Great feedback!" / "Thanks for the review!" (performative)
- Immediately implementing without checking RFC constraints
- Defensive responses to valid criticism
- Accepting suggestions that contradict Abandoned Ideas

**INSTEAD:**
- Restate the technical concern
- Reference specific RFC sections
- Push back with documented reasoning
- Just make the change (actions > words)

## Verification Checklist

For each feedback item:
```
1. Does this contradict Background constraints?
2. Was this alternative already in Abandoned Ideas?
3. Does it conflict with stated design principles?
4. Does it expand scope beyond RFC's purpose?
5. Does the reviewer have full context?
6. Does source material reveal context missing from RFC?
```

If YES to 1-5 → discuss with user before pushing back (see below).
If YES to 6 → update RFC to include missing context, then respond.

## When to Push Back

Push back candidates:
- Feedback contradicts constraint documented in Background
- Alternative already covered in Abandoned Ideas (point them there)
- Change would break consistency with existing systems
- Suggestion adds scope beyond RFC's purpose
- Reviewer misunderstands a constraint (expand Background if so)

**IMPORTANT: Discuss with user before pushing back.** Present the analysis:
```
"This feedback suggests [X]. However, this appears to conflict with
[Background constraint / Abandoned Ideas item / etc.].

My recommendation is to push back because [reason].
Should I push back, or do you want to reconsider the constraint?"
```

The user decides whether to push back or accept. Don't unilaterally reject feedback.

**If user confirms pushback:**
```
"We considered this approach - see Abandoned Ideas.
The blocker was [X]. Is there something we missed?"
```

**If reviewer was right about something you initially pushed back on:**
```
"You're right - I missed [X]. Updated the RFC."
```
No long apology. State correction, move on.

## When to Accept

Accept when:
- Valid technical concern not addressed in RFC
- Clarification that helps readers
- Edge case we missed
- Better phrasing/structure

**How to accept:**
```
"Good catch. Updated [section] to address this."
```
Or just make the change without commentary.

## When to Clarify

If feedback is unclear:
```
STOP - do not implement anything yet
ASK for clarification before proceeding
```

Partial understanding = wrong implementation.

## When Feedback Contains Questions

If a reviewer asks a question (e.g., "How would this work with WebSockets?"), **don't just update the RFC - start a discussion with the author.**

Questions deserve thoughtful analysis, not mechanical responses.

**Start with your analysis:**
```
"The reviewer asks: [question]

My read on this: [Claude's opinion/analysis of the question]

Looking at the source material and RFC, I think [suggested answer or direction].
However, [considerations, tradeoffs, or things that might change the answer].

Options:
1. Update RFC to address this by [specific suggestion]
2. Follow up with reviewer to clarify [what's unclear]
3. This touches on [related concern] - worth discussing further

What's your take?"
```

**The discussion might go several rounds:**
- Author disagrees with Claude's read → debate it
- Author has context Claude doesn't → incorporate it
- Neither is sure → brainstorm together
- Realize it needs reviewer clarification first

**Possible outcomes:**
- An RFC update (after agreeing on what to write)
- A response to the reviewer asking for clarification
- A back-and-forth with the reviewer before resolving
- Adding to Abandoned Ideas if exploring an alternative
- Deferring until more information is available

**Key principle:** Claude brings analysis and opinions to the table, then collaborates with the author. The author makes final decisions, but Claude is an active thinking partner.

## Source-Specific Handling

### From Stakeholders (PM, Leadership)
- May have business context you lack
- Ask if unclear on priority/scope
- Push back on technical impossibilities
- Their concerns often indicate missing Background context

### From Peer Engineers
- Verify against codebase and RFC constraints
- May not have full context - check Background
- Technical discussion is expected
- Good source of edge cases and implementation concerns

### From Security/Platform Teams
- Usually have specialized knowledge
- Verify their concerns are in scope
- If out of scope, explicitly note for future
- May reveal constraints that should be in Background

## Implementation Order

```
FOR multi-item feedback:
  1. Clarify anything unclear FIRST
  2. Then address in this order:
     - Blocking issues (factual errors, security)
     - Clarifications (expand unclear sections)
     - Suggestions (alternative approaches)
     - Polish (wording, formatting)
  3. Track what was changed vs pushed back
```

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Performative agreement | State what changed, or just change it |
| Blind acceptance | Verify against RFC constraints first |
| Defensive pushback | Use documented reasoning, not emotion |
| Ignoring feedback | Every item needs response (even if "no change") |
| Partial implementation | Clarify all items first |
| Not tracking changes | Summarize what changed and why |
| Only checking RFC | Verify against source material too |

## Output Summary

After processing all feedback, provide:

```
## Feedback Summary

**Accepted (X items):**
- [Item]: Updated [section] to [change]
- [Item]: Added [content] to [section]

**Pushed Back (Y items):**
- [Item]: See Abandoned Ideas - [reason]
- [Item]: Contradicts Background constraint [X]

**Clarification Needed (Z items):**
- [Item]: Need to understand [what's unclear]

**Deferred (N items):**
- [Item]: Valid but out of scope. Noted for future.
```

## Updating the RFC

**Delegate updates to the `rfc-writer` agent.** Don't update the RFC directly.

When feedback is accepted, spawn the writer:
```
subagent_type: personal-skills:rfc-writer
prompt: |
  RFC path: [RFC path]

  Change request: [what section, what to add/modify]
  Context: [the feedback item being addressed]

  Update the RFC to address this feedback.
```

**Do NOT use `superpowers:code-reviewer`** - that's for code, not RFC style review.

The `rfc-writer` agent:
- Follows the style guide for tone and structure
- Runs its internal review loop to catch errors
- Returns only when the update passes quality checks

This ensures updates maintain the same quality as the original draft.
