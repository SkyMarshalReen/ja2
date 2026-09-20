# Building Jagged Alliance 2

The original Visual C++ 6 workspace remains in `ja2/Build` for reference. New
development should use the CMake build at the repository root.

## Requirements

- Windows 10 or 11
- Visual Studio 2022 with **Desktop development with C++**
- The x86 MSVC toolset and a Windows SDK
- CMake 3.25 or newer (the Visual Studio C++ workload can install it)
- A legally obtained Jagged Alliance 2 data installation for running the game

The checked-in `mss32.lib` and `SMACKW32.LIB` dependencies are old 32-bit
libraries. Consequently, the supported configuration is **Win32**, not x64.

## Configure and build

From a Developer PowerShell for Visual Studio:

```powershell
cmake --preset vs2022-debug
cmake --build --preset debug
```

For an optimized build:

```powershell
cmake --preset vs2022-release
cmake --build --preset release
```

Build products are written below `out/build`. To stage the executable below
`out/install`, run:

```powershell
cmake --install out/build/vs2022-release --config Release
```

## Runtime data

This source archive does not contain all retail game data. Copy the resulting
`ja2.exe` into a compatible Jagged Alliance 2 installation, preserving that
installation's data and SLF archive files. Do not commit retail game assets.

## Compatibility policy

The build deliberately retains the static MSVC runtime and VC6-compatible
`wchar_t`/for-loop behavior. These settings reduce ABI changes in legacy binary
dependencies and binary save/map formats. They should only be removed as part
of a tested data-format and dependency migration.

Precompiled headers are initially disabled. The historical PCH files contain
absolute include paths and compiler-specific assumptions; correctness is more
important than build speed during the first modernization stage.

The legacy `ja2.rc` resource is also temporarily excluded because it depends on
the obsolete MFC `afxres.h` header. This only omits the embedded icon and version
resource; it does not affect gameplay. It can be restored after converting the
resource script to Windows SDK headers.
