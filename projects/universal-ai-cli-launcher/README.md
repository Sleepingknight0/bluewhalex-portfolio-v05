# Universal AI CLI Launcher

> A portfolio case study for a local Windows developer tool. The public document explains architecture, behavior, safety decisions, and validated outcomes. It intentionally excludes credentials, account identifiers, authentication files, and the private local installer.

## Project snapshot

| Area | Implementation |
|---|---|
| Product | Universal AI CLI Launcher and multi-account selector |
| Command | `ai` |
| Platform | Windows |
| Shells | Windows PowerShell 5.1 and PowerShell 7+ |
| Core language | PowerShell |
| Configuration | Versioned JSON provider registry |
| Account isolation | Provider-supported environment directories or explicit system-default mode |
| Usage data | Official structured interfaces, passive provider metadata, sanitized cache, or explicit unavailable state |
| Startup model | Cache-only and local-only hot path |
| Validation | 75 passing tests on both supported PowerShell editions as of 1 August 2026 |
| Distribution | Private local tool; public architecture case study only |

## What I built

The Universal AI CLI Launcher turns one global command into a consistent control layer for several terminal-based AI tools:

```powershell
ai
ai codex work -- --version
ai claude personal -- -p "Review this project"
ai --last
ai doctor
ai usage
```

The launcher selects a provider, selects an account, applies only the isolation mechanism that the provider genuinely supports, launches the real executable with the original argument boundaries intact, and restores the caller's environment afterward.

It does not reinstall or replace provider CLIs. It orchestrates tools already installed on the machine.

## Why the project exists

Developer AI tools do not share one account model:

- OpenAI Codex can isolate state through `CODEX_HOME`.
- Claude Code can use `CLAUDE_CONFIG_DIR` for launcher-managed profiles.
- Some tools use files, others use API-key environment variables, and others rely on a shared Windows credential store or system keyring.
- Usage limits and reset windows are exposed differently—or not exposed at all.
- A PowerShell wrapper named after a provider can accidentally call itself recursively.
- Automatic discovery and usage refresh can make a simple menu take several seconds to open.

The project solves those differences without pretending every provider behaves the same.

## Product goals

1. One stable `ai` entry point for installed AI terminal applications.
2. Separate account configuration where the provider officially supports it.
3. Safe pass-through of quoted and Unicode arguments.
4. Exact environment restoration and provider exit-code propagation.
5. Usage telemetry that preserves source and freshness instead of inventing quota.
6. A fast terminal UI that performs no provider or network work on ordinary startup.
7. Extensibility through provider metadata rather than one giant provider switch statement.

## High-level architecture

```text
PowerShell profile
      │
      ▼
global ai wrapper
      │
      ▼
ai-launcher.ps1
      │
      ├── hot path
      │     ├── config.json
      │     ├── state.json
      │     ├── provider-index.json
      │     └── menu-index.json
      │
      ├── provider registry
      │     └── providers/<id>/provider.json
      │
      ├── account profiles
      │     └── providers/<id>/accounts/<slug>/profile.json
      │
      ├── explicit usage path
      │     ├── provider usage adapters
      │     ├── normalized cache
      │     └── optional local aggregates
      │
      └── provider process
            ├── temporary environment
            ├── original argument array
            └── original exit code
```

### Architectural style

The launcher uses a modular monolith with a metadata-driven provider registry.

- The main script owns navigation and process launch orchestration.
- focused modules own accounts, paths, indexes, rendering, cache, usage normalization, history, and security checks.
- provider JSON supplies capabilities and strategy configuration.
- provider-specific usage adapters stay outside the generic usage core.
- indexes provide a small read model for the hot terminal UI.

This design keeps provider-specific behavior at the edge while the launch lifecycle remains generic.

## How launching works

### 1. Resolve the selection

The launcher accepts interactive and direct forms:

```powershell
ai
ai codex
ai codex work
ai codex work -- exec "Explain this repository"
```

Everything after `--` belongs to the provider CLI.

### 2. Resolve the real executable

Provider command candidates are stored in JSON. Resolution accepts real applications and external scripts, ignores PowerShell functions with the same name, removes duplicates, and avoids launcher-owned scripts.

The preference order is executable, command file, batch file, then PowerShell script.

### 3. Save the environment

Before applying an account, the launcher records for every changed variable:

- whether the variable existed;
- its original value;
- the replacement required by the provider strategy.

### 4. Apply the account strategy

Supported strategy shapes include:

- environment directory;
- multiple environment variables;
- fixed command arguments;
- API-key environment reference;
- system default;
- explicitly trusted custom script;
- unsupported.

Unknown strategies fail safely.

### 5. Launch without command-string evaluation

The real process receives an argument array. The launcher does not use `Invoke-Expression` and does not join provider arguments into a single command string.

### 6. Restore in `finally`

After the provider exits—or if launch fails—the launcher restores previous values and removes variables that were originally absent. The provider exit code is preserved.

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

Account metadata contains display information and managed paths, never tokens or passwords.

The account lifecycle supports adding, renaming, deleting with exact confirmation, login/launch flow, and migration of existing Codex profiles without copying authentication files.

## Honest capability levels

The launcher classifies account support instead of faking isolation:

- `isolated`
- `partially-isolated`
- `system-default-only`
- `api-key-profile`
- `unsupported`

For example, a provider backed by a shared operating-system credential store remains a System Default account unless an official configurable profile mechanism is found.

## Usage dashboard architecture

Usage is normalized into a provider-independent schema containing:

- provider and account IDs;
- observation and expiry timestamps;
- source type and detail;
- percentage or absolute meters;
- reset timestamps;
- monthly token and cost fields;
- credit fields;
- warnings and errors.

Unavailable numeric values remain `null`; they are never converted to zero.

### Source classifications

- `LIVE`: official data retrieved during the current refresh.
- `CACHED`: official normalized data still inside the cache lifetime.
- `LOCAL`: observed only from launcher-supported local metadata.
- `ESTIMATED`: an explicitly labeled local calculation.
- `STALE`: valid prior data older than its cache lifetime.
- `UNAVAILABLE`: no supported official interface exists.
- `AUTH REQUIRED`, `PERMISSION DENIED`, `ERROR`: retrieval could not produce usable current data.

Provider-wide and locally observed totals are never silently combined.

## Passive Claude telemetry

Claude Code can send a structured Status Line payload after a real response. A launcher-owned collector reads only approved rate-limit, context-window, session-cost, version, and session identifier fields.

The collector:

- receives JSON from standard input;
- checks temporary launcher provider/account identifiers;
- writes a sanitized normalized cache file atomically;
- emits no network request;
- sends no model prompt;
- stores no prompt, response, transcript, working-directory content, or credential value.

This implements a provider-driven update model:

```text
Provider is used
→ provider emits supported metadata
→ collector sanitizes and caches it
→ the next ai menu reads it immediately
```

## Codex usage handling

Codex provider-reported limits remain separate from activity observed on the local machine.

- Official rate-limit percentages and reset timestamps can appear as provider-reported meters when the installed interface returns them.
- Token totals derived from local activity are labeled “Observed on this machine.”
- A locally observed monthly token total is not presented as total account usage.
- Missing cost and credit values remain absent.
- A previously misleading month-shaped rolling window and missing reset-credit zero were removed.

## Grok and Antigravity limitations

The installed Grok Build version did not expose a documented, non-consuming, machine-readable subscription-usage command. The launcher therefore shows no subscription percentage instead of scraping the TUI or calling private endpoints.

xAI Management API data is treated as a separate `xai-api` account type and requires an explicitly referenced management credential.

Antigravity remains system-default-only when its authentication is held in shared provider or operating-system facilities. The launcher does not copy keyring secrets or claim isolated accounts.

## Fast-start architecture

The first implementation performed discovery and usage work while opening menus. Startup took approximately 5–10 seconds.

The optimized hot path reads only:

```text
config.json
state.json
cache/provider-index.json
cache/menu-index.json
```

Ordinary `ai` startup performs no network request, provider executable launch, background job, runspace, recursive session scan, or automatic usage refresh.

Measured warm rendering during the optimization run was approximately 24–29 ms median. This is a local point-in-time measurement, not a universal hardware claim.

## Terminal interface

The current interface is a compact command console with:

- provider groups;
- one usage row per account;
- segmented percentage bars only when a real percentage exists;
- clear fresh, stale, unavailable, and no-account states;
- Unicode rendering in Windows Terminal;
- ASCII fallback when Unicode is unavailable;
- narrow and wide layouts;
- keyboard navigation without recursive menu calls.

The UI formats preloaded state only; it does not refresh usage while rendering.

## Security decisions

### Credential separation

The design distinguishes:

- CLI login credentials;
- inference API credentials;
- usage-read credentials;
- organization administrator credentials;
- management and billing credentials.

The launcher never automatically reuses a CLI OAuth token for a billing API.

### Filesystem defense

- Managed roots are canonicalized.
- Account IDs and slugs are validated against Windows path rules and reserved device names.
- Destructive operations use literal paths and revalidate immediately before deletion.
- The profile root and launcher root are protected deletion targets.
- Reparse-point or junction escapes are rejected.
- JSON updates use temporary files and atomic replacement.

### Data minimization

Usage cache and diagnostic output contain normalized non-secret data only. Authentication file existence may be checked, but authentication file contents are not parsed, printed, copied, merged, or migrated.

## Compatibility engineering

The launcher targets both Windows PowerShell 5.1 and PowerShell 7+ under `Set-StrictMode -Version Latest`.

Compatibility work includes:

- syntax that runs on both editions;
- UTF-8 handling that recognizes Windows Terminal on 5.1;
- ASCII fallback for limited hosts;
- property-access guards for malformed JSON;
- no dependency on WSL, Bash, Python, Node helper scripts, GUI frameworks, or third-party PowerShell modules.

## Testing strategy

The automated suite uses mocked provider responses and isolated temporary roots so it does not require real credentials.

Coverage includes:

- segmented and detailed progress rendering;
- percentage clamping and invalid values;
- UTC, local, Unix-second, and Unix-millisecond reset timestamps;
- cache freshness and stale preservation after failed refresh;
- source-classification separation;
- malformed provider responses;
- authentication and permission failures;
- provider timeout;
- environment restoration;
- Unicode and quoted argument forwarding;
- exact exit-code preservation;
- account naming, collisions, and destructive path guards;
- menu navigation and Back/Quit behavior;
- PowerShell parser validation.

Point-in-time result on 1 August 2026:

```text
PowerShell 7:             PASS=75  FAIL=0  SKIPPED=0
Windows PowerShell 5.1:  PASS=75  FAIL=0  SKIPPED=0
```

## Major bugs and what changed

| Symptom | Root cause | Resolution |
|---|---|---|
| Account menu crashed with an empty usage array | A mandatory parameter rejected an empty collection | The summary path now handles providers with no usage rows |
| Back from Manage passed an empty ProviderId | Navigation used an ambiguous empty return value | Navigation results now separate Back, Quit, and SelectProvider actions |
| Startup took 5–10 seconds | Menu-open refresh and repeated provider discovery blocked rendering | Replaced startup work with provider and menu indexes |
| Multiple accounts were summarized by one bar | Provider-level minimum hid account identity | Main menu now renders one telemetry row per account |
| Missing usage looked like zero | Unavailable numeric fields were coerced | Missing values remain null and render as no telemetry |
| A 730-hour “rolling” window appeared | A month-like duration was derived into a misleading label | Removed the invented period label and kept only supported provenance |
| Cached data was always marked stale | Cache origin and cache age were treated as the same state | Staleness is calculated from observation/expiry time per account |
| Windows Terminal showed ASCII unexpectedly | Legacy code-page detection was used as the only Unicode signal | Added Windows Terminal detection and compatible UTF-8 output handling |

## Engineering outcomes

- Built a single command surface across heterogeneous provider CLIs.
- Preserved account and credential boundaries rather than normalizing them away.
- Moved slow work out of the hot path and measured the result.
- Built a normalized usage model that can express unavailable and local data honestly.
- Added extensibility through JSON provider definitions and adapters.
- Kept behavior compatible with both current and legacy PowerShell editions.
- Turned recurring bugs into regression tests.

## What I would improve next

1. Publish a fully sanitized installer and source package after a separate security review.
2. Add more official provider adapters only when documented structured usage interfaces exist.
3. Expand end-to-end terminal snapshot tests across more console hosts.
4. Add signed release artifacts and reproducible installation verification.
5. Continue reducing backup and cache maintenance overhead without weakening recovery.

## Interview summary

> I built a PowerShell orchestration layer for multiple AI CLIs on Windows. The difficult part was not drawing a menu—it was preserving real provider boundaries: account isolation differs by tool, authentication can live in files or system keyrings, usage data has different provenance, and a wrapper can easily recurse into itself. I designed a JSON registry and adapter architecture, made startup cache-only, implemented exact environment restoration and argument forwarding, and tested the same behavior on PowerShell 5.1 and 7. The result is a fast local command center that is explicit about what it knows and what a provider does not safely expose.

---

## สรุปภาษาไทย

Universal AI CLI Launcher คือเครื่องมือ PowerShell บน Windows ที่รวมการเลือก AI CLI และบัญชีไว้ใต้คำสั่ง `ai` เดียว โดยเน้นสามเรื่องหลัก:

1. **แยกบัญชีตาม capability จริงของ Provider** — ใช้ `CODEX_HOME`, `CLAUDE_CONFIG_DIR` หรือ system-default-only ตามที่ Provider รองรับจริง
2. **เปิดเร็ว** — หน้าเมนูอ่านเฉพาะ index และ cache ในเครื่อง ไม่เรียก API หรือเปิด Provider CLI
3. **ข้อมูล usage ซื่อสัตย์** — แยก provider-reported, local, stale และ unavailable ออกจากกัน ไม่สร้างเปอร์เซ็นต์ปลอมและไม่ใช้ `0` แทนข้อมูลที่ไม่มี

สถาปัตยกรรมเป็น modular monolith ที่ใช้ JSON provider registry และ provider-specific adapters มีการตรวจ path ป้องกันการลบออกนอก managed root ไม่อ่านเนื้อหาไฟล์ authentication และคืน environment/exit code ของ process ได้ถูกต้อง

โปรเจกต์นี้ผ่านชุดทดสอบ 75 รายการทั้ง Windows PowerShell 5.1 และ PowerShell 7 ณ วันที่ 1 สิงหาคม 2026 และลด startup จาก flow ที่เคยใช้เวลา 5–10 วินาทีเหลือการ render แบบ cache-only ระดับมิลลิวินาทีบนเครื่องพัฒนา
