---
name: local-first-server-dev
description: Develop and validate code in an authorized local workspace when authoritative builds, GPU or simulator runs, hardware checks, Linux-only execution, or other final validation must be performed by a human on a restricted server or target device. Use when development and final execution environments are separated; do not trigger merely because a project mentions Linux, CUDA, ROS, or remote systems.
---

# Local-First Server Development

Move every reliable development and validation task that does not require the real target environment into the authorized local workspace. Leave only irreducible environment-dependent validation for a human to execute on the protected server or target.

This workflow does not depend on multi-agent orchestration. A host may use internal sub-tasks or reviewers, but do not require the user to configure multiple agents or perform agent-to-agent handoffs.

## Preserve four invariants

1. **Local first:** analyze, implement, test, and prepare diagnostics locally whenever the result can be trusted without the target environment.
2. **Evidence, not assumption:** distinguish executed checks from static reasoning and from checks that remain unverified.
3. **Human-gated target:** prepare protected-target operations, but never perform them.
4. **Minimize target burden:** reduce target-side work to applying the prepared change, running focused checks, starting the real workload when appropriate, and collecting results.

Do not turn this into a remote execution, environment replication, deployment automation, technology cookbook, or multi-agent workflow.

## Define the boundary by effect

Treat a server, device, production system, remote shell, GPU or HPC environment, robot, embedded target, real hardware, or any environment designated by the user as human-controlled as a **protected target**.

The boundary depends on where data and commands take effect, not on how a path looks. Remote SSH workspaces, SSHFS, SMB/NFS mounts, mapped drives, and similar views of a protected target remain protected-target access.

Before writing, building, testing, installing, or launching anything, establish from the available context that the current workspace is an authorized local copy. Do not assume that Windows means local or that Linux means remote.

If the workspace is confirmed to be on a protected target or its remote mount:

- stop modifying files and do not build, test, install dependencies, create patches there, or launch programs;
- stop inspecting protected project files beyond the minimal read-only information needed to establish the boundary;
- explain that this workflow requires a genuine local repository copy and ask the user to provide one;
- continue only with non-mutating guidance based on information the user has already supplied.

If the location remains ambiguous and that ambiguity affects the safety boundary, inspect only existing read-only environment indicators, then ask one minimal confirmation question if necessary.

Do not use an indirect action to cross the gate. Ordinary commits, pushes, pull requests, or releases follow the host's permissions and the user's authorization, but first consider their downstream effects. If an external write would directly or indirectly trigger protected-target deployment, training, production release, hardware execution, or another gated action, do not perform it. Present it as a human step. Do not assume that a human initiating CI or another automation automatically satisfies the target policy; use only a trigger path the user has identified as permitted. If the effect or permitted trigger path is materially uncertain, ask the minimum question needed to resolve it.

## Work locally first

### 1. Understand the task and repository

Start from the local repository. Read the relevant project instructions, source, configuration, tests, build files, entry points, and existing operational documentation. Trace the affected call and configuration paths far enough to identify the intended change and plausible regressions.

Do not ask for server paths, scheduler settings, or target credentials while they do not block local work. If required source or generated artifacts exist only on the protected target, ask the user to bring the minimum non-sensitive material into the local workspace rather than accessing it remotely.

### 2. Complete the engineering locally

Perform locally all work that does not require authoritative target conditions, including implementation, refactoring, configuration changes, interface adjustments, logs, unit tests, mocks, and small diagnostic helpers when the task justifies them.

Use repository-native tools first, then existing local environments, then lightweight isolated checks. Existing WSL, containers, virtual environments, and toolchains may be used when they are already available and appropriate. Do not create a replacement for the target environment merely to claim complete local execution.

Installing a validation dependency never follows from this skill alone. Do so only when the host permissions and user authorization allow it, the source and benefit are clear, and the change is low-risk and preferably project-scoped or temporary. Avoid global installation, unnecessary lockfile changes, large images, new operating-system environments, drivers, simulators, complete robotics stacks, and large SDKs.

Use mocks only to validate locally owned logic or contracts. Never present a mock as evidence that the real simulator, driver, network, device, GPU, or SDK works.

### 3. Maximize reliable local validation

Choose checks in proportion to the change. Prefer existing syntax checks, linters, type checks, configuration parsing, unit tests, interface checks, shape or dimension checks, mock tests, build-system checks, and module-level runs that can execute reliably in the local environment.

Record evidence in these categories:

- **Executed locally:** the relevant command or check and its observed pass or failure result.
- **Static review only:** a conclusion supported by source or configuration analysis but not by execution.
- **Not run locally:** a locally possible check that was skipped, with the concrete reason.
- **Target validation required:** a check that genuinely requires the protected target, naming the missing condition.

Do not classify a check as target-only merely because its first local attempt failed. Determine whether the failure is a code defect, a local setup issue, or a genuinely unavailable target dependency. Never convert static reasoning or mock success into runtime PASS evidence.

## Pass the server-readiness gate

Do not hand the task to the protected target until the following are true to the extent relevant:

- the main analysis and implementation are complete;
- reasonable locally available checks have run, and any omission has a stated reason;
- remaining checks genuinely require the authoritative environment;
- the human should not need to inspect, design, or edit source code on the target;
- a low-cost preflight can catch likely integration problems before an expensive run;
- success is observable and failure will return enough evidence for local diagnosis;
- the exact revision, patch, or file set to validate can be identified.

If target-side source edits, configuration discovery, temporary scripting, or complex debugging are still expected, continue preparing locally instead of handing off prematurely.

## Prepare the human handoff

Never establish SSH or another remote session, access or mutate the protected filesystem, upload through SCP or rsync, run a remote shell, modify the target environment, control jobs, operate checkpoints, or start or stop target processes. This remains true even when a tool technically makes the operation possible. If the user wants agent-controlled remote development, explain that it is a different workflow rather than weakening this skill's boundary.

Generate the shortest safe human procedure that fits the repository:

1. identify and apply or synchronize the prepared revision, patch, or files;
2. confirm the expected revision or change is present;
3. enter the known working directory and activate the existing environment;
4. run the cheapest meaningful preflight;
5. run the expensive build, simulation, training, launch, or hardware test only after preflight passes;
6. collect the smallest diagnostic set needed for another local iteration.

Base commands on local repository documentation, scripts, and configuration. Do not invent paths, environment names, scheduler partitions, accounts, device IDs, checkpoint locations, task names, image tags, or credentials. If missing information does not block local work, finish local work first. When the user needs a directly executable handoff and a required value cannot be derived, ask for the minimum missing information at handoff time. Use conspicuous placeholders only for an explicitly acceptable template, an example, or provisional guidance, and never present a template as a final executable handoff.

Make commands ordered and directly copyable when the required facts are known. Separate low-cost preflight from high-cost execution. State the expected observable result for each key step and what must be returned after failure. Include the code revision, invoked command, relevant traceback or focused log section, and environment versions when they affect diagnosis. Ask the user to remove credentials, tokens, private data, and other secrets before returning logs.

Prefer non-destructive, repeatable checks. Do not suggest resets, cleans, overwrites, deletion of checkpoints or data, or other destructive target operations unless the user explicitly requests them and their scope is exact.

Keep the handoff in the response by default. Create `SERVER_HANDOFF.md` only when the user requests a persistent handoff or the current task explicitly requires a shareable artifact; do not add it merely because the command list is long.

## Continue from returned evidence

Treat returned target logs and results as feedback on the previous local revision, not as an unrelated task. Confirm which revision, patch, command, and environment produced the result. Compare the evidence with the preceding change, diagnose locally, make the next local revision, rerun meaningful local checks, and replace the previous handoff with a smaller corrected one.

Do not rely on the same model instance or uninterrupted conversation history. Reconstruct continuity from the local diff or revision, the executed command, and the returned evidence when necessary.

## Use the completion and handoff contract

Use this contract when an engineering task is complete or the local phase is ready for protected-target validation. Do not force it into ordinary discussion, design analysis, status updates, or conceptual explanations.

Cover these outcomes, using headings only when they improve clarity:

1. **Local Completed:** what changed locally.
2. **Local Validation:** executed results, static-only conclusions, and locally skipped checks with reasons.
3. **Remaining Validation:** only the checks that still require the protected target.
4. **Server / Target Steps:** the ordered human procedure, including revision identity and preflight.
5. **Expected Result:** the observable success criteria for key target steps.
6. **Return on Failure:** the exact, minimal, sanitized evidence to bring back.

When no protected-target validation is needed, say so directly: `No server or target-environment validation is required for this task.` Do not manufacture a server handoff.
