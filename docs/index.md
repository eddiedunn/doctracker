# doctrack Documentation

## What is doctrack?

doctrack is a stateless tool for creating documentation from source code across multi-repository ecosystems. It operates on target projects without maintaining its own state files, reading configuration, analyzing source code, and generating documentation tasks that can be executed with automation tools like grind.

**Key principle:** Source code is truth. We don't audit existing documentation - we read source code and create documentation from what the code actually does.

## Operational Modes

doctrack supports two operational modes to accommodate different project structures:

### Single Mode
For working within a single repository or when all repositories share a parent directory.

- `.doctrack/` directory created in the working directory
- Session configuration in `.doctrack/session.yaml`
- State tracked in `.doctrack/state.yaml`
- All repositories referenced with relative or absolute paths
- **Ideal for:** Single repo projects, monorepos, or co-located multi-repos

### Hub Mode
For managing documentation across multiple distributed repositories from a central location.

- Central `.doctrack/` hub manages multiple remote repositories
- Each repo maintains its own `docs/.doctrack/` working files
- Use `dt-collate` to gather completed documentation from repos
- Use `dt-build` to compile final documentation output
- **Ideal for:** Multi-repo ecosystems, microservices, distributed teams

## Quick Navigation

### Getting Started
- [Quick Start Guide](../README.md#quick-start) - Get up and running quickly
- [Process Overview](../PROCESS.md#overview) - Understand the 7-step workflow
- [Core Principles](../PROCESS.md#core-principles) - Source code is truth

### The 7-Step Process
1. [Initialization](../PROCESS.md#step-0-initialization) - Set up doctrack system
2. [Chunk Repository](../PROCESS.md#step-1-chunk-repository) - Inventory source files
3. [Review Chunks](../PROCESS.md#step-2-review-chunks) - Identify documentation needs
4. [Create Tasks](../PROCESS.md#step-3-create-tasks) - Generate actionable tasks
5. [Generate Tasks YAML](../PROCESS.md#step-4-generate-tasks-yaml) - Compile for grind
6. [Execute Tasks](../PROCESS.md#step-5-execute-tasks) - Run with grind automation
7. [Mark Complete](../PROCESS.md#step-7-mark-complete) - Verify and track completion

### Key Concepts
- [Operational Modes](../PROCESS.md#operational-modes) - Single vs Hub mode
- [Session Management](../PROCESS.md#session-management) - Working across conversations
- [State Transitions](../PROCESS.md#state-transitions) - How progress is tracked
- [Audience Checklists](../PROCESS.md#audience-checklist-usage) - Meeting user needs
- [Documentation Standards](../PROCESS.md#standards-application) - Ensuring consistency

### Commands Reference
| Command | Purpose |
|---------|---------|
| `dt-init` | Initialize session from session.yaml |
| `dt-status` | Show progress dashboard |
| `dt-sessions` | List and manage sessions |
| `dt-use <path>` | Switch to different session |
| `dt-chunk <repo>` | Inventory source files |
| `dt-review <chunk>` | Review chunk, identify doc needs |
| `dt-task <chunk>` | Create documentation tasks |
| `dt-generate <repo>` | Generate tasks.yaml for grind |
| `dt-complete <chunk>` | Mark chunk complete |
| `dt-collate` | Gather docs from repos (hub mode) |
| `dt-build` | Build final documentation (hub mode) |

### Configuration
- [session.yaml.example](../session.yaml.example) - Configuration template
- [File Locations](../PROCESS.md#file-locations) - Where files are stored
- [Repository Configuration](../PROCESS.md#operational-modes) - Single vs Hub setup

### Workflow Examples
- [Single Repository Example](../PROCESS.md#example-1-single-repository-start-to-finish)
- [Multiple Repositories Example](../PROCESS.md#example-2-multiple-repositories-prioritized)
- [Resuming Work Example](../PROCESS.md#example-3-resuming-after-interruption)

## Next Steps

Ready to start? Follow the [Quick Start Guide](../README.md#quick-start) for your operational mode:
- **Single Mode:** Perfect for individual repos or monorepos
- **Hub Mode:** Ideal for managing multiple distributed repositories
