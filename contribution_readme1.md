# Contribution 1: DatePicker — typing a date in "YYYY-MM-DD" format shifts it one day behind

**Contribution Number:** 1  
**Student:** Prabin Bajgai  
**Issue:** https://github.com/angular/components/issues/27412  
**Status:** Phase II Complete

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

Typing a date like `2023-07-06` into a Material datepicker and clicking away turns it into
`7/5/2023`, a day early. It only happens when your timezone is behind UTC. The cause is that two
parts of `NativeDateAdapter` read the date in different timezones: `parse()` reads the string as UTC,
and `format()` later reads it back by its local calendar day. Behind UTC those two readings are off
by the offset, so the day drops by one.

### Expected Behavior

Type `2023-07-06`, get `7/6/2023`. Same day, just reformatted.

### Current Behavior

Type `2023-07-06`, get `7/5/2023` (a day behind), but only in behind-UTC timezones. Zones ahead of
UTC like Europe/Brussels show the right day.

### Affected Components

- `src/material/core/datetime/native-date-adapter.ts`. `parse()` is where the bug is, and
  `_format()` is the formatting code it conflicts with.
- `src/material/datepicker/datepicker-input-base.ts`. The `_onInput` → `_onBlur` → `_formatValue`
  chain that calls `parse()` and then `format()`.

---

## Reproduction Process

### Environment Setup

Cloned my fork (Bazel monorepo, datepicker demo runs with `yarn dev-app`). The thing that tripped me
up is that the bug depends on your timezone, so it won't show up on a normal machine that's ahead of
UTC. I got a behind-UTC zone two ways: by running the logic under Node with the `TZ` variable set,
which works no matter what my laptop's clock says, and by overriding the browser timezone in Chrome
DevTools. Worth noting that the date math runs in the browser, not the Node devserver, so a `TZ=`
prefix on `yarn dev-app` doesn't change anything on its own.

### Steps to Reproduce

1. Run `yarn dev-app` and open the datepicker overview demo.
2. Set the browser timezone to a behind-UTC zone (DevTools, "Show Sensors", America/Chicago).
3. Type `2023-07-06` into the input and click away.
4. Expected `7/6/2023`, but you get `7/5/2023`.

If you'd rather skip the browser settings, this runs the same `parse()` and `_format()` logic:

```bash
TZ="America/Chicago" node -e '
  function parse(v){ return v ? new Date(Date.parse(v)) : null; }
  function format(date){
    const dtf = new Intl.DateTimeFormat("en-US",{year:"numeric",month:"numeric",day:"numeric",timeZone:"utc"});
    const d = new Date();
    d.setUTCFullYear(date.getFullYear(), date.getMonth(), date.getDate());
    d.setUTCHours(date.getHours(), date.getMinutes(), date.getSeconds(), date.getMilliseconds());
    return dtf.format(d);
  }
  console.log(format(parse("2023-07-06")));   // -> 7/5/2023 (bug); Europe/Brussels -> 7/6/2023
'
```

### Reproduction Evidence

- **Branch:** https://github.com/prabinb19/components/tree/fix-issue-27412
- **What I found:** `Date.parse("2023-07-06")` gives UTC midnight, which in GMT-5 is `Jul 05 19:00`
  on the local clock. I traced the path the value takes: `parse()`, then `getValidDateOrNull()`
  which only validates and doesn't change the date, then `format()`, then `_format()`. Nothing in
  there converts back to local time. `_format()` reads the local getters and copies them onto a UTC
  instant, so whatever local day the date has is the day you see. Since `parse` produced a date whose
  local day is the 5th, that's what shows up. The two methods just aren't using the same timezone.
  Fuller notes and a multi-timezone check are in `phase2_reproduce_and_plan.md`.

---

## Solution Approach

### Analysis

JavaScript has an awkward rule here. A bare date string like `2023-07-06` is parsed as UTC midnight,
but a date with a time and no offset is parsed as local. `parse()` hands the string straight to
`Date.parse()`, so it picks up the UTC reading. The rest of the adapter works in local time, which
leaves this one value as the odd one out, and behind UTC it reads back as yesterday.

### Proposed Solution

Get `parse()` working in local time like everything else in the adapter. When the input is a plain
date with no time, build it from its pieces (`new Date(year, month - 1, day)`) instead of calling
`Date.parse()`. Then the local day always matches what was typed, in any timezone, and `_format()`
shows it unchanged. I want to keep this small, so only bare `YYYY-MM-DD` strings take the new path.
Anything with a time or an explicit `Z`/offset still goes through `Date.parse()` since those already
say what timezone they mean. Two older PRs (#27495 and #27835) tried a larger locale-aware rewrite; I
don't want to go that far, since a maintainer noted the adapter is meant to stay simple.

### Implementation Plan

Using UMPIRE framework (adapted):

**Understand:** `2023-07-06` shows up as `7/5/2023` in behind-UTC zones because `parse()` reads the
string as UTC midnight while `format()` reads dates by their local day.

**Match:** `deserialize()` already checks strings against `ISO_8601_REGEX` before building a `Date`,
so there's precedent for testing the string's shape first. `_createDateWithOverflow()` is a good
reference for the local-component style used elsewhere in the file.

**Plan:**
1. Add a date-only check (`^\d{4}-\d{2}-\d{2}$`) inside `parse()` in `native-date-adapter.ts`.
2. If it matches, split out year/month/day and build the date locally. If not, leave the existing
   `Date.parse()` path alone.
3. Add tests in `native-date-adapter.spec.ts`.

**Implement:** https://github.com/prabinb19/components/tree/fix-issue-27412 *(Phase III)*

**Review:** Read `CONTRIBUTING.md` and the Google TypeScript style guide before the PR, and follow
the repo's commit format (`type(scope): summary`, e.g. `fix(material/core): parse date-only ISO
strings in local time`). Keep the change small.

**Evaluate:** Unit tests in `native-date-adapter.spec.ts`; re-run the Node check across
Chicago/Brussels/Kolkata/Kiritimati to confirm `2023-07-06` gives `7/6/2023` everywhere without
breaking the zones that already worked; make sure the existing `deserialize` ISO tests still pass.

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
