# Windows build (MSVC) — produce ofs-ng.exe with XY select + batch edit

This Linux box cannot produce a native Windows GUI `.exe`. Official Windows
builds use **MSVC** (see `.github/workflows/release.yml`). MinGW / cross-compile
is not a supported path for this SDL3 + OpenGL + libmpv + .NET host stack.

## Prerequisites (Windows)

- Visual Studio 2022 with **Desktop development with C++** (MSVC)
- CMake 3.16+
- Ninja (optional; VS generator also works)
- .NET SDK **10.0.x** (`winget install Microsoft.DotNet.SDK.10` or from https://dotnet.microsoft.com)
- Git with submodules

libmpv is **auto-downloaded** on the first Windows configure (no manual install).

## Exact commands

From a **x64 Native Tools** / Developer PowerShell for VS:

```powershell
cd <path-to-patched-ofs-ng>
git submodule update --init --recursive

cmake -S . -B build -G "Visual Studio 17 2022" -A x64 `
  -DCMAKE_BUILD_TYPE=Release `
  -DOFS_CLANG_TIDY=OFF

cmake --build build --config Release -j 8
```

Or with Ninja after `vcvars64.bat`:

```powershell
cmake -S . -B build -G Ninja -DCMAKE_BUILD_TYPE=Release -DOFS_CLANG_TIDY=OFF
cmake --build build -j 8
```

Output executable: `bin\Release\ofs-ng.exe` (multi-config) or `bin\ofs-ng.exe`
(single-config Ninja). Runtime assets (`data.pak`, managed plugins, libmpv DLL)
are placed next to the exe by the build.

## CI alternative

Push a branch/tag to GitHub and run workflow **Release** (`workflow_dispatch`,
`publish` unchecked) to get `dist-*` Windows zip artifacts without publishing a
GitHub Release. Requires pushing this patched tree (not done from the bot box).

## Smoke-tested on Linux (this box)

- Full Release build with Clang 18 succeeded (`bin/release/ofs-ng`).
- Unit tests: `SelectBox*`, `SetSelectionPosition*`, `OffsetSelectionTime*` passed.
