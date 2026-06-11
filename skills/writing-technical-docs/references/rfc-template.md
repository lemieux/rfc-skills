# RFC Template Reference

Detailed guidance for each RFC section. The style guide (`rfc-style-guide.md`) covers style; this covers structure.

## Abstract (required)

- Maximum 2-3 sentences
- Reader with no context should understand the doc in 10 seconds
- State what problem is being solved and the high-level approach

**Example:**
> This RFC proposes migrating user authentication from session cookies to JWT tokens. The change reduces server-side state, enables cross-domain auth for the new mobile app, and aligns with the platform team's token standardization effort.

## Background (required)

- Provide all context needed to understand the RFC
- Reader who knows nothing about the domain should follow the rest
- Include diagrams and links as needed
- If reviewers ask clarifying questions, expand this section

**Checklist:**
- [ ] Current state explained
- [ ] Why current state is insufficient
- [ ] Constraints (timeline, budget, compatibility)
- [ ] Stakeholders identified

## Glossary (optional)

- Define domain-specific jargon and acronyms
- Place at the beginning, not the end
- If Background is long, mention Glossary exists upfront and link to it

## Problem Statement (optional)

- Keep short if Background already makes the problem obvious
- State the problem directly in simple terms

## Proposal (required)

- Free-form, use sub-sections as needed
- Include code examples in the service's language
- Add diagrams with captions
- Break into logical components (e.g., by service or system)

**Structure options:**
- By component/service
- By phase (if time-sequenced)
- By concern (data model, API, migration, etc.)

## Bill of Work (optional)

- Break down implementation by component/team
- List what each system needs to implement
- Use when multiple teams are involved

## Rollout (optional)

- Deployment strategy, feature flags
- Rollback plans, monitoring, remediation
- Use for changes that can't be deployed atomically

## Risks (optional but recommended)

- Document risks and mitigation strategies
- Explain why certain risks are acceptable
- Write as prose with a subheader per risk (not a table), covering likelihood, impact, and mitigation

## Future Steps (optional)

- Document what this enables long-term
- Shows the change fits a larger strategy
- Helps reviewers understand you're not overbuilding

## Abandoned Ideas (required)

- Document alternatives you considered and discarded
- Explain reasoning. This pre-empts reviewer suggestions.
- Shows thoroughness of research

Write each alternative as a short narrative (150-300 words), not a templated format. Each narrative should still cover what the alternative was, why it seemed viable initially, and the specific blocker that killed it - but woven into prose, as if explaining your thought process to a colleague.
