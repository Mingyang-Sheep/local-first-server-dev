# Example: feedback from a protected target

This example shows how returned server evidence remains part of the same engineering task even if the conversation or model instance changes.

## First local revision

Assume the agent completed a local change, ran meaningful local checks, and produced a human handoff containing:

- the exact commit, patch, or changed file set;
- a revision verification step;
- one low-cost target preflight;
- observable success conditions;
- the minimal evidence to return after failure.

The agent did not connect to or operate the protected target.

## Returned result

The user returns:

```text
Revision: <REVISION_ID>
Command: <EXACT_PREFLIGHT_COMMAND>
Environment: <RELEVANT_VERSIONS>
Result: failed
Traceback or focused log: <SANITIZED_EVIDENCE>
```

## Next local iteration

The agent should:

1. confirm that the reported revision matches the intended local change;
2. map the failure to the changed code, configuration, and affected call path;
3. distinguish a code defect from an environment or synchronization problem;
4. implement the next fix in the local repository;
5. rerun relevant local checks;
6. replace the old target handoff with the smallest corrected procedure.

It should not ask the user to open an editor on the server, add temporary source fixes there, or conduct broad exploratory debugging. If another target observation is needed, request the smallest diagnostic that discriminates between the remaining hypotheses.

If prior conversation context is unavailable, reconstruct continuity from the local diff or revision, the exact target command, and the returned evidence. Do not assume that the same AI instance is required.

## Example completion summary

```text
Local Completed:
- Corrected the configuration-to-runtime field mapping.
- Added a regression test for the returned failure.

Local Validation:
- Regression test passed.
- Existing focused unit suite passed.

Remaining Validation:
- The corrected configuration still requires the real target runtime.

Server / Target Steps:
1. Apply <NEW_REVISION>.
2. Verify the revision.
3. Re-run the same low-cost preflight.

Expected Result:
- The runtime accepts the configuration and reaches the first functional check.

Return on Failure:
- New revision, exact command, focused sanitized error, and relevant version output.
```
