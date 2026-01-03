---
name: workflow-log-idea
description: "Log an improvement idea or feature request"
category: feedback
complexity: simple
---

# /sc:log-idea - Log Improvement Idea

> **Purpose**: Capture ideas for improvements, features, or experiments immediately so they don't get lost.

## Usage

```
/sc:log-idea "Description of idea or improvement"
```

## Behavior

When this command is invoked:

1. **Extract description** from user input (everything after "log-idea")
2. **Determine context** from conversation (if available)
3. **Create/append to today's ideas file**: `~/.claude/feedback/ideas/YYYY-MM-DD.md`
4. **Format entry** with timestamp, value, effort, and priority estimates
5. **Categorize** idea type
6. **Confirm to user**

## Entry Format

```markdown
## HH:MM - [Category]

[User's description]

**Value**: [Why this would be helpful - what problem it solves]
**Effort**: [Rough estimate - Quick/Medium/Large]
**Priority**: [High/Medium/Low based on value/effort ratio]
**Type**: [Tool/Workflow/Automation/Documentation/Other]
```

## Examples

### Example 1: Automation Idea

**Input**: `/sc:log-idea "Auto-generate weekly stats from statusline activity log"`

**Output** (saved to `ideas/2025-11-01.md`):
```markdown
## 16:30 - Automation

Auto-generate weekly stats from statusline activity log

**Value**: Track productivity trends over time, see time/cost per project, identify patterns
**Effort**: Medium (need persistent storage, aggregation script, stats formatting)
**Priority**: Medium (valuable insight but not blocking current work)
**Type**: Automation + Metrics
```

### Example 2: Tool Improvement

**Input**: `/sc:log-idea "Create /sc:new-command to scaffold slash commands automatically"`

**Output** (saved to `ideas/2025-11-01.md`):
```markdown
## 10:15 - Tool Development

Create /sc:new-command to scaffold slash commands automatically

**Value**: Eliminate 10min manual setup per command, reduce errors, encourage creating helpful commands
**Effort**: Quick (template-based generation, single file creation)
**Priority**: High (frequent operation, quick win with big payoff)
**Type**: Tool - Scaffolding
```

### Example 3: Workflow Enhancement

**Input**: `/sc:log-idea "Add Python preset to hackathon bootstrap for ML projects"`

**Output** (saved to `ideas/2025-11-01.md`):
```markdown
## 14:20 - Workflow Enhancement

Add Python preset to hackathon bootstrap for ML projects

**Value**: Faster setup for ML/AI hackathons, proper venv/deps structure
**Effort**: Medium (research Python best practices, create template, test)
**Priority**: Low (occasional use case, existing workarounds)
**Type**: Workflow - New Preset
```

## File Structure

**Path**: `~/.claude/feedback/ideas/YYYY-MM-DD.md`

**Header** (created automatically if new file):
```markdown
# Ideas - [Month Day, Year]

Capturing improvement ideas and feature requests.

---
```

**Then entries are appended** chronologically throughout the day.

## Idea Categories

**Automation**
- Scripts, hooks, automatic processes
- Reducing manual repetitive tasks

**Tool Development**
- New slash commands
- MCP integrations
- Utilities and helpers

**Workflow Enhancement**
- Process improvements
- Methodology refinements
- Presets and templates

**Documentation**
- Guides, checklists, references
- Knowledge capture

**Integration**
- Connecting systems
- API usage
- Third-party tools

**Experiment**
- Try something new
- Test hypothesis
- Proof of concept

**Other**
- Doesn't fit above categories

## Effort Estimation

**Quick** (< 2 hours):
- Single file creation
- Simple script
- Configuration tweak
- Copy/adapt existing pattern

**Medium** (2-8 hours):
- Multiple files/components
- Research required
- Testing needed
- Integration work

**Large** (> 8 hours):
- Significant new system
- Multiple dependencies
- Complex logic
- Extensive testing
- Documentation needed

## Priority Calculation

**High Priority**:
- High value + Quick effort = Quick win!
- High value + Medium effort = Worth the investment
- Frequent use case with significant impact

**Medium Priority**:
- Medium value + Quick/Medium effort
- High value but Low frequency
- Nice to have improvement

**Low Priority**:
- Low value (any effort)
- High effort for Medium value
- Rare use case
- Already have workarounds

## Implementation Notes

### Value Analysis

Consider:
- **Problem solved**: What pain point does this address?
- **Frequency**: How often would this be used?
- **Time saved**: Quantify if possible
- **Quality impact**: Better results or fewer errors?
- **Multiplier effect**: Benefits compound over time?

### Type Classification

Helps with:
- Grouping similar ideas
- Identifying patterns (lots of automation ideas = manual work problem)
- Resource allocation (what skills needed)
- Backlog organization

## Related Commands

- `/sc:log-win` - Log things that worked well
- `/sc:log-friction` - Log pain points (ideas often emerge from friction)
- `/sc:weekly-review` - Prioritize and plan implementation

## Ideas → Implementation Pipeline

### During Weekly Review
1. Review week's ideas
2. Combine with friction log (which ideas address friction?)
3. Prioritize top 1-2 ideas
4. Create action items or add to backlog

### Monthly Review
1. Look for idea clusters (common themes?)
2. Evaluate implemented ideas (did they deliver value?)
3. Archive low-priority ideas that no longer make sense
4. Celebrate ideas that became reality

## Success Criteria

**You're using this well when:**
- Capturing 2-5 ideas per week (ideas flow freely)
- Ideas are specific and actionable
- Ideas address real friction points
- Some ideas convert to implementations (not just collecting)
- Ideas file inspires you (exciting possibilities)

**Anti-patterns:**
- Never capturing ideas (they get lost)
- Vague ideas ("make things better")
- Only big ideas (small improvements count!)
- Collecting without implementing (idea graveyard)
- Not revisiting ideas (capture and forget)

## Example Idea Flow

**Week 1**: Friction → "Manual slash command creation takes 10min"
**Week 1**: Idea → "Create /sc:new-command scaffolding tool"
**Week 2**: Weekly review → Prioritize as high (quick win)
**Week 2**: Implementation → Build /sc:new-command
**Week 3**: Win → "New command scaffolding saved 30min this week"

This is the ideal flow: Friction → Idea → Action → Win

---

**Created**: 2025-11-01
**Part of**: Systematic feedback capture system
