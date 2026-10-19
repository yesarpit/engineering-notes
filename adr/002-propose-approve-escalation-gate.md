# ADR-002: An LLM reasoning layer may propose an action, never deliver one

## Context

Several systems I've built let an LLM reason over live operational state (project
status, blockers, history) and decide what should happen next. Once that
reasoning layer exists, it's tempting to let it also act — deliver the
instruction, send the message, make the call. That's also the point where a
model's reasoning error stops being a bad suggestion and starts being a real
consequence.

## Decision

The reasoning/"decide what to do" step and the delivery/"make it happen" step are
always two separate, independently gated operations:

1. A **dry-run by default.** Running the reasoning loop without an explicit
2.    `--apply`/approval flag only prints and logs what it would do — nothing is
3.   sent. This is true for scheduled/batch review runs and for the interactive
4.      chat surface alike: a chat reply can draft a proposal, but a reply by itself
5.     is never sufficient to move anything downstream.
6. 2. **Explicit approval delivers through the same path a human would use.**
   3.    Approving a proposal goes through the identical code path as a person typing
   4.   the instruction in by hand — there is no separate, higher-trust path for
   5.      LLM-originated actions.
   6.  3. **Hard constraints are enforced outside the model's judgment**, not just
       4.    prompted for. Anything touching money, or any real-world irreversible step,
       5.   is excluded from what the reasoning layer is even allowed to decide —
       6.      it comes back as an explicit "hold, here's what approval is needed," never a
       7.     silent no-op and never an autonomous yes.
     
       8. ## Why not trust the model's own self-restraint
     
       9. Prompting a model to "ask before anything risky" is necessary but not
       10. sufficient — it's a request, not a guarantee, and it degrades under edge cases
       11. and prompt drift in ways a hard gate in code does not. Treating the approval
       12. gate as application logic rather than model behavior means the worst case for a
       13. bad model decision is "a human has to dismiss a bad proposal," not "a bad
       14. decision executed."
     
       15. ## Consequences
     
       16. - Every delivered action has a human-in-the-loop moment on record — proposal,
           -   decision, delivery outcome — which is also what makes the system auditable
           -     after the fact, not just in principle.
           - - The reasoning layer is only as fast as the human loop reviewing it for
             -   anything not pre-approved by policy, which is an intentional tradeoff of
             -     velocity for safety on anything with real stakes attached.
             - - Scoping which businesses/projects the reasoning layer is even allowed to see
               -   is itself an opt-in decision, made once, rather than a default-on behavior.
               -   
