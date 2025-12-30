# Linux Build Workflows

This document explains the different Linux build workflows available for TizenTubeCobalt.

## Overview

There are two main build modes for Linux:

1. **Monolithic Build** - Single executable containing all Cobalt code (simpler, suitable for releases)
2. **Modular Build with libcobalt.so** - Separate shared library for faster development iterations (only rebuilds loader for small changes)

## Workflows

### 1. linux_x64_release

**Purpose:** Standard release build as a single monolithic executable.

**When to use:** 
- For production releases
- When you don't need to iterate quickly on the codebase
- Simple build process

**Build command:**
```bash
gn gen out/linux-x64x11_gold --args='
  target_platform="linux-x64x11"
  target_os="linux"
  target_cpu="x64"
  build_type="gold"
  sb_api_version=15'
ninja -C out/linux-x64x11_gold cobalt_install
```

### 2. linux_x64_full_build

**Purpose:** Full modular build that creates libcobalt.so separately.

**When to use:**
- Initial build to create libcobalt.so for development
- When you make changes that affect the core Cobalt library
- To create a reusable libcobalt.so for faster incremental builds

**Build command:**
```bash
gn gen out/linux-x64x11_gold --args='
  target_platform="linux-x64x11"
  target_os="linux"
  target_cpu="x64"
  build_type="gold"
  sb_api_version=15
  build_with_separate_cobalt_toolchain=true'
ninja -C out/linux-x64x11_gold cobalt_install
```

**Output:**
- `out/linux-x64x11_gold/install/lib/libcobalt.so` - The Cobalt shared library
- `out/linux-x64x11_gold/install/bin/loader_app` - Small loader executable
- `out/linux-x64x11_gold/content/` - Content files

## Fast Development Workflow with libcobalt.so

Once you have built libcobalt.so using the full build workflow, you can:

1. **Initial full build** (do this once):
   ```bash
   gn gen out/linux-x64x11_gold --args='
     target_platform="linux-x64x11"
     target_os="linux"
     target_cpu="x64"
     build_type="gold"
     sb_api_version=15
     build_with_separate_cobalt_toolchain=true'
   ninja -C out/linux-x64x11_gold cobalt_install
   ```

2. **Save libcobalt.so** for reuse:
   ```bash
   # Copy the built libcobalt.so to a safe location
   cp out/linux-x64x11_gold/install/lib/libcobalt.so ~/libcobalt-backup.so
   ```

3. **For subsequent builds** (when you haven't changed core Cobalt code):
   - If you only modified loader_app or application-level code, rebuild just that target:
   ```bash
   ninja -C out/linux-x64x11_gold loader_app
   ```

4. **Manually provide libcobalt.so** (if you need to rebuild from scratch but want to skip rebuilding libcobalt.so):
   ```bash
   # Build everything except the core library
   gn gen out/linux-x64x11_gold --args='
     target_platform="linux-x64x11"
     target_os="linux"
     target_cpu="x64"
     build_type="gold"
     sb_api_version=15
     build_with_separate_cobalt_toolchain=true'
   
   # Copy your pre-built libcobalt.so
   mkdir -p out/linux-x64x11_gold/install/lib
   cp ~/libcobalt-backup.so out/linux-x64x11_gold/install/lib/libcobalt.so
   
   # Build only the loader and other components (libcobalt.so won't be rebuilt)
   ninja -C out/linux-x64x11_gold loader_app
   ```

## Understanding Build Time Savings

With the modular build approach:
- **Full build**: ~30-60 minutes (builds libcobalt.so + loader_app)
- **Incremental build**: ~1-5 minutes (builds only loader_app, reuses existing libcobalt.so)

This is particularly useful when:
- Making changes to application logic, UI, or platform-specific code
- Testing different configurations
- Iterating on features that don't touch core Cobalt functionality

## Troubleshooting

### libcobalt.so not found at runtime
Ensure libcobalt.so is in the correct location:
```bash
export LD_LIBRARY_PATH=out/linux-x64x11_gold/install/lib:$LD_LIBRARY_PATH
out/linux-x64x11_gold/install/bin/loader_app
```

### Build errors after copying libcobalt.so
If you get symbol mismatch errors, your libcobalt.so may be out of sync with your code changes. Rebuild libcobalt.so:
```bash
ninja -C out/linux-x64x11_gold //cobalt/browser:cobalt
```

## GitHub Actions Artifacts

The `linux_x64_full_build` workflow uploads libcobalt.so as an artifact. You can:
1. Download it from the GitHub Actions run
2. Extract and place it in your local build directory
3. Use it for local development without rebuilding the core library
