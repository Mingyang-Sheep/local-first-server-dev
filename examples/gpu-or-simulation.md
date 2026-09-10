# Example: GPU training or simulation

This example illustrates the workflow; it is not a technology-specific command guide.

## Request

> Change the observation and reward definitions in a robot learning project. The real environment can only be created on a GPU server with the project's simulator installed.

## Local phase

The agent should inspect the local repository and trace the relevant path, such as:

```text
configuration
    -> observation construction
    -> environment interface
    -> policy input
    -> reward aggregation
    -> training entry point
```

It should implement the change locally, update affected configuration and unit tests, and add narrowly useful diagnostics such as shape, finite-value, or registration checks when justified.

Possible evidence classification:

```text
Executed locally:
- Configuration parsing passed.
- Observation-builder unit tests passed.
- Reward arithmetic tests passed.

Static review only:
- The policy input width matches the updated configuration path.

Target validation required:
- Simulator environment creation: simulator runtime and GPU are unavailable locally.
- Short vectorized rollout: authoritative simulator execution is required.
- Full training: expensive GPU workload.
```

The agent must not install a complete simulator or claim that a mock environment proves simulator integration.

## Human handoff

The real commands must come from the project. If required values are unknown, ask for only those values at handoff time. A clearly marked template might look like this:

```bash
# TEMPLATE — replace every <...> value with verified project information.
cd <SERVER_PROJECT_DIR>
<ACTIVATE_EXISTING_ENVIRONMENT>

# Confirm the intended revision or patch is present.
git rev-parse HEAD

# Low-cost preflight using the project's real entry point.
<PROJECT_PREFLIGHT_COMMAND_WITH_A_SMALL_ENV_COUNT_AND_SHORT_RUN>

# Run only after preflight satisfies the checks below.
<PROJECT_FORMAL_TRAINING_COMMAND>
```

Expected preflight evidence should be concrete, for example:

- the environment is created successfully;
- observation and action shapes match the intended contract;
- the first steps contain no NaN or infinite values;
- no registration, device, or configuration error appears.

If preflight fails, return the revision identity, exact command, full traceback when reasonably sized, the focused surrounding log section, relevant framework and GPU/runtime versions, and printed shape or finite-value diagnostics. Remove tokens, credentials, dataset paths, and other sensitive information.

The user should not need to modify reward or observation source code on the server.
