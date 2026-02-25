<h1 align="center">Pinball Trainer Plus</h1>
<p align="center">
    <a href="https://github.com/rios0rios0/pinball-trainer-plus/releases/latest">
        <img src="https://img.shields.io/github/release/rios0rios0/pinball-trainer-plus.svg?style=for-the-badge&logo=github" alt="Latest Release"/></a>
    <a href="https://github.com/rios0rios0/pinball-trainer-plus/blob/main/LICENSE">
        <img src="https://img.shields.io/github/license/rios0rios0/pinball-trainer-plus.svg?style=for-the-badge&logo=github" alt="License"/></a>
</p>

A game trainer for **3D Pinball for Windows - Space Cadet** (`pinball.exe`) built with Object Pascal in Borland Delphi 7. The trainer reads and writes the game's process memory in real time to modify score, ball count, flipper force, cheat flags, and game speed. A companion DLL implements a speed hack by hooking Windows timing APIs at runtime through inline code patching. Development was discontinued on 2013-03-28.

## Features

- **Score modification** -- read and write the current score by following a pointer chain at memory address `$01025040` + offset `$52`
- **Ball count modification** -- read and write the remaining ball count via address `$01025040` + offset `$146`
- **Freeze score/balls** -- timer-based freeze locks that continuously rewrite the desired value, preventing the game from changing it
- **Flipper force control** -- two trackbar sliders adjust the force of primary flippers (4 address/offset pairs at `$010236E4`-`$010236FC`) and secondary flippers (3 pairs at `$01023758`-`$01023768`)
- **Cheat mode toggle** -- directly writes to the in-game cheat flag at address `$01024FF8`
- **Death counter toggle** -- reads and writes the death tracking flag at address `$01025044`
- **Speed hack via DLL injection** -- extracts a `SpeedHack.dll` from embedded resources, injects it into the pinball process using `CreateRemoteThread` + `LoadLibraryA`, which hooks three Windows timing functions:
  - `GetTickCount` (kernel32.dll)
  - `timeGetTime` (winmm.dll)
  - `QueryPerformanceCounter` (kernel32.dll)
- **Configurable speed** -- trackbar controls for acceleration multiplier (0x to 1000x) and sleep interval, with real-time communication between the trainer and injected DLL via shared memory addresses
- **Auto-detection** -- a polling timer detects whether `pinball.exe` is running by searching for its window class (`1c7c22a0-9576-11ce-bf80-444553540000`) and automatically enables/disables all controls
- **Built-in game installer** -- extracts a bundled Pinball installer from embedded resources and launches it on demand
- **Skinnable UI** -- uses the AlphaControls (`sSkinManager`) component library for themed visual appearance

## Technologies

- Object Pascal (Borland Delphi 7)
- Windows API (`ReadProcessMemory`, `WriteProcessMemory`, `OpenProcess`, `CreateRemoteThread`, `VirtualAllocEx`, `CreateToolHelp32SnapShot`)
- x86 inline assembly (for API function hooking via JMP patches)
- DLL injection (`LoadLibraryA` via remote thread)
- AlphaControls component suite (skinned UI controls)

## Project Structure

```
pinball-trainer-plus/
├── PTP.dpr                    # Delphi project file (application entry point)
├── PTP.res                    # Compiled project resources
├── UPTP.pas                   # Main form unit (trainer logic, memory R/W, DLL injection)
├── UPTP.dfm                   # Main form visual layout (Delphi Form)
├── SpeedHack/
│   ├── SpeedHack.dpr          # DLL project file (speed hack library)
│   ├── SpeedHack.res          # DLL compiled resources
│   └── USpeedHack.pas         # Speed hack implementation (API hooking, timing override)
├── Resources/
│   ├── SpeedHack.res          # Embedded SpeedHack.dll resource
│   └── Installer.res          # Embedded Pinball installer resource
├── Imgs/
│   └── Icon.ico               # Application icon
├── Clear.bat                   # Cleanup script for Delphi build artifacts
├── LICENSE
└── README.md
```

## How It Works

### Memory manipulation

The trainer opens the pinball process with `PROCESS_ALL_ACCESS` and uses `ReadProcessMemory`/`WriteProcessMemory` to follow pointer chains. For example, to read the score:

1. Read a 4-byte pointer from address `$01025040`
2. Add offset `$52` to the pointer value
3. Read the 4-byte score value from the computed address

### Speed hack mechanism

The speed hack works by injecting a DLL into the pinball process that replaces three Windows timing functions with custom implementations:

1. The trainer extracts `SpeedHack.dll` from its embedded resources to disk
2. It allocates memory in the target process with `VirtualAllocEx`
3. It writes the DLL path into the allocated memory with `WriteProcessMemory`
4. It creates a remote thread that calls `LoadLibraryA` with the DLL path
5. Once loaded, the DLL patches the first 5 bytes of `GetTickCount`, `timeGetTime`, and `QueryPerformanceCounter` with a `JMP` instruction (opcode `$E9`) redirecting to custom functions
6. A high-priority background thread increments a virtual tick counter based on the configured acceleration multiplier
7. The hooked functions return the virtual tick value instead of the real system time, making the game run faster or slower

### Trainer-DLL communication

The trainer and injected DLL communicate through shared memory addresses. The DLL reads the trainer's `SpeedHackSpeed`, `SpeedHackSleep`, and `SpeedHackActiv` variables by finding the trainer window (`TFrmPTPPrincipal`) and reading its process memory at fixed offsets.

## Installation

### Prerequisites

- Borland Delphi 7 (or compatible IDE such as Embarcadero RAD Studio)
- AlphaControls component suite installed in the IDE
- Windows XP/7 with 3D Pinball for Windows - Space Cadet installed

### Building

1. Open `PTP.dpr` in Delphi 7
2. Build the SpeedHack DLL: open `SpeedHack/SpeedHack.dpr` and compile
3. Build the main application: compile `PTP.dpr`

Alternatively, use the `Clear.bat` script to remove build artifacts before a clean rebuild.

## Contributing

This project is no longer actively maintained. You may submit small bug fixes or documentation improvements via Pull Request, but reviews may be infrequent and contributions are not guaranteed to be merged.

## License

This project is licensed under the terms specified in the [LICENSE](LICENSE) file.
