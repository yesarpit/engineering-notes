# ADR-001: Blended heartbeat + commit-recency staleness detection

## Context

A monitoring layer sits in front of several kinds of long-running work on the same
machine: agent loops, background daemons, and cron jobs. They don't all report in
the same way. An agent loop can emit a heartbeat on every turn. A cron job that
runs once a day and exits cleanly never will. Treating "no heartbeat" as
automatically stale flagged daemons and cron jobs as broken even while they were
working exactly as designed.

## Decision

Staleness is computed uniformly for every registered project, from whichever of
two signals is more recent:

- **Heartbeat cadence** — if the project reports a heartbeat, staleness is judged
-   against its own declared check-in interval (one missed check-in is grace, a
-     second is treated as a stall).
- - **Commit recency** — if there's a git repo but no heartbeat, staleness falls
  -   back to a looser, fixed window on the last commit timestamp, since commits
  -     aren't a heartbeat cadence and a quiet daemon between commits is normal.
 
  - A project only reads as stalled once *both* applicable signals are past their
  - window. Once a project has sent at least one heartbeat, a commit newer than that
  - heartbeat is judged by the heartbeat cadence instead — so an agent that commits
  - and then dies still reads as stalled after its own missed check-ins, rather than
  - hiding behind a fresh commit timestamp.
 
  - ## Why not just "require a heartbeat from everything"
 
  - It would be simpler to mandate heartbeats everywhere, but that pushes integration
  - cost onto every daemon and cron job for a signal that doesn't match how they
  - actually run, and silently misclassifies correctly-behaving processes as broken.
  - The uniform, signal-agnostic check avoided that without needing a separate
  - code path per project type.
 
  - ## Consequences
 
  - - No facet/tag on a project gates which checks run — checks run based on what's
    -   actually detected (a `.git` folder, a heartbeat ever having been seen), so
    -     mislabeling a project doesn't silently skip monitoring it.
    - - A goal-tracking escape hatch is needed: a project that finished its goal and
      -   intentionally went quiet should not be treated as failing. A manual/self-reported
      -     "done" status short-circuits the staleness check entirely.
      - - This is still an indirect liveness proxy, not a direct process/port probe —
        -   that's a deliberate scope cut for a later phase, not an oversight.
        -   
