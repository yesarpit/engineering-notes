# ADR-003: Pass agent-generated instruction text as environment variables, never interpolate it into a shell command

## Context

Part of the agent-orchestration layer needs to "wake" a target process by running
a configured shell command when an instruction is issued. The instruction text
itself is arbitrary — it can originate from an LLM, from a human typing into a
dashboard, or eventually from a chat message relayed through a third party like
Slack. None of those sources should be trusted as shell-safe input.

## Decision

The instruction text, project path, and project name are always passed to the
wake command as environment variables. They are never string-substituted into
the command itself. This means arbitrary instruction text — quotes, `$`,
backticks, semicolons, none of it — cannot break out of or inject into the shell
command being run, because it's never parsed as part of the command string in
the first place.

This was verified directly with adversarial instruction text (strings
specifically designed to break out of a naive `exec("cmd " + text)` pattern)
before this path was allowed to run unattended.

## Why this matters more here than in a typical form-input case

Standard input-sanitization advice assumes a human is typing into a form. Here,
the text being passed through can originate from an LLM's own output — and an
LLM can be steered, via prompt injection in whatever content it's summarizing,
into producing text that looks like a legitimate instruction but is actually
crafted to be dangerous once it reaches a shell. Because the agent layer here is
explicitly designed to let a model's output drive a real action (gated by
ADR-002), the shell-command boundary has to assume hostile input, not just messy
input.

## Consequences

- Any new delivery channel added later (a wake command, a webhook call, anything
-   that shells out) has to go through the same env-var contract, not whatever is
-     most convenient at the time — this is the kind of rule that's easy to quietly
-   violate under time pressure if it isn't written down.
-   - The equivalent discipline applies to the inbound side: endpoints that accept
    -   external input (e.g. a chat platform's event webhook) are verified against a
    -     signing secret before anything in the request is trusted, and requests older
    -   than a short replay window are rejected outright. Unset secrets fail closed
    -     (reject everything) rather than failing open.
    - 
