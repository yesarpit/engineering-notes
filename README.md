# engineering-notes
Architecture decisions and design notes from backend/AI-agent systems I've built — the reasoning, not just the code.
# engineering-notes

Architecture decisions and design notes from backend/AI-agent systems I've built — the reasoning, not just the code.

These are written as ADRs (architecture decision records): context, the decision, why the obvious alternative was rejected, and consequences. The underlying projects are private (operational systems with real integrations), but the design reasoning is reusable and worth writing down properly.

## Index

- [ADR-001: Blended heartbeat + commit-recency staleness detection](adr/001-blended-staleness-detection.md) — how to decide something's gone stale when not every process reports the same way.
- - [ADR-002: An LLM reasoning layer may propose an action, never deliver one](adr/002-propose-approve-escalation-gate.md) — the approval gate between "the model decided" and "it actually happened."
  - - [ADR-003: Pass agent-generated instruction text as environment variables, never interpolate it into a shell command](adr/003-env-vars-not-string-interpolation.md) — treating LLM-originated text as hostile input at the shell boundary.
   
    - More added as the systems they come from evolve.
    - 
