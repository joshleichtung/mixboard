---
name: log-win
description: "Log a workflow win - something that worked well"
category: feedback
complexity: simple
---

# /sc:log-win - Log a Win

> **Purpose**: Capture things that work well in your workflow so you can reinforce and amplify them.

## Usage

```
/sc:log-win "Description of what worked well"
```

## Behavior

When this command is invoked:

1. **Extract description** from user input (everything after "log-win")
2. **Determine context** from conversation (if available)
3. **Create/append to today's win file**: `~/.claude/feedback/wins/YYYY-MM-DD.md`
4. **Format entry** with timestamp and analysis
5. **Confirm to user**

## Entry Format

```markdown
## HH:MM - [Context Inferred from Conversation]

[User's description]

**Why it worked**: [Brief analysis of what made this effective]
**Keep doing**: [Actionable takeaway to reinforce this pattern]
```

## Examples

### Example 1: Development Workflow

**Input**: `/sc:log-win "BMad checklist pattern caught 3 issues before commit"`

**Output** (saved to `wins/2025-11-01.md`):
```markdown
## 14:23 - Development Workflow

BMad checklist pattern caught 3 issues before commit

**Why it worked**: Structured checklist forces systematic review before committing
**Keep doing**: Use pre-commit checklist for all non-trivial commits, especially work projects
```

### Example 2: Theme/Tooling

**Input**: `/sc:log-win "New Incal Dreams theme significantly reduced eye strain"`

**Output** (saved to `wins/2025-11-01.md`):
```markdown
## 16:45 - Tooling & Environment

New Incal Dreams theme significantly reduced eye strain

**Why it worked**: Lighter background (#251f2e vs #1a1625) and better contrast (#c5b8cc for dim text)
**Keep doing**: Prioritize ergonomics in tool configuration, measure impact over time
```

### Example 3: Productivity Pattern

**Input**: `/sc:log-win "Context monitoring hook prevented overflow 3 times today"`

**Output** (saved to `wins/2025-11-01.md`):
```markdown
## 18:30 - Context Management

Context monitoring hook prevented overflow 3 times today

**Why it worked**: Early warning at 80% gave time to summarize and clear strategically
**Keep doing**: Act on context warnings immediately, don't ignore them
```

## File Structure

**Path**: `~/.claude/feedback/wins/YYYY-MM-DD.md`

**Header** (created automatically if new file):
```markdown
# Wins - [Month Day, Year]

Capturing what's working well to reinforce positive patterns.

---
```

**Then entries are appended** chronologically throughout the day.

## Implementation Notes

### Context Inference

Try to infer context from:
1. Current conversation topic
2. Recent tool usage (Edit, Write, Bash)
3. File paths being discussed
4. Project mentioned

**Context categories**:
- Development Workflow
- Tooling & Environment
- Context Management
- Code Quality
- Communication
- Productivity Pattern
- Learning/Discovery
- Team Collaboration
- Other

### Analysis Generation

**"Why it worked"** should identify:
- The underlying mechanism
- What made this effective
- Any patterns worth noting

**"Keep doing"** should provide:
- Specific actionable advice
- When to apply this pattern
- How to reinforce it

### Timestamp Format

- Use 24-hour format: `HH:MM`
- Local timezone
- Helpful for seeing when wins cluster (morning productivity, afternoon flow, etc.)

## Related Commands

- `/sc:log-friction` - Log pain points and friction
- `/sc:log-idea` - Capture improvement ideas
- `/sc:weekly-review` - Review all feedback for the week

## Success Criteria

**You're using this well when:**
- Logging 1+ wins per day (at minimum)
- Wins are specific, not generic
- Analysis provides actual insight
- You reference past wins to reinforce patterns
- Wins log grows over time (not static)

**Anti-patterns:**
- Only logging major wins (log small ones too!)
- Vague descriptions ("things went well")
- Not acting on "keep doing" advice
- Forgetting to log in the moment (batching at end of day loses context)

---

**Created**: 2025-11-01
**Part of**: Systematic feedback capture system
