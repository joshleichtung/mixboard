---
name: workflow-checklist
description: "Run development checklists interactively"
category: workflow
complexity: simple
---

# /workflow:checklist - Run Development Checklist

> **Purpose**: Interactive checklist runner for development quality gates.

## Usage

```
/workflow:checklist <checklist-name>
/workflow:checklist list
```

## Available Checklists

### pre-commit
**When**: Before every git commit
**Location**: `~/.claude/checklists/pre-commit.md`
**Purpose**: Code quality, testing, documentation, security checks

### pr-ready
**When**: Before creating a pull request
**Location**: `~/.claude/checklists/pr-ready.md`
**Purpose**: Ensure PR is complete, reviewable, and ready for merge

## Behavior

When invoked with a checklist name:

1. **Load checklist** from `~/.claude/checklists/<name>.md`
2. **Present items** interactively
3. **Ask user** to confirm each item (y/n/skip)
4. **Track progress** through checklist
5. **Summarize results** at end
6. **Warn if critical items** are unchecked

## Examples

### Run Pre-Commit Checklist

**Input**: `/workflow:checklist pre-commit`

**Output**:
```
📋 Running Pre-Commit Checklist
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Code Quality (1/4)
══════════════════
□ Code follows project style?
  (y/n/skip) →
```

User responds, Claude continues through all items, then summarizes:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Pre-Commit Checklist Summary
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

✅ Completed: 18/22 items
⚠️  Skipped: 2 items
❌ Unchecked: 2 items

Unchecked Items:
- No debug artifacts (Section: Code Quality)
- Tests pass locally (Section: Testing)

⚠️  Warning: Critical items unchecked!
Recommend addressing before committing.

Proceed with commit? (y/n)
```

### List Available Checklists

**Input**: `/workflow:checklist list`

**Output**:
```
Available Checklists:
━━━━━━━━━━━━━━━━━━━━

1. pre-commit
   Before every git commit
   Location: ~/.claude/checklists/pre-commit.md

2. pr-ready
   Before creating a pull request
   Location: ~/.claude/checklists/pr-ready.md

Usage: /workflow:checklist <name>
```

## Interactive Mode

### Question Format

For each checklist item:
```
□ [Item description]
  (y/n/skip) →
```

### User Responses

- **y** or **yes** → Mark as checked, continue
- **n** or **no** → Mark as unchecked, note for summary
- **skip** → Don't check or uncheck, continue
- **q** or **quit** → Exit checklist early with summary

### Critical Items

Some items marked as critical (security, tests, etc.):
- If unchecked, warning shown in summary
- User prompted: "Continue anyway? (y/n)"
- Helps prevent accidental commits of broken code

## Implementation Notes

### Checklist File Format

Expected markdown format:
```markdown
# Checklist Title

## Section Name

- [ ] Item description
  - Additional details (optional)
  - Sub-items or context

- [ ] Another item

## Another Section

- [ ] Item in different section
```

### Progress Tracking

- Track checked/unchecked/skipped per section
- Show progress: "Section (2/5 complete)"
- Final summary shows overall completion

### Section Handling

- Present sections one at a time
- Allow skipping entire sections if not applicable
- Example: "Skip Security section? (not a work project)"

## Related Commands

- `/workflow:log-friction` - Log checklist pain points
- `/workflow:log-win` - Log when checklist catches issues

## Project-Specific Checklists

You can create project-specific checklists in:
```
<project>/.claude/checklists/
```

These take precedence over global checklists.

**Example project checklists**:
- `hackathon-demo.md` - Pre-demo checklist
- `hipaa-deploy.md` - HIPAA compliance for deployment
- `api-release.md` - API versioning and documentation

## Success Criteria

**Checklist is useful when**:
- Takes < 5 minutes to complete
- Catches real issues regularly
- Items are actionable (clear yes/no)
- Not skipped due to length/complexity

**Anti-patterns**:
- Too long (>30 items = split into multiple)
- Vague items (not clear how to verify)
- Never catches anything (remove or refine)
- Always skipped (not valuable)

## Tips

### Make It Fast
- Keep items concise
- Group related items
- Allow section skipping

### Make It Valuable
- Only include items that catch real issues
- Remove items that are always "yes"
- Add items after post-mortems

### Make It Contextual
- Different checklists for different contexts
- Project-specific checklists override global
- Skip sections that don't apply

---

**Created**: 2025-11-01
**Part of**: Development workflow quality system
