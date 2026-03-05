---
name: finalizing-technical-docs
description: Finalize RFC drafts for circulation. Use when a draft RFC is approved and needs REVIEW markers resolved, ASCII diagrams converted to mermaid, and Draft Status section removed before sharing.
---

# Finalize Technical Documents

## Overview

This skill transitions an RFC from draft to ready-for-circulation state. It handles the mechanical cleanup that happens after the content is approved but before the document is shared.

## When to Use

- RFC draft has been reviewed and content is approved
- Document contains `<!-- REVIEW: ... -->` markers that need resolution
- ASCII diagrams need conversion to mermaid for Lucidchart export
- Draft Status section needs removal

## Workflow

### Phase 1: Scan for REVIEW Markers

Read the document and extract all `<!-- REVIEW: ... -->` markers. Present them as a numbered list:

```
Found 3 items requiring review:

1. [Line 45] Token expiration: "Chose 24h token expiration. Confirm this works for long sessions."

2. [Line 112] Redis cluster: "Assumed Redis cluster of 3 nodes. Platform team should validate."

3. [Line 203] Threshold: "500KB threshold is a guess. Need performance testing to validate."
```

### Phase 2: Resolve Each Marker

For each marker, ask the user how to resolve it:

**Options:**
- **Confirm** - The decision stands as written. Remove the marker.
- **Update** - Change the decision. User provides new text.
- **Defer** - Keep the marker for post-circulation discussion. Convert to visible callout.

For deferred items, convert from hidden comment to visible callout:
```markdown
| **Open Question** | Token expiration is set to 24h. Feedback requested on whether this works for long sessions. |
| :---- |
```

### Phase 3: Convert ASCII Diagrams to Mermaid

Find all ASCII diagrams and convert them to mermaid equivalents. **Keep the table container and caption intact.** Only replace the ASCII content with mermaid.

**Before:**
```markdown
| |
|:---:|
| `Client → Redis Buffer → Delivery Pod → Widget` |
| *Caption: High-level data flow* |
```

**After:**
```markdown
```mermaid
flowchart LR
    A[Client] --> B[Redis Buffer]
    B --> C[Delivery Pod]
    C --> D[Widget]
```
*Caption: High-level data flow*
```

Note: Fenced code blocks don't render inside markdown table cells. When converting, place the mermaid diagram and caption directly in the document (not inside a table). The table container from the ASCII version is dropped—mermaid renders its own visual boundary.

### Phase 4: Remove Draft Status Section

Delete the entire Draft Status section from the top of the document:

```markdown
## Draft Status

**State:** Draft

**Items for review:**
- [ ] <!-- REVIEW: ... -->

---
```

This section was for internal tracking and shouldn't appear in the circulated version.

### Phase 5: Final Validation

Scan the document to confirm:
- [ ] No `<!-- REVIEW:` markers remain (unless intentionally deferred as callouts)
- [ ] No Draft Status section
- [ ] All diagrams are mermaid (not ASCII)
- [ ] No `---` horizontal lines between sections

Report any issues found.

### Phase 6: Output

Save the finalized document. Suggest a filename without "Draft" or version numbers:
- Input: `RFC - Streaming Architecture v3.md`
- Output: `RFC - Streaming Architecture.md`

Provide notes for Google Docs export:
- Mermaid diagrams will need manual recreation in Lucidchart
- Tables render well, no changes needed
- Code blocks may need font adjustment

## Handling Edge Cases

**No REVIEW markers found**: Skip Phase 2, proceed to diagram conversion.

**No ASCII diagrams found**: Skip Phase 3, proceed to Draft Status removal.

**User wants to keep Draft Status**: Unusual, but respect the request. Note that the document will appear unfinished to readers.

**Complex ASCII diagrams**: If an ASCII diagram is too complex to convert accurately to mermaid, flag it for manual recreation in Lucidchart rather than producing a broken mermaid version.
