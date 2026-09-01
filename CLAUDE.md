# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Pinball Trainer Plus is a discontinued (2013) game trainer for 3D Pinball for Windows - Space Cadet. Built with Object Pascal in Borland Delphi 7. No automated builds or tests. CI in `.github/workflows/` delegates to `rios0rios0/pipelines`: `release.yaml` tags releases, while `claude-review.yaml` and `claude-mention.yaml` run Claude PR review and `@claude` mention responses.

## Build

All builds are manual in the Delphi 7 IDE:

1. Compile `SpeedHack/SpeedHack.dpr` first (produces `SpeedHack.dll`)
2. Compile `PTP.dpr` (produces the trainer executable)

Run `Clear.bat` (root) or `SpeedHack/Clear.bat` to remove build artifacts.

## Architecture

Single-form VCL app (`UPTP.pas`) + injected DLL (`SpeedHack/SpeedHack.dpr`).

- **Process detection** — polls for `pinball.exe` via `CreateToolHelp32SnapShot`; finds window by class `1c7c22a0-9576-11ce-bf80-444553540000`
- **Memory R/W** — `ReadProcessMemory`/`WriteProcessMemory` with pointer chains (base `$01025040`)
- **DLL injection** — extracts DLL from embedded resources, injects via `VirtualAllocEx` + `CreateRemoteThread` + `LoadLibraryA`
- **Speed hack** — DLL patches first 5 bytes of `GetTickCount`, `timeGetTime`, `QueryPerformanceCounter` with JMP to custom replacements
- **Trainer-DLL IPC** — DLL reads trainer's `SpeedHackSpeed`, `SpeedHackSleep`, `SpeedHackActiv` variables from fixed memory offsets

## Conventions

- Type prefix `T` (e.g. `TFrmPTPPrincipal`)
- Constants in `SCREAMING_SNAKE_CASE`
- UI control names prefixed with `s` (AlphaControls)
- UI labels in Portuguese (pt-BR); code and docs in English
- Memory addresses as `DWORD` hex constants
- Conventional commits (e.g. `chore:`, `fix:`)

## Prerequisites

- Borland Delphi 7 with AlphaControls component suite
- Windows XP/7 32-bit
- 3D Pinball for Windows - Space Cadet

<!-- chlog:start -->
## Changelog (chlog) — MANDATORY

If the repository you are working in uses chlog (a `.chlog.yaml` or `.chlog.yml`
config file, or a `.changes/` directory, exists at the project root), the
following is binding and ALWAYS applies: whenever you make ANY change, you MUST
create a changelog fragment as part of the same change — automatically, without
being asked, before committing.

- Do NOT edit CHANGELOG.md directly; it is generated from fragments.
- Create the fragment with:
  `chlog new --kind <Kind> --body "<imperative description>"`
- Valid kinds: Added, Changed, Deprecated, Removed, Fixed, Security
- Choose the kind that best matches the change (e.g., new feature → Added,
  bug fix → Fixed, behavior change → Changed, removal → Removed, security fix → Security).
- If the change is backward-INCOMPATIBLE with the public API (a breaking
  change), you MUST add the `--breaking` flag:
  `chlog new --kind <Kind> --breaking --body "<description>"`.
  This is the ONLY thing that triggers a major version bump — the kind alone
  never does (per SemVer, major = incompatible change). When unsure whether a
  change breaks compatibility, ask the user instead of guessing.
- Fragments are YAML files in `.changes/unreleased/`; stage them with your commit.
- `chlog check` fails the build when a fragment is missing — never skip it.
<!-- chlog:end -->
