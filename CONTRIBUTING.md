# Contributing

> **This project was discontinued in March 2013 and is no longer actively maintained.**
> The repository is preserved as a historical reference. No new features or bug fixes are planned.

## Historical Build Information

This project was built using the following tools and technologies:

- **Language:** Object Pascal (Borland Delphi 7)
- **IDE:** Borland Delphi 7
- **UI Components:** AlphaControls component suite (skinned UI)
- **Windows API:** `ReadProcessMemory`, `WriteProcessMemory`, `OpenProcess`, `CreateRemoteThread`, `VirtualAllocEx`
- **DLL Injection:** `CreateRemoteThread` + `LoadLibraryA`; the injected DLL hooks timing APIs with 5-byte JMP patches written directly over each function's entry point

### Build Steps (Historical)

1. Install Borland Delphi 7 with AlphaControls component suite
2. First compile the speed hack DLL: open `SpeedHack/SpeedHack.dpr` and compile
3. Then compile the main application: open `PTP.dpr` and compile

> **Note:** `Clear.bat` removes Delphi build artifacts.
