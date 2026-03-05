# Copilot Instructions

## Project Overview

**Pinball Trainer Plus** is a game trainer for **3D Pinball for Windows - Space Cadet** (`pinball.exe`). Built with Object Pascal in Borland Delphi 7, it reads and writes the game's process memory in real time to modify score, ball count, flipper force, cheat flags, and game speed. A companion DLL implements a speed hack by hooking three Windows timing APIs at runtime through inline x86 assembly code patching. Development was discontinued on 2013-03-28 and the repository is preserved as a historical reference.

**ALWAYS** reference these instructions first and fall back to searching or reading source files only when you encounter information that does not match what is documented here.

## Working Effectively

### Prerequisites

- **Borland Delphi 7** (or compatible IDE such as Embarcadero RAD Studio) with the **AlphaControls** component suite installed
- **Windows XP/7** (32-bit environment — the trainer targets a 32-bit address space)
- **3D Pinball for Windows - Space Cadet** installed (the bundled installer can be extracted by the trainer itself)

### Bootstrap and Build the Project

There is no automated build system or package manager. Builds are done manually inside the Delphi IDE:

1. Open `SpeedHack/SpeedHack.dpr` in Delphi 7 and compile to produce `SpeedHack.dll`
2. Open `PTP.dpr` in Delphi 7 and compile to produce the trainer executable

Run `Clear.bat` to remove all Delphi build artifacts (`.obj`, `.dcu`, `.exe`, etc.) before a clean rebuild.

There is no CI/CD pipeline and no automated test suite.

## Repository Structure

```
pinball-trainer-plus/
├── .github/
│   └── copilot-instructions.md   # This file
├── PTP.dpr                        # Main application entry point (Delphi project file)
├── PTP.res                        # Compiled project resources
├── UPTP.pas                       # Main form unit — trainer logic, memory R/W, DLL injection
├── UPTP.dfm                       # Main form visual layout (Delphi Form Designer file)
├── SpeedHack/
│   ├── SpeedHack.dpr              # Speed hack DLL project file
│   ├── SpeedHack.res              # DLL compiled resources
│   └── USpeedHack.pas             # Speed hack implementation (API hooking, timing override)
├── Resources/
│   ├── SpeedHack.res              # Embedded SpeedHack.dll resource (linked into the EXE)
│   └── Installer.res              # Embedded Pinball installer resource
├── Imgs/
│   └── Icon.ico                   # Application icon
├── Clear.bat                       # Cleanup script for Delphi build artifacts
├── CONTRIBUTING.md
├── LICENSE
└── README.md
```

### Key Source Files

| File | Purpose |
|---|---|
| `UPTP.pas` | All trainer logic: process detection, memory read/write, DLL injection, UI event handlers, freeze timers |
| `USpeedHack.pas` | Speed hack DLL: inline x86 JMP patching of `GetTickCount`, `timeGetTime`, `QueryPerformanceCounter` |
| `PTP.dpr` | Application entry point; references `UPTP` and links resources |
| `SpeedHack/SpeedHack.dpr` | DLL entry point |
| `UPTP.dfm` | Delphi form layout — edit in the Delphi IDE form designer |

## Technology Stack

| Layer | Technology |
|---|---|
| Language | Object Pascal (Borland Delphi 7) |
| UI Framework | VCL (Visual Component Library) |
| UI Skinning | AlphaControls component suite (`sSkinManager`, `sTrackBar`, `sCheckBox`, etc.) |
| Windows API | `ReadProcessMemory`, `WriteProcessMemory`, `OpenProcess`, `CreateRemoteThread`, `VirtualAllocEx`, `CreateToolHelp32SnapShot` |
| Low-level | x86 inline assembly (`asm … end`) for 5-byte JMP patches |
| DLL injection | `LoadLibraryA` called via `CreateRemoteThread` in the target process |
| Embedded resources | Delphi `{$R …}` directives to bundle `SpeedHack.dll` and the Pinball installer |
| Target platform | Windows XP/7, 32-bit |

## Architecture and Design Patterns

### Main Application (`UPTP.pas`)

The entire trainer is implemented as a single VCL form class `TFrmPTPPrincipal`. Key responsibilities:

- **Process detection** — `GetProcessIDbyName()` uses `CreateToolHelp32SnapShot` to find a running `pinball.exe` and also locates its window by class name `1c7c22a0-9576-11ce-bf80-444553540000`. A polling timer (`TmrAtualiza`) enables/disables all UI controls based on detection status.
- **Memory read/write** — `ReadMemoryBytes()` / `WriteMemoryBytes()` wrap `ReadProcessMemory` / `WriteProcessMemory`. Pointer chains are followed by reading a base address and adding a fixed offset (e.g. score: base `$01025040` + offset `$52`).
- **Freeze timers** — `TmrFreezePontos` and `TmrFreezeBolas` continuously rewrite the desired score/ball-count values to prevent the game from changing them.
- **DLL injection** — `InjectDll()` extracts `SpeedHack.dll` from embedded resources via `CreateResource()`, allocates memory in the target process with `VirtualAllocEx`, writes the DLL path with `WriteProcessMemory`, and creates a remote thread that calls `LoadLibraryA`.
- **Trainer–DLL communication** — After injection, the DLL finds the trainer's window (`TFrmPTPPrincipal`) and reads three variables from the trainer's own process memory at fixed offsets: `SpeedHackSpeed` (acceleration multiplier), `SpeedHackSleep` (thread sleep interval), `SpeedHackActiv` (active flag).

### Speed Hack DLL (`USpeedHack.pas`)

The DLL patches three Windows timing functions in-process:

1. Saves the original first 5 bytes of `GetTickCount`, `timeGetTime`, and `QueryPerformanceCounter`
2. Overwrites the first 5 bytes with an `E9` (JMP) instruction redirecting to custom replacements
3. A high-priority background thread increments a virtual tick counter using the configured multiplier
4. Hooked functions return the virtual tick value instead of real system time, causing the game to run faster or slower

### Key Memory Addresses

| Feature | Base Address | Offset |
|---|---|---|
| Score (pointer chain) | `$01025040` | `$52` |
| Ball count (pointer chain) | `$01025040` | `$146` |
| Flipper force — primary (4 addresses) | `$010236E4`–`$010236FC` | — |
| Flipper force — secondary (3 addresses) | `$01023758`–`$01023768` | — |
| Cheat flag | `$01024FF8` | — |
| Death counter flag | `$01025044` | — |

## Build Commands

All builds are performed manually inside the Delphi 7 IDE. There are no command-line build scripts.

| Step | Action |
|---|---|
| Build SpeedHack DLL | Open `SpeedHack/SpeedHack.dpr` → **Project › Compile** |
| Build trainer EXE | Open `PTP.dpr` → **Project › Compile** |
| Clean build artifacts | Run `Clear.bat` from a command prompt |

Typical build time: **< 30 seconds** for each project on a Windows machine.

## Testing

There is no automated test suite. Validation is done manually:

1. Build both projects (SpeedHack DLL first, then the main application).
2. Install or launch **3D Pinball for Windows - Space Cadet**.
3. Run the trainer; verify that controls become enabled once `pinball.exe` is detected.
4. Exercise each feature: score/ball modification, freeze locks, flipper force sliders, cheat toggle, death counter toggle.
5. Test the speed hack: enable it and observe the game running faster/slower.
6. Test the built-in installer: click the install button and verify the Pinball installer launches.

## Coding Conventions

- **Type prefix** — all type declarations use `T` prefix (e.g. `TFrmPTPPrincipal`).
- **Constants** — use `SCREAMING_SNAKE_CASE` (e.g. `ADDRESS_SCORE`, `OFFSET_BALL`).
- **Variables** — use `PascalCase` or `camelCase` (e.g. `PHandle`, `SpeedHackSpeed`, `CETick64`).
- **UI controls** — AlphaControls-skinned controls are prefixed with `s` (e.g. `sTrckbr1`, `sChkFreezeBolas`, `sEdtSpeed`).
- **Procedures/functions** — `PascalCase` verbs (e.g. `GetProcessIDbyName`, `InjectDll`, `WriteMemoryBytes`).
- **Language mix** — UI labels are in Portuguese (pt-BR); comments and documentation are in English.
- **Resource directives** — use `{$R *.res}` and `{$R Resources\SpeedHack.res}` to link compiled resources.
- **Error handling** — wrap memory operations in `try…except` blocks; do not let unhandled exceptions propagate to the user.
- **Hard-coded addresses** — memory addresses are stored as `DWORD` constants in hexadecimal (e.g. `$01025040`). Document any new addresses in a comment explaining what they point to.

## Development Workflow

Because this project is discontinued, changes should be limited to documentation, resource updates, or historical corrections.

1. Create a feature branch:
   ```bash
   git checkout -b chore/my-change
   ```
2. Open the relevant `.pas` or `.dfm` file in Delphi 7 and make changes.
3. Build and test manually (see above).
4. Commit with a conventional commit message, e.g. `chore: update memory address for score`.
5. Open a pull request against `main`.

## Common Tasks

| Task | How |
|---|---|
| Change a memory address | Update the relevant constant in `UPTP.pas` and update the table in `README.md` |
| Add a new freeze target | Add a new timer in the VCL form, hook it to `WriteMemoryBytes` calls |
| Update the speed hack | Edit `USpeedHack.pas`, rebuild `SpeedHack.dpr`, re-embed the resulting DLL into `Resources/SpeedHack.res` |
| Replace the bundled installer | Rebuild `Resources/Installer.res` from the new installer binary |
| Remove build artifacts | Run `Clear.bat` |

## Troubleshooting

- **Controls remain disabled** — `pinball.exe` is not running or the window class name has changed. Verify the process is in the Task Manager and that the class name constant in `UPTP.pas` is correct.
- **`ReadProcessMemory` / `WriteProcessMemory` fails** — The trainer may need to be run as Administrator. Ensure the process handle is opened with `PROCESS_ALL_ACCESS`.
- **DLL injection fails** — Ensure `SpeedHack.dll` was extracted correctly to disk. Check the path passed to `LoadLibraryA` and that `VirtualAllocEx` succeeded.
- **Speed hack has no effect** — The DLL may not have been loaded, or the hooked functions' addresses may have changed in a different version of `kernel32.dll` / `winmm.dll`. Verify with a debugger that the 5-byte JMP patch was applied.
- **Build error: missing AlphaControls components** — Install the AlphaControls suite into the Delphi 7 component palette before compiling.
- **`Clear.bat` removes too much** — Review the script contents before running; it deletes specific file extensions produced by the Delphi compiler.
