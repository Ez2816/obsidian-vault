---
name: obsidian-weekly-connections
description: Review notes from the past week to identify conceptual connections, implications for preferences and projects, contradictions, missing concept pages, and lasting knowledge trapped in daily notes
trigger:
  - "Run my weekly connection review"
  - "Find this week's connections"
  - "Review what I learned this week"
scope: repository
---

# Obsidian Weekly Connection Review

## Trigger

User says things like:
- Run my weekly connection review
- Find this week's connections
- Review what I learned this week

## Workflow

### 1. Identify the Review Window

Determine the current date in `America/Vancouver` timezone.

Look back seven calendar days from today.

Identify all notes that were created or materially changed during this period by checking file modification timestamps.

Scope includes:
- Daily notes from the past 7 days (`50 Daily/YYYY-MM-DD.md`)
- New files created in `30 Wiki/`, `40 Workspace/`, `20 Personal/`
- Source files added to `10 Sources/` (but do not modify them)
- Inbox items added or processed during the week

### 2. Read and Scan Recent Notes

Read all recent notes to understand what was worked on, learned, or decided this week.

Pay special attention to:
- Completed projects or milestones
- New research findings or experimental results
- Course insights or learning breakthroughs
- Decisions made or preferences clarified
- Questions answered or resolved
- Changes in understanding

### 3. Identify Direct Conceptual Connections

Look for direct connections where one note explicitly builds on, extends, or applies knowledge from another.

Examples:
- Course concept applied in research project
- Technical skill used in career planning
- Previous question answered by new source material
- Course material relevant to another course

For each connection, record:
- Which notes are connected (with paths)
- What the connection is
- Why it matters
- Evidence from the notes themselves

### 4. Identify Relationships Using Different Terminology

Look for concepts expressed with different words in different contexts.

Examples:
- "Data standardization" in one course and "schema consistency" in another
- "Decision framework" in Personal notes and "experimental design" in research
- "Career constraint" in Personal and "time commitment" in Workspace

For each:
- Explain the conceptual overlap
- Show how the terminology differs
- Note whether the underlying concept should have a shared Wiki page

### 5. Identify Course-to-Course Connections

If the user is taking multiple courses, look for connections:
- Shared prerequisites or foundational concepts
- Complementary skills or knowledge areas
- Timing or sequencing of learning
- Opportunities to apply one course's content in another

For each connection, explain the relationship and suggest how to use it (e.g., project idea, study material synthesis).

### 6. Identify Course-to-Research Connections

Look for relationships between course material and active research projects.

Examples:
- Course method that could improve experiment design
- Course theory that explains research findings
- Course concept as a lens for research questions
- Research question that course material addresses

For each:
- Cite the course material and research project
- Explain the connection explicitly
- Suggest how to leverage it

### 7. Identify Project-to-Career Connections

Look for connections between active projects and career planning.

Examples:
- Technical skills developed in projects that align with career goals
- Project decisions that clarify career preferences
- Gaps between project work and career direction
- Interview or resume material emerging from projects

For each connection:
- Explain the relationship
- Suggest preservation in Career workspace if relevant

### 8. Identify Implications for Stated Preferences and Goals

Cross-reference recent work with Personal notes that record preferences, goals, and decision frameworks.

Look for:
- Work that aligns with stated preferences (reinforce it)
- Work that contradicts stated preferences (flag the conflict)
- Changes in what matters or in priorities
- Evidence that a preference has evolved or changed
- Opportunities to pursue a stated goal more directly

For each implication:
- Cite the Personal note and the recent work
- Explain why the connection matters
- Do NOT modify the Personal note; note it for potential AI Review Queue entry

### 9. Identify Changes in Thinking

Look for evidence that understanding has shifted over the week.

Examples:
- A question answered or resolved
- A previous assumption reconsidered
- A new perspective introduced
- A decision made that reverses a prior plan
- New evidence that changes interpretation

For each change:
- Describe what was believed before
- Describe the new understanding
- Cite the evidence (which note or source)
- Note the significance

### 10. Identify Contradictions

Look for direct conflicts between recent notes.

Examples:
- A decision that contradicts a stated preference
- A conclusion that conflicts with earlier research
- Two sources with incompatible claims
- A project goal that conflicts with another project

For each contradiction:
- Identify the conflicting items clearly
- Cite the notes and sources
- Explain the nature of the conflict (factual, interpretive, priority-based, etc.)
- Suggest resolution approaches (without modifying source material)

### 11. Identify Isolated Notes

Look for recent notes that have few or no connections to existing work.

Isolated notes are valuable—they may indicate:
- New research areas or interests
- Unique insights that deserve their own concept page
- Gaps in the knowledge system
- Incomplete ingestion of source material

For each isolated note:
- Explain why it appears isolated
- Suggest whether it needs: a new concept page, integration into existing concepts, or further development

### 12. Identify Missing Concept Pages

Look for concepts, methods, or frameworks that are mentioned repeatedly across multiple notes during the week, but do not have a dedicated concept page in `30 Wiki/Concepts/`.

Examples:
- A statistical method mentioned in course notes, research, and career discussions
- A decision-making framework used in multiple projects
- A technical concept that appears in multiple courses
- A theoretical framework that helps interpret research findings

For each missing concept:
- Explain why a dedicated page would be useful
- Show evidence (which notes mention it and how)
- Suggest initial structure or scope for the concept page

Do NOT create the page. Suggest it for the user's consideration.

### 13. Identify Lasting Knowledge Trapped in Daily Notes

Scan daily notes from the week for information that belongs in permanent locations:

- Research findings or experimental results
- Technical solutions or debugging notes
- Course insights or conceptual breakthroughs
- Meeting outcomes or decisions
- Important accomplishments
- Changes in career direction or personal priorities
- Resolved questions

For each piece of lasting knowledge:
- Identify where it currently is (which daily note)
- Suggest where it belongs permanently:
  - `30 Wiki/Concepts/` — Conceptual learning
  - `30 Wiki/Syntheses/` — Cross-topic connections
  - `40 Workspace/[Project]/` — Project documentation
  - `20 Personal/[Category]/` — Preference or decision updates
  - `90 System/index.md` — If it belongs in the index

Do NOT move information. Suggest locations for user review.

### 14. Label Connection Confidence

For each connection you identify, label its confidence level:

- **Direct** — Explicitly stated connection in the notes
- **Inferred** — Strong logical connection based on evidence
- **Tentative** — Possible connection that needs more exploration
- **Contradicted** — Evidence supports and contradicts the connection
- **Uncertain** — Unclear whether the connection holds

Use these labels to help the user prioritize which connections to explore further.

### 15. Update AI-Maintained Wiki

When a connection analysis clearly warrants a Wiki update:

Update existing Wiki pages to add the new connection:
- Add a new `AI Connections` section if one doesn't exist
- Include: the connected note path, the connection, the confidence label
- Link back to the original concept page
- Cite source material clearly

Only update when:
- The connection is direct or strongly inferred (not tentative or uncertain)
- The update adds meaningful relationship that doesn't already exist
- You have evidence from the notes themselves

Do NOT create new concept pages during connection review. Only update existing ones.

Do NOT rewrite sources or Personal notes.

### 16. Handle Proposed Personal Note Changes

If a connection suggests a change to a Personal note (e.g., an update to preferences, goals, or decisions):

- Do NOT modify the Personal note directly
- Add the proposed change to `90 System/AI Review Queue.md` with:
  - The target Personal note
  - What the connection suggests
  - Evidence from this week's work (with note paths)
  - Confidence level (certain / likely / tentative)

### 17. Update Navigation

Update `90 System/index.md` when the connection review materially changes the Wiki:

- Add links to newly updated concept pages
- Add new connections or syntheses pages
- Update section descriptions if knowledge structure has shifted
- Remove obsolete or superseded entries

Only update if substantial new connections or knowledge areas have emerged.

### 18. Log the Review

Append a concise entry to `90 System/log.md`:

```markdown
## YYYY-MM-DD — connection-review

### Request

Review this week's connections.

### Files examined

- Daily notes: [[YYYY-MM-DD]] through [[YYYY-MM-DD]]
- New/modified files: [list key files]
- Source files added: [list if any]

### Files created or modified

- [list modified Wiki pages]
- [updated index.md]: Y/N
- [updated log.md]: Y (this entry)

### Important findings

- X direct connections identified
- Y missing concept pages suggested
- Z pieces of lasting knowledge for permanent storage
- N contradictions identified
- M changes in thinking detected

### Unresolved questions

- [list questions that came up during review]
```

### 19. Produce Connection Report

After the review, provide a comprehensive summary:

**Format**:

```
## Weekly Connection Review — Week of YYYY-MM-DD

### Direct Connections (X)
- [Connection 1]: evidence → [[note1]] + [[note2]]
- [Connection 2]: evidence → [[note1]] + [[note3]]

### Course Connections (X)
- [Course A] ↔ [Course B]: shared concept
- [Course A] → [Project]: applicable method

### Changes in Thinking (X)
- [Previous understanding] → [New understanding] (evidence: [[note]])

### Contradictions (X)
- [[note1]] conflicts with [[note2]]: explain

### Missing Concept Pages (X)
- "[Concept]": mentioned in [[note1]], [[note2]], [[note3]]

### Lasting Knowledge for Permanent Storage (X)
- [[daily-note]]: [finding] → suggest: [location]

### Implications for Personal Preferences/Goals (X)
- [[Personal note]]: [connection] (do not modify; review required)

### Isolated Notes (X)
- [[note]]: consider [action]

### Wiki Updates Made (X)
- [[concept-page]]: added connection to [related concept]

### AI Review Queue Proposals (X)
- [[Personal note]]: proposed update with evidence

### Next Steps
- [User action items, if any]
```

## Constraints and Protections

### Do Not Modify Sources

Do not read, modify, move, or delete files in `10 Sources/`.

You may extract information from sources and cite them in connections, but never change the originals.

### Do Not Rewrite Personal Notes

Do not modify `20 Personal/` notes directly.

If a connection suggests a Personal note should be updated:
- Place the proposal in `90 System/AI Review Queue.md`
- Wait for explicit user authorization before editing

### Only Update Authorized Wiki Pages

Update only:
- Existing concept pages with new connections
- Existing synthesis pages with new relationships

Do NOT create new concept or synthesis pages during connection review.

Do NOT merge or consolidate multiple pages unless explicitly requested.

### Preserve Evidence

Every substantial claim must cite:
- The supporting note path
- The specific evidence (quote or summary)
- The confidence level (direct / inferred / tentative / uncertain)

Do not invent connections. Do not overstate confidence.

### Focus on Meaningful Connections

Do not create a connection merely because two notes share a keyword.

For every proposed connection, explain:
1. What the connection is
2. Which notes support it
3. Why it matters
4. Whether it is direct, inferred, tentative, contradicted, or uncertain

### Report All Changes

List every file created or modified, including:
- Which Wiki pages were updated
- What was added or changed
- Whether index.md was updated
- Entries added to log.md

## Definition of Done

Weekly connection review is complete only when:

- [ ] Notes from the past seven days have been identified and scanned
- [ ] Direct connections have been identified with evidence
- [ ] Course, research, and career connections have been explored
- [ ] Implications for Personal preferences and goals have been flagged
- [ ] Changes in thinking and contradictions have been noted
- [ ] Missing concept pages have been suggested
- [ ] Lasting knowledge has been identified with suggested permanent locations
- [ ] Isolated notes have been analyzed
- [ ] All connections are labeled with confidence levels
- [ ] Wiki updates have been made only where clearly authorized
- [ ] Proposed Personal changes are in AI Review Queue (not silently modified)
- [ ] index.md has been updated if materially necessary
- [ ] log.md has been updated with a connection-review entry
- [ ] User receives a comprehensive report with all files modified listed
