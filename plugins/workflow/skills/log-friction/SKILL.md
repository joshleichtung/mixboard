---
name: workflow-log-friction
description: "Log workflow friction - something that slowed you down or was annoying"
category: feedback
complexity: simple
---

# /sc:log-friction - Log Friction Point

> **Purpose**: Capture pain points immediately so they don't get forgotten and can be systematically addressed.

## Usage

```
/sc:log-friction "Description of what was annoying or slowed you down"
```

## Behavior

When this command is invoked:

1. **Extract description** from user input (everything after "log-friction")
2. **Determine context** from conversation (if available)
3. **Create/append to today's friction file**: `~/.claude/feedback/friction/YYYY-MM-DD.md`
4. **Format entry** with timestamp, impact analysis, and potential fixes
5. **Suggest priority** based on frequency/impact
6. **Confirm to user**

## Entry Format

```markdown
## HH:MM - [Context]

[User's description]

**Impact**: [How this slowed you down or caused problems]
**Potential Fix**: [Ideas for addressing this - brainstorm options]
**Priority**: [High/Medium/Low based on frequency and impact]
```

## Examples

### Example 1: Tool/Process Friction

**Input**: `/sc:log-friction "Had to manually update 3 files every time I add a slash command"`

**Output** (saved to `friction/2025-11-01.md`):
```markdown
## 15:45 - Command Development

Had to manually update 3 files every time I add a slash command

**Impact**: 5-10 minutes per command, error-prone, discourages creating helpful commands
**Potential Fix**:
- Create /sc:new-command scaffolding tool
- Template-based generation
- Single source of truth for command metadata
**Priority**: High (frequent operation, compounds over time)
```

### Example 2: Environment/Setup

**Input**: `/sc:log-friction "Color contrast issues in terminal made diff output hard to read"`

**Output** (saved to `friction/2025-11-01.md`):
```markdown
## 10:20 - Development Environment

Color contrast issues in terminal made diff output hard to read

**Impact**: Strain to read git diffs, missed changed lines, slower code review
**Potential Fix**:
- Adjust ANSI color values in .wezterm.lua
- Test against git diff specifically
- Use color-detective.html to validate all colors
**Priority**: Medium (daily impact but workaround exists - squint harder)
```

### Example 3: Workflow Gap

**Input**: `/sc:log-friction "No way to track long-term productivity trends or costs"`

**Output** (saved to `friction/2025-11-01.md`):
```markdown
## 17:00 - Metrics & Tracking

No way to track long-term productivity trends or costs

**Impact**: Can't see improvement over time, no data for optimizing workflow, session data is ephemeral
**Potential Fix**:
- Enhance statusline to log to persistent file
- Create weekly/monthly aggregation
- Build /sc:stats command for trend analysis
**Priority**: Low (nice to have, not blocking current work)
```

## File Structure

**Path**: `~/.claude/feedback/friction/YYYY-MM-DD.md`

**Header** (created automatically if new file):
```markdown
# Friction Points - [Month Day, Year]

Capturing workflow friction to address systematically.

---
```

**Then entries are appended** chronologically throughout the day.

## Priority Assignment

**High Priority** indicators:
- Happens daily/frequently
- Significant time waste (5+ minutes)
- Causes errors or quality issues
- Blocks or significantly slows critical path
- Affects multiple projects/contexts

**Medium Priority** indicators:
- Happens weekly
- Moderate time waste (2-5 minutes)
- Workaround exists but annoying
- Impacts quality of life but not deliverables

**Low Priority** indicators:
- Happens rarely
- Minor time waste (< 2 minutes)
- Easy workaround
- More of an annoyance than blocker
- Nice-to-have improvement

## Implementation Notes

### Impact Analysis

Should capture:
- Time cost (quantify if possible)
- Frequency (daily, weekly, rare)
- Quality impact (errors, missed things)
- Emotional toll (frustration, discouragement)
- Compound effects (gets worse over time?)

### Potential Fix Brainstorming

Generate 2-4 potential solutions:
1. **Quick fix** (if exists) - workaround or tactical solution
2. **Proper fix** - systematic solution
3. **Ideal fix** - if time/resources unlimited
4. **Alternative** - different approach entirely

Don't filter ideas here - brainstorm freely, prioritize later.

### Context Inference

Same categories as log-win:
- Development Workflow
- Development Environment
- Tooling & Configuration
- Process & Methodology
- Communication
- Context Management
- Code Quality
- Team Collaboration
- Other

## Related Commands

- `/sc:log-win` - Log things that worked well
- `/sc:log-idea` - Capture improvement ideas (more proactive than friction)
- `/sc:weekly-review` - Convert top friction to action items

## Success Criteria

**You're using this well when:**
- Logging friction immediately when it happens (not batching)
- Friction log entries are specific and actionable
- Top friction points get addressed (friction log shrinks)
- Same issue doesn't appear week after week
- Potential fixes lead to actual improvements

**Anti-patterns:**
- Only logging major frustrations (log small ones too - they compound!)
- Vague descriptions ("things were slow")
- Not reviewing or acting on friction log
- Same friction recurring month after month (not addressing root cause)
- Complaining without capturing (tell Claude, not just teammates!)

## Weekly Review Integration

During `/sc:weekly-review`:
1. Top 2-3 friction points highlighted
2. Asked to prioritize: which to fix first?
3. Create action items for high-priority friction
4. Celebrate if friction log is shrinking

## Monthly Patterns

Watch for:
- **Recurring friction** = root cause needs addressing
- **Friction clusters** = systemic issue (process, tools, environment)
- **Friction trends** = is it getting better or worse?
- **Quick wins** = small fixes with big impact

---

**Created**: 2025-11-01
**Part of**: Systematic feedback capture system
