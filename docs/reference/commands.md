# Command Reference

Complete reference for all doctrack slash commands available in Claude Code.

## Overview

doctrack provides 12 slash commands that guide you through the documentation workflow. Commands fall into three categories:

- **Session Management:** Initialize, switch, and view sessions
- **Documentation Workflow:** Create, review, and complete documentation
- **Publishing:** Collate and build final documentation

## Session Management Commands

### dt-init

Initialize a new doctrack session through an interactive interview.

**Usage:**
```
/dt-init [project-path]
```

**Arguments:**
- `[project-path]` (optional): Path to the project directory. If not provided, uses the current directory.

**Description:**

Runs an interactive 5-phase interview to set up a complete doctrack session configuration. This is the first command you'll run for any new project.

**Interview Phases:**

1. **Project Scope** - Define project type (single/hub), name, and description
2. **Repositories** - Configure repository paths, content types, and chunk ID prefixes
3. **Audiences** - Select target audiences and customize their documentation checklists
4. **Standards** - Set docstring style and documentation requirements
5. **Output** - Specify output path for generated documentation

**What Gets Created:**

- `.doctrack/session.yaml` - Session configuration file
- `.doctrack/state.yaml` - Progress tracking state
- `~/.local/state/doctrack/sessions.yaml` - Global sessions registry
- `~/.local/state/doctrack/current` - Current session marker

**Examples:**

Initialize in current directory:
```
/dt-init
```

Initialize specific project:
```
/dt-init /path/to/my-project
```

**Related Commands:** [dt-use](#dt-use), [dt-sessions](#dt-sessions), [dt-status](#dt-status)

---

### dt-use

Switch the current doctrack session context.

**Usage:**
```
/dt-use <project-path>
```

**Arguments:**
- `<project-path>` (required): Path to an existing doctrack project

**Description:**

Sets the active doctrack session by updating the current session pointer. Use this to switch between multiple doctrack projects or when starting a new Claude Code conversation.

**Process:**

1. Validates the provided path exists
2. Verifies `.doctrack/session.yaml` exists at that path
3. Updates `~/.local/state/doctrack/current` with the absolute path
4. Updates `last_accessed` timestamp in sessions registry
5. Displays project name and type confirmation

**Examples:**

Switch to a project:
```
/dt-use /Users/me/projects/my-api
```

Switch using relative path:
```
/dt-use ../jenkins-platform-docs
```

**Output:**
```
CONTEXT SWITCHED
================

Project: My API Documentation
Type: single
Path: /Users/me/projects/my-api

All subsequent commands will operate on this project.
```

**Related Commands:** [dt-sessions](#dt-sessions), [dt-init](#dt-init), [dt-status](#dt-status)

---

### dt-sessions

List all known doctrack sessions from the global registry.

**Usage:**
```
/dt-sessions
```

**Arguments:** None

**Description:**

Displays all registered doctrack sessions with their current status. Automatically cleans up stale entries (sessions whose paths no longer exist). Shows which session is currently active with a `*` marker.

**What It Shows:**

- Session name and type (single/hub)
- Current status (init, in_progress, complete)
- Project path
- Which session is currently active

**Examples:**

```
/dt-sessions
```

**Output:**
```
DOCTRACK SESSIONS
=================

CURRENT  NAME                    TYPE    STATUS       PATH
-------  ----                    ----    ------       ----
*        Jenkins Platform Docs   hub     in_progress  /path/to/jenkins-platform-docs
         My API                  single  complete     /path/to/my-api
         Widget Service          single  init         /path/to/widget-service

Total: 3 sessions

Use /dt-use <path> to switch sessions
```

**Related Commands:** [dt-use](#dt-use), [dt-init](#dt-init), [dt-status](#dt-status)

---

### dt-status

Display documentation tracking progress dashboard.

**Usage:**
```
/dt-status [project-path]
```

**Arguments:**
- `[project-path]` (optional): Path to the doctrack project. If not provided, uses the current session.

**Description:**

Shows a comprehensive dashboard of your documentation progress, including repository status, chunk completion, and recommended next actions. This is your go-to command for understanding where you are in the workflow.

**What It Shows:**

- Project name, mode (single/hub), and current phase
- Repository status summary (pending, in progress, complete)
- Current focus repository and chunk
- Detailed repository progress with chunk counts
- Collation status (for hub mode)
- Recommended next action

**Examples:**

Check current project status:
```
/dt-status
```

Check specific project:
```
/dt-status /path/to/my-project
```

**Output:**
```
DOCTRACK STATUS DASHBOARD
=========================

PROJECT: /Users/me/projects/my-api
MODE: single
PHASE: documentation

PROJECT OVERVIEW
----------------
Total Repositories: 1
- Pending:     0
- In Progress: 1
- Complete:    0

CURRENT FOCUS
-------------
Repo: my-api
Step: 2: Review chunks
Progress: 3/12 chunks reviewed

REPOSITORY STATUS
-----------------
REPO NAME    STATUS         STEP    PROGRESS
---------    ------         ----    --------
my-api       in_progress    2       3/12 chunks

RECOMMENDED NEXT ACTION
-----------------------
Continue reviewing chunks:
    /dt-review API-AUTH-004

Last Updated: 2024-12-07T14:30:00Z
```

**Related Commands:** [dt-next](#dt-next), [dt-sessions](#dt-sessions)

---

### dt-next

Determine and display the next action to take in the documentation workflow.

**Usage:**
```
/dt-next [project-path]
```

**Arguments:**
- `[project-path]` (optional): Path to the project. If not provided, uses the current session.

**Description:**

Analyzes your current progress and recommends the specific next command to run. Uses a decision tree to determine the optimal next step based on repository status, chunk progress, and phase.

**Decision Logic:**

1. No repo in progress? → Start chunking highest-priority pending repo
2. Step 1 done? → Review first unreviewed chunk
3. Chunk reviewed? → Create tasks for that chunk
4. All chunks tasked? → Generate tasks file for grind
5. Tasks executed? → Complete the chunk
6. All chunks complete? → Move to next repo or collate (hub mode)
7. All repos complete? → Collate (hub mode) or build
8. Collation complete? → Build final site

**Examples:**

```
/dt-next
```

**Output:**
```
DOCTRACK NAVIGATION
===================

PROJECT
-------
Name: My API Documentation
Type: single
Path: /Users/me/projects/my-api

CURRENT STATE
-------------
Phase: documentation
In Progress: my-api
Current Step: Review chunks
Current Chunk: N/A

REPOSITORY STATUS
-----------------
  my-api: in_progress (3/12 chunks reviewed)

DECISION ANALYSIS
-----------------
Repository my-api has 3 chunks reviewed but not tasked.
Next step is to review the next unreviewed chunk.

NEXT ACTION
-----------
Run this command:

    /dt-review API-AUTH-004

Status: Chunk API-AUTH-004 is pending review
```

**Related Commands:** [dt-status](#dt-status), [dt-chunk](#dt-chunk), [dt-review](#dt-review)

---

## Documentation Workflow Commands

### dt-chunk

Inventory source files and create chunk list for a repository.

**Usage:**
```
/dt-chunk [repo-path]
```

**Arguments:**
- `[repo-path]` (optional): Path to the repository. If not provided, uses the repository from session config.

**Description:**

Step 1 of the documentation workflow. Scans all source files in configured directories and groups them into logical chunks for systematic documentation. Each chunk contains ~10 files grouped by functional area.

**Process:**

1. Reads repository configuration for source paths and content type
2. Scans source files based on content type (Python, Ansible, JSON Schema, etc.)
3. Groups files by functional area
4. Assigns chunk IDs using configured prefix (e.g., API-AUTH-001)
5. Creates state files and human-readable tracking document

**What Gets Created:**

- `.doctrack/repos/{repo}.yaml` - Chunk state file (source of truth)
- `.doctrack/state.yaml` - Updated with chunk count
- `CHUNK_TRACKING.md` - Human-readable chunk reference in repo

**Content Types Supported:**

- **python**: Scans for `*.py` files
- **ansible**: Scans for `main.yml` in roles/tasks/
- **json-schema**: Scans for `*.json` files
- **yaml-config**: Scans for `*.yaml` and `*.yml` files
- **mixed**: Combines relevant patterns

**Examples:**

Chunk current repo:
```
/dt-chunk
```

Chunk specific repo:
```
/dt-chunk my-api
```

**Output:**

Creates chunks with IDs like:
- `API-AUTH-001` - Authentication module (8 files)
- `API-USERS-002` - User management (10 files)
- `API-DATA-003` - Data models (7 files)

**Related Commands:** [dt-review](#dt-review), [dt-status](#dt-status), [dt-next](#dt-next)

---

### dt-review

Review a chunk's source code and identify documentation needs.

**Usage:**
```
/dt-review <chunk-id>
```

**Arguments:**
- `<chunk-id>` (required): The chunk identifier to review (e.g., API-AUTH-001)

**Description:**

Step 2 of the documentation workflow. Reads all source files in a chunk to understand what the code does and identifies what documentation needs to be written. This is NOT an audit - you're reading source code to determine documentation needs, not comparing against existing docs.

**Core Principle:** Source code is truth. Ignore existing documentation entirely. Document what the code DOES, not what old docs SAY it does.

**Process:**

1. Loads chunk details from `.doctrack/repos/{repo}.yaml`
2. Reads ALL source files in the chunk completely
3. Identifies public APIs, functions, classes, parameters, return types
4. Determines what needs docstrings, guides, examples, diagrams
5. Applies audience checklist requirements
6. Updates chunk status to "reviewed" with documentation needs

**What It Identifies:**

- Missing or inadequate docstrings
- Need for narrative documentation (guides, tutorials)
- Required code examples
- Diagrams needed for understanding
- Audience-specific requirements

**Examples:**

Review a chunk:
```
/dt-review API-AUTH-001
```

**Output:**
```
## Review Results: API-AUTH-001

**Repository:** my-api
**Source:** src/auth/*.py (8 files)
**Review Date:** 2024-12-07

### What This Code Does

Authentication module handling JWT tokens, OAuth flows, and session management.
Provides decorators for route protection and user identity verification.

### Documentation Needed

#### Docstrings
- [ ] authenticate() - needs complete docstring with examples
- [ ] JWTHandler class - needs class docstring
- [ ] verify_token() - needs params and return documentation

#### Narrative Documentation
- [ ] Overview of authentication flow
- [ ] How-to guide for implementing OAuth
- [ ] Session management guide

#### Examples
- [ ] Basic JWT authentication example
- [ ] OAuth integration example

### Audience Coverage

**Admin:** Deployment and configuration requirements
**Developer:** API usage and integration patterns

### State Updated

✓ Updated .doctrack/repos/my-api.yaml - chunk marked as "reviewed"
✓ Updated .doctrack/state.yaml - progress incremented

### Next Steps

Use `/dt-task API-AUTH-001` to create documentation tasks.
```

**Related Commands:** [dt-task](#dt-task), [dt-chunk](#dt-chunk), [dt-complete](#dt-complete)

---

### dt-task

Create documentation tasks for a chunk.

**Usage:**
```
/dt-task <chunk-id>
```

**Arguments:**
- `<chunk-id>` (required): The chunk identifier to create tasks for (e.g., API-AUTH-001)

**Description:**

Step 3 of the documentation workflow. Transforms review findings into specific, actionable documentation tasks with verification steps. Tasks are written in grind-compatible YAML format.

**Process:**

1. Loads chunk details and review results
2. Creates detailed task specifications for each documentation need
3. Includes verification commands or manual verification steps
4. Generates grind-compatible YAML
5. Updates chunk status to "tasked"

**Task Requirements:**

Every task MUST have:
- Clear, specific description of what to document
- Exact file path for output
- Format specification (docstring, markdown, example)
- Verification method (automatic command or manual steps)
- Model selection and iteration limit

**Verification Methods:**

1. **Automatic (Preferred):**
   ```yaml
   verify: |
     test -f docs/auth.md && \
     grep -q "JWT authentication" docs/auth.md
   ```

2. **Manual:**
   ```yaml
   verification: manual
   verification_notes: |
     1. Check authentication flow is documented
     2. Verify examples work
     3. Confirm audience-appropriate language
   ```

**Examples:**

Create tasks for a chunk:
```
/dt-task API-AUTH-001
```

**Output:**

Saves to `.doctrack/tasks/API-AUTH-001-tasks.yaml`:
```yaml
# Documentation Tasks: API-AUTH-001
# Run with: grind batch .doctrack/tasks/API-AUTH-001-tasks.yaml

tasks:
  - id: "doc-API-AUTH-001-001"
    task: |
      Create docstring for authenticate() function.

      Source: src/auth/core.py

      Requirements:
      - NumPy style docstring
      - Document all parameters with types
      - Document return value
      - Include usage example
      - Document exceptions raised

      Standards:
      - Docstrings: NumPy format
      - Examples: Must be tested/testable

      DO NOT reference existing docs. Write from source code.
    verify: |
      grep -A 20 "def authenticate" src/auth/core.py | grep -q "Parameters"
    model: sonnet
    max_iterations: 3

  - id: "doc-API-AUTH-001-002"
    task: |
      Create authentication flow guide for Developer audience.

      Source files: src/auth/core.py, src/auth/jwt.py

      Requirements:
      - Overview of authentication architecture
      - Step-by-step JWT authentication guide
      - OAuth integration instructions
      - Code examples for common use cases

      Location: docs/guides/authentication.md
    verify: |
      test -f docs/guides/authentication.md && \
      grep -q "JWT" docs/guides/authentication.md
    model: sonnet
    max_iterations: 3
```

**Related Commands:** [dt-review](#dt-review), [dt-generate](#dt-generate), [dt-complete](#dt-complete)

---

### dt-generate

Compile all chunk tasks into a single tasks.yaml for grind.

**Usage:**
```
/dt-generate
```

**Arguments:** None (operates on current project)

**Description:**

Step 4 of the documentation workflow. Compiles all task files from chunked repositories into a single grind-ready YAML file. This prepares tasks for batch execution.

**Process:**

1. Reads current project configuration
2. Finds all `*-tasks.yaml` files in `.doctrack/tasks/`
3. Combines all tasks into a single file
4. Orders tasks by dependencies if specified
5. Saves to `.doctrack/tasks/{project-name}-tasks.yaml`
6. Updates state to mark compilation complete

**Examples:**

Generate tasks file:
```
/dt-generate
```

**Output:**
```
TASKS COMPILED
==============

Project: My API Documentation
Task files found: 12
Tasks total: 47
Output: .doctrack/tasks/my-api-tasks.yaml

State updated: .doctrack/state.yaml
Phase: tasks-generated

Next: grind batch .doctrack/tasks/my-api-tasks.yaml
```

**Generated File Structure:**
```yaml
# Generated by doctrack /dt-generate
# Project: My API Documentation
# Generated: 2024-12-07T14:45:00Z

tasks:
  - id: "doc-API-AUTH-001-001"
    task: |
      Create docstring for authenticate() function.
      ...
    verify: |
      grep -A 20 "def authenticate" src/auth/core.py | grep -q "Parameters"
    model: sonnet
    max_iterations: 3

  # ... 46 more tasks
```

**Related Commands:** [dt-task](#dt-task), [dt-complete](#dt-complete)

---

### dt-complete

Verify documentation tasks and mark chunk as complete.

**Usage:**
```
/dt-complete <chunk-id>
```

**Arguments:**
- `<chunk-id>` (required): The chunk identifier to verify (e.g., API-AUTH-001)

**Description:**

Step 5 of the documentation workflow. Verifies that documentation was created correctly by running verification commands or prompting for manual confirmation. Marks chunks as complete, partial, or blocked based on results.

**Process:**

1. Loads chunk and task details
2. For each task, runs verification:
   - **Automatic:** Executes verify command, checks exit code
   - **Manual:** Prompts user for confirmation
3. Determines completion status based on results
4. Updates chunk status in state files
5. If all chunks in repo complete, marks repo as complete

**Completion Status:**

- **COMPLETE:** All tasks verified (automatic pass or user confirmed)
- **PARTIAL:** Some tasks failed verification
- **BLOCKED:** Unable to verify or all tasks failed

**Examples:**

Complete a chunk:
```
/dt-complete API-AUTH-001
```

**Output:**
```
CHUNK VERIFICATION REPORT
=========================

Chunk: API-AUTH-001
Repository: my-api
Verification Date: 2024-12-07T15:00:00Z

TASK VERIFICATION RESULTS
-------------------------

Task: doc-API-AUTH-001-001
Description: Create docstring for authenticate() function
Type: Automatic
Status: ✓ PASS

Task: doc-API-AUTH-001-002
Description: Authentication flow guide
Type: Automatic
Status: ✓ PASS

SUMMARY
-------
Total Tasks: 2
Passed: 2
Failed: 0
Manual: 0

CHUNK STATUS: COMPLETE

✓ All verification checks passed
✓ Chunk marked complete in .doctrack/repos/my-api.yaml

NEXT STEPS
----------
Continue with: /dt-next
```

**Manual Verification Example:**
```
MANUAL VERIFICATION REQUIRED
============================

Task: doc-API-AUTH-001-003
Description: Advanced OAuth integration guide

Verification Steps:
1. Check that OAuth flow is documented in docs/oauth.md
2. Verify examples work and are tested
3. Confirm language is appropriate for Developer audience

Has this task been completed successfully? (yes/no): _
```

**Related Commands:** [dt-task](#dt-task), [dt-review](#dt-review), [dt-next](#dt-next)

---

## Publishing Commands

### dt-collate

Hub aggregation command - collates documentation from all repositories into unified site.

**Usage:**
```
/dt-collate
```

**Arguments:** None (hub mode only)

**Description:**

Hub mode only. After all repositories are complete, aggregates documentation from multiple repositories into a unified MkDocs site with cross-cutting content and audience journeys.

**Prerequisites:**

- Session type must be "hub"
- All repositories must have status "complete"
- Phase must be "documentation"

**Process:**

1. Verifies all repositories are complete
2. Runs interactive collation interview (navigation structure, cross-cutting sections)
3. Detects and resolves naming conflicts across repos
4. Syncs documentation from all repos
5. Generates cross-cutting content tasks (architecture, glossary, security, operations)
6. Builds audience journey index pages
7. Generates or updates mkdocs.yml with navigation structure

**Collation Interview Questions:**

1. **Navigation structure:**
   - Audience journeys (recommended)
   - Component-first
   - Hybrid

2. **Cross-cutting sections:**
   - Architecture overview
   - Glossary
   - Security model
   - Operations guides
   - Getting started guide

3. **Sync documentation now?** (default: yes)

**Examples:**

```
/dt-collate
```

**Output:**
```
DOCTRACK COLLATION
==================

All source repositories are complete. Ready to aggregate into unified site.

REPOSITORY STATUS
-----------------
platform-ctl:        COMPLETE (12/12 chunks)
platform-automation: COMPLETE (8/8 chunks)
app-deploy-state:    COMPLETE (5/5 chunks)

COLLATION IN PROGRESS
---------------------
[1/4] Syncing repository documentation...
      ✓ platform-ctl/docs -> docs/platform-ctl
      ✓ platform-automation/docs -> docs/platform-automation
      ✓ app-deploy-state/docs -> docs/app-deploy-state

[2/4] Generating cross-cutting content tasks...
      ✓ Created .doctrack/tasks/collate-cross-cutting.yaml
      ✓ 4 tasks generated for: architecture, glossary, security, operations

[3/4] Building audience journeys...
      ✓ docs/journeys/admin/index.md
      ✓ docs/journeys/app-dev/index.md
      ✓ docs/journeys/platform-dev/index.md

[4/4] Generating mkdocs.yml...
      ✓ Navigation structure: audience-journeys
      ✓ Theme: material
      ✓ Plugins configured

COLLATION COMPLETE
==================

Files created:
  - docs/concepts/ (cross-cutting content tasks generated)
  - docs/journeys/admin/index.md
  - docs/journeys/app-dev/index.md
  - docs/journeys/platform-dev/index.md
  - mkdocs.yml

State updated:
  - Phase: documentation -> collation
  - Collation status: complete

NEXT STEPS
----------
1. Run cross-cutting content tasks:
   grind batch .doctrack/tasks/collate-cross-cutting.yaml

2. Then build the site:
   /dt-build
```

**Error Conditions:**

- **Not a hub session:** Command only available for hub mode
- **Repositories not complete:** All repos must be complete before collation
- **Wrong phase:** Collation already performed or not ready

**Related Commands:** [dt-build](#dt-build), [dt-status](#dt-status), [dt-complete](#dt-complete)

---

### dt-build

Generate the MkDocs site from documentation.

**Usage:**
```
/dt-build
```

**Arguments:** None (operates on current project)

**Description:**

Final step: Builds the complete MkDocs documentation site from all generated documentation. Validates or generates mkdocs.yml configuration and runs mkdocs build.

**Process:**

1. Loads project configuration from `.doctrack/session.yaml`
2. Checks if `mkdocs.yml` exists:
   - If not: Generates based on session config (hub vs single mode)
   - If exists: Validates required sections
3. Runs `mkdocs build`
4. Reports build status with page count and build time
5. Updates `.doctrack/state.yaml` with build timestamp

**Generated mkdocs.yml Features:**

- Material theme with light/dark mode toggle
- Search plugin with suggestions and highlighting
- Navigation tabs and sections (hub mode)
- Mermaid diagram support
- Code highlighting and copy buttons
- Markdown extensions (tables, admonitions, details)

**Navigation Structures:**

**Hub Mode (Audience Journeys):**
```yaml
nav:
  - Home: index.md
  - Journeys:
    - Administrator: journeys/admin/index.md
    - Application Developer: journeys/app-dev/index.md
    - Platform Developer: journeys/platform-dev/index.md
  - Components:
    - Platform CTL: platform-ctl/index.md
    - Platform Automation: platform-automation/index.md
  - Concepts:
    - Architecture: concepts/architecture-overview.md
    - Glossary: concepts/glossary.md
```

**Single Mode:**
```yaml
nav:
  - Home: index.md
  - Getting Started: getting-started.md
  - API Reference: api/index.md
  - User Guide: guide/index.md
  - Examples: examples/index.md
  - Contributing: contributing.md
```

**Examples:**

Build the site:
```
/dt-build
```

**Output:**
```
BUILD COMPLETE
==============

Site generated at: ./site
Pages: 47
Build time: 2.3s

To preview: mkdocs serve
To publish: mkdocs gh-deploy (if configured)

NEXT STEPS
----------
1. Review site: open site/index.html
2. Test navigation and search
3. Deploy to hosting platform
```

**Error Handling:**

If build fails:
```
BUILD FAILED
============

Error output:
Config file 'mkdocs.yml' is invalid.
Error: The "nav" provided in the MkDocs configuration
is invalid. Expected a list, got a dict.

TROUBLESHOOTING
---------------
1. Check mkdocs.yml syntax
2. Verify all referenced files exist in docs/
3. Ensure MkDocs and plugins are installed:
   pip install mkdocs mkdocs-material pymdown-extensions

Run with verbose: mkdocs build --verbose
```

**Prerequisites:**

MkDocs and dependencies must be installed:
```bash
pip install mkdocs mkdocs-material pymdown-extensions
```

**Related Commands:** [dt-collate](#dt-collate), [dt-status](#dt-status)

---

## Command Quick Reference

### By Workflow Phase

**Session Setup:**
```
/dt-init              # Initialize new session
/dt-use <path>        # Switch sessions
/dt-sessions          # List all sessions
```

**Documentation Creation:**
```
/dt-status            # Check progress
/dt-next              # Get next action
/dt-chunk [repo]      # Create chunks
/dt-review <chunk>    # Review chunk
/dt-task <chunk>      # Create tasks
/dt-generate          # Compile tasks
# Run: grind batch .doctrack/tasks/{project}-tasks.yaml
/dt-complete <chunk>  # Verify and complete
```

**Publishing (Hub Mode):**
```
/dt-collate           # Aggregate documentation
/dt-build             # Build MkDocs site
```

### By Use Case

**Starting a new project:**
```
/dt-init /path/to/project
/dt-chunk
/dt-review {first-chunk}
```

**Resuming work:**
```
/dt-sessions
/dt-use /path/to/project
/dt-status
/dt-next
```

**Checking progress:**
```
/dt-status
/dt-next
```

**Completing documentation:**
```
/dt-complete {chunk}
/dt-status
```

**Publishing (hub mode):**
```
/dt-collate
# Run: grind batch .doctrack/tasks/collate-cross-cutting.yaml
/dt-build
```

## Common Patterns

### First Time Setup
```
/dt-init
# Answer interview questions
/dt-status
/dt-chunk
```

### Normal Workflow Loop
```
/dt-review <chunk-id>
/dt-task <chunk-id>
# Repeat for all chunks, then:
/dt-generate
# Run grind batch command
/dt-complete <chunk-id>
# Repeat complete for all chunks
```

### Resuming After Break
```
/dt-sessions
/dt-use <project-path>
/dt-status
/dt-next
```

### Multi-Repository Hub
```
# Document each repo
/dt-chunk repo-1
/dt-chunk repo-2
# Complete all repos...
/dt-collate
# Run grind for cross-cutting tasks
/dt-build
```

## Tips

- Use `/dt-next` when unsure what to do next
- Use `/dt-status` to see overall progress
- Complete chunks in order shown in `CHUNK_TRACKING.md`
- Review before tasking - ensures tasks are specific
- Verify tasks before marking complete - ensures quality
- Hub mode requires all repos complete before collation
- Session context persists via `~/.local/state/doctrack/current`

## See Also

- [Getting Started Guide](../getting-started.md) - Step-by-step tutorial
- [Core Principles](../../PROCESS.md) - Understanding source-as-truth
- [Workflow Examples](../../PROCESS.md#workflow-examples) - Real-world scenarios
- [Configuration](../../session.yaml.example) - Session configuration reference
