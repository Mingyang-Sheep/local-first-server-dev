# Local-First Server Development

[简体中文](README.zh-CN.md)

`local-first-server-dev` is an agent skill for projects where code can be understood, changed, and partly validated in a local development workspace, while authoritative compilation, simulation, training, hardware access, or runtime validation must happen on a human-controlled server or target device.

Its goal is simple:

> Leave only the work that truly requires the target environment for the target environment. Complete the difficult engineering work locally first.

## Why this skill exists

GPU servers, HPC clusters, robotics workstations, embedded devices, proprietary SDKs, and production-like Linux environments are often necessary for final validation but poor places for interactive AI development. Without an explicit workflow, a coding agent may either try to recreate an unreliable copy of that environment locally or hand an unfinished problem to the user to debug remotely.

This skill establishes a different operating model:

```text
Local analysis
    -> Local modification
    -> Local validation
    -> Human gate
    -> User executes on the protected target
    -> Logs and results return
    -> Continue local iteration
```

The server or device should mostly perform `apply`, `build`, `run`, and `collect`—not source exploration, solution design, or live code editing.

## Core behavior

- **Local first:** use the local repository for analysis, implementation, tests, configuration work, mocks, and diagnostics.
- **Evidence, not assumption:** clearly separate executed checks, static review, skipped local checks, and target-only validation.
- **Human-gated target:** never connect to or operate the protected server, device, hardware, or production environment.
- **Minimal target burden:** prepare an ordered sync, preflight, formal-run, and failure-collection procedure for the user.

The skill also stops mutations when the current workspace is itself a protected remote workspace or remote mount. The boundary is determined by where an operation takes effect, not by whether a path appears local.

## When to use it

Use the skill when development and authoritative execution are separated, for example:

- CUDA or GPU training projects;
- Isaac Lab, Isaac Sim, MJLab, or MuJoCo simulation projects;
- ROS 2 and robotics deployments;
- Linux-only or cross-compiled applications;
- embedded targets, sensors, CAN, cameras, and proprietary SDKs;
- Slurm/HPC jobs and other controlled compute environments.

Technology keywords alone are not a trigger. A local PyTorch, Linux, ROS, or MuJoCo project that can be fully validated in the current authorized workspace does not need this workflow.

## What it does not do

- It does not SSH into, mount, upload to, or modify a protected target.
- It does not start or stop remote builds, jobs, training, simulations, deployments, or hardware processes.
- It does not recreate a complete server environment locally merely to claim runtime validation.
- It does not require orchestrator, reviewer, tester, or handoff agents.
- It does not provide technology-specific setup cookbooks.

Normal Git hosting actions remain subject to the host agent's permissions and the user's authorization. However, an action that triggers protected-target deployment, training, production release, or hardware execution is part of the gated execution chain and must be left for the user.

## Install

The canonical artifact is the repository's [`SKILL.md`](SKILL.md). The skill follows the open agent skills directory shape, while platform compatibility is claimed only after actual testing.

### Codex

For a user-scoped Codex installation, clone the repository into the user skills directory. Replace `<REPOSITORY_URL>` with the published repository URL.

PowerShell:

```powershell
New-Item -ItemType Directory -Force "$HOME\.agents\skills" | Out-Null
git clone <REPOSITORY_URL> "$HOME\.agents\skills\local-first-server-dev"
```

POSIX shell:

```bash
mkdir -p "$HOME/.agents/skills"
git clone <REPOSITORY_URL> "$HOME/.agents/skills/local-first-server-dev"
```

For a repository-scoped installation, place the skill folder below `.agents/skills/` in the relevant repository. Codex supports explicit invocation with `$local-first-server-dev` and may also select the skill implicitly from its description. See the [official OpenAI skill documentation](https://developers.openai.com/codex/skills) for current locations and invocation behavior.

For another coding agent, follow that host's current official method for loading a `SKILL.md`, rule, or project instruction. The presence of generic instructions here is not itself a compatibility claim.

## Compatibility status

| Host | Status |
| --- | --- |
| Codex | Format validation and behavioral smoke test performed locally |

No support claim is made for other hosts until the workflow has been tested there. Platform-specific installation wrappers may be added later only when real use demonstrates a need; the behavior definition remains in one place.

## Use

Invoke it explicitly when needed:

```text
Use $local-first-server-dev to implement this change locally. The final build and run must be performed by me on the GPU server.
```

Or describe the boundary directly:

```text
The robot SDK is not available locally. Modify and validate everything that can be trusted here, then give me the smallest target-device preflight and run procedure.
```

At completion or target handoff, the agent reports what changed, what was actually validated, what remains target-only, the human commands, expected results, and the exact sanitized evidence to return after failure. It does not force this report format into ordinary discussion.

## Examples

- [GPU training or simulation](examples/gpu-or-simulation.md)
- [Target device or cross-compilation](examples/target-device-or-cross-compile.md)
- [Server feedback loop](examples/feedback-loop.md)

The commands in the examples are deliberately marked templates. A real handoff must derive commands from the target repository and ask only for information that is genuinely missing.

## Repository structure

```text
local-first-server-dev/
|-- SKILL.md
|-- README.md
|-- README.zh-CN.md
|-- LICENSE
`-- examples/
    |-- gpu-or-simulation.md
    |-- target-device-or-cross-compile.md
    `-- feedback-loop.md
```

The first version intentionally contains no remote automation, server scripts, platform adapter tree, or technology cookbook. Behavioral evaluations and thin host adapters can be added when observed failures justify them.

## Contributing

Keep changes focused on decisions that improve this workflow across projects. Do not duplicate the behavior specification in platform wrappers or turn one project's environment details into universal rules. Compatibility claims should include the host, invocation method, scenario, and observed result.

## License

[MIT](LICENSE)
