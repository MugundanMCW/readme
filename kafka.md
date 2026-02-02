## Detailed Changes

### Change 1: setup.py - Add LIBRDKAFKA_DIR Support

**File**: `setup.py`  
**Lines Added**: 31  
**Purpose**: Detect custom librdkafka installation and bundle DLLs

#### Original Code (Lines 1-20)
```python
#!/usr/bin/env python

import os
import platform

from setuptools import Extension, setup

work_dir = os.path.dirname(os.path.realpath(__file__))
mod_dir = os.path.join(work_dir, 'src', 'confluent_kafka')
ext_dir = os.path.join(mod_dir, 'src')

# On Un*x the library is linked as -lrdkafka,
# while on windows we need the full librdkafka name.
if platform.system() == 'Windows':
    librdkafka_libname = 'librdkafka'
else:
    librdkafka_libname = 'rdkafka'

module = Extension(
    'confluent_kafka.cimpl',
    libraries=[librdkafka_libname],
    sources=[
        os.path.join(ext_dir, 'confluent_kafka.c'),
        # ... more sources
    ],
)

setup(ext_modules=[module])
```

#### Modified Code
```python
#!/usr/bin/env python

import os
import platform
import shutil

from setuptools import Extension, setup

work_dir = os.path.dirname(os.path.realpath(__file__))
mod_dir = os.path.join(work_dir, 'src', 'confluent_kafka')

# On Un*x the library is linked as -lrdkafka,
# while on windows we need the full librdkafka name.
# Note: CMake builds produce 'rdkafka.lib', NuGet packages use 'librdkafka.lib'
if platform.system() == 'Windows':
    librdkafka_libname = 'librdkafka'
else:
    librdkafka_libname = 'rdkafka'

# Add support for LIBRDKAFKA_DIR environment variable
include_dirs = []
library_dirs = []

if platform.system() == 'Windows':
    librdkafka_dir = os.environ.get('LIBRDKAFKA_DIR')
    if librdkafka_dir:
        include_dirs = [
            os.path.join(librdkafka_dir, 'include', 'librdkafka'),
            os.path.join(librdkafka_dir, 'include')
        ]
        library_dirs = [os.path.join(librdkafka_dir, 'lib')]
        
        # Auto-detect library name (CMake uses 'rdkafka', NuGet uses 'librdkafka')
        if os.path.exists(os.path.join(library_dirs[0], 'rdkafka.lib')):
            librdkafka_libname = 'rdkafka'
        elif os.path.exists(os.path.join(library_dirs[0], 'librdkafka.lib')):
            librdkafka_libname = 'librdkafka'
        
        # Copy DLLs to package directory for bundling
        dll_dir = os.path.join(librdkafka_dir, 'bin')
        if os.path.exists(dll_dir):
            dest_dir = mod_dir
            for dll_file in os.listdir(dll_dir):
                if dll_file.endswith('.dll'):
                    src = os.path.join(dll_dir, dll_file)
                    dst = os.path.join(dest_dir, dll_file)
                    if not os.path.exists(dst) or os.path.getmtime(src) > os.path.getmtime(dst):
                        print(f"Copying {dll_file} to package directory")
                        shutil.copy2(src, dst)

module = Extension(
    'confluent_kafka.cimpl',
    libraries=[librdkafka_libname],
    include_dirs=include_dirs,
    library_dirs=library_dirs,
    sources=[
        # IMPORTANT: Use relative paths with forward slashes
        'src/confluent_kafka/src/confluent_kafka.c',
        'src/confluent_kafka/src/Producer.c',
        'src/confluent_kafka/src/Consumer.c',
        'src/confluent_kafka/src/Metadata.c',
        'src/confluent_kafka/src/AdminTypes.c',
        'src/confluent_kafka/src/Admin.c',
    ],
)

setup(ext_modules=[module])
```

#### Key Changes Explained

1. **Import shutil** (Line 5)
   - **Why**: Needed for copying DLL files
   - **Function**: `shutil.copy2()` preserves file metadata

2. **Check LIBRDKAFKA_DIR** (Lines 24-26)
   - **Why**: Allow users to specify custom librdkafka location
   - **Fallback**: If not set, existing behavior continues (NuGet)

3. **Set include_dirs and library_dirs** (Lines 27-31)
   - **Why**: Tell compiler where to find headers and libs
   - **Structure**: Matches CMake install layout

4. **Auto-detect Library Name** (Lines 33-37)
   - **Problem**: CMake produces `rdkafka.lib`, NuGet uses `librdkafka.lib`
   - **Solution**: Check which file exists and use appropriate name
   - **Why Important**: Linker fails if wrong name is used

5. **Copy DLLs** (Lines 39-48)
   - **Problem**: DLLs must be in wheel for runtime loading
   - **Solution**: Copy from `bin/` to package directory before build
   - **Optimization**: Only copy if source is newer (timestamp check)

6. **Use Relative Paths** (Lines 56-61)
   - **Problem**: `os.path.join()` creates absolute paths on Windows
   - **Error**: `setup() arguments must *always* be /-separated paths relative to the setup.py directory`
   - **Solution**: Use forward-slash strings like `'src/confluent_kafka/src/Admin.c'`

#### Why This Works

```
Before (x86/x64):
setup.py → Finds libs in Python installation → Builds extension → DLLs added by windows-build.bat

After (ARM64):
setup.py → Finds libs via LIBRDKAFKA_DIR → Copies DLLs to package → Builds extension → DLLs already in package
```

---

### Change 2: pyproject.toml - Enable Package Data

**File**: `pyproject.toml`  
**Lines Added**: 4  
**Purpose**: Tell setuptools to include DLL files in wheel

#### Original Code (Line 100)
```toml
[tool.setuptools]
include-package-data = false
```

#### Modified Code
```toml
[tool.setuptools]
include-package-data = true

[tool.setuptools.package-data]
confluent_kafka = ["*.dll"]
```

#### Why This Change is Needed

1. **Default Behavior**: setuptools excludes non-Python files
2. **Our Need**: DLLs must be included for runtime loading
3. **Solution**: Explicitly declare DLLs as package data

#### What Happens Without This

```
Wheel contents:
confluent_kafka/
├── __init__.py          ✅ Included
├── cimpl.pyd            ✅ Included (C extension)
├── rdkafka.dll          ❌ EXCLUDED (not Python file)
└── rdkafka++.dll        ❌ EXCLUDED (not Python file)

Result: ImportError at runtime
```

#### What Happens With This

```
Wheel contents:
confluent_kafka/
├── __init__.py          ✅ Included
├── cimpl.pyd            ✅ Included
├── rdkafka.dll          ✅ Included (via package_data)
└── rdkafka++.dll        ✅ Included (via package_data)

Result: Import succeeds
```

---

### Change 3: tools/windows-install-librdkafka.bat - Skip NuGet for Custom Builds

**File**: `tools/windows-install-librdkafka.bat`  
**Lines Added**: 5  
**Purpose**: Skip NuGet download when using custom librdkafka

#### Original Code
```batch
echo on
set librdkafka_version=%1
set outdir=%2

nuget install librdkafka.redist -version %librdkafka_version% -OutputDirectory %outdir%
```

#### Modified Code
```batch
echo on
set librdkafka_version=%1
set outdir=%2

rem Check if LIBRDKAFKA_DIR is set (for custom builds like ARM64)
if defined LIBRDKAFKA_DIR (
    echo Using custom librdkafka from LIBRDKAFKA_DIR: %LIBRDKAFKA_DIR%
    exit /b 0
)

nuget install librdkafka.redist -version %librdkafka_version% -OutputDirectory %outdir%
```

#### Why This Change is Needed

1. **Problem**: NuGet doesn't have ARM64 binaries
2. **Solution**: If user provides custom librdkafka, skip NuGet entirely
3. **Benefit**: Allows ARM64 builds without modifying NuGet logic

#### Flow Diagram

```
┌─────────────────────────────────────┐
│ windows-install-librdkafka.bat      │
├─────────────────────────────────────┤
│                                      │
│  Is LIBRDKAFKA_DIR set?             │
│  ├─ YES → Use custom build (ARM64)  │
│  │         Exit early                │
│  │                                   │
│  └─ NO  → Download from NuGet       │
│            (x86/x64)                 │
│                                      │
└─────────────────────────────────────┘
```

---

### Change 4: tools/windows-build.bat - Handle Custom librdkafka

**File**: `tools/windows-build.bat`  
**Lines Added**: 24  
**Purpose**: Support custom librdkafka in wheel building process

#### Key Additions

```batch
rem Check if LIBRDKAFKA_DIR is already set (for custom builds like ARM64)
if not defined LIBRDKAFKA_DIR (
    rem Download and install librdkafka from NuGet.
    call tools\windows-install-librdkafka.bat %LIBRDKAFKA_NUGET_VERSION% dest || exit /b 1
) else (
    echo Using custom librdkafka from: %LIBRDKAFKA_DIR%
)
```

And later:

```batch
rem Handle custom LIBRDKAFKA_DIR (for ARM64 and other custom builds)
if defined LIBRDKAFKA_DIR (
    rem Detect architecture
    python -c "import platform; arch = platform.machine().lower(); print('arm64' if 'arm' in arch or 'aarch64' in arch else ('x64' if 'amd64' in arch or 'x86_64' in arch else 'x86'))" > arch.txt
    set /p DETECTED_ARCH=<arch.txt
    del arch.txt
    
    echo Detected architecture: %DETECTED_ARCH%
    
    md stage\%DETECTED_ARCH%\confluent_kafka
    copy %LIBRDKAFKA_DIR%\bin\*.dll stage\%DETECTED_ARCH%\confluent_kafka\ || exit /b 1
    
    cd stage\%DETECTED_ARCH%
    for %%W in (..\..\wheelhouse\*.whl) do (
        7z a -r %%~W confluent_kafka\*.dll || exit /b 1
        unzip -l %%~W
    )
    cd ..\..
) else (
    rem ... existing x86/x64 logic ...
)
```

#### Why This is Important

This script is used by CI/CD (`cibuildwheel`) to create wheels. While our `setup.py` changes handle DLL bundling for `pip install .`, this ensures the same works in automated builds.

---

### Change 5: tools/windows-install-librdkafka-arm64.ps1 (NEW FILE)

**File**: `tools/windows-install-librdkafka-arm64.ps1` (NEW)  
**Lines**: 107  
**Purpose**: Automated script to build librdkafka for ARM64

#### Complete Script

```powershell
<#
.SYNOPSIS
Build librdkafka for Windows ARM64

.DESCRIPTION
This script builds librdkafka from source for Windows ARM64 architecture.
It requires Visual Studio 2022 with ARM64 build tools and CMake.

.PARAMETER Version
librdkafka version tag to build (default: v2.6.1)

.PARAMETER InstallDir
Installation directory (default: C:\librdkafka-ARM64)

.EXAMPLE
.\windows-install-librdkafka-arm64.ps1
.\windows-install-librdkafka-arm64.ps1 -Version v2.6.1 -InstallDir C:\librdkafka-ARM64
#>

param(
    [string]$Version = "v2.6.1",
    [string]$InstallDir = "C:\librdkafka-ARM64"
)

$ErrorActionPreference = "Stop"

Write-Host "Building librdkafka $Version for Windows ARM64..." -ForegroundColor Cyan

# Check if already exists
if (Test-Path $InstallDir) {
    Write-Host "Using existing installation at $InstallDir" -ForegroundColor Green
    $env:LIBRDKAFKA_DIR = $InstallDir
    exit 0
}

# Create temp directory
$TempDir = Join-Path $env:TEMP "librdkafka-build-$(Get-Random)"
New-Item -ItemType Directory -Force -Path $TempDir | Out-Null
Write-Host "Build directory: $TempDir" -ForegroundColor Gray

try {
    Set-Location $TempDir
    
    # Clone librdkafka
    Write-Host "Cloning librdkafka..." -ForegroundColor Yellow
    git clone --depth 1 --branch $Version https://github.com/confluentinc/librdkafka.git
    if ($LASTEXITCODE -ne 0) { throw "Git clone failed" }
    
    Set-Location librdkafka
    
    # Create build directory
    New-Item -ItemType Directory -Force -Path "build-arm64" | Out-Null
    Set-Location build-arm64
    
    # Configure with CMake for ARM64
    Write-Host "Configuring CMake for ARM64..." -ForegroundColor Yellow
    cmake .. -G "Visual Studio 17 2022" -A ARM64 `
        -DCMAKE_BUILD_TYPE=Release `
        -DCMAKE_INSTALL_PREFIX="$InstallDir" `
        -DRDKAFKA_BUILD_STATIC=OFF `
        -DRDKAFKA_BUILD_EXAMPLES=OFF `
        -DRDKAFKA_BUILD_TESTS=OFF `
        -DWITH_SSL=OFF `
        -DWITH_ZLIB=OFF `
        -DWITH_ZSTD=OFF `
        -DWITH_SASL=OFF `
        -DENABLE_LZ4_EXT=OFF
    
    if ($LASTEXITCODE -ne 0) { throw "CMake configuration failed" }
    
    # Build
    Write-Host "Building librdkafka..." -ForegroundColor Yellow
    cmake --build . --config Release --parallel
    if ($LASTEXITCODE -ne 0) { throw "Build failed" }
    
    # Install
    Write-Host "Installing to $InstallDir..." -ForegroundColor Yellow
    cmake --install . --config Release
    if ($LASTEXITCODE -ne 0) { throw "Installation failed" }
    
    Write-Host "`nBuild successful!" -ForegroundColor Green
    Write-Host "Installation directory: $InstallDir" -ForegroundColor Cyan
    
    # Set environment variable
    $env:LIBRDKAFKA_DIR = $InstallDir
    [Environment]::SetEnvironmentVariable("LIBRDKAFKA_DIR", $InstallDir, "Process")
    
    # Verify installation
    $dllPath = Join-Path $InstallDir "bin\rdkafka.dll"
    if (Test-Path $dllPath) {
        Write-Host "Verified: DLL found at $dllPath" -ForegroundColor Green
    } else {
        Write-Warning "DLL not found at expected location"
    }
    
} catch {
    Write-Host "`nBuild failed: $($_.Exception.Message)" -ForegroundColor Red
    exit 1
} finally {
    # Cleanup
    Set-Location $env:TEMP
    if (Test-Path $TempDir) {
        Remove-Item -Recurse -Force $TempDir -ErrorAction SilentlyContinue
    }
}

Write-Host "`nTo use this build, set: `$env:LIBRDKAFKA_DIR = '$InstallDir'" -ForegroundColor Cyan
exit 0
```

#### Script Breakdown

1. **Parameters** (Lines 20-23)
   - `Version`: Git tag to checkout (default: v2.6.1)
   - `InstallDir`: Where to install (default: C:\librdkafka-ARM64)

2. **Check Existing Installation** (Lines 29-33)
   - Avoids rebuilding if already exists
   - Saves time during development

3. **Clone Repository** (Lines 43-46)
   - Uses `--depth 1` for faster clone
   - Checks out specific version tag

4. **CMake Configuration** (Lines 54-66)
   - `-G "Visual Studio 17 2022"`: Use VS2022 generator
   - `-A ARM64`: Target ARM64 architecture
   - `-DCMAKE_BUILD_TYPE=Release`: Optimized build
   - `-DRDKAFKA_BUILD_STATIC=OFF`: Build shared library (DLL)
   - `-DWITH_SSL=OFF`: Disable SSL (simplifies dependencies)
   - Other flags: Disable optional features

5. **Build** (Lines 71-72)
   - `--config Release`: Build release configuration
   - `--parallel`: Use multiple CPU cores

6. **Install** (Lines 75-77)
   - Copies files to `$InstallDir`
   - Creates proper directory structure

7. **Verification** (Lines 88-92)
   - Checks if DLL was created
   - Provides feedback to user

8. **Cleanup** (Lines 96-100)
   - Removes temporary build directory
   - Saves disk space

#### Why CMake Configuration Matters

```cmake
-DWITH_SSL=OFF          # Avoids needing OpenSSL for ARM64
-DWITH_ZLIB=OFF         # Avoids needing zlib for ARM64
-DWITH_ZSTD=OFF         # Avoids needing zstd for ARM64
-DWITH_SASL=OFF         # Avoids needing SASL for ARM64
-DENABLE_LZ4_EXT=OFF    # Avoids needing lz4 for ARM64
```

**Trade-off**: Minimal build without compression/encryption support  
**Benefit**: No external dependencies needed  
**For Production**: Enable these features and build dependencies for ARM64

---

### Change 6: README.md - Document ARM64 Support

**File**: `README.md`  
**Lines Added**: 20  
**Purpose**: User-facing documentation

#### Addition (Insert after "Install from source" section)

```markdown
#### Windows ARM64

Windows ARM64 requires building librdkafka from source as pre-built binaries are not yet available:

**Prerequisites:**
- Visual Studio 2022 with ARM64 build tools
- CMake

**Build steps:**
```powershell
# Build librdkafka for ARM64
.\tools\windows-install-librdkafka-arm64.ps1

# Set environment variable
$env:LIBRDKAFKA_DIR = "C:\librdkafka-ARM64"

# Install confluent-kafka-python
pip install .
```

For more details, see [INSTALL.md](INSTALL.md#windows-arm64).
```

---

### Change 7: INSTALL.md - Detailed ARM64 Instructions

**File**: `INSTALL.md`  
**Lines Added**: 60  
**Purpose**: Comprehensive installation guide

#### Addition (Insert in Windows section)

```markdown
### Windows ARM64

Windows ARM64 support requires building librdkafka from source.

#### Prerequisites

- **Visual Studio 2022** with:
  - Desktop development with C++
  - ARM64 build tools
  - Windows 11 SDK
- **CMake** 3.15 or later
- **Git**
- **Python 3.8+** for ARM64

#### Build librdkafka

Run the provided PowerShell script:

```powershell
.\tools\windows-install-librdkafka-arm64.ps1
```

This will:
1. Clone librdkafka from GitHub
2. Build it for ARM64 using CMake
3. Install to `C:\librdkafka-ARM64`
4. Set the `LIBRDKAFKA_DIR` environment variable

#### Build confluent-kafka-python

```powershell
# Ensure LIBRDKAFKA_DIR is set
$env:LIBRDKAFKA_DIR = "C:\librdkafka-ARM64"

# Add DLLs to PATH for runtime
$env:PATH = "C:\librdkafka-ARM64\bin;$env:PATH"

# Install
pip install .
```

#### Verify Installation

```powershell
python -c "from confluent_kafka import libversion; print(libversion())"
```

#### Troubleshooting

**Issue:** `librdkafka not found during build`

**Solution:** Ensure `LIBRDKAFKA_DIR` is set:
```powershell
$env:LIBRDKAFKA_DIR = "C:\librdkafka-ARM64"
```

**Issue:** `DLL not found at runtime`

**Solution:** Add librdkafka bin directory to PATH:
```powershell
$env:PATH = "C:\librdkafka-ARM64\bin;$env:PATH"
```

**Issue:** `CMake not found`

**Solution:** Install CMake:
```powershell
winget install Kitware.CMake
```
```

---
# Building confluent-kafka-python for Windows ARM64 (Continued)

---

## Build Process

### Step-by-Step Build Instructions

#### Step 1: Clone Repository

```powershell
# Clone confluent-kafka-python
git clone https://github.com/confluentinc/confluent-kafka-python.git
cd confluent-kafka-python

# Verify you're on ARM64
python -c "import platform; print(f'Architecture: {platform.machine()}')"
# Expected output: Architecture: ARM64
```

#### Step 2: Build librdkafka for ARM64

```powershell
# Run the automated build script
.\tools\windows-install-librdkafka-arm64.ps1

# Expected output:
# Building librdkafka v2.6.1 for Windows ARM64...
# Build directory: C:\Users\...\Temp\librdkafka-build-...
# Cloning librdkafka...
# Configuring CMake for ARM64...
# Building librdkafka...
# Installing to C:\librdkafka-ARM64...
# Build successful!
# Installation directory: C:\librdkafka-ARM64
```

**What This Does:**
1. Creates temporary build directory in `%TEMP%`
2. Clones librdkafka repository from GitHub
3. Checks out version tag `v2.6.1`
4. Configures CMake for ARM64 architecture
5. Compiles librdkafka using Visual Studio 2022
6. Installs to `C:\librdkafka-ARM64`
7. Cleans up temporary files

**Build Time:** Approximately 5-10 minutes depending on your system

**Disk Space Required:** 
- Temporary: ~500 MB (deleted after build)
- Final installation: ~50 MB

**Verification:**

```powershell
# Check installation directory
Get-ChildItem C:\librdkafka-ARM64

# Expected structure:
# C:\librdkafka-ARM64\
# ├── bin\
# │   ├── rdkafka.dll       (7 MB)
# │   └── rdkafka++.dll     (94 KB)
# ├── lib\
# │   ├── rdkafka.lib
# │   ├── rdkafka++.lib
# │   ├── cmake\
# │   └── pkgconfig\
# ├── include\
# │   └── librdkafka\
# │       ├── rdkafka.h
# │       ├── rdkafka_mock.h
# │       └── rdkafkacpp.h
# └── share\
#     └── licenses\

# Verify DLL architecture
dumpbin /headers C:\librdkafka-ARM64\bin\rdkafka.dll | findstr "machine"
# Expected output: AA64 machine (ARM64)
```

**Troubleshooting Build Failures:**

**Error: "CMake not found"**
```powershell
# Install CMake
winget install Kitware.CMake

# Verify installation
cmake --version
```

**Error: "Visual Studio not found" or "ARM64 build tools not found"**
```powershell
# Check Visual Studio installation
& "C:\Program Files\Microsoft Visual Studio\2022\Community\Common7\Tools\VsDevCmd.bat" -arch=arm64

# If this fails, install ARM64 build tools:
# 1. Open Visual Studio Installer
# 2. Click "Modify" on Visual Studio 2022
# 3. Go to "Individual components"
# 4. Search for "ARM64"
# 5. Check "MSVC v143 - VS 2022 C++ ARM64 build tools"
# 6. Check "Windows 11 SDK (10.0.22621.0)"
# 7. Click "Modify" to install
```

**Error: "Git clone failed"**
```powershell
# Check internet connection
Test-NetConnection github.com -Port 443

# Try manual clone
git clone --depth 1 --branch v2.6.1 https://github.com/confluentinc/librdkafka.git
```

**Error: "Build failed" during compilation**
```
# Check build log for specific errors
# Common issues:
# 1. Insufficient disk space (need ~500 MB free)
# 2. Antivirus blocking compiler
# 3. Corrupted Visual Studio installation

# Try clean rebuild:
Remove-Item -Recurse -Force C:\librdkafka-ARM64 -ErrorAction SilentlyContinue
.\tools\windows-install-librdkafka-arm64.ps1
```

#### Step 3: Set Environment Variables

```powershell
# Set LIBRDKAFKA_DIR (required for build)
$env:LIBRDKAFKA_DIR = "C:\librdkafka-ARM64"

# Add to PATH (required for runtime)
$env:PATH = "C:\librdkafka-ARM64\bin;$env:PATH"

# Verify environment variables
Write-Host "LIBRDKAFKA_DIR: $env:LIBRDKAFKA_DIR"
Write-Host "PATH includes librdkafka: $($env:PATH -like '*librdkafka*')"
```

**Why Both Variables Are Needed:**

| Variable | Purpose | Used By | When |
|----------|---------|---------|------|
| `LIBRDKAFKA_DIR` | Tell setup.py where to find headers and libs | setup.py | Build time |
| `PATH` | Tell Windows where to find DLLs | Python runtime | Runtime |

**Making Environment Variables Persistent:**

For the current session only (temporary):
```powershell
$env:LIBRDKAFKA_DIR = "C:\librdkafka-ARM64"
$env:PATH = "C:\librdkafka-ARM64\bin;$env:PATH"
```

For all future sessions (permanent):
```powershell
# Set user environment variable
[Environment]::SetEnvironmentVariable("LIBRDKAFKA_DIR", "C:\librdkafka-ARM64", "User")

# Add to user PATH
$currentPath = [Environment]::GetEnvironmentVariable("PATH", "User")
$newPath = "C:\librdkafka-ARM64\bin;$currentPath"
[Environment]::SetEnvironmentVariable("PATH", $newPath, "User")

# Restart PowerShell for changes to take effect
```

#### Step 4: Clean Previous Builds (Optional but Recommended)

```powershell
# Remove any previous build artifacts
Remove-Item -Recurse -Force build, dist, *.egg-info -ErrorAction SilentlyContinue

# Remove any previously copied DLLs
Remove-Item src\confluent_kafka\*.dll -ErrorAction SilentlyContinue

# Verify clean state
Get-ChildItem -Recurse -Include *.dll, *.pyd | Where-Object { $_.FullName -like "*confluent_kafka*" }
# Should return nothing
```

#### Step 5: Build confluent-kafka-python Wheel

```powershell
# Upgrade build tools (recommended)
pip install --upgrade pip setuptools wheel

# Build the wheel
python setup.py bdist_wheel

# Expected output:
# Copying rdkafka++.dll to package directory
# Copying rdkafka.dll to package directory
# running bdist_wheel
# running build
# running build_py
# creating build\lib.win-arm64-cpython-312\confluent_kafka
# ...
# running build_ext
# building 'confluent_kafka.cimpl' extension
# ...
# creating dist\confluent_kafka-2.13.0-cp312-cp312-win_arm64.whl
```

**Build Output Explained:**

1. **"Copying rdkafka.dll to package directory"**
   - This is our setup.py copying DLLs from `C:\librdkafka-ARM64\bin\`
   - Destination: `src\confluent_kafka\`
   - These will be bundled into the wheel

2. **"running build_py"**
   - Copies Python source files to build directory
   - Creates package structure

3. **"running build_ext"**
   - Compiles C extension (`cimpl.pyd`)
   - Links against `rdkafka.lib`
   - This is the core Kafka client implementation

4. **"creating dist\confluent_kafka-2.13.0-cp312-cp312-win_arm64.whl"**
   - Final wheel file created
   - `cp312`: Python 3.12
   - `win_arm64`: Windows ARM64 platform tag

**Build Time:** 1-2 minutes

**Troubleshooting Build Errors:**

**Error: "error: Error: setup script specifies an absolute path"**
```
Cause: Using os.path.join() for source files in setup.py
Solution: Already fixed in our setup.py (uses relative paths with forward slashes)
```

**Error: "librdkafka not found" or "rdkafka.h: No such file or directory"**
```powershell
# Verify LIBRDKAFKA_DIR is set
echo $env:LIBRDKAFKA_DIR
# Should output: C:\librdkafka-ARM64

# Verify headers exist
Test-Path C:\librdkafka-ARM64\include\librdkafka\rdkafka.h
# Should output: True

# If not set, set it:
$env:LIBRDKAFKA_DIR = "C:\librdkafka-ARM64"
```

**Error: "LINK : fatal error LNK1181: cannot open input file 'rdkafka.lib'"**
```powershell
# Verify lib file exists
Test-Path C:\librdkafka-ARM64\lib\rdkafka.lib
# Should output: True

# Check if setup.py detected it correctly
python -c "import os; print(os.path.exists(os.path.join(os.environ.get('LIBRDKAFKA_DIR', ''), 'lib', 'rdkafka.lib')))"
# Should output: True
```

**Error: "Microsoft Visual C++ 14.0 or greater is required"**
```powershell
# Install Visual Studio 2022 Build Tools
winget install Microsoft.VisualStudio.2022.BuildTools

# Or use full Visual Studio 2022
winget install Microsoft.VisualStudio.2022.Community
```

#### Step 6: Verify Wheel Contents

```powershell
# List wheel contents
$wheel = Get-ChildItem dist\*.whl | Select-Object -First 1
python -m zipfile -l $wheel.FullName

# Check for DLLs specifically
python -m zipfile -l $wheel.FullName | Select-String "\.dll"

# Expected output:
# confluent_kafka/rdkafka++.dll                  2026-01-07 08:10:50        94720
# confluent_kafka/rdkafka.dll                    2026-01-07 08:10:30      7031808
```

**What Should Be in the Wheel:**

```
confluent_kafka-2.13.0-cp312-cp312-win_arm64.whl
├── confluent_kafka/
│   ├── __init__.py                    ✅ Python package
│   ├── cimpl.pyd                      ✅ C extension (compiled)
│   ├── rdkafka.dll                    ✅ librdkafka runtime (7 MB)
│   ├── rdkafka++.dll                  ✅ librdkafka C++ runtime (94 KB)
│   ├── admin/                         ✅ Admin API
│   ├── schema_registry/               ✅ Schema Registry support
│   └── ... (other Python modules)
└── confluent_kafka-2.13.0.dist-info/
    ├── METADATA                       ✅ Package metadata
    ├── WHEEL                          ✅ Wheel metadata
    └── ... (other metadata files)
```

**Verify Wheel Metadata:**

```powershell
# Extract wheel metadata
$wheel = Get-ChildItem dist\*.whl | Select-Object -First 1
python -m zipfile -e $wheel.FullName temp_wheel
Get-Content temp_wheel\confluent_kafka-2.13.0.dist-info\WHEEL

# Expected output:
# Wheel-Version: 1.0
# Generator: bdist_wheel (0.XX.X)
# Root-Is-Purelib: false
# Tag: cp312-cp312-win_arm64

# Clean up
Remove-Item -Recurse -Force temp_wheel
```

**Important:** The `Tag: cp312-cp312-win_arm64` confirms this is an ARM64 wheel.

#### Step 7: Install the Wheel

```powershell
# Uninstall any existing installation
pip uninstall -y confluent-kafka

# Install from the wheel
pip install dist\confluent_kafka-2.13.0-cp312-cp312-win_arm64.whl

# Expected output:
# Processing c:\confluent-kafka-python\dist\confluent_kafka-2.13.0-cp312-cp312-win_arm64.whl
# Installing collected packages: confluent-kafka
# Successfully installed confluent-kafka-2.13.0
```

**Installation Locations:**

```powershell
# Find where package was installed
python -c "import confluent_kafka; print(confluent_kafka.__file__)"
# Example output: C:\Python-3.12\Lib\site-packages\confluent_kafka\__init__.py

# Verify DLLs were installed
$pkgDir = python -c "import os, confluent_kafka; print(os.path.dirname(confluent_kafka.__file__))"
Get-ChildItem $pkgDir\*.dll

# Expected output:
# rdkafka.dll
# rdkafka++.dll
```

#### Step 8: Test the Installation

```powershell
# Test 1: Import the module
python -c "import confluent_kafka; print('✓ Import successful')"

# Test 2: Check librdkafka version
python -c "from confluent_kafka import libversion; print(f'librdkafka version: {libversion()}')"
# Expected output: librdkafka version: ('2.13.0', 34406655)

# Test 3: Create a producer (basic functionality test)
python -c "from confluent_kafka import Producer; p = Producer({'bootstrap.servers': 'localhost:9092'}); print('✓ Producer created')"
# Note: This will work even without a Kafka broker running

# Test 4: Create a consumer
python -c "from confluent_kafka import Consumer; c = Consumer({'bootstrap.servers': 'localhost:9092', 'group.id': 'test'}); print('✓ Consumer created')"
```

**Expected Results:**

```
✓ Import successful
librdkafka version: ('2.13.0', 34406655)
✓ Producer created
✓ Consumer created
```

**If Tests Fail:**

**Error: "ImportError: DLL load failed while importing cimpl"**
```powershell
# This means DLLs are not in the wheel or PATH is wrong

# Check if DLLs are in package directory
$pkgDir = python -c "import os, confluent_kafka; print(os.path.dirname(confluent_kafka.__file__))" 2>$null
if ($pkgDir) {
    Get-ChildItem $pkgDir\*.dll
} else {
    Write-Host "Package not properly installed"
}

# If DLLs are missing, rebuild wheel:
Remove-Item -Recurse -Force build, dist
$env:LIBRDKAFKA_DIR = "C:\librdkafka-ARM64"
python setup.py bdist_wheel
pip install --force-reinstall dist\*.whl
```

**Error: "ModuleNotFoundError: No module named 'confluent_kafka'"**
```powershell
# Package not installed
pip install dist\confluent_kafka-*.whl
```

#### Step 9: Run Comprehensive Tests (Optional)

```powershell
# Install test dependencies
pip install pytest

# Run unit tests (excluding integration tests that need Kafka broker)
python -m pytest tests/ --ignore=tests/integration --ignore=tests/schema_registry -v

# Expected output:
# tests/test_*.py::test_* PASSED
# ...
# ====== X passed in Y.YY seconds ======
```
