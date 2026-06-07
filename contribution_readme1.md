# Contribution 1: DatePicker — typing a date in "YYYY-MM-DD" format shifts it one day behind

**Contribution Number:** 1  
**Student:** Prabin Bajgai  
**Issue:** https://github.com/angular/components/issues/27412  
**Status:** Phase I Complete

---

## Why I Chose This Issue

In Angular Material's `NativeDateAdapter`, typing a date like `2023-07-06` into the datepicker
input and blurring it formats the value to `7/5/2023` — one day behind. The bug only shows up in
negative-offset (behind-UTC) timezones. It matters because date-entry is one of the most common
form interactions, and silently being off by a day is the kind of subtle data-corruption bug that
slips into production and is painful to debug later.

I chose it because the scope is small and well-bounded (a date-parsing edge case in one adapter,
not a redesign), the root cause is already well documented in the issue thread, and it's a great
way to learn about a classic real-world gotcha: JavaScript's `Date` parses date-only ISO strings
(`YYYY-MM-DD`) as UTC midnight, so converting back to local time in a behind-UTC zone rolls the
date back a day. It's labeled `help wanted` / `P3` and uses TypeScript, which I'm comfortable with.

**Note for Phase II:** Two earlier PRs (#27495 and #27835 by `wartab`) attempted a fix but appear
to have stalled (last activity ~Mar 2024). I'll review them in Phase II to learn from the approach
taken and confirm the issue is still genuinely open before investing further.

---

## Understanding the Issue

### Problem Description

[In your own words, what's broken or missing?]

### Expected Behavior

[What should happen?]

### Current Behavior

[What actually happens?]

### Affected Components

[Which parts of the codebase are involved?]

---

## Reproduction Process

### Environment Setup

[Notes on setting up your local development environment - challenges you faced, how you solved them]

### Steps to Reproduce

1. [Step 1]
2. [Step 2]
3. [Observed result]

### Reproduction Evidence

- **Commit showing reproduction:** [Link to commit in your fork]
- **Screenshots/logs:** [If applicable]
- **My findings:** [What you discovered during reproduction]

---

## Solution Approach

### Analysis

[Your analysis of the root cause - what's causing the issue?]

### Proposed Solution

[High-level description of your fix approach]

### Implementation Plan

Using UMPIRE framework (adapted):

**Understand:** [Restate the problem]

**Match:** [What similar patterns/solutions exist in the codebase?]

**Plan:** [Step-by-step implementation plan]
1. [Modify file X to do Y]
2. [Add function Z]
3. [Update tests]

**Implement:** [Link to your branch/commits as you work]

**Review:** [Self-review checklist - does it follow the project's contribution guidelines?]

**Evaluate:** [How will you verify it works?]

---

## Testing Strategy

### Unit Tests

- [ ] Test case 1: [Description]
- [ ] Test case 2: [Description]
- [ ] Test case 3: [Description]

### Integration Tests

- [ ] Integration scenario 1
- [ ] Integration scenario 2

### Manual Testing

[What you tested manually and results]

---

## Implementation Notes

### Week [X] Progress

[What you built this week, challenges faced, decisions made]

### Week [Y] Progress

[Continue documenting as you work]

### Code Changes

- **Files modified:** [List]
- **Key commits:** [Links to important commits]
- **Approach decisions:** [Why you chose certain approaches]

---

## Pull Request

**PR Link:** [GitHub PR URL when submitted]

**PR Description:** [Draft or final PR description - much of the content above can be adapted]

**Maintainer Feedback:**
- [Date]: [Summary of feedback received]
- [Date]: [How you addressed it]

**Status:** [Awaiting review / Iterating / Approved / Merged]

---

## Learnings & Reflections

### Technical Skills Gained

[What you learned technically]

### Challenges Overcome

[What was hard and how you solved it]

### What I'd Do Differently Next Time

[Reflection on your process]

---

## Resources Used

- [Link to helpful documentation]
- [Tutorial or Stack Overflow post that helped]
- [GitHub issues or discussions that helped]
