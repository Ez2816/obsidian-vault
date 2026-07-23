---
name: obsidian-end-day
description: Review today's work, identify lasting knowledge and blockers, flag issues for reconsideration without moving tasks or creating artificial productivity scores
trigger:
  - "Close my day"
  - "End my day"
  - "Review today's tasks"
  - "Run my evening review"
scope: repository
---

# Obsidian End-of-Day Review

## Trigger

User says things like:
- Close my day
- End my day
- Review today's tasks
- Run my evening review

## Workflow

### 1. Determine Today's Date

Determine the current date in `America/Vancouver` timezone using YYYY-MM-DD format.

**CRITICAL**: All dates must be calculated in Pacific time, not UTC. This prevents the skill from trying to access tomorrow's notebook when running after midnight UTC.

### 2. Read Today's Daily Note

Locate and read today's daily note: `50 Daily/YYYY-MM-DD.md`

### 3. Count and Summarize Completed Work

Count tasks marked as completed (`- [x]`).

Summarize meaningful accomplishments without creating a productivity score.

Focus on:
- What was actually finished
- Work that moved projects forward
- Challenges overcome
- Useful discoveries or decisions made

Do NOT include:
- A productivity rating or score
- Judgmental language
- Comparison to a "should" baseline
- Guilt framing

### 4. Identify Unfinished Tasks and Blockers

List unfinished tasks (`- [ ]`) that remain in the note.

Identify tasks with blockers:
- External dependencies
- Missing information
- Technical obstacles
- Resource constraints
- Unclear next steps

### 5. Suggest Task Improvements

For unfinished tasks, suggest whether they should be:

- **Broken down**: Too large or unclear; needs a smaller next action
- **Scheduled**: Needs a calendar block or specific time commitment
- **Delegated**: Someone else should do this
- **Moved to backlog**: Not a priority right now; belongs in project planning
- **Clarified**: Intent is unclear or conditions have changed
- **Removed**: No longer needed or relevant

Suggest only—do not modify the tasks.

### 6. Identify Lasting Knowledge

Scan today's note for information that belongs in the permanent vault:

- Research decisions or findings
- Experimental results or technical solutions
- Course insights or learning breakthroughs
- Meeting outcomes or important communications
- Accomplishments worth preserving
- Changes in career direction or personal judgment
- Resolved questions or confirmed preferences

### 7. Suggest Preservation Locations

For each lasting piece of knowledge, suggest where it should be stored:

- `30 Wiki/Concepts/` — New concept pages or updates to existing concepts
- `30 Wiki/Syntheses/` — Connections between ideas
- `40 Workspace/[Project]/` — Permanent project documentation
- `20 Personal/[Category]/` — Changes to preferences, goals, or decisions
- `90 System/index.md` — If it belongs in the knowledge index

Do NOT actually move or copy information—only suggest.

### 8. Check for Personal Note Changes

If today's work suggests a change to a Personal note (e.g., a decision, preference, or goal update):

- Note the connection clearly
- Do NOT modify the Personal note directly
- Place the proposed change in `90 System/AI Review Queue.md` with:
  - Which Personal note it affects
  - What changed or what should be updated
  - Why (evidence from today's work)
  - Confidence level (certain / likely / tentative)

### 9. Perform Lightweight Daily Health Checks

Check for only these issues:

**Duplicate Tasks in Today's Note**
- Scan `Carried Forward` and `Today's Tasks` sections
- Flag any tasks that appear twice or are equivalent
- Note in `Questions for Review`

**Malformed Links**
- Check for broken internal links in sections changed today
- Flag syntax errors like `[[ ]]` or `[[missing]]` where the target doesn't exist
- Do NOT fix broken links; report them

**Captures Needing Classification**
- Look in `Notes and Captures` section
- Identify rough notes that should be moved to:
  - `00 Inbox/` (if raw capture)
  - `10 Sources/` (if extracted source material)
  - `30 Wiki/` (if conceptual learning)
  - `40 Workspace/` (if project-related)

**Lasting Knowledge Trapped in Daily Note**
- Identify research findings, decisions, course insights, or project knowledge that should be in permanent locations
- Suggest preservation (already done in Step 5)

**Repeatedly Carried Tasks**
- Identify tasks with `⚠ Repeated carry-forward` markers
- Note if they appeared again today and were not completed
- Report in summary

**Proposed Personal-Note Changes**
- Identify any updates to `20 Personal/` that should go to AI Review Queue
- Do NOT modify Personal notes directly

Do NOT perform a complete vault-wide audit. Focus only on today's changes.

### 10. Do Not Carry Forward

Do NOT move unfinished tasks into tomorrow's note.

The morning skill (`obsidian-start-day`) handles task carryover.

Unfinished tasks remain in today's note as a historical record.

### 11. Add End-of-Day Reflection

Add a concise summary to the `End-of-Day Reflection` section:

- What went well
- What didn't go as planned
- Useful observations or insights
- What should be prioritized tomorrow

Keep it brief (3-5 bullet points max).

Do NOT include productivity judgment or guilt framing.

## Report Structure

After the review, provide a summary covering:

1. **Completed**: X tasks finished today
2. **Unfinished**: Y tasks remaining; list key blockers
3. **Lasting Knowledge Identified**: List items needing preservation + suggested locations
4. **Suggested Task Improvements**: Which tasks should be broken down, scheduled, delegated, or removed
5. **Health Check Findings**: Duplicate tasks, broken links, captures needing classification, repeated carries
6. **Personal Note Changes**: Proposed updates with reasoning (if any)
7. **No Action Needed**: Blockers, questions, or issues that are already tracked

## Constraints and Protections

### Do Not Silently Rewrite Personal Notes

Never modify `20 Personal/` directly during end-of-day review.

If you identify a change that should be made to a Personal note:
- Place it in `90 System/AI Review Queue.md`
- Wait for explicit user authorization

### Do Not Create Productivity Scoring

Do not compare work to:
- A daily target
- Yesterday's work
- A "should" baseline
- A productivity metric

Summarize what happened. That is sufficient.

### Do Not Carry Forward

The morning workflow carries forward unfinished tasks.

Your job is to review, identify, and suggest—not to move tasks into tomorrow's note.

Leave unfinished tasks exactly where they are in today's note.

### Do Not Perform Complete Audits

Focus on today's activity only.

Do not scan the entire vault for broken links, orphaned notes, or structural issues.

A complete health audit is a separate workflow (`obsidian-vault-health`).

### Do Not Modify Sources

Do not read or modify `10 Sources/` during end-of-day review.

## User Action Reminder

After end-of-day review completes, remind the user:

```
✅ YOUR PART — Close Out Your Day

Before tomorrow morning, please:

1. **Mark completed tasks**: Check off all tasks you finished today with [x]
2. **Review the reflection**: Read the End-of-Day Reflection summary I added
3. **Address blockers**: Note any blockers that need action tomorrow or escalation
4. **Consider suggestions**: If I suggested breaking down, scheduling, or delegating tasks:
   - Move tasks to appropriate backlog/calendar if needed
   - Mark #no-carryover if you don't want a task carried forward
5. **Preserve lasting knowledge**: Any important findings from today should be:
   - Moved to permanent locations (Wiki, Workspace, Personal)
   - Or left as-is if you want to address it later
6. **Review proposed changes**: Check AI Review Queue for any Personal note changes I proposed
   - Approve, reject, or request modifications
7. **Reflect**: Add any personal observations to "End-of-Day Reflection" if you want

Tomorrow's morning setup will carry forward only incomplete tasks (not marked [x] or [-]).

Your daily note: 50 Daily/YYYY-MM-DD.md
```

### 12. Commit and Push Changes

After the end-of-day review is complete, commit and push changes to GitHub:

```bash
git add -A
git commit -m "end-of-day review: record completion status and observations"
git push origin master
```

Do not push if there are no changes (git commit will have failed).

## Definition of Done

End-of-day review is complete only when:

- [ ] Today's completed and unfinished tasks have been counted
- [ ] Accomplishments are summarized (without productivity scoring)
- [ ] Blockers are identified
- [ ] Task improvement suggestions are made
- [ ] Lasting knowledge is identified with suggested preservation locations
- [ ] Personal note changes are placed in AI Review Queue (not silently modified)
- [ ] Lightweight health checks are completed (duplicates, broken links, captures, repeated carries)
- [ ] End-of-Day Reflection section contains a concise summary
- [ ] User receives a clear report of findings and suggestions
- [ ] User receives action reminder about their responsibilities for the evening
