# Windows Build Tools Setup

This repository expects these commands to be available in `PATH`:

- `mingw32-make`
- `x86_64-w64-mingw32-clang`
- `clang-format`
- `clang-tidy`
- `python`

The steps below prepare that toolchain in a regular PowerShell environment.

## 1) Install MSYS2

Run PowerShell as your normal user and install MSYS2:

```powershell
winget install --id MSYS2.MSYS2 -e --accept-source-agreements --accept-package-agreements
```

Default install location used below: `C:\msys64`.

## 2) Update MSYS2 and install required packages

```powershell
$Bash = "C:\msys64\usr\bin\bash.exe"

# Core update (run again if MSYS2 asks for another pass)
& $Bash -lc "pacman -Syu --noconfirm"
& $Bash -lc "pacman -Su --noconfirm"

# Toolchain for this project
& $Bash -lc "pacman -S --needed --noconfirm \
  mingw-w64-ucrt-x86_64-clang \
  mingw-w64-ucrt-x86_64-clang-tools-extra \
  mingw-w64-ucrt-x86_64-make \
  mingw-w64-ucrt-x86_64-python"
```

## 3) Add required folders to user PATH

```powershell
$pathsToAdd = @(
  "$env:USERPROFILE\\bin",
  "C:\msys64\ucrt64\bin",
  "C:\msys64\usr\bin"
)

$current = [Environment]::GetEnvironmentVariable("Path", "User")
$newPath = $current
foreach ($p in $pathsToAdd) {
  if ($newPath -notlike "*$p*") {
    if ([string]::IsNullOrEmpty($newPath)) { $newPath = $p } else { $newPath = "$newPath;$p" }
  }
}
[Environment]::SetEnvironmentVariable("Path", $newPath, "User")
```

Close and reopen PowerShell after updating PATH.

## 4) Add compiler name shims expected by this Makefile

MSYS2 provides `clang.exe`, but this project's `Makefile` calls prefixed compiler names.
Create wrappers in `%USERPROFILE%\bin`:

```powershell
$shimDir = "$env:USERPROFILE\bin"
New-Item -ItemType Directory -Force -Path $shimDir | Out-Null

@'
@echo off
clang --target=x86_64-w64-mingw32 %*
'@ | Set-Content -Encoding Ascii "$shimDir\x86_64-w64-mingw32-clang.cmd"

@'
@echo off
clang --target=aarch64-w64-mingw32 %*
'@ | Set-Content -Encoding Ascii "$shimDir\aarch64-w64-mingw32-clang.cmd"
```

`mingw32-make.exe` is installed by `mingw-w64-ucrt-x86_64-make`.

## 5) Verify toolchain

```powershell
Get-Command python, mingw32-make, x86_64-w64-mingw32-clang, clang-format, clang-tidy
```

If all commands resolve, test the build:

```powershell
mingw32-make ARCH=x86_64 CONF=Debug
```

Expected outputs:

- binaries in `Output/Debug_x86_64/AAS/`
- no `command not found` errors for make/clang/clang-tidy/clang-format

## 6) Optional: deploy prerequisites

`deploy` target requires `mt.exe` (Windows SDK manifest tool). If `Get-Command mt.exe`
fails, install Windows SDK:

```powershell
winget install --id Microsoft.WindowsSDK.10.0.18362 -e --accept-source-agreements --accept-package-agreements
```

Then add the installed `mt.exe` directory to PATH:

```powershell
$mt = Get-ChildItem "C:\Program Files (x86)\Windows Kits\10\bin" -Recurse -File -Filter mt.exe |
  Where-Object { $_.FullName -match '\\x64\\mt\.exe$' } |
  Sort-Object FullName -Descending |
  Select-Object -First 1

if (-not $mt) { throw "mt.exe not found. Re-run SDK install." }

$mtDir = $mt.Directory.FullName
$userPath = [Environment]::GetEnvironmentVariable("Path", "User")
if ($userPath -notlike "*$mtDir*") {
  [Environment]::SetEnvironmentVariable("Path", "$userPath;$mtDir", "User")
}
$env:Path = "$env:Path;$mtDir"
Get-Command mt.exe
```

## Troubleshooting

- `mingw32-make` not found:
  - confirm `C:\msys64\ucrt64\bin` is in PATH and reopen shell.
- `x86_64-w64-mingw32-clang` not found:
  - confirm `%USERPROFILE%\bin` is in PATH and wrapper files exist.
- `clang-tidy` or `clang-format` not found:
  - reinstall `mingw-w64-ucrt-x86_64-clang-tools-extra` and `mingw-w64-ucrt-x86_64-clang`.
- `mt.exe` not found:
  - install `Microsoft.WindowsSDK.10.0.18362`, then run the PATH snippet in section 6.
