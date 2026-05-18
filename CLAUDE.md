# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Pinball Trainer Plus is a discontinued (2013) game trainer for 3D Pinball for Windows - Space Cadet. Built with Object Pascal in Borland Delphi 7. No automated builds or tests; CI handles release tagging only (`.github/workflows/release.yaml`).

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
