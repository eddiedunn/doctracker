# Getting Started with doctrack

This guide will walk you through using doctrack for the first time, from installation to completing your first documentation cycle.

## Prerequisites

Before starting, ensure you have:

1. **grind** - The task automation tool used to execute documentation tasks
   - Install from: [grind repository](https://github.com/anthropics/grind)
   - Used for: Running automated documentation generation tasks

2. **Claude Code** - The AI assistant that runs doctrack commands
   - doctrack operates as a set of Claude Code slash commands
   - All `/dt-*` commands run within Claude Code conversations

3. **A project to document** - One or more repositories with source code

## Choosing Your Mode

doctrack supports two operational modes. Choose based on your project structure:

### Single Mode

**Best for:**
- Single repository projects
- Monorepos with multiple components
- Multiple repositories in the same parent directory
- Projects where you're documenting from within the repository

**Structure:**
```
my-project/
├── .doctrack/              # Configuration and state
│   ├── session.yaml        # Session configuration
│   └── state.yaml          # Progress tracking
├── src/                    # Your source code
└── docs/                   # Generated documentation
```

### Hub Mode

**Best for:**
- Multiple distributed repositories
- Microservices architectures
- Multi-repo ecosystems
- Projects with separate documentation repository

**Structure:**
```
doctrack-hub/               # Central documentation hub
├── .doctrack/              # Hub configuration
│   ├── session.yaml        # Multi-repo configuration
│   ├── state.yaml          # Overall progress
│   ├── collated/           # Gathered documentation
│   └── output/             # Built documentation site

../repos/                   # Your distributed repositories
├── repo-a/
├── repo-b/
└── repo-c/
```

## Running /dt-init

The initialization process sets up your doctrack session through an interactive interview.

### Starting Initialization

In Claude Code, run:

```
/dt-init
```

Or specify a project path:

```
/dt-init /path/to/my-project
```

### The 5-Phase Interview

The initialization will ask you questions across 5 phases:

#### Phase 1: Project Scope
- Project type (single repository or documentation hub)
- Project name
- Project description (optional)

#### Phase 2: Repositories
For **Single Mode:**
- Repository path (defaults to current directory)
- Content type (Python, Ansible, JSON Schema, YAML Config, Mixed)
- Source directories to scan
- Chunk ID prefix (e.g., "API" for API-AUTH-001 style IDs)

For **Hub Mode:**
- Number of repositories
- For each repository:
  - Name and path
  - Content type
  - Priority (high, medium, low)
  - Chunk ID prefix

#### Phase 3: Audiences
- Select target audiences (Administrator, Developer, End User, or Custom)
- Customize checklists for each audience
- Define custom audiences if needed

Audiences ensure documentation meets the needs of different user groups.

#### Phase 4: Standards
- Docstring style (NumPy, Google, or Sphinx)
- Whether to require examples in all documentation

Standards ensure consistency across all generated documentation.

#### Phase 5: Output
- Output path for generated documentation site

### What Gets Created

After completing the interview, doctrack creates:

1. **`.doctrack/session.yaml`** - Your session configuration
2. **`.doctrack/state.yaml`** - Progress tracking file
3. **`~/.local/state/doctrack/sessions.yaml`** - Global session registry
4. **`~/.local/state/doctrack/current`** - Current active session marker

## Your First Documentation Cycle

Now that you're initialized, let's document your first chunk of code.

### Step 1: Check Status

```
/dt-status
```

This shows your repositories and their current status. All will be "pending" initially.

### Step 2: Chunk a Repository

Break a repository into manageable documentation chunks:

```
/dt-chunk my-repo
```

This will:
- Scan all source files in configured directories
- Group files into logical chunks (~10 files each)
- Create a `CHUNK_TRACKING.md` file
- Assign chunk IDs like `API-AUTH-001`

### Step 3: Review a Chunk

Review the first chunk to identify documentation needs:

```
/dt-review API-AUTH-001
```

This will:
- Read all source files in the chunk
- Understand what the code does
- Identify missing docstrings, guides, and examples
- Apply audience checklist requirements
- Display review results

**Important:** This is NOT an audit. You're reading source code to understand what it does and determining what documentation needs to be written.

### Step 4: Create Documentation Tasks

Transform review findings into actionable tasks:

```
/dt-task API-AUTH-001
```

This will:
- Create detailed task specifications
- Include verification steps
- Generate grind-compatible YAML
- Save to `{repo}/docs/.doctrack/chunks/API-AUTH-001-task.md`

### Step 5: Repeat for All Chunks

Continue reviewing and creating tasks for each chunk:

```
/dt-review API-AUTH-002
/dt-task API-AUTH-002

/dt-review API-USERS-003
/dt-task API-USERS-003

# ... and so on
```

### Step 6: Generate Tasks File

Once all chunks have tasks, compile them into a single file:

```
/dt-generate my-repo
```

This creates `.doctrack/my-repo-tasks.yaml` ready for grind.

### Step 7: Execute with grind

Run the automated documentation generation:

```bash
grind batch .doctrack/my-repo-tasks.yaml
```

grind will:
- Read each task specification
- Analyze source code
- Write docstrings, guides, and examples
- Run verification commands
- Report results

### Step 8: Human Review

Manually review the generated documentation:
- Check docstrings match actual code behavior
- Verify examples run correctly
- Ensure guides are accurate and clear
- Validate audience requirements are met

### Step 9: Mark Complete

After reviewing and approving the documentation:

```
/dt-complete API-AUTH-001
```

Repeat for each chunk:

```
/dt-complete API-AUTH-002
/dt-complete API-USERS-003
# ... and so on
```

This verifies documentation was created correctly and updates progress tracking.

### Step 10: Check Final Status

```
/dt-status
```

When all chunks are complete, the repository status will show "complete"!

## Next Steps

### For Single Mode Projects

You're done! Your documentation is in:
- **Docstrings:** In your source files
- **Guides:** `{repo}/docs/guides/`
- **Examples:** `{repo}/docs/examples/`

### For Hub Mode Projects

After completing all repositories:

1. **Collate documentation:**
   ```
   /dt-collate
   ```
   Gathers all documentation into `.doctrack/collated/`

2. **Build final output:**
   ```
   /dt-build
   ```
   Compiles into unified documentation site in `.doctrack/output/`

### Multiple Repositories

If you have more repositories to document:

```
/dt-chunk next-repo
/dt-review NEXT-AUTH-001
# ... repeat the process
```

### Resuming Work Later

In a new Claude Code conversation:

```
/dt-sessions          # List all sessions
/dt-use my-project    # Switch to your session
/dt-status           # See where you left off
```

### Advanced Features

- **Session Management:** Use `/dt-sessions` and `/dt-use` to work across multiple projects
- **Priority Order:** Configure repository priorities to guide your workflow
- **Custom Audiences:** Define audiences specific to your project needs
- **Verification Commands:** Customize verification in task files

## Common Questions

### What if I stop mid-process?

No problem! Your progress is saved in `.doctrack/state.yaml`. Use `/dt-status` to see where you left off, and continue from there.

### Can I customize the task specifications?

Yes! Task files are in `{repo}/docs/.doctrack/chunks/`. Edit them before running `/dt-generate`.

### What if grind tasks fail verification?

You can:
1. Fix task specifications and re-run `/dt-generate` and grind
2. Manually fix the generated documentation
3. Adjust verification commands in the task files

### Do I need to complete all chunks before using grind?

No, but it's recommended. You can run `/dt-generate` with partial chunks, but it's more efficient to batch them together.

## Getting Help

- **Process details:** See [PROCESS.md](../PROCESS.md)
- **Configuration:** See [session.yaml.example](../session.yaml.example)
- **Commands reference:** Run `/dt-help` or see [docs/index.md](index.md)
- **Workflow examples:** See [PROCESS.md workflow examples](../PROCESS.md#workflow-examples)

## Quick Reference

### Essential Commands

```
/dt-init              # Initialize new session
/dt-status            # Check progress
/dt-chunk <repo>      # Inventory source files
/dt-review <chunk>    # Identify documentation needs
/dt-task <chunk>      # Create documentation tasks
/dt-generate <repo>   # Compile tasks for grind
/dt-complete <chunk>  # Mark chunk complete
```

### The Core Principle

**Source code is truth.** Always document what the code actually does, not what old documentation says. Never audit or compare against existing docs - read the source and write documentation from what you understand.

---

Ready to start? Run `/dt-init` and begin your first documentation cycle!
