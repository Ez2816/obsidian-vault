---
name: obsidian-workspace-to-wiki
description: Weekly scan of Workspace for lasting knowledge and connections; consolidate insights, solutions, and patterns into Wiki organized by logical topic
trigger:
  - "Review my workspace weekly"
  - "Extract workspace knowledge"
  - "Run my workspace-to-wiki review"
  - "Consolidate workspace insights"
scope: repository
---

# Workspace-to-Wiki Consolidation Skill

## Purpose

Extract lasting knowledge from active workspace projects and consolidate it into the Wiki, organized by logical topic rather than workspace folder structure. Focus on solutions, insights, code patterns, and meaningful connections that enhance understanding.

---

## Workflow

### Step 1: Scan Workspace for Lasting Knowledge

Read all files in `40 Workspace/` recursively.

For each file, identify:
- **Solutions** — problems solved, working implementations, proven approaches
- **Insights** — conceptual understanding, patterns discovered, principles learned
- **Code patterns** — reusable technical solutions, algorithms, implementations
- **Connections** — how workspace findings relate to existing Wiki concepts, Personal goals, or other projects
- **Open questions** — unresolved problems worth documenting

Do not move or delete workspace content. Extract understanding only.

---

### Step 2: Identify Logical Topic Groupings

Group extracted knowledge into topics that make conceptual sense, NOT workspace folder structure.

**Examples of logical grouping** (not exhaustive):
- If workspace contains pgmpy code + Bayesian theory + sampling methods → topic: "Bayesian Networks & Inference"
- If workspace contains multiple project experiments → topic: "Experimental Methods & Results"
- If workspace contains solutions to recurring technical problems → topic: "Technical Solutions"
- If workspace contains decision-making processes → topic: "Decision Frameworks"

Ask: "What idea connects this to that?" not "What folder is this in?"

Topic names should be:
- Clear and descriptive
- Conceptually coherent
- Useful for future reference

---

### Step 3: Create or Update Wiki Pages

For each logical topic:

**Check if Wiki page exists**:
1. Search `30 Wiki/` for existing pages on this topic
2. If found, read the current page
3. Decide: update existing page or create new one?

**Prefer updating** existing pages when:
- The page covers the same conceptual ground
- Adding workspace insights extends existing understanding
- The connection is clear and natural

**Create new page** when:
- No existing page covers this topic
- The topic represents new conceptual territory
- The page will have clear, sustained purpose

**Page structure**:

```markdown
# Topic Name

[Brief description of what this topic covers]

## Workspace Findings

[Extracted knowledge from workspace]

### Solutions
[Working implementations, proven approaches]

### Insights
[Conceptual understanding, patterns, principles]

### Code Patterns / Technical Approaches
[Reusable technical solutions if applicable]

## Connections

[How does this relate to other Wiki concepts, Personal goals, or research?]

- **Related concepts**: [[Concept A]], [[Concept B]]
- **Related projects**: [[Project X]]
- **Related Personal goals**: (reference without modifying)

## Open Questions

[Unresolved problems, uncertainties, future directions]

---

*AI-generated consolidation from Workspace; draws on workspace notes and active projects. See [[40 Workspace/...]] for detailed working notes.*
```

**Tag clarity**:
- Mark each substantial claim with confidence:
  - `[direct]` — user directly stated or implemented this
  - `[inferred]` — AI interpretation of workspace content
  - `[tentative]` — uncertain connection
  - `[open]` — unresolved question
  - `[contradicted]` — conflicting evidence exists

---

### Step 4: Add Cross-References

For each new or updated Wiki page:

1. Identify connections to existing Wiki pages
2. Add links: `[[Other Concept]]`, `[[Related Project]]`
3. Identify connections to Personal notes (reference without editing)
4. Update related concept pages with backlinks if appropriate

Do not create pages merely to increase link density.

---

### Step 5: Handle Ambiguous or Substantial Proposals

For proposals that are unclear, substantial, or may be sensitive:

1. Create entry in `90 System/AI Review Queue.md`
2. Include:
   - Proposed Wiki page or update
   - Source workspace files
   - Reason for the proposal
   - Confidence level
   - Alternative organization if relevant

Example:
```
### Proposed: "Bayesian Inference Methods" (new Wiki page)

**Source**: `40 Workspace/Research/PGMpy & Graph Networks/` multiple files

**Reason**: Workspace contains multiple documents on sampling, inference, implementation. Represents conceptual territory not yet covered in Wiki.

**Confidence**: high — clear pattern across multiple workspace notes

**Alternative**: Could merge into existing "Bayesian Networks" page if that exists; recommend separate page for depth.

**Status**: pending approval
```

---

### Step 6: Update Navigation

Check if `90 System/index.md` should be updated:

1. If new topic represents major knowledge area, add link
2. If topic strengthens understanding of existing index items, update those entries
3. Keep index concise and navigable—update only if materially useful

---

### Step 7: Log the Operation

Append entry to `90 System/log.md` using this format:

```markdown
## YYYY-MM-DD — workspace-to-wiki

### Request
Weekly workspace-to-wiki review

### Files examined
[List workspace files scanned]

### Files created or modified
[List Wiki pages created/modified]

### Important findings
[Key topics extracted, major connections identified]

### Unresolved questions
[Uncertain topics, ambiguous connections, proposals in AI Review Queue]
```

---

### Step 8: Report Results

Report:
- **Topics identified**: [count and names]
- **Wiki pages created**: [names]
- **Wiki pages updated**: [names]
- **Major connections found**: [brief list]
- **Proposals in AI Review Queue**: [count]
- **Unresolved questions**: [brief list]

---

## Key Principles

1. **No workspace content is moved or deleted** — only understanding is extracted
2. **Topic organization prioritizes conceptual logic** over folder structure
3. **All AI-generated Wiki content is clearly marked** as interpretation, not authority
4. **Each page has clear purpose** — created only when it adds sustained value
5. **Ambiguous proposals go to AI Review Queue** — user reviews before finalizing
6. **Personal notes are referenced, never edited** — connections noted but judgments protected
7. **Confidence levels are explicit** — direct, inferred, tentative, open, contradicted
8. **Connections explain the "why"** — not just keyword overlap

---

## Protections

- Never modify `10 Sources/`
- Never silently rewrite `20 Personal/`
- Never move or delete `40 Workspace/` content
- Place substantial or ambiguous proposals in AI Review Queue
- Preserve all workspace historical records

---

## When to Invoke

User requests like:
- "Review my workspace weekly"
- "Extract workspace knowledge"
- "Run my workspace-to-wiki review"
- "Consolidate workspace insights"

Do not invoke automatically. User must explicitly request via trigger phrase or schedule.
