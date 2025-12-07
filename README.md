# doctrack

A stateless tool for creating documentation from source code across multi-repository ecosystems.

## What It Is

Doctrack operates on target projects without maintaining its own state files. It reads configuration, analyzes source code, and generates documentation tasks that can be executed with automation tools like grind.

**Key principle:** Source code is truth. We don't audit existing documentation - we read source code and create documentation from what the code actually does.

## Quick Start

### Single Repository Mode

For working within a single repository or when all repos share a parent directory:

```bash
# Initialize with interactive interview
/dt-init /path/to/my-project

# Check status and start documenting
/dt-status
/dt-chunk my-repo
```

See [docs/getting-started.md](docs/getting-started.md) for the full tutorial.

### Hub Mode

For managing documentation across multiple distributed repositories:

```bash
# Initialize hub with interactive interview
/dt-init /path/to/doctrack-hub

# Work through repositories
/dt-chunk <repo-name>
/dt-review <chunk-id>
/dt-task <chunk-id>
/dt-generate <repo-name>

# Collate and build final output
/dt-collate
/dt-build
```

See [docs/getting-started.md](docs/getting-started.md) for the full tutorial.

## Commands

| Command | Purpose |
|---------|---------|
| `/dt-init` | Initialize session with interactive interview |
| `/dt-status` | Show progress dashboard |
| `/dt-sessions` | List and manage sessions |
| `/dt-use <path>` | Switch to different session |
| `/dt-chunk <repo>` | Inventory source files |
| `/dt-review <chunk>` | Review chunk, identify doc needs |
| `/dt-task <chunk>` | Create documentation tasks |
| `/dt-generate <repo>` | Generate tasks.yaml for grind |
| `/dt-complete <chunk>` | Mark chunk complete |
| `/dt-collate` | Gather docs from repos (hub mode) |
| `/dt-build` | Build final documentation (hub mode) |

## Core Principles

1. **Source code is truth** - Documentation is created from reading and understanding source code
2. **Not an audit system** - We don't compare against existing docs
3. **Stateless operation** - Operates on target projects without internal state
4. **Automation-ready** - Generates grind-compatible task files
5. **Systematic process** - 7-step workflow per repository

## Documentation

- [PROCESS.md](PROCESS.md) - Complete process documentation
- [docs/getting-started.md](docs/getting-started.md) - Getting started tutorial
- `docs/` - Additional guides and examples

## Configuration

Configuration is created through the `/dt-init` interactive interview which guides you through:
- Repository definitions
- Audience checklists
- Documentation standards
- Hub mode settings

## State Tracking

Progress is tracked in `.doctrack/state.yaml` within your project directory. This file can be committed to git to preserve progress across sessions.
