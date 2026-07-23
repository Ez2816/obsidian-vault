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

This skill executes a complete workflow using Bash, Read, Write, and Edit tools. All paths are quoted to handle spaces correctly.

## Workflow Execution

### Step 1: Determine Today's Date

Get current date in America/Vancouver timezone:

```bash
TODAY=$(TZ='America/Vancouver' date +%Y-%m-%d)
YESTERDAY=$(TZ='America/Vancouver' date -d 'yesterday' +%Y-%m-%d)
echo "Today: $TODAY, Yesterday: $YESTERDAY"
```

### Step 2: Check or Create Today's Daily Note

Read the template and create today's note if it doesn't exist:

```bash
DAILY_DIR="./50 Daily"
TODAY_FILE="$DAILY_DIR/$TODAY.md"
TEMPLATE_FILE="./90 System/Templates/Daily Note.md"

# Check if today's note exists
if [ -f "$TODAY_FILE" ]; then
    echo "Today's note already exists: $TODAY_FILE"
    # Read existing note to prepare for task merging
else
    echo "Creating new daily note for $TODAY"
    # Read template and create note with {{date}} replaced
    TEMPLATE_CONTENT=$(cat "$TEMPLATE_FILE")
    NEW_CONTENT="${TEMPLATE_CONTENT//\{\{date\}\}/$TODAY}"
    # Write the new note
    echo "$NEW_CONTENT" > "$TODAY_FILE"
    echo "Created: $TODAY_FILE"
fi
```

### Step 3: Find Previous Daily Note

Locate the most recent daily note before today:

```bash
YESTERDAY_FILE="$DAILY_DIR/$YESTERDAY.md"

# If yesterday's file doesn't exist, scan backwards
if [ ! -f "$YESTERDAY_FILE" ]; then
    echo "Yesterday's note doesn't exist, scanning backwards..."
    for i in {1..7}; do
        SCAN_DATE=$(TZ='America/Vancouver' date -d "$i days ago" +%Y-%m-%d)
        SCAN_FILE="$DAILY_DIR/$SCAN_DATE.md"
        if [ -f "$SCAN_FILE" ]; then
            YESTERDAY_FILE="$SCAN_FILE"
            YESTERDAY="$SCAN_DATE"
            echo "Found previous note: $YESTERDAY_FILE"
            break
        fi
    done
fi
```

### Step 4: Extract and Carry Forward Eligible Tasks

Read tasks from the previous note and filter eligible ones:

**Read previous note** using the Read tool and extract all `- [ ]` tasks (incomplete).

**Eligibility criteria** — Task is eligible if:
- Incomplete: `- [ ]` (not `- [x]` or `- [-]`)
- No `#no-carryover` tag
- Not already in today's note
- Not a delegated/waiting task with no action
- Not explicitly marked as intentionally postponed

**Special handling**:
- Do NOT modify or mark complete in the previous note (preserve history)
- Track carry count by counting `↪ from` markers in the task's history
- Flag tasks with 3+ carries: add `⚠ Repeated carry-forward` marker

### Step 5: Build Today's Task Section

For each eligible task, add to today's note in the `Tasks` section:

**Format for carried tasks**:
```
- [ ] Task description ↪ from [[YYYY-MM-DD]]
```

**For tasks with 3+ carries**, also add to `Needs Reconsideration` section with options:
- Break into smaller steps?
- Schedule on calendar?
- Delegate it?
- Mark #no-carryover?
- Clarify priority or remove it?

### Step 6: Add Repeated Carries Section

If any tasks have 3+ carries, populate the `Needs Reconsideration` section with each flagged task and suggested actions.

### Step 7: Update Previous Note's History (Optional)

For each carried task, optionally append carry marker to the previous note:
```
↪ carried to [[YYYY-MM-DD]]
```

**Do NOT do this if it would cause merge conflicts or make the previous note too cluttered.** Preserve previous notes as immutable historical records. Only add carry markers if they don't disrupt the note's original content.

### Step 8: Commit and Push

After all changes are complete:

```bash
cd "$(git rev-parse --show-toplevel)"
if [ -n "$(git status --porcelain)" ]; then
    git add "50 Daily/$TODAY.md"
    git commit -m "morning setup: create daily note and carry forward tasks"
    git push origin master
    echo "Changes committed and pushed"
else
    echo "No changes to commit"
fi
```

### Step 9: Report Summary

Provide a concise summary:

```
Morning setup complete ✓
- Today's note: 50 Daily/2026-07-23.md
- Tasks carried forward: X
- Repeated carries flagged (3+): Y
- Previous note source: [[YYYY-MM-DD]]
- Changes: committed and pushed to GitHub
```

## Implementation Notes

### Path Handling
All file paths with spaces use proper bash quoting:
- `"./50 Daily"` (quoted)
- `"$DAILY_DIR/$TODAY.md"` (quoted variable)
- `"./90 System/Templates/Daily Note.md"` (quoted)

### Timezone Safety
Always use explicit `TZ='America/Vancouver'` for date calculations:
```bash
TODAY=$(TZ='America/Vancouver' date +%Y-%m-%d)
```

### Error Handling
- Check file existence before reading: `[ -f "$FILE" ]`
- Verify git operations didn't fail: check exit codes
- Report errors clearly so the user knows what went wrong

### Git Safety
- Only commit if there are changes: `git status --porcelain`
- Use full paths in git operations to avoid cwd issues
- Always push after commit

## Constraints

### Never Modify
- `10 Sources/` — Read-only
- Previous daily notes' original tasks — Preserve as historical records

### Always Preserve
- Original task wording exactly
- All tags, due dates, priority markers
- Previous note's history (don't mark tasks completed in earlier notes)
- Carry count accuracy

## Definition of Done

- [ ] Today's daily note exists at `50 Daily/YYYY-MM-DD.md`
- [ ] All eligible unfinished tasks carried forward with `↪ from` markers
- [ ] Tasks with 3+ carries flagged and moved to `Needs Reconsideration`
- [ ] Previous note remains an accurate historical record
- [ ] Changes committed and pushed to GitHub
- [ ] Summary provided to user

