# AGENTS.md

This file provides guidance to AI coding agents (Claude Code, Codex, Cursor, GitHub Copilot) when working with code in this repository.

## Project

Pinball Trainer Plus is a discontinued (2013) game trainer for 3D Pinball for Windows - Space Cadet. Built with Object Pascal in Borland Delphi 7. No automated builds or tests. CI in `.github/workflows/` delegates to `rios0rios0/pipelines`: `checks.yaml` runs the shared pull-request checks, `release.yaml` tags releases, and `claude-review.yaml` and `claude-mention.yaml` run Claude PR review and `@claude` mention responses.

## Build

All builds are manual in the Delphi 7 IDE:

1. Compile `SpeedHack/SpeedHack.dpr` first (produces `SpeedHack.dll`)
2. Compile `PTP.dpr` (produces the trainer executable)

`UPTP.pas` embeds the committed `Resources/SpeedHack.res`, not the freshly built DLL: a change to `SpeedHack.dpr` reaches the trainer only after that resource is rebuilt from the new DLL.

Run `Clear.bat` (root) and `SpeedHack/Clear.bat` to remove IDE and compiler intermediates (`.dcu`, `.~??`, `.cfg`, `.dof`, …); neither deletes the built `.exe` or `.dll`.

## Architecture

Single-form VCL app (`UPTP.pas`) that reads and writes `pinball.exe` memory, plus a DLL (`SpeedHack/SpeedHack.dpr`) the trainer injects into the game to hook its timing APIs. `SpeedHack/USpeedHack.pas` is an unused inline-`asm` variant referenced by neither project.

The DLL finds the trainer by its window class `TFrmPTPPrincipal` and reads `SpeedHackSpeed`, `SpeedHackSleep`, and `SpeedHackActiv` from hard-coded absolute addresses in the trainer's process (procedure `Speed` in `SpeedHack.dpr`). Renaming the form class, or any change that moves those globals in the compiled trainer, breaks the speed hack.

## Conventions

- The original identifiers, comments, and UI strings mix Portuguese (pt-BR) and English; leave them as they are. Write new documentation in English.
- `UPTP.pas` is Windows-1252, not UTF-8; edit it in a way that keeps that encoding.
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
  `chlog new --kind <Kind> --body '<past-tense description>'`
- Write an apostrophe inside the single-quoted body as `'\''`.
- Valid kinds: Added, Changed, Deprecated, Removed, Fixed, Security
- Choose the kind that best matches the change (e.g., new feature → Added,
  bug fix → Fixed, behavior change → Changed, removal → Removed, security fix → Security).
- If the change is backward-INCOMPATIBLE with the public API (a breaking
  change), you MUST add the `--breaking` flag:
  `chlog new --kind <Kind> --breaking --body '<past-tense description>'`.
  This is the ONLY thing that triggers a major version bump — the kind alone
  never does (per SemVer, major = incompatible change). When unsure whether a
  change breaks compatibility, ask the user instead of guessing.
- Fragments are YAML files in `.changes/unreleased/`; stage them with your commit.
- `chlog check` fails the build when a fragment is missing — never skip it.
<!-- chlog:end -->
