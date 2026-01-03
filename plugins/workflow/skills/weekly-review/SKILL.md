---
name: workflow-weekly-review
description: "Interactive weekly review of workflow, wins, friction, and ideas"
category: feedback
complexity: standard
---

# /sc:weekly-review - Weekly Workflow Review

> **Purpose**: Systematic weekly reflection to convert feedback into action and measure continuous improvement.

## Usage

```
/sc:weekly-review
```

## Behavior

When this command is invoked:

1. **Scan feedback files** from past 7 days
   - `~/.claude/feedback/wins/*.md`
   - `~/.claude/feedback/friction/*.md`
   - `~/.claude/feedback/ideas/*.md`

2. **Aggregate and analyze**
   - Count wins, friction, ideas by day
   - Identify top items in each category
   - Detect patterns and trends

3. **Present summary**
   - Stats overview
   - Top wins (celebrate!)
   - Top friction (needs addressing)
   - Top ideas (for prioritization)

4. **Interactive reflection**
   - Ask 5 key questions
   - Capture answers
   - Generate action items

5. **Save review**
   - Create `~/.claude/feedback/weekly/YYYY-WW.md`
   - Include all analysis and answers

6. **Generate action items**
   - Based on friction and ideas
   - Prioritized and specific

## Review Structure

### Part 1: Summary Stats (Auto-generated)

```markdown
# Weekly Review - Week 44, 2025

**Period**: Oct 28 - Nov 3, 2025

## Summary Stats
- Days active: 5
- Wins logged: 8
- Friction points: 4
- Ideas captured: 6
- Most active day: Wednesday (6 entries)
```

### Part 2: Top Wins (Celebrate!)

```markdown
## Top Wins 🎉

1. **Context monitoring prevented overflow 3 times** (Wed)
   - Why: Early warning gave time for strategic clearing
   - Keep doing: Act on warnings immediately

2. **New Incal Dreams theme reduced eye strain significantly** (Mon)
   - Why: Better contrast and lighter background
   - Keep doing: Prioritize ergonomics in tooling

3. **BMad checklist caught 3 issues before commit** (Tue)
   - Why: Systematic review process
   - Keep doing: Use checklist for non-trivial commits
```

### Part 3: Top Friction (Needs Attention)

```markdown
## Top Friction Points 😤

1. **Manual slash command scaffolding (3 files, 10min each)** (Priority: High)
   - Impact: Discourages creating helpful commands
   - Fix: Create /sc:new-command scaffolding tool
   - Status: Ready to implement

2. **Color contrast in terminal for git diffs** (Priority: Medium)
   - Impact: Strain to read diffs, slower review
   - Fix: Adjust ANSI colors in .wezterm.lua
   - Status: Partially addressed, needs validation

3. **No long-term productivity tracking** (Priority: Low)
   - Impact: Can't see trends or measure improvement
   - Fix: Persistent activity log from statusline
   - Status: Idea stage, not blocking
```

### Part 4: Top Ideas (For Prioritization)

```markdown
## Ideas Worth Pursuing 💡

**High Priority** (Quick wins or high value):
1. Create /sc:new-command scaffolding tool
   - Effort: Quick | Value: High | Type: Automation

**Medium Priority** (Valuable but more effort):
2. Auto-generate weekly stats from statusline
   - Effort: Medium | Value: Medium | Type: Metrics

**Low Priority** (Nice to have):
3. Add Python preset for ML hackathons
   - Effort: Medium | Value: Low (occasional use)
```

### Part 5: Reflection Questions (Interactive)

Claude asks these questions and captures your answers:

```markdown
## Reflection

### Q1: Which improvement had the biggest impact this week?

**Your answer**: New theme - way less eye strain during long sessions

**Analysis**: Environment optimization paying off. Continue prioritizing ergonomics.

---

### Q2: What was your biggest friction point this week?

**Your answer**: Manual command scaffolding every time I want to add a slash command

**Analysis**: Clear automation opportunity. High frequency, measurable time waste.

---

### Q3: What's one thing you'll change or implement next week?

**Your answer**: Build /sc:new-command to automate scaffolding

**Analysis**: Addresses #2 friction point, quick win with recurring benefit.

---

### Q4: Any wins worth celebrating or sharing?

**Your answer**: Context monitoring actually prevented me from losing important context 3 times!

**Analysis**: Systematic improvements working as designed. Pattern worth reinforcing.

---

### Q5: Any ideas you want to prioritize for implementation?

**Your answer**: Yes - the command scaffolding tool. It's a quick win that removes daily friction.

**Analysis**: Alignment between friction and priority. Good candidate for next week.
```

### Part 6: Action Items (Generated)

```markdown
## Action Items for Next Week

### High Priority (Do These)
- [ ] Build /sc:new-command scaffolding tool
  - Addresses: Friction #1 (manual command creation)
  - Estimated effort: 2-3 hours
  - Expected benefit: Save 10min per command, remove barrier

- [ ] Validate all terminal colors with actual usage
  - Addresses: Friction #2 (git diff readability)
  - Estimated effort: 30min
  - Expected benefit: Easier code review, less strain

### Medium Priority (If Time)
- [ ] Document "Incal Dreams" theme impact for future reference
  - Addresses: Win #2 (preserve what worked)
  - Estimated effort: 15min
  - Expected benefit: Reference for future optimizations

### Backlog (Track but Don't Start)
- [ ] Design persistent activity logging system
  - Addresses: Idea #2 (long-term tracking)
  - Estimated effort: 4-6 hours
  - Expected benefit: Trend analysis, productivity insights

---

## Patterns & Insights

**Emerging patterns**:
- Environment/tooling improvements having big ergonomic impact
- Automation ideas clustering around command/workflow management
- Context management systems proving valuable

**What's working**:
- Systematic feedback capture (8 wins, 4 friction, 6 ideas)
- Acting on improvements (theme, monitoring)
- Reinforcing wins through recognition

**Areas for growth**:
- Convert more friction to action (only 50% addressed this week)
- Implement quick wins faster (ideas sit too long)
- More granular time tracking for better effort estimates

---

**Review completed**: 2025-11-03 16:30
**Next review**: 2025-11-10
```

## Implementation Notes

### File Scanning Logic

```
For past 7 days (today - 6 to today):
  - Check wins/YYYY-MM-DD.md
  - Check friction/YYYY-MM-DD.md
  - Check ideas/YYYY-MM-DD.md
  - Count entries per day
  - Extract all entries for analysis
```

### Priority Scoring

**For friction**:
- Mentioned multiple times = Higher priority
- High impact + frequent = High priority
- Has potential fix = More actionable

**For ideas**:
- Quick effort + High value = High priority
- Addresses friction = Higher priority
- Mentioned multiple times = Rising priority

### Question Customization

Adapt questions based on what was captured:
- If no wins: "What prevented wins this week?"
- If lots of friction: Focus on top 2, not overwhelming
- If no ideas: "What would make next week better?"

### Action Item Generation

Should be:
- **Specific** ("Build /sc:new-command") not ("Improve workflow")
- **Linked** to friction/ideas being addressed
- **Estimated** for effort and benefit
- **Prioritized** clearly (High/Medium/Backlog)
- **Actionable** (can start immediately)

## Related Commands

- `/sc:log-win` - Capture wins during the week
- `/sc:log-friction` - Capture friction during the week
- `/sc:log-idea` - Capture ideas during the week

## Weekly Review Best Practices

### Timing

**Best time**: Sunday afternoon/evening
- Week is complete
- Fresh perspective
- Time to plan next week
- Not rushed

**Duration**: 10-15 minutes
- Don't overthink
- Answer instinctively
- Focus on top items

### Mindset

**Do**:
- Be honest (especially about friction)
- Celebrate wins (even small ones)
- Prioritize ruthlessly (can't do everything)
- Generate specific actions

**Don't**:
- Beat yourself up (retrospective, not blame)
- Over-analyze (patterns > perfection)
- Create impossible action items
- Skip if busy (consistency > completeness)

### Evolution

**First 4 weeks**: Establish habit
- Just do it weekly
- Get comfortable with format
- Don't optimize yet

**Months 2-3**: Refine based on usage
- Adjust questions if not valuable
- Add/remove stats
- Tune action item generation

**Ongoing**: Systematic improvement
- Review monthly patterns
- Measure friction reduction
- Track idea conversion rate
- Adapt to changing needs

## Success Criteria

**Working well when**:
- Takes 10-15 minutes consistently
- You look forward to it (positive ritual)
- Action items actually get done
- Friction log shrinks over time
- Wins log grows
- Ideas convert to implementations

**Needs adjustment when**:
- Skipping regularly (too long? not valuable?)
- Action items ignored (too many? not specific?)
- Same friction every week (not addressing root cause)
- No wins (too high bar? not capturing?)

---

**Created**: 2025-11-01
**Part of**: Systematic feedback capture system
