# Linux Build Workflows

This directory contains GitHub Actions workflows for building TizenTubeCobalt on Linux.

## Available Workflows

### 1. `linux_x64_release.yaml`
**Purpose:** Standard monolithic executable build

**Use when:**
- Building production releases
- Need simple build without separate library
- Don't need fast incremental builds

**Output:** Single `cobalt` executable

---

### 2. `linux_x64_full_build.yaml` ⚡ RECOMMENDED FOR DEVELOPMENT
**Purpose:** Modular build with separate libcobalt.so

**Use when:**
- Setting up development environment
- Need fast incremental builds
- Want to manually reuse libcobalt.so

**Output:** 
- `libcobalt.so` (core Cobalt library)
- `loader_app` (small loader executable)
- Available as GitHub Actions artifact

**Benefits:**
- ⚡ **Much faster incremental builds** (1-5 min vs 30-60 min)
- 🔧 **Manually replace libcobalt.so** without rebuilding
- 📦 **Download from GitHub Actions** for local dev

---

## Quick Start

### For Development:

1. **First build** - Run `linux_x64_full_build` workflow:
   ```
   Trigger: workflow_dispatch
   Output: Download libcobalt.so artifact
   ```

2. **Local development** - See [docs/linux_build_workflows.md](../../docs/linux_build_workflows.md)

### For Releases:

Run `linux_x64_release` workflow with tag push or manual trigger.

---

## Documentation

See [docs/linux_build_workflows.md](../../docs/linux_build_workflows.md) for:
- Detailed build instructions
- Fast development workflow guide
- Troubleshooting tips
- Build time comparisons
