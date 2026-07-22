---
name: obsidian-start-day
description: Create today's daily note and carry forward eligible unfinished tasks from yesterday, preserving history and flagging repeated carries
trigger:
  - "Start my day"
  - "Run my morning setup"
  - "Create today's daily note"
  - "Carry forward yesterday's tasks"
scope: repository
---

# Obsidian Morning Setup

## ⚠️ CRITICAL REMINDER

This skill describes a workflow, but **YOU MUST ACTUALLY EXECUTE IT USING TOOLS**:
- Use **Read** tool to check if files exist
- Use **Write** tool to create/modify files
- Use **Bash** for file scanning if needed
- Do not just describe actions; actually perform them

Do not treat this skill as narrative instructions. Treat it as a checklist of actual operations to perform.

## Trigger

User says things like:
- Start my day
- Run my morning setup
- Create today's daily note
- Carry forward yesterday's tasks

## Workflow

### 1. Determine Today's Date

Determine the current date in `America/Vancouver` timezone using YYYY-MM-DD format.

### 2. Check or Create Today's Daily Note

**CRITICAL**: Use Read and Write tools to actually create/modify files.

Check whether `50 Daily/YYYY-MM-DD.md` exists using Read.

If it does not exist:
- **USE WRITE TOOL** to create `50 Daily/YYYY-MM-DD.md`
- Copy the template structure from `90 System/Templates/Daily Note.md`
- Replace `{{date}}` with today's date in YYYY-MM-DD format
- Do not just describe the file; actually write it

If it already exists:
- Read it to check its contents
- Proceed to task carryover

### 3. Find the Most Recent Previous Daily Note

Locate the most recent daily note dated before today in `50 Daily/`.

Scan backwards from yesterday until you find a file with eligible tasks.

### 4. Read Eligible Tasks

Read all incomplete tasks from the previous daily note.

A task is eligible for carryover if it:
- Is an incomplete Markdown task: `- [ ] Task description`
- Does NOT have `#no-carryover` tag
- Is NOT marked completed: `- [x]`
- Is NOT marked cancelled: `- [-]`
- Is NOT a non-actionable checklist item
- Is NOT already present in today's note
- Is NOT a delegated or waiting task with no current action
- Was NOT explicitly marked by the user as intentionally postponed to a later date

If uncertain whether a task is actionable, place it in `Questions for Review` section rather than carrying it forward.

### 5. Prevent Duplicates

Before adding a task to today's note, check if an equivalent task already exists in today's `Tasks` section.

Treat as duplicates:
- Identical wording
- Equivalent wording
- Same project and intended outcome
- Only minor punctuation or spacing differences

If two tasks may be duplicates but the distinction is unclear, preserve both and place a note under `Questions for Review`.

### 6. Project-Link Review

After identifying eligible tasks, review project associations:

1. **Preserve existing links**: If a carried task already contains a project link (e.g., `[[Project Name]]`), keep it exactly as written.

2. **Check project fit**: For each carried task, consider whether it belongs to an existing project note in `40 Workspace/`.

3. **Do not invent projects**: Do not add links to projects that are not clearly documented in `40 Workspace/`.

4. **Unambiguous project**: When the correct project is clear and unambiguous:
   - Add one relevant project link while preserving the original task wording
   - Example: `- [ ] Review methodology [[Research/Project Name]] ↪ from [[YYYY-MM-DD]]`
   - Place the link at the end, after the carry marker

5. **Ambiguous project fit**: When multiple projects could apply or the connection is unclear:
   - Leave the task unchanged
   - List the possible project links under `Questions for Review` section
   - Example: `- [ ] Design experiment | Possible projects: [[Research/ProjectA]], [[Research/ProjectB]]`

6. **Standalone tasks**: Do not force project links onto:
   - Routine maintenance tasks
   - Personal errands
   - One-off administrative items
   - Tasks that do not advance any ongoing project

7. **Linking criterion**: A task should be linked to a project when:
   - Completing it meaningfully advances that project
   - It resolves one of the project's open questions
   - It produces information the project should retain
   - The connection is direct and substantive, not incidental

8. **Report additions**: Track any project links added and note proposed links under Questions for Review.

### 8. Copy Eligible Tasks to Today's Note

For each eligible task, add it to today's `Tasks` section with this format:

```markdown
- [ ] Task description ↪ from [[YYYY-MM-DD]]
```

If a project link was added during Step 6, include it:

```markdown
- [ ] Task description [[Project Name]] ↪ from [[YYYY-MM-DD]]
```

Preserve:
- Original task wording exactly
- Project links (e.g., `[[Project Name]]`)
- All tags
- Due dates
- Priority markers
- Useful context
- Checkbox status as `[ ]` (incomplete)

### 9. Preserve Earlier Note's History

Do NOT remove the unfinished task from the earlier daily note.

Do NOT mark the earlier task as completed merely because it was copied into today's note.

The earlier note must remain an accurate record of what was planned and not completed on that date.

Optionally, append this marker to the earlier task to track where it went:
```markdown
↪ carried to [[YYYY-MM-DD]]
```

### 10. Create Link to Previous Day

When carrying forward tasks from the previous day, include the source date marker in the task itself:

```markdown
- [ ] Task description ↪ from [[YYYY-MM-DD-previous]]
```

### 11. Flag Repeated Carryovers

For each carried task, track how many times it has been carried forward by counting `↪ from` markers in its history.

When a task has been carried forward three or more times:
- Add this marker: `⚠ Repeated carry-forward`
- Move the task to the `Needs Reconsideration` section
- Suggest one or more of these actions:
  - Break it into a smaller next action
  - Schedule it on a calendar
  - Move it to a project backlog
  - Assign a real due date
  - Delegate it
  - Remove it
  - Clarify why it remains important
  - Mark it `#no-carryover` (exempt from future carries)
  - Convert it into a waiting item

### 12. Report Results

After completing carryover, provide a summary:

- Number of tasks carried forward
- Number of tasks skipped (and why)
- Number of repeated carries detected (3+)
- Ambiguous duplicates flagged for review
- Project links added or proposed
- Name of previous daily note used

Use a concise format:

```
Morning setup complete:
- Carried forward: X tasks
- Skipped: Y tasks (reasons)
- Repeated carries (⚠): Z tasks
- Ambiguous duplicates: N items in Questions for Review
- Project links added: M tasks | Proposed: N items under Questions for Review
- Source: [[YYYY-MM-DD]]
```

## Constraints and Protections

### Never Modify `10 Sources/`

Do not read, modify, move, rename, or delete any files in `10 Sources/`.

### Do Not Silently Modify `20 Personal/`

Do not modify Personal notes during morning setup.

If a carried task references a Personal note or implies an update to Personal context:
- Note the connection for potential AI Review Queue entry
- Do not modify the Personal note
- Report the potential connection in the summary

### Preserve Complete History

Never erase or rewrite an earlier day's task history merely because a task was carried forward.

Earlier daily notes are immutable historical records.

### Only Routine Carryover During Morning

Do not create or modify other vault notes during morning setup unless explicitly requested.

The morning workflow touches only:
- Create today's daily note (if missing)
- Read previous daily note(s)
- Modify today's daily note (add carried tasks)

### Track Carry Count Accurately

Count carries by scanning back through daily notes and counting `↪ from` markers.

Do not reset carry counts or modify past dates' carry markers.

## User Action Reminder

After morning setup completes, remind the user:

```
📋 YOUR PART — Review and Update Today's Note

After the morning setup, please:

1. **Review carried tasks**: Check that carried tasks still make sense for today
2. **Add new tasks**: Add any additional tasks for today under "Tasks"
3. **Check project links**: Verify that project links added are accurate
4. **Update schedule**: Add any deadlines or time-blocked events to "Schedule and Deadlines"
5. **Note changes**: If circumstances changed overnight, update context in "Notes and Captures"
6. **Challenge repeated carries**: Any tasks with ⚠ Repeated carry-forward should be reconsidered:
   - Break it into smaller steps?
   - Schedule it on calendar?
   - Delegate it?
   - Remove it?
   - Mark #no-carryover?

Your daily note is ready at: 50 Daily/YYYY-MM-DD.md
```

### 13. Commit and Push Changes

After the morning setup is complete, commit and push changes to GitHub:

```bash
git add -A
git commit -m "morning setup: create daily note and carry forward tasks"
git push origin master
```

Do not push if there are no changes (git commit will have failed).

## Definition of Done

Morning setup is complete only when:

- [ ] Today's daily note exists
- [ ] Eligible unfinished tasks have been carried forward
- [ ] Duplicate tasks have been checked
- [ ] Repeated carries (3+) have been flagged and moved to `Needs Reconsideration`
- [ ] Yesterday's history remains intact (no tasks marked as completed in earlier notes)
- [ ] Changes are committed and pushed to GitHub
- [ ] User receives a concise summary of the result
- [ ] User receives action reminder about their responsibilities for the day
