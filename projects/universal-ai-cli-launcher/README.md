# Universal AI CLI Launcher

> A public architecture case study for a private Windows developer tool. Credentials, account identifiers, authentication files, local configuration, source code, and the installer are intentionally excluded.

## Project summary

| Area                | Implementation                                                                                      |
| ------------------- | --------------------------------------------------------------------------------------------------- |
| Product             | Universal AI CLI Launcher and multi-account selector                                                |
| Entry command       | `ai`                                                                                                |
| Platform            | Windows                                                                                             |
| Shell support       | Windows PowerShell 5.1 and PowerShell 7 or newer                                                    |
| Core language       | PowerShell                                                                                          |
| Configuration       | Versioned JSON provider registry                                                                    |
| Account isolation   | Provider-supported environment directories or explicit system-default mode                          |
| Usage data          | Official structured interfaces, passive metadata, sanitized cache, or an explicit unavailable state |
| Startup model       | Cache-only, local-only hot path                                                                     |
| Recorded validation | 75 passing tests on both supported PowerShell editions on 1 August 2026                             |
| Distribution        | Private local tool; public case study only                                                          |

## Overview

The launcher turns one global command into a consistent control layer for several terminal-based AI tools.

It selects a provider and account, applies only the isolation strategy that the provider supports, launches the real executable, preserves argument boundaries, and restores the caller's environment.

The launcher does not install, replace, or impersonate provider command-line tools.

## Example commands

```powershell
ai
ai codex work -- --version
ai claude personal -- -p "Review this project"
ai --last
ai doctor
ai usage
```

Everything after `--` is passed to the selected provider executable.

## Problem statement

AI command-line tools do not share one account, authentication, or usage model.

- Some providers isolate state through a configurable directory.
- Some use environment variables or fixed command arguments.
- Others rely on a shared Windows credential store or system keyring.
- Usage limits and reset windows may be official, local, unavailable, or not machine-readable.
- A wrapper named after a provider can accidentally call itself.
- Live discovery and usage refresh can make a simple menu unacceptably slow.

The design preserves these differences instead of forcing every provider into one false abstraction.

## Product goals

1. Provide one stable entry point for installed AI terminal tools.
2. Isolate accounts only through officially supported mechanisms.
3. Preserve quoted, Unicode, and pass-through arguments exactly.
4. Restore the environment and propagate provider exit codes.
5. Preserve usage provenance, freshness, and unavailable states.
6. Keep ordinary startup local, deterministic, and fast.
7. Add providers through metadata and focused adapters.

## Architecture

```text
PowerShell profile
      |
      v
global ai wrapper
      |
      v
ai-launcher.ps1
      |
      |-- hot path
      |   |-- config.json
      |   |-- state.json
      |   |-- provider-index.json
      |   `-- menu-index.json
      |
      |-- provider registry
      |   `-- providers/<id>/provider.json
      |
      |-- account profiles
      |   `-- providers/<id>/accounts/<slug>/profile.json
      |
      |-- explicit usage path
      |   |-- provider adapters
      |   |-- normalized cache
      |   `-- optional local aggregates
      |
      `-- provider process
          |-- temporary environment
          |-- original argument array
          `-- original exit code
```

The implementation is a modular monolith with a metadata-driven provider registry.

The main script owns navigation and launch orchestration. Focused modules own account management, paths, indexes, cache, rendering, usage normalization, history, and security checks.

Provider-specific behavior remains at the system edge. The launch lifecycle and safety rules stay generic.

## Launch lifecycle

### 1. Resolve the selection

The launcher accepts interactive, provider-only, provider-and-account, and fully direct command forms.

### 2. Resolve the executable

Command candidates are stored in provider metadata. Resolution ignores conflicting PowerShell functions, removes duplicates, and rejects launcher-owned scripts.

The preference order is executable, command file, batch file, then PowerShell script.

### 3. Save the environment

Before applying an account, the launcher records whether every affected variable existed and stores its original value.

### 4. Apply the account strategy

Supported strategies include environment directories, environment variables, fixed arguments, API-key references, system-default mode, trusted custom scripts, and explicit unsupported states.

Unknown strategies fail safely.

### 5. Launch the provider

The provider receives an argument array. The launcher does not use `Invoke-Expression` or convert the arguments into a command string.

### 6. Restore state

A `finally` block restores previous values and removes variables that were originally absent. The provider's exit code is preserved.

## Provider and account model

Each provider has non-sensitive metadata:

```json
{
  "id": "codex",
  "displayName": "OpenAI Codex",
  "commandCandidates": ["codex.exe", "codex.cmd", "codex.ps1", "codex"],
  "profileStrategy": {
    "type": "environmentDirectory",
    "environmentVariable": "CODEX_HOME"
  }
}
```

Account metadata contains display information and managed paths, never passwords or tokens.

The account lifecycle supports creation, renaming, exact-confirmation deletion, login and launch flows, and safe migration of supported profile structures.

## Capability levels

The launcher describes real isolation support with explicit capability levels:

- `isolated`
- `partially-isolated`
- `system-default-only`
- `api-key-profile`
- `unsupported`

A provider backed by one shared operating-system credential store remains system-default-only until an official configurable profile mechanism is available.

## Usage data model

Provider usage is normalized into a common schema containing provider and account IDs, observation and expiry timestamps, source details, meters, reset times, optional totals, warnings, and errors.

Unavailable numbers remain `null`; they are never displayed as zero.

### Source classifications

| State               | Meaning                                                  |
| ------------------- | -------------------------------------------------------- |
| `LIVE`              | Official data retrieved during the current refresh       |
| `CACHED`            | Official normalized data still within its cache lifetime |
| `LOCAL`             | Activity observed only through supported local metadata  |
| `ESTIMATED`         | Explicitly labelled local calculation                    |
| `STALE`             | Prior valid data older than its cache lifetime           |
| `UNAVAILABLE`       | No supported official interface exists                   |
| `AUTH REQUIRED`     | Authentication is required before retrieval              |
| `PERMISSION DENIED` | Current credentials cannot access the data               |
| `ERROR`             | Retrieval failed without usable current data             |

Provider-wide totals and locally observed activity are never silently combined.

## Passive telemetry

Supported provider status payloads can update a sanitized local cache after real tool use.

The collector accepts approved rate-limit, context-window, session-cost, version, and session identifier fields. It writes normalized data atomically and makes no network request.

It does not store prompts, responses, transcripts, working-directory content, or credential values.

```text
Provider is used
-> provider emits supported metadata
-> collector sanitizes and caches the payload
-> the next launcher menu reads the cache
```

## Provider-specific boundaries

Codex provider-reported limits remain separate from activity observed on the local computer. Missing cost and credit values remain absent.

Claude status data is collected only through its supported passive payload. Authentication file contents are not parsed.

Providers without a documented, non-consuming usage interface display an unavailable state instead of a fabricated subscription percentage.

System-keyring providers remain system-default-only when safe account isolation is unavailable.

## Fast-start design

The original menu performed discovery and usage work during startup and took approximately five to ten seconds on the development computer.

The optimized hot path reads only configuration, state, and prebuilt provider and menu indexes.

Ordinary startup performs no network request, provider launch, background job, recursive session scan, or automatic usage refresh.

Recorded warm rendering during the optimization run was approximately 24 to 29 milliseconds median. This is a local point-in-time result, not a universal performance claim.

## Terminal interface

The interface provides provider groups, one usage row per account, segmented meters only for real percentages, explicit freshness states, Unicode rendering, ASCII fallback, and keyboard navigation.

The menu formats preloaded state only. It does not refresh usage while rendering.

## Security model

### Credential separation

The design distinguishes CLI login, inference API, usage-read, organization administrator, and management or billing credentials.

A CLI OAuth token is never automatically reused for a billing API.

### Filesystem safeguards

- Managed roots are canonicalized.
- Account identifiers are validated against Windows path rules and reserved names.
- Destructive operations use literal paths and revalidate immediately before deletion.
- The profile and launcher roots are protected deletion targets.
- Reparse-point and junction escapes are rejected.
- JSON updates use temporary files and atomic replacement.

### Data minimization

Caches and diagnostics contain normalized non-secret data only.

Authentication file existence may be checked, but authentication contents are not printed, copied, merged, migrated, or used as a general-purpose API credential.

## Compatibility

The launcher targets Windows PowerShell 5.1 and PowerShell 7 or newer under strict mode.

Compatibility work includes cross-edition syntax, Windows Terminal detection, UTF-8 output, ASCII fallback, malformed JSON guards, and independence from WSL, Bash, Python, Node helpers, GUI frameworks, or external modules.

## Validation strategy

The automated suite uses mocked provider responses and isolated temporary roots. It does not require real credentials.

Coverage includes rendering, reset timestamps, cache freshness, source separation, malformed responses, authentication failures, timeouts, environment restoration, argument forwarding, exit codes, path guards, and navigation.

Recorded result on 1 August 2026:

```text
PowerShell 7:             PASS=75  FAIL=0  SKIPPED=0
Windows PowerShell 5.1:  PASS=75  FAIL=0  SKIPPED=0
```

## Representative defects resolved

| Symptom                                     | Cause                                              | Resolution                                                   |
| ------------------------------------------- | -------------------------------------------------- | ------------------------------------------------------------ |
| Empty usage arrays crashed the account menu | A mandatory parameter rejected an empty collection | Added a valid no-usage summary path                          |
| Back navigation passed an empty provider ID | Navigation used one ambiguous empty return value   | Introduced distinct Back, Quit, and Select actions           |
| Startup took five to ten seconds            | Discovery and usage refresh blocked menu rendering | Replaced startup work with local indexes                     |
| Multiple accounts appeared as one meter     | Provider-level aggregation removed identity        | Rendered one telemetry row per account                       |
| Missing values appeared as zero             | Unavailable numeric fields were coerced            | Preserved null and unavailable states                        |
| Cached data always appeared stale           | Origin and age were treated as one property        | Calculated freshness from observation and expiry per account |
| Windows Terminal fell back to ASCII         | Legacy code-page detection was the only signal     | Added compatible terminal and UTF-8 detection                |

## Engineering outcomes

- One command surface across heterogeneous provider tools.
- Account and credential boundaries preserved explicitly.
- Slow work removed from the interactive hot path.
- Honest usage states with provenance and freshness.
- Metadata-driven provider extensibility.
- Cross-edition PowerShell compatibility.
- Regression tests for recurring failure modes.

## Future improvements

1. Publish a sanitized installer and source package after a separate security review.
2. Add adapters only for documented provider interfaces.
3. Expand terminal snapshot coverage across more console hosts.
4. Add signed releases and reproducible installation checks.
5. Reduce cache maintenance overhead without weakening recovery.

## Distribution note

This directory documents the architecture and engineering outcomes only. The private launcher implementation, credentials, local profiles, and installer are not included.
