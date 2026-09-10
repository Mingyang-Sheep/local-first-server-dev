# Example: target device or cross-compilation

This example covers Linux-only toolchains, ROS workspaces, embedded devices, proprietary SDKs, and real hardware without turning the skill into a platform cookbook.

## Request

> Update a C++ component and its configuration on Windows. The authoritative build uses a Linux cross-compiler, and final behavior must be checked on an embedded target with a sensor attached.

## Local phase

The agent should inspect the source, headers, build configuration, protocol definitions, tests, and existing deployment documentation. It should implement the change locally and run reliable checks already available in the repository, such as formatting, static analysis, host-side unit tests, configuration parsing, protocol fixtures, and CMake configuration checks that do not require the target toolchain.

Possible evidence classification:

```text
Executed locally:
- Host-side unit tests passed.
- Configuration fixture parsed successfully.
- Static analysis reported no new findings.

Static review only:
- The changed interface is wired into the target executable.

Target validation required:
- Cross-compilation with the authoritative sysroot and SDK.
- Dynamic linking on the target image.
- Sensor communication and timing on real hardware.
```

A host-side protocol mock may validate serialization and error handling. It does not validate the device driver, bus, sensor, timing, or target ABI.

## Human handoff

Use the repository's real build and launch scripts. The following is only a shape for the handoff:

```bash
# TEMPLATE — values and commands must be verified from the project.
cd <TARGET_BUILD_WORKSPACE>
<ACTIVATE_EXISTING_TOOLCHAIN_OR_CONTAINER>

# Confirm the prepared change.
<VERIFY_REVISION_OR_PATCH_COMMAND>

# Cheapest authoritative build or configuration check.
<TARGET_PREFLIGHT_BUILD_COMMAND>

# Run only after preflight succeeds.
<TARGET_DEVICE_LAUNCH_OR_HARDWARE_TEST_COMMAND>
```

Expected results should identify the build artifact, successful link or package creation, process startup, device discovery, and the narrow functional signal relevant to the change.

On failure, return the revision identity, exact command, compiler or linker diagnostics, toolchain and SDK versions, target architecture, focused runtime logs, and the smallest relevant hardware observation. Sanitize hostnames, credentials, serial numbers, private endpoints, and proprietary data.

The handoff should not ask the user to repair CMake files, change headers, or write an ad-hoc target script. If that work is still necessary, return to the local phase.
