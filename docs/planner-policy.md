# Planner Policy

This document describes the deterministic task-selection policy used by Kestrel's current daily
planner.

The canonical executable implementation is:

[`flows/build-kestrel-today.json`](../flows/build-kestrel-today.json)

This policy is intentionally deterministic. It does not currently use an LLM to decide which tasks
appear in the daily plan.

## Goals

The planner is designed to balance several competing needs:

- never hide work that is overdue or immediately due;
- respect task start dates as "do not surface before this date";
- give academic work enough lead time to avoid last-minute deadlines;
- keep important work visible even when its deadline is not immediate;
- prevent old, undated work from disappearing indefinitely;
- avoid allowing neglected low-priority tasks to become infinitely urgent merely because they are
  old;
- keep an ordinary daily plan small enough to remain useful.

The planner produces a ranked projection of work to consider today. It does not change the
authoritative Nextcloud tasks.

## Current task sources

The current planner reads two configured Nextcloud Tasks collections:

- **Personal**
- **Classes**

These names describe their role within Kestrel. The actual CalDAV collection names are configured
separately during installation.

Nextcloud remains the authoritative task store.

## Date semantics

Kestrel interprets task dates according to the user's configured `KESTREL_TIMEZONE`.

Planning is based primarily on calendar dates rather than time-of-day.

### Start date

A task's start date means:

> Do not surface this task before this date.

A task with a start date later than the planning date is therefore normally excluded from the daily
plan, even if its score would otherwise be high.

If a task has a start date later than its due date, Kestrel treats that as a date conflict rather
than silently hiding the task.

### Due date

A due date represents a real deadline.

Due dates affect both mandatory inclusion and discretionary scoring.

### Created date

When available, the task creation date is used to estimate age and, when no start date exists, the
amount of the task's available work window that has elapsed.

## Priority interpretation

Kestrel follows the iCalendar priority scale exposed by Nextcloud/Thunderbird.

| Raw priority | Kestrel band | Score contribution |
| --- | --- | ---: |
| 1–4 | High | 30 |
| 5 | Normal | 15 |
| 6–9 | Low | 0 |
| 0, missing, or invalid | Unspecified | 10 |

Priority affects discretionary ranking. Mandatory deadline rules take precedence over score.

## Mandatory inclusion

Mandatory work is always included, even when this causes the daily plan to exceed its normal target
size.

### Overdue tasks

Every overdue task is mandatory.

### General tasks

A non-Class task becomes mandatory when it is due on any of these three calendar dates:

- today;
- tomorrow;
- the day after tomorrow.

In implementation terms, the general mandatory window is **today + 2 days**.

### Class tasks

Class tasks receive a larger deadline window.

A Class task becomes mandatory when it is due:

- today; or
- within the next 6 days.

This is a total visibility window of **7 calendar dates including today**.

### Date conflicts

A task whose start date occurs after its due date is treated as mandatory so the inconsistent
metadata is visible to the user rather than causing the task to disappear.

## High-priority Class early attention

High-priority Class tasks can receive guaranteed early consideration before entering the mandatory
deadline window.

A task qualifies for **early attention** when all of the following are true:

- it belongs to Classes;
- its priority is High;
- it is not blocked by a future start date;
- it is not already mandatory;
- it is due within 21 days.

Early-attention tasks fill ordinary daily capacity but do not expand an already overloaded mandatory
day.

## Normal daily target

Kestrel targets approximately **6 actionable tasks** on an ordinary day.

This is a target, not a hard limit.

The planner:

1. includes all mandatory tasks;
2. fills remaining capacity with qualifying high-priority Class early-attention tasks;
3. fills any remaining capacity with the highest-scoring available tasks.

If there are already 6 or more mandatory tasks, Kestrel does not hide any of them merely to satisfy
the target.

## Discretionary score

Tasks that are not already mandatory compete for remaining daily capacity using a deterministic
score.

The current score is:

```text
priority
+ class weighting
+ deadline proximity
+ elapsed work window
+ age
````

The theoretical maximum is 105 points.

### Priority score

| Priority band | Points |
| ------------- | -----: |
| High          |     30 |
| Normal        |     15 |
| Unspecified   |     10 |
| Low           |      0 |

### Class weighting

A task in the Classes collection receives:

```text
+15
```

This reflects the additional scheduling pressure associated with academic deadlines.

### Deadline proximity

A task can receive up to:

```text
+25
```

for deadline proximity.

The score rises linearly during the final 30 days before the due date.

Conceptually:

```text
deadline score = 25 × (1 - remaining days / 30)
```

within the final 30 days.

A task due in 30 or more days receives little or no deadline contribution, while a task at its
deadline receives the full contribution.

Mandatory inclusion is handled separately, so this score does not determine whether immediately due
or overdue tasks are shown.

### Elapsed work window

A task can receive up to:

```text
+25
```

based on how much of its available work period has elapsed.

Kestrel uses:

```text
start date → due date
```

when a start date exists.

Otherwise it uses:

```text
created date → due date
```

For example, if a task became available 14 days ago and is due in 7 days, its total work window is
21 days and approximately two-thirds of that window has elapsed.

That increasing pressure raises the task's score even before the deadline becomes immediate.

### Age

A task can receive up to:

```text
+10
```

for age.

The contribution rises over the first 30 days and then saturates.

Conceptually:

```text
age score = min(age in days, 30) / 30 × 10
```

This lets neglected work gradually receive more attention without allowing a very old, low-priority
task to become arbitrarily urgent solely because of age.

## Ranking

### Mandatory work

Overdue tasks are placed before not-yet-due mandatory work.

Among overdue tasks, score is considered first so importance can influence ordering. Remaining
deadline age is used as a tie-breaker.

For upcoming mandatory work, the nearest deadline appears first.

### Discretionary work

Non-mandatory candidates are ordered by:

1. higher total score;
2. nearer due date;
3. greater age;
4. title as a stable final tie-breaker.

High-priority Class early-attention tasks and ordinary Focus tasks ultimately share this score-based
ordering after selection.

## Backlog review

Kestrel separately addresses old work that has no deadline.

When the normal plan is not already overloaded, it may append one **Backlog Review** item after the
actionable tasks.

A task qualifies for backlog review when it:

- is currently available;
- is not already selected;
- has no due date;
- is at least 30 days old.

The review task is not automatically declared urgent. Instead, the user is prompted to reconsider
it:

> Do it, reprioritize it, defer it, break it down, or remove it.

Kestrel selects one review candidate per calendar day. The choice remains stable throughout that day
and rotates across eligible tasks over time.

The review item is appended after normal actionable work.

On a typical day this means the projection can contain:

```text
6 actionable tasks
+ 1 backlog-review task
= 7 displayed items
```

No backlog-review item is added when the mandatory workload is already 6 or more tasks.

## Planner roles

Projected tasks currently receive one of four roles.

| Role              | Meaning                                               |
| ----------------- | ----------------------------------------------------- |
| `must_address`    | Mandatory because of deadline or inconsistent dates   |
| `early_attention` | High-priority Class work receiving advance visibility |
| `focus`           | Discretionary work selected by score                  |
| `backlog_review`  | Old, non-deadline work selected for reconsideration   |

These roles are included in the Google Tasks projection metadata so the user can understand why a
task was selected.

## Google Tasks projection

The planner output is projected into a Google Tasks list named:

```text
Kestrel Today
```

This list is not authoritative.

Kestrel may create, update, delete, and reorder its own managed projections in this list so that it
matches the current daily plan.

The canonical task remains in Nextcloud.

Kestrel identifies its managed Google projections using metadata stored in the task notes, including
the source Nextcloud UID.

Unmanaged Google tasks are ignored.

## Current limitations

The deterministic planner does not yet account for:

- estimated effort;
- lead-time requirements distinct from effort;
- task dependencies or blocked state;
- calendar workload;
- available free time;
- user energy or preferred work periods;
- project-level balancing;
- recurring-task semantics beyond the currently supported source collections;
- LLM-based judgment or task decomposition.

These are potential future planning inputs rather than assumptions hidden inside the current score.

## Future planner direction

Kestrel may eventually use a local LLM for judgment-oriented planning tasks such as:

- identifying tasks that are too large or vague;
- suggesting decomposition;
- estimating qualitative effort;
- detecting likely dependencies;
- recognizing stale tasks that need a decision;
- balancing work against calendar load;
- explaining tradeoffs between competing priorities.

The deterministic policy should remain the safety and scheduling foundation.

An LLM should augment that policy rather than silently replacing hard rules such as deadlines,
future start dates, permission boundaries, or authoritative-source semantics.
