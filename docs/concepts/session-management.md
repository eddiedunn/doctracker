# Session Management

doctrack uses a session-based architecture to manage documentation projects. This design allows you to work on multiple projects simultaneously while maintaining isolated state and configuration for each project.

## Overview

The session management system consists of three key components:

1. **Global state directory** - XDG-compliant location tracking all sessions
2. **Session registry** - Central registry of known projects
3. **Per-project state** - Isolated configuration and progress tracking for each project

This architecture ensures:
- Clean separation between doctrack tool code and project state
- Multiple concurrent projects without conflicts
- Easy switching between projects
- Persistent session tracking across Claude Code conversations

## XDG State Location

doctrack follows the [XDG Base Directory Specification](https://specifications.freedesktop.org/basedir-spec/basedir-spec-latest.html) for storing global state.

### Default Location

```
~/.local/state/doctrack/
```

This expands to:
- **macOS/Linux:** `/Users/{username}/.local/state/doctrack/`
- **With XDG_STATE_HOME set:** `$XDG_STATE_HOME/doctrack/`

### Directory Structure

```
~/.local/state/doctrack/
├── sessions.yaml          # Registry of all known sessions
└── current               # Current active session path (text file)
```

### Why XDG?

Using the XDG standard provides:
- **Consistency** - Follows platform conventions for state storage
- **Discoverability** - State lives in a well-known, standard location
- **Separation** - User state is separate from application code
- **Portability** - Works across Unix-like systems

## Session Registry (sessions.yaml)

The global session registry tracks all known doctrack projects.

### Location

```
~/.local/state/doctrack/sessions.yaml
```

### Structure

```yaml
version: 1
sessions:
  - path: /Users/gdunn6/code/jenkins-project/jenkins-platform-docs
    name: "Jenkins Platform Docs"
    type: hub
    last_accessed: 2025-12-07T10:30:00Z
    status: in_progress

  - path: /Users/gdunn6/code/my-api-server
    name: "My API Server"
    type: single
    last_accessed: 2025-12-06T14:00:00Z
    status: complete
```

### Fields

- **path** - Absolute path to the project directory (contains `.doctrack/`)
- **name** - Human-readable project name from session configuration
- **type** - Session mode: `single` or `hub`
- **last_accessed** - ISO 8601 timestamp of most recent use
- **status** - Overall progress: `pending`, `in_progress`, or `complete`

### Automatic Management

The registry is automatically maintained:

- **Created** - When you run `/dt-init` for the first time
- **Updated** - When you switch sessions with `/dt-use`
- **Cleaned** - Stale entries (paths that no longer exist) are removed when listing sessions

### Session Discovery

When you run `/dt-sessions`:

1. Read `sessions.yaml` for known sessions
2. Validate each path still exists
3. Remove stale entries from the registry
4. Display active sessions with their status

## Current Project Context

The active session is tracked in a simple text file.

### Location

```
~/.local/state/doctrack/current
```

### Contents

A single line containing the absolute path to the current project:

```
/Users/gdunn6/code/jenkins-project/jenkins-platform-docs
```

### Usage

This file allows doctrack commands to determine which project you're working on:

```bash
/dt-status              # Uses current session
/dt-chunk my-repo       # Operates on current session's repos
/dt-use other-project   # Updates this file to switch sessions
```

### Benefits

- **Persistent across conversations** - New Claude Code sessions automatically resume your work
- **Explicit switching** - Use `/dt-use` to change projects
- **Simple and reliable** - Plain text file, easy to inspect or manually edit if needed

## Per-Project .doctrack/ Structure

Each project has its own `.doctrack/` directory containing all session-specific data.

### Location

```
{project-root}/.doctrack/
```

For example:
```
/Users/gdunn6/code/my-api-server/.doctrack/
```

### Directory Structure

```
my-project/.doctrack/
├── session.yaml                    # Project configuration
├── STATE.yaml                      # Progress tracking
├── repos/                          # Per-repository state (hub mode)
│   ├── repo-a.yaml
│   ├── repo-b.yaml
│   └── repo-c.yaml
├── my-project-tasks.yaml           # Generated grind tasks
├── collated/                       # Hub mode: gathered docs
│   ├── repo-a/
│   ├── repo-b/
│   └── repo-c/
└── output/                         # Hub mode: built documentation site
    └── index.md
```

### session.yaml

Created by `/dt-init`, contains all project configuration:

```yaml
version: 1
created: 2025-12-07T10:30:00Z
type: single  # or "hub"

project:
  name: "My API Server"
  description: "REST API for user management"
  output_path: docs/

repositories:
  - name: my-api-server
    path: .
    content_type: python
    prefix: API
    priority: high
    source_dirs:
      - src/
    doc_dir: docs/

audiences:
  - id: developer
    name: Developer
    checklist:
      - "Code examples for common use cases"
      - "API reference with parameters and return values"

standards:
  docstring_style: google
  require_examples: true
```

### STATE.yaml

Tracks documentation progress:

```yaml
version: 1
repositories:
  my-api-server:
    status: in_progress
    chunks:
      API-AUTH-001:
        status: complete
        created: 2025-12-07T10:00:00Z
        completed: 2025-12-07T11:30:00Z
      API-AUTH-002:
        status: in_progress
        created: 2025-12-07T10:05:00Z
      API-USERS-003:
        status: pending
        created: 2025-12-07T10:10:00Z
```

### Generated Files

- **`{repo}-tasks.yaml`** - grind batch task file created by `/dt-generate`
- **`collated/`** - Hub mode only: documentation gathered from distributed repos
- **`output/`** - Hub mode only: final built documentation site

## State Isolation Between Projects

Each project's state is completely isolated from others.

### What This Means

**Different projects never interfere:**
```
project-a/.doctrack/
  session.yaml          # Project A configuration
  STATE.yaml            # Project A progress

project-b/.doctrack/
  session.yaml          # Project B configuration
  STATE.yaml            # Project B progress
```

**You can work on multiple projects:**
```bash
# In project A
/dt-use project-a
/dt-status              # Shows project A progress
/dt-chunk repo-a        # Creates chunks in project A

# Switch to project B
/dt-use project-b
/dt-status              # Shows project B progress
/dt-chunk repo-b        # Creates chunks in project B
```

### Benefits of Isolation

1. **No conflicts** - Projects can have different configurations without interference
2. **Independent progress** - Complete one project while another is in progress
3. **Clear boundaries** - Each project is self-contained
4. **Easy cleanup** - Delete a project's `.doctrack/` to reset it completely
5. **Version control friendly** - Each project controls whether to commit `.doctrack/`

### Switching Projects

List available sessions:
```bash
/dt-sessions
```

Switch to a different project:
```bash
/dt-use jenkins-platform-docs
```

Check which project is active:
```bash
/dt-status              # Shows current project at the top
```

## Session Lifecycle

### 1. Creation (/dt-init)

When you initialize a new session:

1. Interactive interview collects configuration
2. Creates `{project}/.doctrack/session.yaml`
3. Creates `{project}/.doctrack/STATE.yaml`
4. Registers session in `~/.local/state/doctrack/sessions.yaml`
5. Sets as current in `~/.local/state/doctrack/current`

### 2. Active Use

During documentation work:

1. Commands read `~/.local/state/doctrack/current` to find the active project
2. Load configuration from `{project}/.doctrack/session.yaml`
3. Read/update progress in `{project}/.doctrack/STATE.yaml`
4. Update `last_accessed` in global `sessions.yaml`

### 3. Switching (/dt-use)

When changing projects:

1. Validate target project exists and has `.doctrack/`
2. Update `~/.local/state/doctrack/current` with new path
3. Update `last_accessed` timestamp for the new session
4. All subsequent commands operate on the new project

### 4. Resuming Work

In a new Claude Code conversation:

1. Commands automatically read `~/.local/state/doctrack/current`
2. Continue working on the same project
3. Or explicitly switch with `/dt-use` if needed

### 5. Completion

When a project is finished:

1. Mark all chunks complete with `/dt-complete`
2. Session status updates to `complete` in `sessions.yaml`
3. Session remains registered for future reference
4. `.doctrack/` directory can be committed, archived, or deleted

## Common Scenarios

### Scenario 1: Working on One Project

```bash
# Initialize
/dt-init /path/to/my-project

# Work proceeds normally
/dt-chunk my-repo
/dt-review API-001
# ... documentation work ...

# In a new conversation later
/dt-status              # Automatically resumes my-project
```

The `current` file ensures continuity across conversations.

### Scenario 2: Multiple Concurrent Projects

```bash
# Start project A
/dt-init /path/to/project-a
/dt-chunk repo-a

# Start project B
/dt-init /path/to/project-b
/dt-chunk repo-b

# Switch between them
/dt-use project-a
/dt-status              # Shows project-a

/dt-use project-b
/dt-status              # Shows project-b
```

Each project maintains independent state in its own `.doctrack/` directory.

### Scenario 3: Resuming After Weeks

```bash
# List all sessions
/dt-sessions

# Output shows:
# 1. Jenkins Platform Docs (hub) - in_progress - /path/to/jenkins-docs
# 2. My API Server (single) - complete - /path/to/api-server
# 3. Analytics Service (single) - in_progress - /path/to/analytics

# Resume an old project
/dt-use jenkins-platform-docs
/dt-status              # Shows exactly where you left off
```

The global registry preserves all sessions.

### Scenario 4: Cleaning Up Old Sessions

```bash
# Remove a project
rm -rf /path/to/old-project/.doctrack/

# Next time you run /dt-sessions
# The stale entry is automatically removed from sessions.yaml
```

Stale sessions are automatically cleaned up.

## Best Practices

### Version Control

**Consider committing `.doctrack/` when:**
- You want team members to share the same configuration
- You want to preserve session history
- The project is stable and configuration is finalized

**Consider .gitignore for `.doctrack/` when:**
- Each team member has different configuration preferences
- You're experimenting with different settings
- Progress tracking is personal, not shared

Example `.gitignore`:
```gitignore
# Ignore all session state
.doctrack/

# But keep the configuration for the team
!.doctrack/session.yaml
```

### Organization

**Use descriptive project names:**
```yaml
# Good
name: "Jenkins Platform Documentation"
name: "MyCompany API Server Docs"

# Less helpful
name: "Docs"
name: "Project"
```

This makes `/dt-sessions` output more useful.

**Use consistent directory structures:**
```
~/code/projects/
├── project-a/
│   └── .doctrack/
├── project-b/
│   └── .doctrack/
└── documentation-hubs/
    └── platform-docs/
        └── .doctrack/
```

### Cleanup

**Periodically review sessions:**
```bash
/dt-sessions            # Review all sessions
# Remove completed or abandoned projects
rm -rf /path/to/old-project/.doctrack/
```

**Reset a project's state:**
```bash
# Keep configuration, reset progress
rm .doctrack/STATE.yaml

# Complete reset
rm -rf .doctrack/
/dt-init .              # Re-initialize
```

## Troubleshooting

### Session Not Found

**Problem:** Commands say "No active session"

**Solution:**
```bash
/dt-sessions            # List available sessions
/dt-use project-name    # Activate the session
```

Or check if `~/.local/state/doctrack/current` exists and points to a valid path.

### Wrong Project Active

**Problem:** Commands operate on the wrong project

**Solution:**
```bash
/dt-status              # Verify which project is active
/dt-use correct-project # Switch to the right one
```

### Stale Sessions

**Problem:** `/dt-sessions` shows old, deleted projects

**Solution:**
Sessions are automatically cleaned when you run `/dt-sessions`. If not:
```bash
# Manually edit the registry
nano ~/.local/state/doctrack/sessions.yaml
# Remove the stale entry
```

### Multiple Sessions for Same Project

**Problem:** Accidentally initialized twice

**Solution:**
```bash
# Keep one, remove the other
rm -rf /path/to/duplicate/.doctrack/

# Or merge them by choosing one as primary
/dt-use primary-location
```

## Summary

doctrack's session management provides:

- **XDG-compliant state** in `~/.local/state/doctrack/`
- **Global session registry** tracking all projects in `sessions.yaml`
- **Current project context** for seamless conversation continuity
- **Per-project isolation** via `.doctrack/` directories
- **Clean state separation** preventing conflicts between projects

This architecture enables efficient multi-project workflows while maintaining clear boundaries and persistent state across all your documentation work.
