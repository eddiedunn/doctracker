# Single-Repo vs Hub Mode

doctrack supports two operational modes designed to accommodate different project structures and documentation workflows. This guide explains the differences, when to use each mode, and how they affect your workflow and configuration.

## Overview

Both modes follow the same core principles and 7-step documentation process, but differ in where documentation is aggregated and how the final output is built.

| Aspect | Single-Repo Mode | Hub Mode |
|--------|------------------|----------|
| **Best for** | Individual repositories, monorepos | Multi-repo ecosystems, microservices |
| **Configuration** | `type: single` | `type: hub` |
| **Repository count** | One | Multiple |
| **Collation** | Not needed | Required (`/dt-collate`) |
| **Final output** | In repository's docs/ | In hub's output directory |
| **Complexity** | Simpler workflow | Additional aggregation steps |

## Single-Repo Mode

### Purpose

Generate documentation for a single repository or monorepo from within the repository itself.

### When to Use Single-Repo Mode

Choose single-repo mode when:

- **Working with one repository** - Your project is contained in a single repository
- **Monorepo structure** - Multiple components live in one repository
- **Co-located repositories** - Multiple repos share a parent directory and you want to document each separately
- **Documenting from within** - You're working inside the repository being documented
- **Simpler workflows** - You want the most straightforward documentation process

### Directory Structure

```
my-project/
├── .doctrack/              # doctrack session files
│   ├── session.yaml        # Configuration (type: single)
│   ├── state.yaml          # Progress tracking
│   └── my-project-tasks.yaml   # Generated grind tasks
├── src/                    # Your source code
│   ├── auth/
│   └── users/
└── docs/                   # Documentation lives here
    ├── .doctrack/          # Working files
    │   └── chunks/         # Chunk task specifications
    ├── guides/             # Generated guides
    └── examples/           # Generated examples
```

### Configuration

In your `session.yaml`:

```yaml
type: single        # Specifies single-repo mode
name: "My Project"
description: "API server for user management"

repositories:
  - name: my-project
    path: .           # Repository is current directory
    content_type: python
    priority: high
    source_directories:
      - src
    chunk_id_prefix: "API"

audiences:
  - name: Developer
    checklist:
      - "Code examples for common use cases"
      - "API reference with parameters and return values"

standards:
  docstring_style: google
  require_examples: true

output_path: docs/
```

### Workflow

The complete workflow for single-repo mode:

```bash
# 1. Initialize session
/dt-init /path/to/my-project

# 2. Inventory source files into chunks
/dt-chunk my-project

# 3. Review each chunk to identify documentation needs
/dt-review API-AUTH-001
/dt-review API-AUTH-002
/dt-review API-USERS-003

# 4. Create documentation task specifications
/dt-task API-AUTH-001
/dt-task API-AUTH-002
/dt-task API-USERS-003

# 5. Generate tasks.yaml for grind
/dt-generate my-project

# 6. Execute automated documentation generation
grind batch .doctrack/my-project-tasks.yaml

# 7. Verify and mark complete
/dt-complete API-AUTH-001
/dt-complete API-AUTH-002
/dt-complete API-USERS-003

# 8. Check final status
/dt-status
```

### Key Characteristics

- **No collation step** - Documentation stays where it's written
- **Direct output** - Generated docs go straight to `docs/`
- **Simpler state** - Track one repository's progress
- **Self-contained** - Everything in one location

## Hub Mode

### Purpose

Aggregate documentation from multiple distributed repositories into a unified documentation site from a central hub location.

### When to Use Hub Mode

Choose hub mode when:

- **Multiple distributed repositories** - Repos are in different directories or locations
- **Microservices architecture** - Each service is a separate repository
- **Multi-repo ecosystems** - Related projects that need unified documentation
- **Separate documentation repository** - You maintain docs separately from code
- **Cross-cutting documentation** - You need overview guides that span multiple repos
- **Distributed teams** - Different teams own different repositories

### Directory Structure

```
doctrack-hub/               # Central documentation hub
├── .doctrack/              # Hub session files
│   ├── session.yaml        # Configuration (type: hub)
│   ├── state.yaml          # Overall progress tracking
│   ├── collated/           # Gathered documentation
│   │   ├── api-server/     # From first repo
│   │   ├── auth-service/   # From second repo
│   │   └── user-service/   # From third repo
│   ├── output/             # Final unified documentation site
│   │   ├── index.md
│   │   ├── api-server/
│   │   ├── auth-service/
│   │   └── guides/         # Cross-cutting guides
│   └── *-tasks.yaml        # Generated grind tasks per repo

../repos/                   # Your repositories (elsewhere)
├── api-server/
│   └── docs/
│       ├── .doctrack/      # Repository working files
│       │   └── chunks/
│       ├── guides/         # Generated for this repo
│       └── examples/
├── auth-service/
│   └── docs/.doctrack/
└── user-service/
    └── docs/.doctrack/
```

### Configuration

In your `session.yaml`:

```yaml
type: hub           # Specifies hub mode
name: "Platform Docs"
description: "Unified documentation for microservices platform"

repositories:
  - name: api-server
    path: /Users/me/repos/api-server
    content_type: python
    priority: high
    source_directories:
      - src
    chunk_id_prefix: "API"

  - name: auth-service
    path: /Users/me/repos/auth-service
    content_type: python
    priority: high
    source_directories:
      - src/auth
    chunk_id_prefix: "AUTH"

  - name: user-service
    path: /Users/me/repos/user-service
    content_type: python
    priority: medium
    source_directories:
      - src
    chunk_id_prefix: "USER"

audiences:
  - name: Developer
    checklist:
      - "Code examples for common use cases"
      - "API reference with parameters and return values"
  - name: Administrator
    checklist:
      - "Deployment guides"
      - "Configuration reference"

standards:
  docstring_style: google
  require_examples: true

output_path: .doctrack/output/
```

### Workflow

The complete workflow for hub mode includes per-repository documentation followed by hub-level aggregation:

```bash
# 1. Initialize hub session
/dt-init /path/to/doctrack-hub

# 2-7. Document each repository (repeat for all repos)

# Repository 1: api-server
/dt-chunk api-server
/dt-review API-CORE-001
/dt-task API-CORE-001
/dt-generate api-server
grind batch .doctrack/api-server-tasks.yaml
/dt-complete API-CORE-001

# Repository 2: auth-service
/dt-chunk auth-service
/dt-review AUTH-OAUTH-001
/dt-task AUTH-OAUTH-001
/dt-generate auth-service
grind batch .doctrack/auth-service-tasks.yaml
/dt-complete AUTH-OAUTH-001

# Repository 3: user-service
/dt-chunk user-service
/dt-review USER-API-001
/dt-task USER-API-001
/dt-generate user-service
grind batch .doctrack/user-service-tasks.yaml
/dt-complete USER-API-001

# 8. Check all repositories are complete
/dt-status

# 9. Collate documentation from all repos
/dt-collate

# 10. Build unified documentation site
/dt-build
```

### Key Characteristics

- **Multi-repository tracking** - Progress across all repos in one session
- **Collation step** - Gather documentation from distributed repos
- **Unified output** - Single documentation site spans all repositories
- **Cross-cutting content** - Add overview guides that reference multiple repos
- **Priority ordering** - Configure which repos to document first

## Key Differences in Workflow

### Initialization

**Single-Repo Mode:**
```bash
/dt-init /path/to/my-project
# Creates .doctrack/ inside the project
```

**Hub Mode:**
```bash
/dt-init /path/to/doctrack-hub
# Creates central hub, references external repos
```

### Documentation Location

**Single-Repo Mode:**
- Docstrings: Added directly to source files in `src/`
- Guides: Written to `docs/guides/`
- Examples: Written to `docs/examples/`
- All paths relative to the repository root

**Hub Mode:**
- Docstrings: Added to source files in each repository
- Guides: Initially written to each repo's `docs/guides/`
- Examples: Initially written to each repo's `docs/examples/`
- Collation: Copied to `.doctrack/collated/{repo-name}/`
- Final output: Compiled to `.doctrack/output/`

### Completion Workflow

**Single-Repo Mode:**
```bash
# After documenting all chunks:
/dt-status
# You're done! Documentation is in docs/
```

**Hub Mode:**
```bash
# After documenting all chunks in all repos:
/dt-status          # Verify all repos complete
/dt-collate         # Gather from distributed repos
/dt-build           # Compile unified site
# Documentation is in .doctrack/output/
```

### State Tracking

**Single-Repo Mode:**
- One `STATE.yaml` tracking one repository
- Simpler progress tracking
- One set of chunks to manage

**Hub Mode:**
- One `STATE.yaml` tracking multiple repositories
- Per-repository progress and overall progress
- Multiple sets of chunks across repos
- Priority-based workflow guidance

## Configuration Differences

### Minimal Single-Repo Configuration

```yaml
type: single
name: "My Project"
repositories:
  - name: my-project
    path: .
    content_type: python
    source_directories:
      - src
audiences:
  - name: Developer
standards:
  docstring_style: google
output_path: docs/
```

### Minimal Hub Configuration

```yaml
type: hub
name: "Platform Docs"
repositories:
  - name: repo-a
    path: /path/to/repo-a
    content_type: python
    priority: high

  - name: repo-b
    path: /path/to/repo-b
    content_type: python
    priority: medium

audiences:
  - name: Developer
standards:
  docstring_style: google
output_path: .doctrack/output/
```

**Key difference:** Hub mode requires absolute paths for repositories since they're external to the hub directory.

## Choosing Between Modes

### Use Single-Repo Mode if:

✅ You have one repository to document
✅ Your project is a monorepo with all code in one place
✅ You're documenting from within the repository
✅ You want the simplest possible workflow
✅ Documentation consumers work with individual repos

### Use Hub Mode if:

✅ You have multiple repositories in different locations
✅ You're building a unified documentation site
✅ You need cross-cutting guides that span repos
✅ You maintain a separate documentation repository
✅ You want to prioritize which repos to document first
✅ Different teams own different repositories

## Switching Between Modes

You cannot switch modes after initialization. If you need to change modes:

1. Complete or abandon the current session
2. Run `/dt-init` again with a new configuration
3. Choose the appropriate mode during the initialization interview

## Common Patterns

### Pattern 1: Monorepo with Single Mode

```
my-monorepo/
├── .doctrack/          # Session in repo root
├── services/
│   ├── api/
│   ├── auth/
│   └── users/
└── docs/               # All docs here
```

Document each service as separate chunks within one session.

### Pattern 2: Microservices with Hub Mode

```
documentation-hub/      # Hub for all services
├── .doctrack/

../services/           # Services elsewhere
├── api-service/
├── auth-service/
└── user-service/
```

Document each service individually, then aggregate into unified site.

### Pattern 3: Multiple Single-Mode Sessions

```
projects/
├── project-a/
│   └── .doctrack/     # Separate session
└── project-b/
    └── .doctrack/     # Separate session
```

Use `/dt-sessions` and `/dt-use` to switch between independent projects.

## Summary

Both operational modes follow the same core documentation process but differ in scope and output:

- **Single-Repo Mode** is simpler, self-contained, and ideal for individual projects
- **Hub Mode** is more powerful for multi-repository ecosystems requiring unified documentation

Choose based on your project structure, team organization, and documentation goals. The mode you select affects configuration, workflow steps, and where your final documentation lives.
