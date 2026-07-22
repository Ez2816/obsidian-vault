---
name: obsidian-vault-health
description: Audit the entire vault for broken links, orphans, duplicates, stale index entries, misplaced files, old inbox items, unresolved review items, conflicting personal notes, missing concept pages, trapped project knowledge, repeated carries, and inconsistent daily structure
trigger:
  - "Run my vault health check"
  - "Audit my Obsidian vault"
  - "Lint my vault"
  - "Check my knowledge system"
scope: repository
---

# Obsidian Vault Health Check

## Trigger

User says things like:
- Run my vault health check
- Audit my Obsidian vault
- Lint my vault
- Check my knowledge system

## Workflow

### 1. Broken Internal Links

Scan all `.md` files for internal links in Obsidian format: `[[link]]` or `[[link|display text]]`.

For each link:
- Check whether the target file exists in the vault
- Report link type: file link, anchor link, image link
- Note which file contains the broken link

**Broken link categories**:
- Points to non-existent file (e.g., `[[Concept that doesn't exist]]`)
- Points to non-existent anchor in existing file (e.g., `[[File#section]]` where the anchor doesn't exist)
- Malformed syntax (e.g., `[[ ]]`, `[[|]]`, missing closing brackets)

**Report format**:
```
Broken Links: X
- [[NonExistent]] in [[50 Daily/2026-07-15.md]]
- [[Concept#Missing Section]] in [[30 Wiki/Concepts/Example.md]]
```

Do NOT fix links. Only report them.

### 2. Orphan Notes

Identify notes with no incoming or outgoing internal links.

Scan all `.md` files (excluding daily notes) for:
- Files that are not linked from other notes
- Files that do not link to other notes

Orphan notes are not necessarily problematic—they may be:
- Intentionally standalone
- Not yet integrated into the system
- Ready for connection or deletion

**Report format**:
```
Orphan Notes: X
- [[30 Wiki/Concepts/Isolated Concept.md]]: no incoming/outgoing links
- [[40 Workspace/Project/Notes.md]]: only linked from daily notes
```

Do NOT delete orphans. Only report them.

### 3. Duplicate or Overlapping Wiki Pages

Scan `30 Wiki/` for concept pages that cover similar topics.

Look for:
- Pages with nearly identical titles (e.g., "Concept" and "Concepts")
- Pages covering the same domain (e.g., two pages about statistical methods)
- Pages with redundant content or overlapping scope
- Multiple pages that could be merged into a single, clearer page

For each potential duplicate:
- List both pages
- Explain the overlap
- Suggest merge or clarification
- Note which page should be primary if merged

**Report format**:
```
Duplicate Concept Pages: X
- [[30 Wiki/Concepts/Statistical Analysis.md]] and [[30 Wiki/Concepts/Statistics.md]]
  → Overlap: both cover basic statistical methods; suggest merge with [[Statistics.md]] as primary
```

Do NOT merge pages. Only suggest consolidation.

### 4. Unsupported AI-Generated Claims

Scan all Wiki pages for claims that lack supporting evidence or source citations.

Specifically look for:
- Factual claims without a source link (file path or `[[]]`)
- Definitions without attribution
- Methods or frameworks without a course or research project reference
- Conclusions marked as "AI interpretation" without evidence cited

Wiki pages should distinguish:
- Source-supported facts (with source path)
- User judgments (with link to Personal note)
- AI interpretations (marked as such, with supporting notes cited)
- Tentative connections (flagged as uncertain)
- Open questions (explicitly marked)

**Report format**:
```
Unsupported Claims: X
- [[30 Wiki/Concepts/Example.md]] line 15: "This method is most effective..." lacks source
- [[30 Wiki/Synthesis/Topic.md]]: defines term without source attribution
```

### 5. Missing Source References

For each fact in Wiki pages that references a source, verify the source exists and is correctly cited.

Check:
- Source file paths are valid and readable
- Citations use correct format: `[[10 Sources/...]]`
- Source material is in the correct directory

**Report format**:
```
Missing Source References: X
- [[30 Wiki/Concepts/Example.md]]: cites [[10 Sources/Papers/NonExistent.pdf]] which doesn't exist
```

### 6. Stale or Inaccurate Index Entries

Read `90 System/index.md`.

For each section, verify:
- Links point to valid files
- Descriptions match current content
- Pages listed still exist and are relevant
- Links are in appropriate sections
- No dead or obsolete references

Look for:
- Entries for pages that no longer exist
- Outdated descriptions of page content
- Missing entries for major knowledge areas
- Sections that should be reorganized based on current vault structure

**Report format**:
```
Stale Index Entries: X
- Links to [[30 Wiki/Projects/Old Project.md]] which was merged into another page
- "Major Knowledge Areas" section lacks entry for [[30 Wiki/Concepts/Important Concept.md]]
```

### 7. Files Placed in Inappropriate Directories

Scan the vault structure to check whether files are in logical locations.

Verify:
- Source material is in `10 Sources/` with appropriate subcategory
- Personal notes are in `20 Personal/` subcategories (Preferences, Decisions, Reflections, Goals)
- Wiki pages are in `30 Wiki/` subcategories (Concepts, Projects, People and Organizations, Comparisons, Syntheses)
- Workspace notes are in `40 Workspace/` project directories (School, Research, Career, Finance, Personal Systems)
- Daily notes are in `50 Daily/` with YYYY-MM-DD naming
- System files are in `90 System/`
- Captures belong in `00 Inbox/`

**Report format**:
```
Misplaced Files: X
- [[00 Inbox/2026-03-15 - Research Notes.md]]: should be in 30 Wiki/ or 40 Workspace/
- [[40 Workspace/Old Meeting Notes.pdf]]: should be in 10 Sources/Meetings/
```

Do NOT move files. Only identify misplacements.

### 8. Old Inbox Items

Scan `00 Inbox/` for items that have not been processed or classified.

For each item:
- Estimate when it was added (by file timestamp if possible)
- Assess whether it has been classified or is still waiting
- Determine next action (move to appropriate directory, delete, or clarify)

Items in Inbox more than 30 days old without progress should be flagged for decision.

**Report format**:
```
Old Inbox Items: X
- [[00 Inbox/Q2 Research Notes.md]]: added 2026-04-10, unclear status
- [[00 Inbox/Course Materials.pdf]]: added 2026-05-20, should be moved to 10 Sources/Lectures/
```

### 9. Unresolved AI Review Queue Items

Read `90 System/AI Review Queue.md`.

For each item, check status:
- Is it marked Pending, Approved, or Rejected?
- How old is it (when was it created)?
- Does it need action or follow-up?
- Is the evidence still valid?

Items pending for more than two weeks without action should be flagged.

**Report format**:
```
Unresolved Review Queue Items: X
- Target: [[20 Personal/Goals.md]] | Pending since 2026-07-01 | Status: awaiting user decision
- Target: [[30 Wiki/Concepts/Example.md]] | Approved on 2026-07-10 | Status: changes not yet applied
```

### 10. Personal Judgments That May Conflict with Newer Writing

Compare Personal notes with recent work to identify potential conflicts.

For each Personal note (in `20 Personal/`):
- Check recent Daily notes, Workspace updates, and new sources
- Identify whether recent work:
  - Supports the stated preference/goal/decision
  - Contradicts it
  - Suggests it may have changed
  - Provides evidence that updates it

**Report format**:
```
Potential Personal Note Conflicts: X
- [[20 Personal/Goals/Career Goals.md]] (2026-03-15): states preference for X, but recent work focuses on Y
  → Evidence: [[50 Daily/2026-07-10.md]], [[40 Workspace/Project/Notes.md]]
  → Action: review and possibly update
```

Do NOT modify Personal notes. Only flag potential conflicts.

### 11. Important Concepts Repeatedly Mentioned Without Dedicated Pages

Scan all notes for concepts, methods, frameworks, or techniques that appear frequently but lack a dedicated concept page.

Look for:
- Methods mentioned in multiple courses and projects
- Theoretical frameworks used in research and career planning
- Technical skills mentioned across multiple contexts
- Decision frameworks or mental models repeated in Personal notes

For each frequently mentioned concept lacking a page:
- List where it appears (note paths)
- Explain why a dedicated concept page would be valuable
- Suggest initial scope or structure

**Report format**:
```
Missing Concept Pages: X
- "Risk assessment framework": appears in [[40 Workspace/Finance/Notes.md]], [[40 Workspace/Research/Method.md]], [[50 Daily/2026-07-10.md]]
  → Suggestion: create [[30 Wiki/Concepts/Risk Assessment.md]]
```

### 12. Project Knowledge Trapped in Daily Notes

Scan recent Daily notes for information that should be permanent project documentation.

Look for:
- Research findings or experimental results
- Project decisions or significant milestones
- Technical solutions or architectural choices
- Important conversations or meeting outcomes
- Project progress or status updates that update the record

For each piece of lasting project knowledge:
- Identify which Daily note contains it
- Suggest which Workspace project note should contain the permanent record
- Note the significance (should this update the project's permanent documentation?)

**Report format**:
```
Project Knowledge in Daily Notes: X
- [[50 Daily/2026-07-12.md]]: "Decided to use approach X because Y" → should update [[40 Workspace/Research/Project/Design Decisions.md]]
- [[50 Daily/2026-07-14.md]]: Meeting outcome "Team agreed on timeline Z" → should update [[40 Workspace/Project/Project Plan.md]]
```

Do NOT move information. Only identify and suggest.

### 13. Repeatedly Carried Tasks

Scan Daily notes to identify tasks that have been carried forward multiple times.

Look for tasks with `⚠ Repeated carry-forward` markers or with multiple `↪ from` markers.

For each:
- Count how many times it has been carried
- Note the date range (first appearance to most recent)
- Assess current status (still unfinished? ongoing concern?)

**Report format**:
```
Repeatedly Carried Tasks: X
- "Finish analysis report": carried 5 times (2026-06-10 → 2026-07-15)
  → Status: still unfinished; suggestion: assign due date or break into smaller action
```

### 14. Inconsistent Daily Note Structure

Scan `50 Daily/` notes to verify they follow the standard structure:

1. Carried Forward
2. Today's Tasks
3. Schedule and Deadlines
4. Notes and Captures
5. Completed Today
6. Blockers
7. Needs Reconsideration
8. Questions for Review
9. End-of-Day Reflection

Check for:
- Notes missing major sections
- Sections with inconsistent naming
- Notes that combine or reorder sections
- Missing End-of-Day Reflection

**Report format**:
```
Inconsistent Daily Structure: X
- [[50 Daily/2026-07-10.md]]: missing "Blockers" section
- [[50 Daily/2026-07-12.md]]: "Tasks" used instead of "Today's Tasks"
- [[50 Daily/2026-07-15.md]]: missing "End-of-Day Reflection"
```

Do NOT rewrite notes. Only flag structure issues.

### 15. Distinguish Harmless Observations from Necessary Action

As you audit, categorize findings:

**Harmless (no action needed)**:
- Isolated notes that are deliberately standalone
- Intentional orphans waiting for connection
- Older Inbox items that are waiting for user decision
- Personal conflicts that reflect natural evolution of thinking

**Worthwhile (should address when convenient)**:
- Missing concept pages that would improve navigation
- Project knowledge in daily notes that should be documented
- Stale index entries that should be updated
- Misplaced files that could be organized better

**Urgent (should address soon)**:
- Broken links that point to critical notes
- Unsupported AI claims that could mislead
- Unresolved AI Review Queue items blocking decisions
- Serious file placement errors that break expected structure

**Strictly Avoid**:
- Do NOT modify Sources
- Do NOT rewrite Personal notes
- Do NOT make large structural changes automatically
- Do NOT merge or consolidate pages without authorization
- Do NOT delete files

### 16. Put Substantial Changes into AI Review Queue

For any changes that require judgment or involve Personal notes:

Add entry to `90 System/AI Review Queue.md`:
- Type of change (merge pages, update index, move file, etc.)
- Target note(s)
- Proposed change with reasoning
- Impact of change
- Confidence level
- Status: Pending (awaiting user decision)

Wait for explicit authorization before making changes.

Only make changes if clearly authorized by AGENTS.md:
- Small repairs to broken links if the target is unambiguous
- Fixing malformed link syntax
- Updating stale index entries if you understand the current structure

### 17. Make Small Safe Repairs Only

Authorized repairs (require no user decision):
- Fix malformed link syntax: `[[` to `[[link]]` (if target is clear)
- Fix anchor link formatting
- Standardize date formats in daily notes if inconsistent
- Remove duplicate blank sections in daily notes

Do NOT make repairs that involve:
- Choosing which duplicate page to keep (user decision)
- Moving files (structural choice)
- Merging content (semantic choice)
- Updating Personal notes (protected content)

Report all repairs made.

### 18. Update Activity Log

Append a concise entry to `90 System/log.md`:

```markdown
## YYYY-MM-DD — lint

### Request

Run vault health check.

### Files examined

- Total files scanned: N
- Daily notes: M
- Wiki pages: P
- Workspace files: Q
- Personal notes: R

### Files created or modified

- `90 System/AI Review Queue.md`: updated with X new proposals
- [list any small repairs made]

### Important findings

- Broken links: X
- Orphan notes: Y
- Duplicate concept pages: Z
- Unsupported claims: A
- Old Inbox items: B
- Misplaced files: C
- Unresolved review items: D
- Missing concept pages: E
- Project knowledge in daily notes: F
- Repeatedly carried tasks: G
- Inconsistent daily structure: H

### Unresolved questions

- [any issues that need user guidance to resolve]
```

### 19. Produce Prioritized Health Report

After the complete audit, provide a comprehensive report organized by priority:

**Format**:

```
## Vault Health Check — YYYY-MM-DD

### URGENT (address soon)
[List items requiring immediate attention with specific file paths and actions]

### WORTHWHILE (address when convenient)
[List improvements that would enhance vault quality without blocking work]

### OPTIONAL (nice-to-have improvements)
[List low-impact observations and suggestions]

### NO ACTION NEEDED
[Observations that are harmless or intentional]

---

## Detailed Findings

### Broken Links (X total)
[List each]

### Orphan Notes (X total)
[List each with type]

### Duplicate/Overlapping Wiki Pages (X total)
[List each with merge suggestions]

### Unsupported AI Claims (X total)
[List each with location]

### Missing Source References (X total)
[List each]

### Stale Index Entries (X total)
[List each]

### Misplaced Files (X total)
[List each with suggested location]

### Old Inbox Items (X total)
[List each with age and suggested action]

### Unresolved AI Review Queue Items (X total)
[List each with status and age]

### Potential Personal Conflicts (X total)
[List each with evidence]

### Missing Concept Pages (X total)
[List each with supporting evidence]

### Project Knowledge in Daily Notes (X total)
[List each with suggested location]

### Repeatedly Carried Tasks (X total)
[List each with carry count]

### Inconsistent Daily Structure (X total)
[List each with detail]

---

## Repairs Made
[List small safe repairs actually applied]

## Proposals in AI Review Queue
[List items added for user decision]

## Next Steps
[Recommended actions for user]
```

### 20. List All Files Created or Modified

At the end of the report, explicitly list:

- Files read (do not need to list individually, just count)
- Files modified (list with specific changes)
- Files created (list each)
- Entries added to AI Review Queue
- Entries added to log.md

## Constraints and Protections

### Do Not Modify Sources

Do not read or modify `10 Sources/`.

Sources are read-only. Observations can cite sources but never change them.

### Do Not Rewrite Personal Notes

Do not modify `20 Personal/` directly.

Flag conflicts and proposed changes in AI Review Queue.

Wait for explicit authorization before editing Personal notes.

### Do Not Make Large Structural Changes

Do not:
- Bulk-rename directories
- Move large numbers of files
- Reorganize the vault structure without authorization
- Consolidate multiple concept pages without user decision
- Delete files or folders

Small repairs are okay (broken link fixes). Structural changes require AI Review Queue proposal + user approval.

### Preserve All Evidence

For every finding, cite:
- File path
- Specific evidence (quote, line number, or description)
- Why it matters

Do not make claims without backing evidence.

### Report Accurately

If something is unclear or could be interpreted multiple ways, say so.

Do not hide uncertainty or ambiguity.

Provide confidence levels: certain / likely / tentative.

## Definition of Done

Vault health check is complete only when:

- [ ] All `.md` files have been scanned for broken links
- [ ] Orphan notes have been identified
- [ ] Duplicate/overlapping Wiki pages have been identified
- [ ] Unsupported AI claims have been flagged
- [ ] Missing source references have been found
- [ ] Stale index entries have been identified
- [ ] Misplaced files have been flagged
- [ ] Old Inbox items have been assessed
- [ ] Unresolved AI Review Queue items have been checked
- [ ] Personal conflicts have been detected
- [ ] Missing concept pages have been suggested
- [ ] Project knowledge in daily notes has been identified
- [ ] Repeatedly carried tasks have been catalogued
- [ ] Daily note structure inconsistencies have been noted
- [ ] Findings are prioritized (urgent / worthwhile / optional / no action)
- [ ] Small safe repairs have been applied (if any)
- [ ] Substantial changes are in AI Review Queue
- [ ] log.md has been updated with a lint entry
- [ ] User receives a prioritized health report with all files listed
