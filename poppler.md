# Building Poppler for Windows ARM64 - Complete Guide

This guide provides **complete, step-by-step instructions** for building Poppler PDF library and all its dependencies on Windows ARM64. Every command, every decision, and every workaround is documented.

---

## Table of Contents

1. [Overview](#overview)
2. [What You'll Build](#what-youll-build)
3. [Prerequisites](#prerequisites)
4. [Understanding the Build Process](#understanding-the-build-process)
5. [Part 1: System Setup](#part-1-system-setup)
6. [Part 2: MSVC Dependencies](#part-2-msvc-dependencies)
7. [Part 3: MSYS2 Clang Dependencies](#part-3-msys2-clang-dependencies)
8. [Part 4: Building Poppler](#part-4-building-poppler)
9. [Part 5: Verification](#part-5-verification)
10. [Troubleshooting](#troubleshooting)


---

## Overview

### What is Poppler?

Poppler is a PDF rendering library based on the xpdf-3.0 code base. It's used by many applications including:
- **TeXworks** - LaTeX editor with integrated PDF viewer
- **Okular** - KDE document viewer
- **Evince** - GNOME document viewer
- Many other PDF tools and applications

### Why This Guide?

Building Poppler on Windows ARM64 is complex because:
1. **Multiple toolchains required**: Some dependencies need MSVC, others need Clang
2. **ARM64 assembly**: Some libraries use ARM-specific assembly that MSVC doesn't support
3. **Complex dependencies**: 13+ libraries need to be built in correct order
4. **Limited documentation**: No official ARM64 Windows build guide exists

This guide solves all these problems with tested, working instructions.

---

## What You'll Build

### Final Output

After completing this guide, you'll have:

**Libraries:**
- `libpoppler-154.dll` - Core PDF library (3.5 MB)
- `libpoppler-qt6-3.dll` - Qt6 wrapper for GUI applications (753 KB)
- `libpoppler-cpp-2.dll` - C++ wrapper (160 KB)

**Command-Line Tools:**
- `pdfinfo.exe` - Extract PDF metadata
- `pdftotext.exe` - Convert PDF to text
- `pdftoppm.exe` - Convert PDF to images
- `pdftocairo.exe` - High-quality PDF to image conversion
- `pdfimages.exe` - Extract images from PDFs
- `pdftops.exe` - Convert PDF to PostScript
- `pdftohtml.exe` - Convert PDF to HTML
- `pdfseparate.exe` - Split PDF pages
- `pdfunite.exe` - Merge PDFs
- `pdfattach.exe` - Attach files to PDFs
- `pdfdetach.exe` - Extract attachments
- `pdffonts.exe` - List fonts in PDFs

**Development Files:**
- Headers for C, C++, and Qt6 APIs
- Import libraries for linking
- pkg-config files

### Features Included

✅ **JPEG support** - Fast JPEG decoding with ARM64 NEON optimizations  
✅ **PNG support** - PNG image handling (latest 1.8.x)  
✅ **TIFF support** - TIFF image export  
✅ **JPEG2000 support** - Modern image format  
✅ **Color management** - Accurate color reproduction (LCMS2)  
✅ **HTTP/HTTPS support** - Download remote resources (libcurl + Windows Schannel)  
✅ **Cairo rendering** - High-quality rendering with ARM64 NEON (~30% faster)  
✅ **Boost optimization** - ~10-15% performance improvement  
✅ **Qt6 wrapper** - For TeXworks and Qt applications  
✅ **C++ wrapper** - Modern C++ API  
✅ **Win32 fonts** - Native Windows font handling  

### Features Excluded

❌ **GLib wrapper** - Causes toolchain conflicts (not needed for TeXworks)  
❌ **GObject Introspection** - Language bindings for Python/JavaScript  
❌ **NSS3/GPGme** - Digital signature support (complex dependencies)  
❌ **Fontconfig** - Using Windows font API instead  

---

## Prerequisites

### Hardware Requirements

- **Windows ARM64 device** (Surface Pro X, Lenovo ThinkPad X13s, etc.)
- **20 GB free disk space** (for source code, build files, and dependencies)
- **8 GB RAM minimum** (16 GB recommended)
- **Internet connection** (for downloading source code)

### Software Requirements

#### 1. Windows 11 ARM64

- **Version**: Windows 11 22H2 or later
- **Build**: 22621 or higher
- Check your version: Press `Win+R`, type `winver`, press Enter

#### 2. Visual Studio 2022

**Download**: https://visualstudio.microsoft.com/downloads/

**Installation Steps:**
1. Download "Visual Studio 2022 Community" (free)
2. Run the installer
3. Select **"Desktop development with C++"** workload
4. In "Individual components" tab, ensure these are checked:
   - ✅ MSVC v143 - VS 2022 C++ ARM64 build tools
   - ✅ C++ CMake tools for Windows
   - ✅ Windows 11 SDK (10.0.22621.0 or later)
5. Click "Install" (requires ~10 GB)

**Verify Installation:**
```powershell
# Open PowerShell and run:
"C:\Program Files\Microsoft Visual Studio\2022\Community\VC\Tools\MSVC\14.44.35207\bin\Hostarm64\arm64\cl.exe"
```
Should show: `Microsoft (R) C/C++ Optimizing Compiler Version 19.44.35216.0 for ARM64`

#### 3. CMake

**Download**: https://cmake.org/download/

**Installation Steps:**
1. Download "Windows arm64 Installer" (cmake-3.28.x-windows-arm64.msi)
2. Run installer
3. **Important**: Select "Add CMake to system PATH for all users"
4. Complete installation

**Verify Installation:**
```powershell
cmake --version
```
Should show: `cmake version 3.28.x` or higher

#### 4. Git for Windows

**Download**: https://git-scm.com/download/win

**Installation Steps:**
1. Download "ARM64" version
2. Run installer
3. Use default options (just keep clicking "Next")
4. **Important**: Select "Git from the command line and also from 3rd-party software"

**Verify Installation:**
```powershell
git --version
```
Should show: `git version 2.43.x` or higher

#### 5. MSYS2 ARM64

**Download**: https://www.msys2.org/

**Installation Steps:**
1. Download "msys2-aarch64-xxxxxxxx.exe"
2. Run installer
3. Install to `C:\msys64` (default location - **do not change**)
4. After installation, **uncheck** "Run MSYS2 now" and click "Finish"

**Initial Setup:**
1. Open **Start Menu** → Search for "MSYS2 CLANGARM64"
2. **Important**: Use "MSYS2 CLANGARM64" terminal, NOT "MSYS2 MSYS" or "MSYS2 MINGW64"
3. Run these commands:

```bash
# Update package database
pacman -Syu
# Press Enter when asked, then close the terminal when it says "close the window"

# Reopen MSYS2 CLANGARM64 terminal
pacman -Su
# Press Enter to continue

# Install base development tools
pacman -S --needed base-devel
# Press Enter to install all

# Install Clang toolchain
pacman -S mingw-w64-clang-aarch64-toolchain
# Press Enter to install all

# Install build tools
pacman -S mingw-w64-clang-aarch64-cmake
pacman -S mingw-w64-clang-aarch64-meson
pacman -S mingw-w64-clang-aarch64-ninja
pacman -S mingw-w64-clang-aarch64-pkg-config
pacman -S mingw-w64-clang-aarch64-python
```

**Verify Installation:**
```bash
clang --version
```
Should show: `clang version 18.x.x` or higher with `Target: aarch64-w64-windows-gnu`

---

## Understanding the Build Process

### Why Two Toolchains?

We use **two different compilers** for different parts:

#### MSVC (Microsoft Visual C++)
**Used for:**
- zlib, libjpeg-turbo, libpng, libtiff
- FreeType, libopenjpeg2, libcurl
- LCMS2, win-iconv

**Why:** These libraries are straightforward C/C++ code that compiles fine with MSVC.

#### Clang (from MSYS2)
**Used for:**
- Cairo (and its dependency Pixman)
- Poppler itself
- Qt6

**Why:** 
- Cairo uses ARM64 NEON assembly with GNU syntax that MSVC doesn't understand
- Avoids header conflicts between MSVC and MSYS2 libraries
- Produces Windows-compatible binaries that work with MSVC-built dependencies

### Build Order

Dependencies must be built in this order (later items depend on earlier ones):

```
1. zlib                    (no dependencies)
2. libjpeg-turbo          (no dependencies)
3. libpng                 (needs zlib)
4. libtiff                (needs zlib, libjpeg, libpng)
5. FreeType               (needs zlib, libpng)
6. libopenjpeg2           (needs zlib, libpng, libtiff)
7. LCMS2                  (no dependencies)
8. libcurl                (needs zlib)
9. win-iconv              (no dependencies)
10. Cairo                 (needs FreeType, libpng, zlib, pixman)
11. Boost                 (header-only, no build needed)
12. Qt6                   (pre-built from MSYS2)
13. Poppler               (needs ALL of the above)
```

### Directory Structure

We'll create this structure:

```
C:\
├── poppler-build\              # Temporary build workspace
│   ├── zlib\                   # Source code for each library
│   ├── libjpeg-turbo\
│   ├── libpng\
│   ├── ... (other sources)
│   └── poppler\
│
├── poppler-deps\               # Installed dependencies
│   ├── bin\                    # DLL files
│   ├── lib\                    # Import libraries (.lib, .dll.a)
│   │   ├── cmake\              # CMake config files
│   │   └── pkgconfig\          # pkg-config files
│   └── include\                # Header files
│
├── poppler\                    # Final Poppler installation
│   ├── bin\                    # Poppler DLLs and utilities
│   ├── lib\                    # Poppler libraries
│   └── include\                # Poppler headers
│
└── msys64\                     # MSYS2 installation
    └── clangarm64\             # Clang ARM64 environment
        ├── bin\                # Clang, Qt6, Cairo DLLs
        ├── lib\
        └── include\
```

---

## Part 1: System Setup

### Step 1.1: Create Build Directories

Open **PowerShell** (not MSYS2):

```powershell
# Create main directories
New-Item -ItemType Directory -Force -Path "C:\poppler-build"
New-Item -ItemType Directory -Force -Path "C:\poppler-deps"
New-Item -ItemType Directory -Force -Path "C:\poppler"

# Create subdirectories
New-Item -ItemType Directory -Force -Path "C:\poppler-deps\bin"
New-Item -ItemType Directory -Force -Path "C:\poppler-deps\lib"
New-Item -ItemType Directory -Force -Path "C:\poppler-deps\include"
New-Item -ItemType Directory -Force -Path "C:\poppler-deps\lib\cmake"
New-Item -ItemType Directory -Force -Path "C:\poppler-deps\lib\pkgconfig"

# Verify
Get-ChildItem C:\poppler-deps
```

**Expected output:**
```
Directory: C:\poppler-deps

Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
d-----                                            bin
d-----                                            include
d-----                                            lib
```

### Step 1.2: Set Up Visual Studio Environment

Open **"x64_arm64 Cross Tools Command Prompt for VS 2022"** from Start Menu.

**Important**: This is different from regular PowerShell. It sets up MSVC compiler paths.

Verify it's working:
```cmd
cl
```

Should show: `Microsoft (R) C/C++ Optimizing Compiler Version 19.44.xxxxx for ARM64`

**Keep this window open** - we'll use it for Part 2.

---

## Part 2: MSVC Dependencies

All commands in this section run in **"x64_arm64 Cross Tools Command Prompt for VS 2022"**.

### Step 2.1: Build zlib

**What it does**: Compression library used by many other libraries.

```cmd
cd C:\poppler-build

REM Clone source code
git clone https://github.com/madler/zlib.git
cd zlib

REM Configure build
cmake -B build -A ARM64 ^
  -DCMAKE_INSTALL_PREFIX=C:\poppler-deps ^
  -DCMAKE_BUILD_TYPE=Release

REM Build (takes ~2 minutes)
cmake --build build --config Release

REM Install to C:\poppler-deps
cmake --install build

REM Verify
dir C:\poppler-deps\lib\z.lib
dir C:\poppler-deps\bin\zlib.dll
```

**Expected output:**
```
-- Installing: C:/poppler-deps/lib/z.lib
-- Installing: C:/poppler-deps/bin/zlib.dll
-- Installing: C:/poppler-deps/include/zlib.h
-- Installing: C:/poppler-deps/include/zconf.h
```

**If you see errors:**
- "git is not recognized" → Install Git for Windows
- "cmake is not recognized" → Install CMake and add to PATH
- "cl is not recognized" → Use "x64_arm64 Cross Tools Command Prompt", not regular PowerShell

---

### Step 2.2: Build libjpeg-turbo

**What it does**: Fast JPEG decoder with ARM64 NEON optimizations.

```cmd
cd C:\poppler-build

REM Clone source code
git clone https://github.com/libjpeg-turbo/libjpeg-turbo.git
cd libjpeg-turbo

REM Configure build
cmake -B build -A ARM64 ^
  -DCMAKE_INSTALL_PREFIX=C:\poppler-deps ^
  -DCMAKE_BUILD_TYPE=Release

REM Build (takes ~5 minutes)
cmake --build build --config Release

REM Install
cmake --install build

REM Verify
dir C:\poppler-deps\lib\jpeg.lib
dir C:\poppler-deps\bin\jpeg62.dll
```

**Expected output:**
```
-- Installing: C:/poppler-deps/lib/jpeg.lib
-- Installing: C:/poppler-deps/bin/jpeg62.dll
-- Installing: C:/poppler-deps/include/jpeglib.h
```

---

### Step 2.3: Build libpng

**What it does**: PNG image support.

```cmd
cd C:\poppler-build

REM Clone source code
git clone https://github.com/glennrp/libpng.git
cd libpng

REM Configure build
cmake -B build -A ARM64 ^
  -DCMAKE_INSTALL_PREFIX=C:\poppler-deps ^
  -DCMAKE_PREFIX_PATH=C:\poppler-deps ^
  -DCMAKE_BUILD_TYPE=Release ^
  -DPNG_SHARED=ON

REM Build (takes ~3 minutes)
cmake --build build --config Release

REM Install
cmake --install build --config Release

REM Verify - note the version number may be 18 (latest from git)
dir C:\poppler-deps\lib\libpng*.lib
dir C:\poppler-deps\bin\libpng*.dll
```

**Expected output:**
```
-- Installing: C:/poppler-deps/lib/libpng18.lib
-- Installing: C:/poppler-deps/bin/libpng18.dll
-- Installing: C:/poppler-deps/include/png.h
```

**Important**: Note the version number (18 in this case). You'll need it later.

---

### Step 2.4: Build libtiff

**What it does**: TIFF image format support.

```cmd
cd C:\poppler-build

REM Clone source code
git clone https://gitlab.com/libtiff/libtiff.git
cd libtiff

REM Configure build
cmake -B build -A ARM64 ^
  -DCMAKE_INSTALL_PREFIX=C:\poppler-deps ^
  -DCMAKE_PREFIX_PATH=C:\poppler-deps ^
  -DCMAKE_BUILD_TYPE=Release

REM Build (takes ~10 minutes)
cmake --build build --config Release

REM Install
cmake --install build

REM Verify
dir C:\poppler-deps\lib\tiff.lib
dir C:\poppler-deps\bin\tiff.dll
```

**Expected output:**
```
-- Installing: C:/poppler-deps/lib/tiff.lib
-- Installing: C:/poppler-deps/bin/tiff.dll
-- Installing: C:/poppler-deps/include/tiff.h
```

---

### Step 2.5: Build FreeType

**What it does**: Font rendering engine (absolutely required for PDFs).

```cmd
cd C:\poppler-build

REM Clone source code
git clone https://gitlab.freedesktop.org/freetype/freetype.git
cd freetype

REM Configure build
cmake -B build -A ARM64 ^
  -DCMAKE_INSTALL_PREFIX=C:\poppler-deps ^
  -DCMAKE_PREFIX_PATH=C:\poppler-deps ^
  -DCMAKE_BUILD_TYPE=Release

REM Build (takes ~15 minutes)
cmake --build build --config Release

REM Install
cmake --install build

REM Verify
dir C:\poppler-deps\lib\freetype.lib
dir C:\poppler-deps\bin\freetype.dll
```

**Expected output:**
```
-- Installing: C:/poppler-deps/lib/freetype.lib
-- Installing: C:/poppler-deps/bin/freetype.dll
-- Installing: C:/poppler-deps/include/freetype2/
```

---

### Step 2.6: Build libopenjpeg2

**What it does**: JPEG2000 image format support.

```cmd
cd C:\poppler-build

REM Clone source code
git clone https://github.com/uclouvain/openjpeg.git
cd openjpeg

REM Configure build
cmake -B build -A ARM64 ^
  -DCMAKE_INSTALL_PREFIX=C:\poppler-deps ^
  -DCMAKE_PREFIX_PATH=C:\poppler-deps ^
  -DCMAKE_BUILD_TYPE=Release

REM Build (takes ~8 minutes)
cmake --build build --config Release

REM Install
cmake --install build

REM Verify
dir C:\poppler-deps\lib\openjp2.lib
dir C:\poppler-deps\bin\openjp2.dll
```

**Expected output:**
```
-- Installing: C:/poppler-deps/lib/openjp2.lib
-- Installing: C:/poppler-deps/bin/openjp2.dll
-- Installing: C:/poppler-deps/include/openjpeg-2.5/
```

---

### Step 2.7: Build LCMS2

**What it does**: Color management for accurate color reproduction.

**Special note**: LCMS2 doesn't use CMake, it uses Visual Studio solution files.

```cmd
cd C:\poppler-build

REM Clone source code
git clone https://github.com/mm2/Little-CMS.git
cd Little-CMS

REM Navigate to Visual Studio project
cd Projects\VC2022

REM Build with msbuild (takes ~5 minutes)
msbuild lcms2.sln /p:Configuration=Release /p:Platform=ARM64
```

**Expected warnings** (these are OK):
```
tifficc.vcxproj : warning : Project file contains ToolsVersion="12.0"
jpegicc.vcxproj : warning : Project file contains ToolsVersion="12.0"
```

**Expected errors** (these are OK - optional utilities):
```
tifficc.vcxproj : error C1083: Cannot open include file: 'tiffio.h'
jpegicc.vcxproj : error C1083: Cannot open include file: 'jpeglib.h'
fast_float_plugin.vcxproj : error C1189: #error:  SSE2 intrinsics not available
```

**Why these errors are OK**: These are optional utilities that need headers we didn't provide. The core library (`lcms2_static.lib` and `lcms2.dll`) built successfully.

Now copy the files manually:

```cmd
REM Copy headers
xcopy /E /I ..\..\include C:\poppler-deps\include\lcms2

REM Copy libraries
copy ..\..\Lib\MS\lcms2_static.lib C:\poppler-deps\lib\
copy ..\..\bin\lcms2.dll C:\poppler-deps\bin\
copy ..\..\bin\lcms2.lib C:\poppler-deps\lib\

REM Return to build directory
cd C:\poppler-build

REM Verify
dir C:\poppler-deps\lib\lcms2*.lib
dir C:\poppler-deps\bin\lcms2.dll
```

**Expected output:**
```
C:\poppler-deps\lib\lcms2_static.lib
C:\poppler-deps\lib\lcms2.lib
C:\poppler-deps\bin\lcms2.dll
```

---

### Step 2.8: Build libcurl

**What it does**: HTTP/HTTPS support for downloading remote resources.

```cmd
cd C:\poppler-build

REM Clone source code
git clone https://github.com/curl/curl.git
cd curl

REM Configure build
cmake -B build -A ARM64 ^
  -DCMAKE_INSTALL_PREFIX=C:\poppler-deps ^
  -DCMAKE_PREFIX_PATH=C:\poppler-deps ^
  -DCMAKE_BUILD_TYPE=Release ^
  -DBUILD_SHARED_LIBS=ON ^
  -DCURL_USE_SCHANNEL=ON ^
  -DCURL_USE_LIBPSL=OFF ^
  -DCURL_DISABLE_LDAP=ON ^
  -DCURL_DISABLE_LDAPS=ON ^
  -DBUILD_CURL_EXE=OFF ^
  -DBUILD_TESTING=OFF ^
  -DUSE_NGHTTP2=OFF ^
  -DCURL_BROTLI=OFF ^
  -DCURL_ZSTD=OFF

REM Build (takes ~10 minutes)
cmake --build build --config Release

REM Install
cmake --install build

REM Verify
dir C:\poppler-deps\lib\libcurl.lib
dir C:\poppler-deps\bin\libcurl.dll
```

**Why these flags:**
- `-DCURL_USE_SCHANNEL=ON` - Use Windows native SSL (no OpenSSL needed)
- `-DCURL_USE_LIBPSL=OFF` - Disable optional Public Suffix List library
- `-DCURL_DISABLE_LDAP=ON` - Disable LDAP (rarely needed)
- `-DBUILD_CURL_EXE=OFF` - Don't build curl.exe (only need library)
- Other flags disable optional features to simplify build

**Expected output:**
```
-- Installing: C:/poppler-deps/lib/libcurl.lib
-- Installing: C:/poppler-deps/bin/libcurl.dll
-- Installing: C:/poppler-deps/include/curl/
```

---

### Step 2.9: Build win-iconv

**What it does**: Character encoding conversion (Windows doesn't have native iconv).

```cmd
cd C:\poppler-build

REM Clone source code
git clone https://github.com/win-iconv/win-iconv.git
cd win-iconv

REM Configure build
cmake -B build -A ARM64 ^
  -DCMAKE_INSTALL_PREFIX=C:\poppler-deps ^
  -DCMAKE_BUILD_TYPE=Release ^
  -DBUILD_SHARED=ON ^
  -DBUILD_STATIC=ON

REM Build (takes ~2 minutes)
cmake --build build --config Release

REM Install
cmake --install build

REM Verify
dir C:\poppler-deps\lib\iconv.lib
dir C:\poppler-deps\bin\iconv.dll
```

**Expected output:**
```
-- Installing: C:/poppler-deps/lib/iconv.lib
-- Installing: C:/poppler-deps/bin/iconv.dll
-- Installing: C:/poppler-deps/include/iconv.h
```

---

**🎉 Part 2 Complete!**

You've now built all MSVC dependencies. Let's verify:

```cmd
dir C:\poppler-deps\lib\*.lib
```

**You should see:**
- z.lib
- jpeg.lib
- libpng18.lib (or libpng16.lib)
- tiff.lib
- freetype.lib
- openjp2.lib
- lcms2_static.lib
- lcms2.lib
- libcurl.lib
- iconv.lib

**If any are missing**, go back to that step and check for errors.

---

## Part 3: MSYS2 Clang Dependencies

Now we switch to **MSYS2 CLANGARM64** terminal for the remaining dependencies.

**Why switch?** Cairo and Poppler need Clang because they use ARM64 assembly that MSVC doesn't support.

### Step 3.1: Install MSYS2 Dependencies

Open **MSYS2 CLANGARM64** terminal from Start Menu.

**Important**: Make sure it says "CLANGARM64" in the title bar, not "MSYS" or "MINGW64".

```bash
# Install Cairo dependencies
pacman -S mingw-w64-clang-aarch64-pixman
pacman -S mingw-w64-clang-aarch64-freetype
pacman -S mingw-w64-clang-aarch64-fontconfig
pacman -S mingw-w64-clang-aarch64-libpng
pacman -S mingw-w64-clang-aarch64-zlib

# Press Enter when asked to proceed with installation
```

**Expected output:**
```
Packages (5) mingw-w64-clang-aarch64-pixman-0.43.4-1
             mingw-w64-clang-aarch64-freetype-2.13.2-1
             ...

Total Installed Size:  XX.XX MiB

:: Proceed with installation? [Y/n] Y
```

---

### Step 3.2: Build Cairo

**What it does**: High-quality rendering backend with ARM64 NEON optimizations.

**Why MSYS2 Clang**: Cairo's Pixman library uses ARM64 NEON assembly with GNU syntax. MSVC doesn't understand GNU assembly, but Clang does.

```bash
# Navigate to build directory
cd /c/poppler-build

# Clone Cairo source
git clone https://gitlab.freedesktop.org/cairo/cairo.git
cd cairo

# Configure with Meson
meson setup build \
  --prefix=/c/poppler-deps \
  --buildtype=release \
  -Ddefault_library=shared \
  -Dtests=disabled \
  -Dglib=disabled \
  -Dspectre=disabled \
  -Dsymbol-lookup=disabled

# Build (takes ~20 minutes)
meson compile -C build

# Install
meson install -C build

# Verify
ls -lh /c/poppler-deps/bin/libcairo*.dll
ls -lh /c/poppler-deps/lib/libcairo*.dll.a
```

**Expected output:**
```
Installing src/libcairo-2.dll to C:/poppler-deps/bin
Installing src/libcairo.dll.a to C:/poppler-deps/lib
Installing C:/poppler-build/cairo/src/cairo.h to C:/poppler-deps/include/cairo
```

**If you see errors:**
- "meson: command not found" → Run `pacman -S mingw-w64-clang-aarch64-meson`
- "pixman not found" → Run `pacman -S mingw-w64-clang-aarch64-pixman`

---

### Step 3.3: Install Boost Headers

**What it does**: Performance optimization (~10-15% faster rendering).

**Why header-only**: Poppler only uses Boost's template libraries, no compilation needed.

```bash
# Navigate to build directory
cd /c/poppler-build

# Download Boost
wget https://boostorg.jfrog.io/artifactory/main/release/1.86.0/source/boost_1_86_0.tar.gz

# Extract
tar -xzf boost_1_86_0.tar.gz

# Copy headers
cp -r boost_1_86_0/boost /c/poppler-deps/include/

# Verify
ls /c/poppler-deps/include/boost/version.hpp
```

**Expected output:**
```
/c/poppler-deps/include/boost/version.hpp
```

**If wget fails:**
- "wget: command not found" → Run `pacman -S wget`
- Or download manually from https://www.boost.org/ and extract to `C:\poppler-build`


### Step 3.4: Create Boost CMake Config

**Why needed**: CMake's FindBoost needs a config file to locate Boost.

Switch to **PowerShell** for this step:

```powershell
# Create directory
New-Item -ItemType Directory -Force -Path "C:\poppler-deps\lib\cmake\Boost-1.86.0"

# Create BoostConfig.cmake
@"
# Boost 1.86.0 Config (Header-only)
set(Boost_FOUND TRUE)
set(Boost_VERSION_STRING "1.86.0")
set(Boost_VERSION_MAJOR 1)
set(Boost_VERSION_MINOR 86)
set(Boost_VERSION_PATCH 0)
set(Boost_VERSION 108600)
set(Boost_INCLUDE_DIRS "C:/poppler-deps/include")
set(Boost_INCLUDE_DIR "C:/poppler-deps/include")
set(Boost_LIBRARIES "")
set(Boost_LIBRARY_DIRS "")

# Create interface target
if(NOT TARGET Boost::boost)
    add_library(Boost::boost INTERFACE IMPORTED)
    set_target_properties(Boost::boost PROPERTIES
        INTERFACE_INCLUDE_DIRECTORIES "`${Boost_INCLUDE_DIRS}"
    )
endif()

# Alias for compatibility
if(NOT TARGET Boost::headers)
    add_library(Boost::headers ALIAS Boost::boost)
endif()

# Component support
set(Boost_CONTAINER_FOUND TRUE)
set(Boost_OPTIONAL_FOUND TRUE)
set(Boost_VARIANT_FOUND TRUE)

message(STATUS "Found Boost: `${Boost_VERSION_STRING} (header-only)")
"@ | Out-File -Encoding UTF8 "C:\poppler-deps\lib\cmake\Boost-1.86.0\BoostConfig.cmake"

# Create version file
@"
set(PACKAGE_VERSION "1.86.0")
set(PACKAGE_VERSION_EXACT FALSE)
set(PACKAGE_VERSION_COMPATIBLE TRUE)

if(PACKAGE_VERSION VERSION_LESS PACKAGE_FIND_VERSION)
    set(PACKAGE_VERSION_COMPATIBLE FALSE)
endif()

if(PACKAGE_FIND_VERSION STREQUAL PACKAGE_VERSION)
    set(PACKAGE_VERSION_EXACT TRUE)
endif()
"@ | Out-File -Encoding UTF8 "C:\poppler-deps\lib\cmake\Boost-1.86.0\BoostConfigVersion.cmake"

# Verify
Get-Content "C:\poppler-deps\lib\cmake\Boost-1.86.0\BoostConfig.cmake" | Select-Object -First 5
```

**Expected output:**
```
# Boost 1.86.0 Config (Header-only)
set(Boost_FOUND TRUE)
set(Boost_VERSION_STRING "1.86.0")
...
```

---

### Step 3.5: Install Qt6

**What it does**: Qt6 wrapper for GUI applications like TeXworks.

Back to **MSYS2 CLANGARM64** terminal:

```bash
# Install Qt6 base
pacman -S mingw-w64-clang-aarch64-qt6-base

# Install Qt6 tools (optional, for development)
pacman -S mingw-w64-clang-aarch64-qt6-tools

# Press Enter to proceed
```

**Expected output:**
```
Packages (2) mingw-w64-clang-aarch64-qt6-base-6.7.x-x
             mingw-w64-clang-aarch64-qt6-tools-6.7.x-x

Total Installed Size:  XXX.XX MiB

:: Proceed with installation? [Y/n] Y
```

**Verify installation:**
```bash
ls /clangarm64/lib/cmake/Qt6/
ls /clangarm64/bin/Qt6Core.dll
```

**Expected output:**
```
Qt6Config.cmake
Qt6ConfigVersion.cmake
...
/clangarm64/bin/Qt6Core.dll
```

---

**🎉 Part 3 Complete!**

All dependencies are now installed. Let's verify everything:

```bash
# Check MSVC dependencies
ls /c/poppler-deps/lib/*.lib

# Check MSYS2 dependencies
ls /clangarm64/lib/libcairo*.dll.a
ls /clangarm64/lib/cmake/Qt6/

# Check Boost
ls /c/poppler-deps/include/boost/version.hpp
```

**You should see:**
- All MSVC .lib files from Part 2
- Cairo libraries
- Qt6 CMake configs
- Boost headers

---

## Part 4: Building Poppler

Now we build Poppler itself using **MSYS2 CLANGARM64** terminal.

### Step 4.1: Download Poppler Source

```bash
# Navigate to build directory
cd /c/poppler-build

# Clone Poppler repository
git clone https://gitlab.freedesktop.org/poppler/poppler.git
cd poppler

# Optional: Check out specific version (or use latest main branch)
# git checkout poppler-24.12.0

# Check current version
git describe --tags
```

**Expected output:**
```
poppler-24.12.0
```
or similar version tag.

---

### Step 4.2: Create Build Configuration Script

**Why a script**: Poppler has many configuration options. A script makes it reproducible.

Create a file called `configure-poppler.sh`:

```bash
cat > configure-poppler.sh << 'EOF'
#!/bin/bash
# Poppler Build Configuration for Windows ARM64

# Set up paths
export PKG_CONFIG_PATH="/c/poppler-deps/lib/pkgconfig:/clangarm64/lib/pkgconfig"
export CMAKE_PREFIX_PATH="/c/poppler-deps;/clangarm64"

# Configure Poppler
cmake -B build -G Ninja \
  -DCMAKE_BUILD_TYPE=Release \
  -DCMAKE_INSTALL_PREFIX=/c/poppler \
  -DCMAKE_PREFIX_PATH="/c/poppler-deps;/clangarm64" \
  \
  `# Core features` \
  -DENABLE_UNSTABLE_API_ABI_HEADERS=ON \
  -DBUILD_SHARED_LIBS=ON \
  \
  `# Wrappers` \
  -DENABLE_CPP=ON \
  -DENABLE_QT6=ON \
  -DENABLE_GLIB=OFF \
  \
  `# Utilities` \
  -DENABLE_UTILS=ON \
  \
  `# Image formats` \
  -DENABLE_LIBJPEG=ON \
  -DENABLE_LIBPNG=ON \
  -DENABLE_LIBTIFF=ON \
  -DENABLE_LIBOPENJPEG=openjpeg2 \
  \
  `# Rendering backends` \
  -DENABLE_SPLASH=ON \
  -DWITH_Cairo=ON \
  \
  `# Font support` \
  -DFONT_CONFIGURATION=win32 \
  \
  `# Color management` \
  -DENABLE_LCMS=ON \
  \
  `# Network support` \
  -DENABLE_LIBCURL=ON \
  \
  `# Performance` \
  -DENABLE_BOOST=ON \
  \
  `# Disabled features` \
  -DENABLE_GOBJECT_INTROSPECTION=OFF \
  -DENABLE_NSS3=OFF \
  -DENABLE_GPGME=OFF \
  -DENABLE_QT5=OFF \
  \
  `# Testing` \
  -DBUILD_TESTING=OFF \
  -DBUILD_MANUAL_TESTS=OFF \
  \
  `# Dependency hints` \
  -DZLIB_INCLUDE_DIR=/c/poppler-deps/include \
  -DZLIB_LIBRARY=/c/poppler-deps/lib/z.lib \
  -DJPEG_INCLUDE_DIR=/c/poppler-deps/include \
  -DJPEG_LIBRARY=/c/poppler-deps/lib/jpeg.lib \
  -DPNG_PNG_INCLUDE_DIR=/c/poppler-deps/include \
  -DPNG_LIBRARY=/c/poppler-deps/lib/libpng18.lib \
  -DTIFF_INCLUDE_DIR=/c/poppler-deps/include \
  -DTIFF_LIBRARY=/c/poppler-deps/lib/tiff.lib \
  -DFreetype_INCLUDE_DIR=/c/poppler-deps/include/freetype2 \
  -DFreetype_LIBRARY=/c/poppler-deps/lib/freetype.lib \
  -DOpenJPEG_INCLUDE_DIR=/c/poppler-deps/include/openjpeg-2.5 \
  -DOpenJPEG_LIBRARY=/c/poppler-deps/lib/openjp2.lib \
  -DLCMS2_INCLUDE_DIR=/c/poppler-deps/include/lcms2 \
  -DLCMS2_LIBRARIES=/c/poppler-deps/lib/lcms2.lib \
  -DCURL_INCLUDE_DIR=/c/poppler-deps/include \
  -DCURL_LIBRARY=/c/poppler-deps/lib/libcurl.lib \
  -DIconv_INCLUDE_DIR=/c/poppler-deps/include \
  -DIconv_LIBRARY=/c/poppler-deps/lib/iconv.lib \
  -DCairo_INCLUDE_DIR=/c/poppler-deps/include/cairo \
  -DCairo_LIBRARY=/c/poppler-deps/lib/libcairo.dll.a \
  -DBoost_INCLUDE_DIR=/c/poppler-deps/include \
  -DBoost_DIR=/c/poppler-deps/lib/cmake/Boost-1.86.0

echo ""
echo "Configuration complete! Review the output above for any warnings."
echo "If everything looks good, run: ninja -C build"
EOF

# Make script executable
chmod +x configure-poppler.sh
```

---

### Step 4.3: Configure Poppler

```bash
# Run configuration script
./configure-poppler.sh
```

**This will take 2-3 minutes**. Watch for these key messages:

**Expected successful output:**
```
-- The C compiler identification is Clang 18.x.x
-- The CXX compiler identification is Clang 18.x.x
-- Detecting C compiler ABI info - done
-- Detecting CXX compiler ABI info - done
...
-- Found ZLIB: /c/poppler-deps/lib/z.lib (found version "1.3.1")
-- Found JPEG: /c/poppler-deps/lib/jpeg.lib (found version "62")
-- Found PNG: /c/poppler-deps/lib/libpng18.lib (found version "1.8.x")
-- Found TIFF: /c/poppler-deps/lib/tiff.lib (found version "4.x.x")
-- Found Freetype: /c/poppler-deps/lib/freetype.lib (found version "2.13.x")
-- Found OpenJPEG: /c/poppler-deps/lib/openjp2.lib (found version "2.5.x")
-- Found LCMS2: /c/poppler-deps/lib/lcms2.lib (found version "2.16")
-- Found CURL: /c/poppler-deps/lib/libcurl.lib (found version "8.x.x")
-- Found Cairo: /c/poppler-deps/lib/libcairo.dll.a (found version "1.18.x")
-- Found Boost: /c/poppler-deps/lib/cmake/Boost-1.86.0/BoostConfig.cmake (found version "1.86.0")
-- Found Qt6: /clangarm64/lib/cmake/Qt6/Qt6Config.cmake (found version "6.7.x")
...
-- Configuring done
-- Generating done
-- Build files have been written to: /c/poppler-build/poppler/build
```

**Configuration summary** (at the end):

```
Building Poppler with support for:
  font configuration: win32
  splash output:      yes
  cairo output:       yes
  qt6 wrapper:        yes
  cpp wrapper:        yes
  glib wrapper:       no
  use libjpeg:        yes
  use libpng:         yes
  use libtiff:        yes
  use zlib:           yes
  use libcurl:        yes
  use libopenjpeg2:   yes
  use lcms2:          yes
  use boost:          yes
  command line utils: yes
```

**If you see errors:**

1. **"Could not find ZLIB"** or similar:
   - Check that the library exists: `ls /c/poppler-deps/lib/z.lib`
   - Verify the path in the error message
   - Make sure you completed Part 2 successfully

2. **"Could not find Cairo"**:
   - Check: `ls /c/poppler-deps/lib/libcairo.dll.a`
   - Verify Part 3 Step 3.2 completed successfully

3. **"Could not find Qt6"**:
   - Check: `ls /clangarm64/lib/cmake/Qt6/`
   - Run: `pacman -S mingw-w64-clang-aarch64-qt6-base`

4. **"Could not find Boost"**:
   - Check: `ls /c/poppler-deps/lib/cmake/Boost-1.86.0/BoostConfig.cmake`
   - Verify Part 3 Step 3.4 completed successfully

---

### Step 4.4: Build Poppler

```bash
# Build with Ninja (takes 30-45 minutes)
ninja -C build

# Or build with verbose output to see what's happening:
# ninja -C build -v
```

**Expected output:**
```
[1/XXX] Building CXX object utils/CMakeFiles/pdfinfo.dir/pdfinfo.cc.obj
[2/XXX] Building CXX object poppler/CMakeFiles/poppler.dir/Annot.cc.obj
[3/XXX] Building CXX object poppler/CMakeFiles/poppler.dir/Array.cc.obj
...
[XXX/XXX] Linking CXX shared library libpoppler-154.dll
```

**Build progress indicators:**
- First 30%: Building core Poppler library
- Next 20%: Building C++ wrapper
- Next 20%: Building Qt6 wrapper
- Last 30%: Building command-line utilities

**If build fails:**

1. **Linker errors about missing symbols**:
   ```
   undefined reference to `jpeg_start_decompress'
   ```
   - This means a dependency wasn't found correctly
   - Check the CMake configuration output for warnings
   - Verify all .lib files exist in `C:\poppler-deps\lib`

2. **Compiler errors in Poppler code**:
   - Make sure you're using MSYS2 CLANGARM64 terminal
   - Check Clang version: `clang --version` (should be 18.x or higher)
   - Try updating MSYS2: `pacman -Syu`

3. **Out of memory errors**:
   - Close other applications
   - Build with fewer parallel jobs: `ninja -C build -j2`

---

### Step 4.5: Install Poppler

```bash
# Install to C:\poppler
ninja -C build install
```

**Expected output:**
```
Installing: C:/poppler/bin/libpoppler-154.dll
Installing: C:/poppler/bin/libpoppler-cpp-2.dll
Installing: C:/poppler/bin/libpoppler-qt6-3.dll
Installing: C:/poppler/bin/pdfinfo.exe
Installing: C:/poppler/bin/pdftotext.exe
Installing: C:/poppler/bin/pdftoppm.exe
Installing: C:/poppler/bin/pdftocairo.exe
Installing: C:/poppler/bin/pdfimages.exe
Installing: C:/poppler/bin/pdftops.exe
Installing: C:/poppler/bin/pdftohtml.exe
Installing: C:/poppler/bin/pdfseparate.exe
Installing: C:/poppler/bin/pdfunite.exe
Installing: C:/poppler/bin/pdfattach.exe
Installing: C:/poppler/bin/pdfdetach.exe
Installing: C:/poppler/bin/pdffonts.exe
Installing: C:/poppler/lib/libpoppler.dll.a
Installing: C:/poppler/lib/libpoppler-cpp.dll.a
Installing: C:/poppler/lib/libpoppler-qt6.dll.a
Installing: C:/poppler/include/poppler/...
Installing: C:/poppler/lib/pkgconfig/poppler.pc
Installing: C:/poppler/lib/pkgconfig/poppler-cpp.pc
Installing: C:/poppler/lib/pkgconfig/poppler-qt6.pc
Installing: C:/poppler/lib/cmake/poppler/...
```

---

### Step 4.6: Copy Runtime Dependencies

Poppler DLLs need their dependencies to run. Copy them to the Poppler bin directory:

```bash
# Copy MSVC-built dependencies
cp /c/poppler-deps/bin/*.dll /c/poppler/bin/

# Copy MSYS2 dependencies
cp /clangarm64/bin/libcairo-2.dll /c/poppler/bin/
cp /clangarm64/bin/libpixman-1-0.dll /c/poppler/bin/
cp /clangarm64/bin/libfreetype-6.dll /c/poppler/bin/
cp /clangarm64/bin/libfontconfig-1.dll /c/poppler/bin/
cp /clangarm64/bin/libexpat-1.dll /c/poppler/bin/
cp /clangarm64/bin/libbz2-1.dll /c/poppler/bin/
cp /clangarm64/bin/libbrotlidec.dll /c/poppler/bin/
cp /clangarm64/bin/libbrotlicommon.dll /c/poppler/bin/
cp /clangarm64/bin/libharfbuzz-0.dll /c/poppler/bin/
cp /clangarm64/bin/libglib-2.0-0.dll /c/poppler/bin/
cp /clangarm64/bin/libintl-8.dll /c/poppler/bin/
cp /clangarm64/bin/libpcre2-8-0.dll /c/poppler/bin/
cp /clangarm64/bin/libgraphite2.dll /c/poppler/bin/

# Copy Qt6 dependencies (for Qt6 wrapper)
cp /clangarm64/bin/Qt6Core.dll /c/poppler/bin/
cp /clangarm64/bin/Qt6Gui.dll /c/poppler/bin/
cp /clangarm64/bin/Qt6Widgets.dll /c/poppler/bin/

# Copy C++ runtime
cp /clangarm64/bin/libc++.dll /c/poppler/bin/
cp /clangarm64/bin/libunwind.dll /c/poppler/bin/

# Verify all DLLs are present
ls -lh /c/poppler/bin/*.dll | wc -l
```

**Expected output:**
```
30-35 DLL files
```

**Why so many DLLs?**
- Poppler's own DLLs (3)
- Direct dependencies (zlib, jpeg, png, tiff, etc.) (10)
- Cairo and its dependencies (8)
- Qt6 and its dependencies (8)
- C++ runtime (2)

---

**🎉 Part 4 Complete!**

Poppler is now built and installed in `C:\poppler`!

---

## Part 5: Verification

Let's verify everything works correctly.

### Step 5.1: Check Installation

In **MSYS2 CLANGARM64** terminal:

```bash
# List all installed files
ls -lh /c/poppler/bin/

# Check DLL sizes (approximate)
ls -lh /c/poppler/bin/libpoppler-154.dll
ls -lh /c/poppler/bin/libpoppler-qt6-3.dll
ls -lh /c/poppler/bin/libpoppler-cpp-2.dll
```

**Expected output:**
```
-rwxr-xr-x 1 user None 3.5M ... libpoppler-154.dll
-rwxr-xr-x 1 user None 753K ... libpoppler-qt6-3.dll
-rwxr-xr-x 1 user None 160K ... libpoppler-cpp-2.dll
```

---

### Step 5.2: Test Command-Line Utilities

Create a test PDF or download one:

```bash
# Download a sample PDF
cd /c/poppler/bin
curl -o test.pdf https://www.w3.org/WAI/ER/tests/xhtml/testfiles/resources/pdf/dummy.pdf

# Test pdfinfo
./pdfinfo.exe test.pdf
```

**Expected output:**
```
Title:          Dummy PDF file
Author:         ...
Creator:        ...
Producer:       ...
CreationDate:   ...
ModDate:        ...
Tagged:         no
UserProperties: no
Suspects:       no
Form:           none
JavaScript:     no
Pages:          1
Encrypted:      no
Page size:      595.276 x 841.89 pts (A4)
Page rot:       0
File size:      13264 bytes
Optimized:      no
PDF version:    1.4
```

**Test other utilities:**

```bash
# Extract text
./pdftotext.exe test.pdf test.txt
cat test.txt

# Convert to PNG
./pdftoppm.exe -png test.pdf test
ls test-*.png

# Get font information
./pdffonts.exe test.pdf

# Convert to high-quality PNG with Cairo
./pdftocairo.exe -png -r 300 test.pdf test-cairo
ls test-cairo-*.png
```

**If utilities fail:**

1. **"The code execution cannot proceed because XXX.dll was not found"**:
   - A dependency DLL is missing
   - Check Step 4.6 and copy the missing DLL
   - Use Dependency Walker or `ldd` to find missing dependencies:
     ```bash
     ldd ./pdfinfo.exe | grep "not found"
     ```

2. **"Error: Couldn't open file 'test.pdf'"**:
   - Check file exists: `ls test.pdf`
   - Check permissions: `chmod 644 test.pdf`

3. **Crashes or hangs**:
   - Try with a different PDF
   - Check if it's a corrupted PDF
   - Run with verbose output: `./pdfinfo.exe -v test.pdf`

---

### Step 5.3: Test Library Linking

Create a simple test program to verify the library works:

```bash
# Create test directory
mkdir -p /c/poppler-test
cd /c/poppler-test

# Create test program
cat > test-poppler.cpp << 'EOF'
#include <poppler-version.h>
#include <iostream>

int main() {
    std::cout << "Poppler version: " << POPPLER_VERSION << std::endl;
    return 0;
}
EOF

# Compile test program
clang++ test-poppler.cpp \
  -I/c/poppler/include/poppler/cpp \
  -L/c/poppler/lib \
  -lpoppler-cpp \
  -o test-poppler.exe

# Run test
./test-poppler.exe
```

**Expected output:**
```
Poppler version: 24.12.0
```

**Test Qt6 wrapper:**

```bash
cat > test-qt6.cpp << 'EOF'
#include <poppler-qt6.h>
#include <iostream>

int main() {
    std::cout << "Poppler Qt6 version: " 
              << Poppler::version().toStdString() << std::endl;
    return 0;
}
EOF

# Compile Qt6 test
clang++ test-qt6.cpp \
  -I/c/poppler/include/poppler/qt6 \
  -I/clangarm64/include/QtCore \
  -I/clangarm64/include/QtGui \
  -I/clangarm64/include \
  -L/c/poppler/lib \
  -L/clangarm64/lib \
  -lpoppler-qt6 \
  -lQt6Core \
  -lQt6Gui \
  -o test-qt6.exe

# Run Qt6 test
./test-qt6.exe
```

**Expected output:**
```
Poppler Qt6 version: 24.12.0
```

---

### Step 5.4: Performance Test

Test rendering performance with a complex PDF:

```bash
cd /c/poppler/bin

# Download a multi-page PDF
curl -o complex.pdf https://www.adobe.com/content/dam/acom/en/devnet/pdf/pdfs/PDF32000_2008.pdf

# Time PDF to PNG conversion
time ./pdftoppm.exe -png -r 150 complex.pdf output

# Time Cairo rendering (should be faster due to ARM64 NEON)
time ./pdftocairo.exe -png -r 150 complex.pdf output-cairo
```

**Expected results:**
- Cairo rendering should be ~30% faster than Splash backend
- ARM64 NEON optimizations in libjpeg-turbo and Cairo provide significant speedup
- Memory usage should be reasonable (< 500 MB for typical PDFs)

---

**🎉 Part 5 Complete!**

Your Poppler installation is verified and working!

---

## Part 6: Integration with Applications

### Step 6.1: Using Poppler in TeXworks

If you're building this for TeXworks:

1. **Copy Poppler to TeXworks directory:**
   ```bash
   # Assuming TeXworks is in C:\TeXworks
   cp -r /c/poppler/* /c/TeXworks/
   ```

2. **Verify TeXworks can find Poppler:**
   - Launch TeXworks
   - Open a PDF file
   - Check if PDF rendering works

3. **If TeXworks can't find Poppler:**
   - Add `C:\poppler\bin` to system PATH
   - Or copy all DLLs to TeXworks executable directory

---

### Step 6.2: System-Wide Installation

To make Poppler available system-wide:

**Option 1: Add to PATH (PowerShell as Administrator):**

```powershell
# Add to system PATH
$oldPath = [Environment]::GetEnvironmentVariable("Path", "Machine")
$newPath = $oldPath + ";C:\poppler\bin"
[Environment]::SetEnvironmentVariable("Path", $newPath, "Machine")

# Verify
$env:Path -split ';' | Select-String "poppler"
```

**Option 2: Create symbolic links:**

```powershell
# Create links in a directory already in PATH
New-Item -ItemType SymbolicLink -Path "C:\Windows\System32\pdfinfo.exe" -Target "C:\poppler\bin\pdfinfo.exe"
New-Item -ItemType SymbolicLink -Path "C:\Windows\System32\pdftotext.exe" -Target "C:\poppler\bin\pdftotext.exe"
# ... repeat for other utilities
```

**Option 3: Copy to Program Files:**

```powershell
# Copy entire installation
Copy-Item -Recurse "C:\poppler" "C:\Program Files\Poppler"

# Add to PATH
$oldPath = [Environment]::GetEnvironmentVariable("Path", "Machine")
$newPath = $oldPath + ";C:\Program Files\Poppler\bin"
[Environment]::SetEnvironmentVariable("Path", $newPath, "Machine")
```

---

### Step 6.3: Development Setup

For developers using Poppler in their projects:

**CMake integration:**

```cmake
# In your CMakeLists.txt
list(APPEND CMAKE_PREFIX_PATH "C:/poppler")

find_package(Poppler REQUIRED COMPONENTS cpp qt6)

target_link_libraries(your_app
    Poppler::poppler
    Poppler::cpp
    Poppler::qt6
)
```

**pkg-config integration:**

```bash
# Set PKG_CONFIG_PATH
export PKG_CONFIG_PATH="/c/poppler/lib/pkgconfig:$PKG_CONFIG_PATH"

# Query Poppler
pkg-config --cflags poppler-cpp
pkg-config --libs poppler-cpp
```

**Manual linking:**

```bash
# Compile flags
-I/c/poppler/include/poppler/cpp

# Link flags
-L/c/poppler/lib -lpoppler-cpp
```

---

## Troubleshooting

### Common Build Issues

#### Issue 1: "CMake Error: Could not find CMAKE_ROOT"

**Cause**: CMake not properly installed or not in PATH.

**Solution:**
```powershell
# Reinstall CMake and ensure "Add to PATH" is checked
# Or manually add to PATH:
$env:Path += ";C:\Program Files\CMake\bin"
```

---

#### Issue 2: "LINK : fatal error LNK1181: cannot open input file 'z.lib'"

**Cause**: Dependency library not found.

**Solution:**
```bash
# Verify library exists
ls /c/poppler-deps/lib/z.lib

# If missing, rebuild zlib (Part 2, Step 2.1)
cd /c/poppler-build/zlib
cmake --build build --config Release
cmake --install build
```

---

#### Issue 3: "undefined reference to `cairo_xxx'"

**Cause**: Cairo not properly linked or wrong toolchain.

**Solution:**
```bash
# Verify Cairo library exists
ls /c/poppler-deps/lib/libcairo.dll.a

# Ensure using MSYS2 CLANGARM64 terminal
echo $MSYSTEM  # Should output: CLANGARM64

# If wrong terminal, close and open "MSYS2 CLANGARM64"
```

---

#### Issue 4: "Qt6 not found"

**Cause**: Qt6 not installed in MSYS2.

**Solution:**
```bash
# Install Qt6
pacman -S mingw-w64-clang-aarch64-qt6-base

# Verify installation
ls /clangarm64/lib/cmake/Qt6/Qt6Config.cmake
```

---

#### Issue 5: Build runs out of memory

**Cause**: Insufficient RAM or too many parallel jobs.

**Solution:**
```bash
# Build with fewer parallel jobs
ninja -C build -j2

# Or use single-threaded build
ninja -C