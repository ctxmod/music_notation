# Writing progress-tracker snapshots

Progress snapshots are condensed carry-forward files; record mastery only when proven by reasoning, not by a correct label.

Snapshot files are condensed records of the learner's musical understanding, one per session-day (`YYYY-MM-DD.md`), kept in the tracker location the consuming repository names.

## Principles

- **Carry-forward snapshot, not a log.** Start a session's file from the most recent snapshot, then revise: promote, demote, correct, prune. The latest file alone is the authoritative current picture; older files are history.
- **Condensed.** One line per bullet; the whole file readable in under a minute.
- **Four sections:** Deep mastery, Weak / shaky, Misconceptions to correct, Not yet assessed.
- **The mastery bar: think critically.** A correct label is not evidence of understanding; it may be a guess or rote. Record under Deep mastery only when the learner explained *why*, heard it in a real passage, handled an edge case or a deliberate trap, connected it to neighbouring ideas, and reconstructed rather than recited. When in doubt, log it as Weak: under-claiming is safe; over-claiming makes the next session skip a real gap. Periodically re-verify old entries and demote any that no longer hold.

## Template

```markdown
---
type: tutor-progress-tracker
date: YYYY-MM-DD
carried-from: YYYY-MM-DD   # previous snapshot forked, or null
---

# Tutor Progress Tracker — YYYY-MM-DD

Condensed snapshot of current understanding. Carried forward and revised each session; not a complete log.

## Deep mastery (probed, not just answered)
- <concept> — <one-line evidence of deep understanding> (YYYY-MM-DD)

## Weak / shaky (revisit and re-probe)
- <concept> — <what is missing or hesitant> (YYYY-MM-DD)

## Misconceptions to correct (keep until re-verified)
- <wrong belief> → <the correct idea> (YYYY-MM-DD)

## Not yet assessed
- <areas not yet probed>
```
