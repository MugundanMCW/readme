# Building PyMuPDF for Windows ARM64

This document details all changes required to build PyMuPDF for Windows ARM64 architecture.

## Overview

PyMuPDF and its underlying MuPDF library were originally designed for x86/x64 architectures. Building for ARM64 on Windows requires modifications to:

1. **MuPDF source code** - CPU detection and build system
2. **PyMuPDF build scripts** - Architecture handling and wheel generation

## Prerequisites

- Windows ARM64 machine or VM
- Python 3.11+ for ARM64 (from python.org or Microsoft Store)
- Visual Studio 2022 with ARM64 build tools
- Git
- SWIG (will be installed automatically by build process)

## Part 1: MuPDF Changes

MuPDF requires patches to detect and build for ARM64 architecture.

### 1.1 File: `mupdf/scripts/wdev.py`

**Location**: Lines 150-200

**Purpose**: Add ARM64 CPU detection and configuration

**Changes**:

```python
# Around line 150, in WindowsCpu.__init__() method:

class WindowsCpu:
    def __init__(self, name=None):
        if name is None:
            name = _cpu_name()
        self.name = name
        
        if name in ('x32', 'x86', 'Win32'):
            self.bits = 32
            self.windows_subdir = ''
            self.windows_name = 'Win32'
            self.windows_config = 'Win32'
            self.windows_suffix = ''
        elif name in ('x64', 'x86_64', 'AMD64'):
            self.bits = 64
            self.windows_subdir = 'x64/'
            self.windows_name = 'x64'
            self.windows_config = 'x64'
            self.windows_suffix = '64'
        elif name in ('arm64', 'ARM64', 'aarch64'):  # <-- ADD THIS BLOCK
            self.bits = 64
            self.windows_subdir = 'ARM64/'
            self.windows_name = 'ARM64'
            self.windows_config = 'ARM64'
            self.windows_suffix = '64'
        else:
            assert 0, f'Unrecognised cpu name: {name}'
```

**Around line 200, in `_cpu_name()` function**:

```python
def _cpu_name():
    '''
    Returns name of CPU that we are running on.
    '''
    # Check if building for ARM64 via environment variable
    target_arch = os.environ.get('CIBW_ARCHS_WINDOWS', '').upper()
    if target_arch == 'ARM64':
        return 'arm64'
    
    # Check platform machine type
    machine = platform.machine().lower()
    if machine in ('arm64', 'aarch64'):
        return 'arm64'
    
    # Default x32/x64 detection
    return f'x{32 if sys.maxsize == 2**31 - 1 else 64}'
```

### 1.2 File: `mupdf/scripts/wrap/state.py`

**Location**: Lines 200-250

**Purpose**: Add ARM64 support to MuPDF's wrapper generation system

**Changes**:

```python
# Around line 200, in Cpu.__init__() method:

class Cpu:
    def __init__(self, name=None):
        if name is None:
            name = cpu_name()
        self.name = name
        
        if name in ('x32', 'x86', 'Win32'):
            self.bits = 32
            self.windows_subdir = ''
            self.windows_name = 'Win32'
            self.windows_config = 'Win32'
            self.windows_suffix = ''
        elif name in ('x64', 'x86_64', 'AMD64'):
            self.bits = 64
            self.windows_subdir = 'x64/'
            self.windows_name = 'x64'
            self.windows_config = 'x64'
            self.windows_suffix = '64'
        elif name in ('arm64', 'ARM64', 'aarch64'):  # <-- ADD THIS BLOCK
            self.bits = 64
            self.windows_subdir = 'ARM64/'
            self.windows_name = 'ARM64'
            self.windows_config = 'ARM64'
            self.windows_suffix = '64'
        else:
            assert 0, f'Unrecognised cpu name: {name}'
```

**Around line 220, in `cpu_name()` function**:

```python
def cpu_name():
    '''
    Returns name of CPU.
    '''
    # Check if building for ARM64 via environment variable
    target_arch = os.environ.get('CIBW_ARCHS_WINDOWS', '').upper()
    if target_arch == 'ARM64':
        ret = 'arm64'
    else:
        # Check platform machine type
        machine = platform.machine().lower()
        if machine in ('arm64', 'aarch64'):
            ret = 'arm64'
        else:
            # Default x32/x64 detection
            ret = f'x{32 if sys.maxsize == 2**31 - 1 else 64}'
    return ret
```

**Around line 350, in `BuildDirs.__init__()` method**:

```python
# Find the line that checks for x32/x64 and update it:

# BEFORE:
if flag in ('x32', 'x64'):
    self.cpu = Cpu(flag)

# AFTER:
if flag in ('x32', 'x64', 'arm64', 'ARM64'):
    self.cpu = Cpu(flag)
```

### 1.3 File: `mupdf/scripts/wrap/__main__.py`

**Location**: Lines 1750-1850

**Purpose**: Fix hardcoded x64 library paths in build system

**Changes**:

```python
# Around line 1750-1850, in build() function, action '3':

# BEFORE:
if state.state_.windows:
    libdir = f'{build_dirs.dir_mupdf}/platform/win32/'
    mupdfcpp_lib = f'{build_dirs.dir_mupdf}/platform/win32/'
    if build_dirs.cpu.bits == 64:
        libdir += 'x64/'
    libdir += 'Debug/' if debug else 'Memento/' if memento else 'Release/'
    libs = list()
    libs.append(libdir + ('mupdfcpp64.lib' if build_dirs.cpu.bits == 64 else 'mupdfcpp.lib'))

# AFTER:
if state.state_.windows:
    # Use CPU-aware paths
    libdir = f'{build_dirs.dir_mupdf}/platform/win32/'
    
    # Add CPU-specific subdirectory
    if build_dirs.cpu.name == 'arm64':
        libdir += 'ARM64/'
    elif build_dirs.cpu.bits == 64:
        libdir += 'x64/'
    # else: x32, no subdirectory
    
    # Add build type
    libdir += 'Debug/' if debug else 'Memento/' if memento else 'Release/'
    
    libs = list()
    
    # Use CPU-aware library name
    if build_dirs.cpu.name in ('arm64', 'x64'):
        libs.append(libdir + f'mupdfcpp{build_dirs.cpu.windows_suffix}.lib')
    else:
        libs.append(libdir + 'mupdfcpp.lib')
```

## Part 2: PyMuPDF Changes

### 2.1 File: `setup.py`

**Location**: Multiple sections

#### 2.1.1 Add MuPDF Patch Application Function

**Location**: After imports, around line 200

**Purpose**: Automatically apply ARM64 patches to MuPDF when building

```python
def apply_mupdf_arm64_patches(mupdf_local):
    '''
    Applies ARM64 patches to MuPDF source using git apply.
    '''
    log(f'Applying ARM64 patches to MuPDF...')
    
    # Patch for scripts/wdev.py
    wdev_patch = '''diff --git a/scripts/wdev.py b/scripts/wdev.py
index 1234567..abcdefg 100644
--- a/scripts/wdev.py
+++ b/scripts/wdev.py
@@ -150,6 +150,12 @@ class WindowsCpu:
             self.windows_name = 'x64'
             self.windows_config = 'x64'
             self.windows_suffix = '64'
+        elif name in ('arm64', 'ARM64', 'aarch64'):
+            self.bits = 64
+            self.windows_subdir = 'ARM64/'
+            self.windows_name = 'ARM64'
+            self.windows_config = 'ARM64'
+            self.windows_suffix = '64'
         else:
             assert 0, f'Unrecognised cpu name: {name}'
 
@@ -200,7 +206,18 @@ def _cpu_name():
     '''
     Returns name of CPU that we are running on.
     '''
-    return f'x{32 if sys.maxsize == 2**31 - 1 else 64}'
+    # Check if building for ARM64 via environment variable
+    target_arch = os.environ.get('CIBW_ARCHS_WINDOWS', '').upper()
+    if target_arch == 'ARM64':
+        return 'arm64'
+    
+    # Check platform machine type
+    machine = platform.machine().lower()
+    if machine in ('arm64', 'aarch64'):
+        return 'arm64'
+    
+    # Default x32/x64 detection
+    return f'x{32 if sys.maxsize == 2**31 - 1 else 64}'
'''
    
    # Similar patch for scripts/wrap/state.py
    state_patch = '''diff --git a/scripts/wrap/state.py b/scripts/wrap/state.py
index 1234567..abcdefg 100644
--- a/scripts/wrap/state.py
+++ b/scripts/wrap/state.py
@@ -200,6 +200,12 @@ class Cpu:
             self.windows_name = 'x64'
             self.windows_config = 'x64'
             self.windows_suffix = '64'
+        elif name in ('arm64', 'ARM64', 'aarch64'):
+            self.bits = 64
+            self.windows_subdir = 'ARM64/'
+            self.windows_name = 'ARM64'
+            self.windows_config = 'ARM64'
+            self.windows_suffix = '64'
         else:
             assert 0, f'Unrecognised cpu name: {name}'
 
@@ -220,7 +226,18 @@ def cpu_name():
     '''
     Returns name of CPU.
     '''
-    ret = f'x{32 if sys.maxsize == 2**31 - 1 else 64}'
+    # Check if building for ARM64 via environment variable
+    target_arch = os.environ.get('CIBW_ARCHS_WINDOWS', '').upper()
+    if target_arch == 'ARM64':
+        ret = 'arm64'
+    else:
+        # Check platform machine type
+        machine = platform.machine().lower()
+        if machine in ('arm64', 'aarch64'):
+            ret = 'arm64'
+        else:
+            # Default x32/x64 detection
+            ret = f'x{32 if sys.maxsize == 2**31 - 1 else 64}'
     return ret
 
@@ -350,7 +367,7 @@ class BuildDirs:
             self.python_version = None
             for flag in flags:
-                if flag in ('x32', 'x64'):
+                if flag in ('x32', 'x64', 'arm64', 'ARM64'):
                     self.cpu = Cpu(flag)
'''
    
    # Patch for scripts/wrap/__main__.py
    main_patch = '''diff --git a/scripts/wrap/__main__.py b/scripts/wrap/__main__.py
index 1234567..abcdefg 100644
--- a/scripts/wrap/__main__.py
+++ b/scripts/wrap/__main__.py
@@ -1750,14 +1750,24 @@ def build():
     if state.state_.windows:
-        libdir = f'{build_dirs.dir_mupdf}/platform/win32/'
-        mupdfcpp_lib = f'{build_dirs.dir_mupdf}/platform/win32/'
-        if build_dirs.cpu.bits == 64:
+        # Use CPU-aware paths
+        libdir = f'{build_dirs.dir_mupdf}/platform/win32/'
+        
+        # Add CPU-specific subdirectory
+        if build_dirs.cpu.name == 'arm64':
+            libdir += 'ARM64/'
+        elif build_dirs.cpu.bits == 64:
             libdir += 'x64/'
+        # else: x32, no subdirectory
+        
+        # Add build type
         libdir += 'Debug/' if debug else 'Memento/' if memento else 'Release/'
+        
         libs = list()
-        libs.append(libdir + ('mupdfcpp64.lib' if build_dirs.cpu.bits == 64 else 'mupdfcpp.lib'))
+        
+        # Use CPU-aware library name
+        if build_dirs.cpu.name in ('arm64', 'x64'):
+            libs.append(libdir + f'mupdfcpp{build_dirs.cpu.windows_suffix}.lib')
+        else:
+            libs.append(libdir + 'mupdfcpp.lib')
'''
    
    # Apply patches
    for patch_content, filename in [
        (wdev_patch, 'wdev.py'),
        (state_patch, 'state.py'),
        (main_patch, '__main__.py')
    ]:
        patch_file = os.path.join(mupdf_local, f'pymupdf_{filename}.patch')
        with open(patch_file, 'w', newline='\n') as f:
            f.write(patch_content)
        
        # Check if already applied
        result = subprocess.run(
            f'cd {mupdf_local} && git apply --check --reverse {patch_file}',
            shell=True,
            capture_output=True
        )
        
        if result.returncode == 0:
            log(f'Patch for {filename} already applied')
        else:
            log(f'Applying patch for {filename}')
            subprocess.run(
                f'cd {mupdf_local} && git apply {patch_file}',
                shell=True,
                check=True
            )
```

**Call this function after the imports section**:

```python
# After all imports and before other functions:
apply_mupdf_arm64_patches()
```

#### 2.1.2 Fix `retarget_vs_solution()` Function

**Location**: Around line 600

**Purpose**: Retarget Visual Studio solution for ARM64

```python
def retarget_vs_solution(mupdf_local, devenv, cpu_config):
    '''
    Retarget Visual Studio solution to current VS version and platform.
    This fixes the "v142 build tools not found" error.
    '''
    solution_path = os.path.join(mupdf_local, 'platform', 'win32', 'mupdf.sln')
    
    if not os.path.exists(solution_path):
        log(f'Solution not found: {solution_path}')
        return
    
    log(f'Retargeting solution to current VS version with platform {cpu_config}...')
    
    # Use devenv to upgrade/retarget the solution
    command = f'"{devenv}" "{solution_path}" /Upgrade'
    
    try:
        result = subprocess.run(command, shell=True, capture_output=True, text=True, timeout=120)
        if result.returncode == 0 or 'successfully' in result.stdout.lower():
            log(f'Solution retargeted successfully')
        else:
            log(f'Solution retargeting returned code {result.returncode}')
            if result.stdout:
                log(f'stdout: {result.stdout[:500]}')
            if result.stderr:
                log(f'stderr: {result.stderr[:500]}')
    except subprocess.TimeoutExpired:
        log(f'Warning: Solution retargeting timed out (may be normal)')
    except Exception as e:
        log(f'Warning: Solution retargeting failed: {e}')
```

#### 2.1.3 Update `build_mupdf_windows()` Function

**Location**: Around line 700-850

**Purpose**: Detect ARM64 architecture and use correct build paths

```python
def build_mupdf_windows(
        mupdf_local,
        build_type,
        overwrite_config,
        g_py_limited_api,
        PYMUPDF_SETUP_MUPDF_REFCHECK_IF,
        PYMUPDF_SETUP_MUPDF_TRACE_IF,
        ):
    
    assert mupdf_local

    if overwrite_config:
        # ... existing config code ...
    
    # Detect target architecture from environment (for cibuildwheel)
    target_arch = os.environ.get('CIBW_ARCHS_WINDOWS', '').upper()
    if target_arch == 'ARM64':
        log(f'Building for Windows ARM64 (target: {target_arch})')
        cpu = pipcl.wdev.WindowsCpu('arm64')
    else:
        # Use default CPU detection
        cpu = pipcl.wdev.WindowsCpu()
    
    log(f'Using CPU configuration: {cpu.name} ({cpu.windows_config})')
    
    wp = pipcl.wdev.WindowsPython(cpu=cpu)
    tesseract = '' if os.environ.get('PYMUPDF_SETUP_MUPDF_TESSERACT') == '0' else 'tesseract-'
    windows_build_tail = f'build\\shared-{tesseract}{build_type}'
    if g_py_limited_api:
        windows_build_tail += f'-Py_LIMITED_API_{pipcl.current_py_limited_api()}'
    windows_build_tail += f'-{cpu.windows_config}-py{wp.version}'
    windows_build_dir = f'{mupdf_local}\\{windows_build_tail}'
    
    devenv = os.environ.get('PYMUPDF_SETUP_DEVENV')
    if not devenv:
        try:
            log(f'Looking for Visual Studio 2022.')
            vs = pipcl.wdev.WindowsVS(year=2022, cpu=cpu)
        except Exception as e:
            log(f'Failed to find VS-2022, looking for any Visual Studio.')
            vs = pipcl.wdev.WindowsVS(cpu=cpu)
        log(f'vs:\n{vs.description_ml("    ")}')
        devenv = vs.devenv
    if not devenv:
        devenv = 'devenv.com'
        log(f'Cannot find devenv.com in default locations, using: {devenv!r}')
    
    # Retarget solution to fix v142 toolset error
    retarget_vs_solution(mupdf_local, devenv, cpu.windows_config)

    # Build the command with environment variables for ARM64
    env_vars = ''
    if cpu.name == 'arm64':
        # Set environment variables to force ARM64 build
        env_vars = f'set VSCMD_ARG_TGT_ARCH=arm64 && set PROCESSOR_ARCHITECTURE=ARM64 && '
    
    command = f'{env_vars}cd "{mupdf_local}" && "{sys.executable}" ./scripts/mupdfwrap.py'
    if os.environ.get('PYMUPDF_SETUP_MUPDF_VS_UPGRADE') == '1':
        command += ' --vs-upgrade 1'
    
    # ... rest of existing command building code ...
    
    return windows_build_dir
```

#### 2.1.4 Update `_windows_lib_directory()` Function

**Location**: Around line 900

**Purpose**: Return correct library directory for ARM64

```python
def _windows_lib_directory(mupdf_local, build_type):
    ret = f'{mupdf_local}/platform/win32/'
    
    # Detect CPU architecture
    target_arch = os.environ.get('CIBW_ARCHS_WINDOWS', '').upper()
    if target_arch == 'ARM64':
        ret += 'ARM64/'
    elif _cpu_bits() == 64:
        ret += 'x64/'
    # else: 32-bit, no subdirectory
    
    if build_type == 'release':
        ret += 'Release/'
    elif build_type == 'debug':
        ret += 'Debug/'
    else:
        assert 0, f'Unrecognised {build_type=}.'
    return ret
```

#### 2.1.5 Update `_extension_flags()` Function

**Location**: Around line 950-1050

**Purpose**: Use correct library paths for ARM64

```python
def _extension_flags(mupdf_local, mupdf_build_dir, build_type):
    '''
    Returns various flags to pass to pipcl.build_extension().
    '''
    # ... existing code ...
    
    if windows():
        defines.append('FZ_DLL_CLIENT')
        # Detect CPU architecture
        target_arch = os.environ.get('CIBW_ARCHS_WINDOWS', '').upper()
        if target_arch == 'ARM64':
            cpu = pipcl.wdev.WindowsCpu('arm64')
        else:
            cpu = pipcl.wdev.WindowsCpu()
        wp = pipcl.wdev.WindowsPython(cpu=cpu)
        if os.environ.get('PYMUPDF_SETUP_MUPDF_VS_UPGRADE') == '1':
            infix = 'win32-vs-upgrade'
        else:
            infix = 'win32'
        build_type_infix = 'Debug' if debug else 'Release'
        libpaths = (
                f'{mupdf_local}\\platform\\{infix}\\{wp.cpu.windows_subdir}{build_type_infix}',
                f'{mupdf_local}\\platform\\{infix}\\{wp.cpu.windows_subdir}{build_type_infix}Tesseract',
                )
        libs = f'mupdfcpp{wp.cpu.windows_suffix}.lib'
        libraries = f'{mupdf_local}\\platform\\{infix}\\{wp.cpu.windows_subdir}{build_type_infix}\\{libs}'
        compiler_extra = ''
    else:
        # ... existing Unix code ...
    
    # ... rest of function ...
```

#### 2.1.6 Fix Wheel Tag Generation

**Location**: Around line 1050-1070

**Purpose**: Remove hardcoded `py3` tag that prevents correct wheel naming

```python
# BEFORE:
if 'p' in PYMUPDF_SETUP_FLAVOUR:
    version = version_p
    name = 'PyMuPDF'
    readme = readme_p
    summary = '...'
    tag_python = None
    
elif 'b' in PYMUPDF_SETUP_FLAVOUR:
    version = version_b
    name = 'PyMuPDFb'
    readme = readme_b
    summary = 'MuPDF shared libraries for PyMuPDF.'
    tag_python = 'py3'  # <-- REMOVE THIS
    
elif 'd' in PYMUPDF_SETUP_FLAVOUR:
    version = version_b
    name = 'PyMuPDFd'
    readme = readme_d
    summary = 'MuPDF build-time files for PyMuPDF.'
    tag_python = 'py3'  # <-- REMOVE THIS

# AFTER:
if 'p' in PYMUPDF_SETUP_FLAVOUR:
    version = version_p
    name = 'PyMuPDF'
    readme = readme_p
    summary = '...'
    # tag_python will be None, letting pipcl.py determine it
    
elif 'b' in PYMUPDF_SETUP_FLAVOUR:
    version = version_b
    name = 'PyMuPDFb'
    readme = readme_b
    summary = 'MuPDF shared libraries for PyMuPDF.'
    # Don't set tag_python - let pipcl.py determine it based on py_limited_api
    
elif 'd' in PYMUPDF_SETUP_FLAVOUR:
    version = version_b
    name = 'PyMuPDFd'
    readme = readme_d
    summary = 'MuPDF build-time files for PyMuPDF.'
    # Don't set tag_python - let pipcl.py determine it based on py_limited_api
```

### 2.2 File: `pipcl.py`

**Location**: Around line 1045-1055

**Purpose**: Fix `tag_python()` method to correctly handle `py_limited_api`

```python
# BEFORE:
def tag_python(self):
    '''
    Get two-digit python version, e.g. 'cp3.8' for python-3.8.6.
    '''
    if self.tag_python_:
        ret = self.tag_python_
    else:
        ret = 'cp' + ''.join(platform.python_version().split('.')[:2])
    assert '-' not in ret
    return ret

# AFTER:
def tag_python(self):
    '''
    Get two-digit python version, e.g. 'cp38' for python-3.8.6.
    For py_limited_api, returns minimum supported version from Py_LIMITED_API.
    '''
    if self.tag_python_:
        ret = self.tag_python_
    elif self.py_limited_api:
        # For stable ABI, use minimum Python version from Py_LIMITED_API
        # Format is typically '0x030X0000' where X is minor version
        if isinstance(self.py_limited_api, str) and self.py_limited_api.startswith('0x'):
            value = int(self.py_limited_api, 16)
            major = (value >> 24) & 0xFF
            minor = (value >> 16) & 0xFF
            ret = f'cp{major}{minor}'
        else:
            # Fallback to current Python version
            ret = 'cp' + ''.join(platform.python_version().split('.')[:2])
    else:
        ret = 'cp' + ''.join(platform.python_version().split('.')[:2])
    assert '-' not in ret
    return ret
```

### 2.3 File: `wdev.py` (in PyMuPDF root)

**Location**: Lines 150-200

**Purpose**: Add ARM64 CPU detection (same as MuPDF's wdev.py)

Apply the same changes as described in section 1.1 above.

## Part 3: Build Instructions

### 3.1 Environment Setup

```powershell
# Set architecture for cibuildwheel
$env:CIBW_ARCHS_WINDOWS = "ARM64"

# Optional: Set Visual Studio year if you have multiple versions
$env:WDEV_VS_YEAR = "2022"
```

### 3.2 Clean Previous Builds

```powershell
# Remove old wheels to avoid confusion
Remove-Item wheelhouse\*.whl -ErrorAction SilentlyContinue

# Clean MuPDF build artifacts (optional)
Remove-Item -Recurse -Force mupdf-*-source\build -ErrorAction SilentlyContinue
```

### 3.3 Build Command

```powershell
# Using cibuildwheel (recommended)
python -m pip install cibuildwheel
cibuildwheel --platform windows

# Or using setup.py directly
python setup.py bdist_wheel
```

### 3.4 Verify the Wheel

```powershell
# Check wheel name - should be cp3XX-abi3-win_arm64
dir wheelhouse\*.whl

# Check wheel contents
python -c "import zipfile; z = zipfile.ZipFile('wheelhouse/pymupdfb-1.26.3-cp311-abi3-win_arm64.whl'); print(z.read([n for n in z.namelist() if n.endswith('WHEEL')][0]).decode())"

# Should show:
# Wheel-Version: 1.0
# Generator: pipcl
# Root-Is-Purelib: false
# Tag: cp311-abi3-win_arm64
```

### 3.5 Install and Test

```powershell
# Install the wheel
pip install wheelhouse\pymupdfb-1.26.3-cp311-abi3-win_arm64.whl

# Test import
python -c "import pymupdf; print(pymupdf.__version__)"
```

## Part 4: Patch Files

For convenience, here are standalone patch files you can apply with `git apply`.

### 4.1 MuPDF wdev.py Patch

Save as `mupdf_wdev.patch`:

```diff
diff --git a/scripts/wdev.py b/scripts/wdev.py
index 1234567..abcdefg 100644
--- a/scripts/wdev.py
+++ b/scripts/wdev.py
@@ -150,6 +150,12 @@ class WindowsCpu:
             self.windows_name = 'x64'
             self.windows_config = 'x64'
             self.windows_suffix = '64'
+        elif name in ('arm64', 'ARM64', 'aarch64'):
+            self.bits = 64
+            self.windows_subdir = 'ARM64/'
+            self.windows_name = 'ARM64'
+            self.windows_config = 'ARM64'
+            self.windows_suffix = '64'
         else:
             assert 0, f'Unrecognised cpu name: {name}'
 
@@ -200,7 +206,18 @@ def _cpu_name():
     '''
     Returns name of CPU that we are running on.
     '''
-    return f'x{32 if sys.maxsize == 2**31 - 1 else 64}'
+    # Check if building for ARM64 via environment variable
+    target_arch = os.environ.get('CIBW_ARCHS_WINDOWS', '').upper()
+    if target_arch == 'ARM64':
+        return 'arm64'
+    
+    # Check platform machine type
+    machine = platform.machine().lower()
+    if machine in ('arm64', 'aarch64'):
+        return 'arm64'
+    
+    # Default x32/x64 detection
+    return f'x{32 if sys.maxsize == 2**31 - 1 else 64}'
```

Apply with:
```bash
cd mupdf-1.27.0-source
git apply mupdf_wdev.patch
```

### 4.2 MuPDF state.py Patch

Save as `mupdf_state.patch`:

```diff
diff --git a/scripts/wrap/state.py b/scripts/wrap/state.py
index 1234567..abcdefg 100644
--- a/scripts/wrap/state.py
+++ b/scripts/wrap/state.py
@@ -200,6 +200,12 @@ class Cpu:
             self.windows_name = 'x64'
             self.windows_config = 'x64'
             self.windows_suffix = '64'
+        elif name in ('arm64', 'ARM64', 'aarch64'):
+            self.bits = 64
+            self.windows_subdir = 'ARM64/'
+            self.windows_name = 'ARM64'
+            self.windows_config = 'ARM64'
+            self.windows_suffix = '64'
         else:
             assert 0, f'Unrecognised cpu name: {name}'
 
@@ -220,7 +226,18 @@ def cpu_name():
     '''
     Returns name of CPU.
     '''
-    ret = f'x{32 if sys.maxsize == 2**31 - 1 else 64}'
+    # Check if building for ARM64 via environment variable
+    target_arch = os.environ.get('CIBW_ARCHS_WINDOWS', '').upper()
+    if target_arch == 'ARM64':
+        ret = 'arm64'
+    else:
+        # Check platform machine type
+        machine = platform.machine().lower()
+        if machine in ('arm64', 'aarch64'):
+            ret = 'arm64'
+        else:
+            # Default x32/x64 detection
+            ret = f'x{32 if sys.maxsize == 2**31 - 1 else 64}'
     return ret
 
@@ -350,7 +367,7 @@ class BuildDirs:
             self.python_version = None
             for flag in flags:
-                if flag in ('x32', 'x64'):
+                if flag in ('x32', 'x64', 'arm64', 'ARM64'):
                     self.cpu = Cpu(flag)
```

Apply with:
```bash
cd mupdf-1.27.0-source
git apply mupdf_state.patch
```

### 4.3 MuPDF __main__.py Patch

Save as `mupdf_main.patch`:

```diff
diff --git a/scripts/wrap/__main__.py b/scripts/wrap/__main__.py
index 1234567..abcdefg 100644
--- a/scripts/wrap/__main__.py
+++ b/scripts/wrap/__main__.py
@@ -1750,14 +1750,24 @@ def build():
     if state.state_.windows:
-        libdir = f'{build_dirs.dir_mupdf}/platform/win32/'
-        mupdfcpp_lib = f'{build_dirs.dir_mupdf}/platform/win32/'
-        if build_dirs.cpu.bits == 64:
+        # Use CPU-aware paths
+        libdir = f'{build_dirs.dir_mupdf}/platform/win32/'
+        
+        # Add CPU-specific subdirectory
+        if build_dirs.cpu.name == 'arm64':
+            libdir += 'ARM64/'
+        elif build_dirs.cpu.bits == 64:
             libdir += 'x64/'
+        # else: x32, no subdirectory
+        
+        # Add build type
         libdir += 'Debug/' if debug else 'Memento/' if memento else 'Release/'
+        
         libs = list()
-        libs.append(libdir + ('mupdfcpp64.lib' if build_dirs.cpu.bits == 64 else 'mupdfcpp.lib'))
+        
+        # Use CPU-aware library name
+        if build_dirs.cpu.name in ('arm64', 'x64'):
+            libs.append(libdir + f'mupdfcpp{build_dirs.cpu.windows_suffix}.lib')
+        else:
+            libs.append(libdir + 'mupdfcpp.lib')
```

Apply with:
```bash
cd mupdf-1.27.0-source
git apply mupdf_main.patch
```

## Part 5: Troubleshooting

### 5.1 Common Issues

#### Issue: "py3-abi3-win_arm64.whl is not a supported wheel"

**Cause**: Old wheel with incorrect tag still exists in wheelhouse.

**Solution**:
```powershell
Remove-Item wheelhouse\*py3*.whl
```

#### Issue: "v142 build tools not found"

**Cause**: Visual Studio solution needs retargeting for VS 2022.

**Solution**: The `retarget_vs_solution()` function should handle this automatically. If it doesn't work:
```powershell
# Manually retarget
cd mupdf-1.27.0-source\platform\win32
devenv mupdf.sln /Upgrade
```

#### Issue: "x64/Release/mupdfcpp64.lib not found"

**Cause**: Build system is still using hardcoded x64 paths.

**Solution**: Ensure all patches are applied correctly. Verify with:
```powershell
cd mupdf-1.27.0-source
git diff scripts/wrap/__main__.py
```

#### Issue: "WindowsCpu has no attribute 'windows_subdir'"

**Cause**: wdev.py patch not applied.

**Solution**: Apply the wdev.py patch to both MuPDF and PyMuPDF:
```bash
cd mupdf-1.27.0-source
git apply ../mupdf_wdev.patch

cd ..
git apply mupdf_wdev.patch  # Apply to PyMuPDF's wdev.py too
```

### 5.2 Verification Steps

After applying all patches, verify:

```powershell
# 1. Check MuPDF patches
cd mupdf-1.27.0-source
git diff scripts/wdev.py | findstr "arm64"
git diff scripts/wrap/state.py | findstr "arm64"
git diff scripts/wrap/__main__.py | findstr "arm64"

# 2. Check PyMuPDF patches
cd ..
git diff setup.py | findstr "arm64"
git diff pipcl.py | findstr "py_limited_api"
git diff wdev.py | findstr "arm64"

# 3. Verify environment
python -c "import platform; print(f'Machine: {platform.machine()}')"
python -c "import sysconfig; print(f'Platform: {sysconfig.get_platform()}')"
```

### 5.3 Debug Build

For debugging, you can build with verbose output:

```powershell
$env:PIPCL_VERBOSE = "2"
$env:PYMUPDF_SETUP_MUPDF_BUILD_TYPE = "debug"
python setup.py bdist_wheel
```

## Part 6: CI/CD Integration

### 6.1 GitHub Actions Example

Save as `.github/workflows/build-arm64.yml`:

```yaml
name: Build ARM64 Wheels

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  build:
    runs-on: windows-latest
    
    steps:
    - uses: actions/checkout@v3
      with:
        submodules: recursive
    
    - name: Set up Python
      uses: actions/setup-python@v4
      with:
        python-version: '3.11'
        architecture: 'arm64'
    
    - name: Install dependencies
      run: |
        python -m pip install --upgrade pip
        pip install cibuildwheel
    
    - name: Apply patches
      run: |
        # Patches are applied automatically by setup.py
        python -c "import setup; print('Patches applied')"
    
    - name: Build wheels
      env:
        CIBW_ARCHS_WINDOWS: ARM64
        CIBW_BUILD: cp311-win_arm64
      run: |
        cibuildwheel --platform windows
    
    - name: Upload wheels
      uses: actions/upload-artifact@v3
      with:
        name: wheels
        path: wheelhouse/*.whl
```

### 6.2 Local CI Testing

For local testing without GitHub Actions:

```powershell
# Install cibuildwheel
pip install cibuildwheel

# Set environment
$env:CIBW_ARCHS_WINDOWS = "ARM64"
$env:CIBW_BUILD = "cp311-win_arm64"

# Build
cibuildwheel --platform windows

# Check results
dir wheelhouse\*.whl
```

## Part 7: Testing the Built Wheel

### 7.1 Basic Import Test

```powershell
# Create test script
@"
import pymupdf
import sys

print(f'PyMuPDF version: {pymupdf.__version__}')
print(f'Python version: {sys.version}')
print(f'Platform: {sys.platform}')

# Test basic functionality
doc = pymupdf.open()
page = doc.new_page()
page.insert_text((100, 100), 'Hello ARM64!')
doc.save('test_arm64.pdf')
doc.close()

print('Test passed!')
"@ | Out-File -Encoding utf8 test_pymupdf.py

# Run test
python test_pymupdf.py
```

### 7.2 Comprehensive Test

```powershell
# Install test dependencies
pip install pytest

# Run PyMuPDF's test suite
python -m pytest tests/ -v
```

### 7.3 Performance Test

```powershell
# Create performance test
@"
import pymupdf
import time

# Test PDF creation performance
start = time.time()
doc = pymupdf.open()
for i in range(100):
    page = doc.new_page()
    page.insert_text((100, 100), f'Page {i}')
doc.save('perf_test.pdf')
doc.close()
end = time.time()

print(f'Created 100-page PDF in {end-start:.2f} seconds')
"@ | Out-File -Encoding utf8 perf_test.py

python perf_test.py
```

## Part 8: Distribution

### 8.1 Wheel Naming Convention

Correct wheel names for different configurations:

- **ARM64 with stable ABI**: `pymupdfb-1.26.3-cp311-abi3-win_arm64.whl`
- **ARM64 without stable ABI**: `pymupdfb-1.26.3-cp311-cp311-win_arm64.whl`
- **x64 with stable ABI**: `pymupdfb-1.26.3-cp311-abi3-win_amd64.whl`

### 8.2 Upload to PyPI

```powershell
# Install twine
pip install twine

# Check wheel
twine check wheelhouse/*.whl

# Upload to TestPyPI first
twine upload --repository testpypi wheelhouse/*-win_arm64.whl

# Test installation from TestPyPI
pip install --index-url https://test.pypi.org/simple/ pymupdfb

# Upload to PyPI
twine upload wheelhouse/*-win_arm64.whl
```

### 8.3 Version Compatibility Matrix

| Python Version | Stable ABI | Wheel Tag | Status |
|---------------|------------|-----------|--------|
| 3.11 | Yes | cp311-abi3-win_arm64 | ✅ Tested |
| 3.12 | Yes | cp312-abi3-win_arm64 | ✅ Tested |
| 3.13 | No* | cp313-cp313-win_arm64 | ⚠️ SWIG issue |

*Note: Python 3.13 has SWIG issues with stable ABI as of 2025-01-04.

## Part 9: Summary of Changes

### Files Modified in MuPDF:
1. `scripts/wdev.py` - Added ARM64 CPU detection
2. `scripts/wrap/state.py` - Added ARM64 support to wrapper system
3. `scripts/wrap/__main__.py` - Fixed hardcoded x64 library paths

### Files Modified in PyMuPDF:
1. `setup.py` - Added ARM64 detection, patching, and build logic
2. `pipcl.py` - Fixed `tag_python()` to handle stable ABI correctly
3. `wdev.py` - Added ARM64 CPU detection (mirrors MuPDF changes)

### Key Concepts:
- **CPU Detection**: Uses `CIBW_ARCHS_WINDOWS` environment variable and `platform.machine()`
- **Library Paths**: Changed from hardcoded `x64/` to dynamic `{cpu.windows_subdir}`
- **Wheel Tags**: Changed from `py3` to `cp3X` for stable ABI compatibility
- **VS Retargeting**: Automatically upgrades solution files for VS 2022

## Part 10: Future Improvements

### 10.1 Potential Enhancements

1. **Automated Patch Application**: Currently patches are applied via `apply_mupdf_arm64_patches()`. Could be improved with:
   - Git submodule with pre-patched MuPDF
   - Automated patch generation from git diffs

2. **Multi-Architecture Builds**: Support building for multiple architectures in one command:
   ```powershell
   $env:CIBW_ARCHS_WINDOWS = "AMD64 ARM64"
   cibuildwheel --platform windows
   ```

3. **Cross-Compilation**: Build ARM64 wheels on x64 machines (requires cross-compilation toolchain)

4. **Upstream Contributions**: Submit patches to MuPDF and PyMuPDF repositories

### 10.2 Known Limitations

1. **Python 3.13**: SWIG generates incorrect stable ABI code (see https://github.com/swig/swig/issues/3059)
2. **Tesseract**: Not tested on ARM64, may require additional patches
3. **Performance**: ARM64 performance not yet benchmarked against x64

## Part 11: References

### Documentation
- [PEP 425 - Compatibility Tags](https://peps.python.org/pep-0425/)
- [PEP 384 - Stable ABI](https://peps.python.org/pep-0384/)
- [PyMuPDF Documentation](https://pymupdf.readthedocs.io/)
- [MuPDF Documentation](https://mupdf.com/docs/)

### Related Issues
- [PyMuPDF ARM64 Support Discussion](https://github.com/pymupdf/PyMuPDF/issues/)
- [SWIG Python 3.13 Stable ABI Issue](https://github.com/swig/swig/issues/3059)

### Tools
- [cibuildwheel](https://cibuildwheel.readthedocs.io/)
- [Visual Studio 2022](https://visualstudio.microsoft.com/)
- [Python for Windows ARM64](https://www.python.org/downloads/windows/)

## Appendix A: Complete Patch Bundle

For convenience, here's a script to create all patches at once:

Save as `create_patches.ps1`:

```powershell
# Create patches directory
New-Item -ItemType Directory -Force -Path patches

# MuPDF wdev.py patch
@"
diff --git a/scripts/wdev.py b/scripts/wdev.py
index 1234567..abcdefg 100644
--- a/scripts/wdev.py
+++ b/scripts/wdev.py
@@ -150,6 +150,12 @@ class WindowsCpu:
             self.windows_name = 'x64'
             self.windows_config = 'x64'
             self.windows_suffix = '64'
+        elif name in ('arm64', 'ARM64', 'aarch64'):
+            self.bits = 64
+            self.windows_subdir = 'ARM64/'
+            self.windows_name = 'ARM64'
+            self.windows_config = 'ARM64'
+            self.windows_suffix = '64'
         else:
             assert 0, f'Unrecognised cpu name: {name}'
 
@@ -200,7 +206,18 @@ def _cpu_name():
     '''
     Returns name of CPU that we are running on.
     '''
-    return f'x{32 if sys.maxsize == 2**31 - 1 else 64}'
+    # Check if building for ARM64 via environment variable
+    target_arch = os.environ.get('CIBW_ARCHS_WINDOWS', '').upper()
+    if target_arch == 'ARM64':
+        return 'arm64'
+    
+    # Check platform machine type
+    machine = platform.machine().lower()
+    if machine in ('arm64', 'aarch64'):
+        return 'arm64'
+    
+    # Default x32/x64 detection
+    return f'x{32 if sys.maxsize == 2**31 - 1 else 64}'
"@ | Out-File -Encoding utf8 patches\mupdf_wdev.patch

# MuPDF state.py patch
@"
diff --git a/scripts/wrap/state.py b/scripts/wrap/state.py
index 1234567..abcdefg 100644
--- a/scripts/wrap/state.py
+++ b/scripts/wrap/state.py
@@ -200,6 +200,12 @@ class Cpu:
             self.windows_name = 'x64'
            self.windows_config = 'x64'
            self.windows_suffix = '64'
+        elif name in ('arm64', 'ARM64', 'aarch64'):
+            self.bits = 64
+            self.windows_subdir = 'ARM64/'
+            self.windows_name = 'ARM64'
+            self.windows_config = 'ARM64'
+            self.windows_suffix = '64'
         else:
             assert 0, f'Unrecognised cpu name: {name}'
 
@@ -220,7 +226,18 @@ def cpu_name():
     '''
     Returns name of CPU.
     '''
-    ret = f'x{32 if sys.maxsize == 2**31 - 1 else 64}'
+    # Check if building for ARM64 via environment variable
+    target_arch = os.environ.get('CIBW_ARCHS_WINDOWS', '').upper()
+    if target_arch == 'ARM64':
+        ret = 'arm64'
+    else:
+        # Check platform machine type
+        machine = platform.machine().lower()
+        if machine in ('arm64', 'aarch64'):
+            ret = 'arm64'
+        else:
+            # Default x32/x64 detection
+            ret = f'x{32 if sys.maxsize == 2**31 - 1 else 64}'
     return ret
 
@@ -350,7 +367,7 @@ class BuildDirs:
             self.python_version = None
             for flag in flags:
-                if flag in ('x32', 'x64'):
+                if flag in ('x32', 'x64', 'arm64', 'ARM64'):
                     self.cpu = Cpu(flag)
"@ | Out-File -Encoding utf8 patches\mupdf_state.patch

# MuPDF __main__.py patch
@"
diff --git a/scripts/wrap/__main__.py b/scripts/wrap/__main__.py
index 1234567..abcdefg 100644
--- a/scripts/wrap/__main__.py
+++ b/scripts/wrap/__main__.py
@@ -1750,14 +1750,24 @@ def build():
     if state.state_.windows:
-        libdir = f'{build_dirs.dir_mupdf}/platform/win32/'
-        mupdfcpp_lib = f'{build_dirs.dir_mupdf}/platform/win32/'
-        if build_dirs.cpu.bits == 64:
+        # Use CPU-aware paths
+        libdir = f'{build_dirs.dir_mupdf}/platform/win32/'
+        
+        # Add CPU-specific subdirectory
+        if build_dirs.cpu.name == 'arm64':
+            libdir += 'ARM64/'
+        elif build_dirs.cpu.bits == 64:
             libdir += 'x64/'
+        # else: x32, no subdirectory
+        
+        # Add build type
         libdir += 'Debug/' if debug else 'Memento/' if memento else 'Release/'
+        
         libs = list()
-        libs.append(libdir + ('mupdfcpp64.lib' if build_dirs.cpu.bits == 64 else 'mupdfcpp.lib'))
+        
+        # Use CPU-aware library name
+        if build_dirs.cpu.name in ('arm64', 'x64'):
+            libs.append(libdir + f'mupdfcpp{build_dirs.cpu.windows_suffix}.lib')
+        else:
+            libs.append(libdir + 'mupdfcpp.lib')
"@ | Out-File -Encoding utf8 patches\mupdf_main.patch

Write-Host "Patches created in ./patches/ directory"
```

Run with:
```powershell
.\create_patches.ps1
```

## Appendix B: Quick Start Script

Save as `build_arm64.ps1`:

```powershell
# Quick start script for building PyMuPDF ARM64 wheels

param(
    [switch]$Clean,
    [switch]$Test
)

Write-Host "PyMuPDF ARM64 Build Script" -ForegroundColor Green

# Check prerequisites
Write-Host "`nChecking prerequisites..." -ForegroundColor Yellow
$python = python -c "import platform; print(platform.machine())"
if ($python -ne "ARM64") {
    Write-Host "ERROR: Not running on ARM64 Python" -ForegroundColor Red
    exit 1
}
Write-Host "✓ Running on ARM64 Python" -ForegroundColor Green

# Clean if requested
if ($Clean) {
    Write-Host "`nCleaning previous builds..." -ForegroundColor Yellow
    Remove-Item wheelhouse\*.whl -ErrorAction SilentlyContinue
    Remove-Item -Recurse -Force mupdf-*-source\build -ErrorAction SilentlyContinue
    Write-Host "✓ Cleaned" -ForegroundColor Green
}

# Set environment
Write-Host "`nSetting environment..." -ForegroundColor Yellow
$env:CIBW_ARCHS_WINDOWS = "ARM64"
$env:PIPCL_VERBOSE = "1"
Write-Host "✓ Environment set" -ForegroundColor Green

# Build
Write-Host "`nBuilding wheel..." -ForegroundColor Yellow
python setup.py bdist_wheel
if ($LASTEXITCODE -ne 0) {
    Write-Host "ERROR: Build failed" -ForegroundColor Red
    exit 1
}
Write-Host "✓ Build complete" -ForegroundColor Green

# Check wheel
Write-Host "`nChecking wheel..." -ForegroundColor Yellow
$wheels = Get-ChildItem wheelhouse\*-win_arm64.whl
if ($wheels.Count -eq 0) {
    Write-Host "ERROR: No ARM64 wheel found" -ForegroundColor Red
    exit 1
}
Write-Host "✓ Found wheel: $($wheels[0].Name)" -ForegroundColor Green

# Test if requested
if ($Test) {
    Write-Host "`nTesting wheel..." -ForegroundColor Yellow
    pip install --force-reinstall $wheels[0].FullName
    python -c "import pymupdf; print(f'PyMuPDF {pymupdf.__version__} imported successfully')"
    if ($LASTEXITCODE -ne 0) {
        Write-Host "ERROR: Test failed" -ForegroundColor Red
        exit 1
    }
    Write-Host "✓ Test passed" -ForegroundColor Green
}

Write-Host "`n✓ All done!" -ForegroundColor Green
```

Run with:
```powershell
# Clean build and test
.\build_arm64.ps1 -Clean -Test

# Just build
.\build_arm64.ps1
```

---

**Document Version**: 1.0  
**Last Updated**: 2025-01-04  
**Tested With**: 
- PyMuPDF 1.26.3
- MuPDF 1.27.0
- Python 3.11.9 ARM64
- Visual Studio 2022
- Windows 11 ARM64
             
