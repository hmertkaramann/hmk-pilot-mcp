---
name: navisworks-wf-timeliner
description: TimeLiner 4D — tasks, attachment rules, dates and progress. Requires the HMK Pilot connector for Navisworks.
---

## The model

TASKS carry planned and actual start/end dates and a TASK TYPE (Construct,
Demolish, Temporary). A task drives geometry only once it is ATTACHED to
items — usually via a SET, occasionally to items directly.

Attaching to a SEARCH set is what makes a 4D model survive model updates: the
set re-evaluates, so newly added items join the right task automatically.
Attaching to a selection set freezes it. Prefer search sets and say which you
used.

## Recipe — build a sequence

1. Make (or identify) the sets that carve the model into work packages.
2. Create tasks with the dates and task types.
3. Attach each task to its set — `auto_attach_tasks_by_rule` when the naming
   is systematic, explicit links otherwise.
4. Recalculate dates and report the resulting span.

## Reporting

Give the span and the task count, then the shape of the schedule — not a
50-row dump:

> "42 görev, 12.03.2026 – 08.11.2026. En uzun kalem 'Kaba İnşaat' (94 gün).
> 6 görevin bağlı seti yok."

That last number matters more than the rest: an unattached task is a task that
drives nothing, and it is the most common defect in a handed-over 4D model.
Always count and report it.

## Progress

Actual dates versus planned is what a progress snapshot means. When the user
asks "nerede kaldık", compare the two and report the delta in days, per work
package — not per task.
