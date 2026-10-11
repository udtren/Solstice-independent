# Executable name (solstice.exe) — agent reference

Status: done and manually checked 2026-10-11 at the user's request (Windows). User-facing note:
[docs/test-builds.md](../test-builds.md).

## What changed

On Windows the application is started by two small stubs that call
`krita_main()` in `krita.dll` (`krita/windows_stub_main.cpp`). Their output
names changed from `krita` to `solstice`:

| File | Before | Now |
| --- | --- | --- |
| GUI launcher (`krita_windows_stub_exe`) | `bin/krita.exe` | `bin/solstice.exe` |
| Console launcher (`krita_windows_stub_com`) | `bin/krita.com` | `bin/solstice.com` |
| Version resource (`krita/versioninfo.rc.in`) | `InternalName` `krita`, `OriginalFilename` `krita.exe` | `solstice`, `solstice.exe` |

Kept on purpose:

- `krita.dll`, `libkrita*.dll`, plugin names, `kritarunner`, the CMake
  target names, the embedded manifest resource file name and the manifest
  assembly identity. They are invisible to users and renaming them touches
  every library and plugin.
- The application name `krita`. `krita_main()` now calls
  `QCoreApplication::setApplicationName("krita")` first: before `KAboutData`
  (component name `krita`) Qt would otherwise derive the name from the
  executable, `solstice`. The profile paths and config names are already
  explicit (`KisSolsticePaths`, `KConfig::setMainConfigName()`), so the
  settings stay in `%APPDATA%\Solstice\config\kritarc`.
- The single-instance key (`"Krita5"` + home) and Python's `Krita.instance()`.
- Linux and macOS: the executable target `krita` is unchanged (not validated
  for Solstice).

## Touch points

- `krita/CMakeLists.txt`: `OUTPUT_NAME "solstice"` for both stubs.
- `krita/versioninfo.rc.in`: internal and original file names.
- `krita/main.cc`: the application name at the start of `krita_main()`.
- `packaging/windows/package-complete.py`: copies, `objdump` checks,
  `windeployqt` inputs and debug splitting use `solstice.exe`/`solstice.com`.
  The unused `package-complete-msvc.py`, the NSIS installer and the MSIX
  scripts still name `krita.exe`; update them if they are ever used.
- `build-tools/github-actions/build-windows.ps1`: required outputs.
- `run-krita.bat`, `build-tools/paint-trace/run.cmd` (launcher and the
  running-process check `solstice`, `krita`).

## Development install

A rebuilt `_install\bin` contains `solstice.exe` and `solstice.com`. The old
`krita.exe`/`krita.com` from earlier installs are not removed by CMake and
still start the same `krita.dll`; delete them only with the user's consent.

Before installing, check that neither process runs:
`Get-Process | Where-Object { $_.ProcessName -match '^(krita|solstice)' }`.

## Checks

- `(Get-Item <install>\bin\solstice.exe).VersionInfo`: `ProductName`
  Solstice, `InternalName` solstice, `OriginalFilename` solstice.exe.
- Manual: start `solstice.exe`; the existing profile, settings, workspaces
  and resources are used (no first-start question); the taskbar and Task
  Manager show Solstice; `solstice.com --version` prints the version.
- Trial build: the ZIP has `bin/solstice.exe` and `bin/solstice.com`.
