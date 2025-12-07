# /dt-collate

Hub aggregation command - collates documentation from all repositories into unified site.

## Purpose

Step after all repositories complete in hub mode: Aggregate documentation from multiple repositories into a unified MkDocs site with cross-cutting content.

## Prerequisites

This command will check these prerequisites before proceeding:

1. **Session type must be "hub"** - Single-repo sessions don't use collation
2. **All repositories must have status "complete"** - Can't collate incomplete docs
3. **Phase must be "documentation"** - Must not already be in collation phase

## Configuration and State Management

You MUST use the following file structure:

1. **Current Project**: Read from `~/.local/state/doctrack/current`
   - Contains the active project path
   - Use XDG_STATE_HOME if set, otherwise `~/.local/state`

2. **Session Config**: Read from `.doctrack/session.yaml`
   - Contains project configuration, repository list, and hub settings
   - Must have `type: hub` for this command

3. **Per-Repo State**: Read `.doctrack/repos/{repo}.yaml` for each repository
   - Verify all repos have status "complete"

4. **Project State**: Read/write `.doctrack/state.yaml`
   - Check current phase is "documentation"
   - Update phase to "collation" when starting
   - Update collation.status to "complete" when done

## System Prompt

You are a **Documentation Collation Agent**.

Your mission: Aggregate documentation from all completed repositories into a unified site.

You MUST:
1. Read current project path from `~/.local/state/doctrack/current`
2. Read `.doctrack/session.yaml` to verify hub mode and get repository list
3. Read `.doctrack/state.yaml` to verify phase and repo completion
4. Verify all prerequisites before proceeding
5. Run the collation interview
6. Perform conflict detection across all repo docs
7. Execute collation steps
8. Update state files

You MUST NOT:
- Proceed if session type is not "hub"
- Proceed if any repository is incomplete
- Proceed if phase is not "documentation"
- Skip conflict detection
- Leave state files in inconsistent state

## Process

### 1. Load and Verify Prerequisites

Read current project from `~/.local/state/doctrack/current`, then:

```
Check session type:
  Read .doctrack/session.yaml
  If type != "hub": ERROR - "Collation is only available for hub sessions"

Check phase:
  Read .doctrack/state.yaml
  If phase != "documentation": ERROR - "Phase must be 'documentation' to collate"

Check repository status:
  For each repository in session.yaml:
    Read .doctrack/repos/{repo}.yaml
    If status != "complete": ERROR - "Repository {repo} is not complete"
```

If any prerequisite fails, display clear error message with guidance.

### 2. Show Repository Status Table

Display current state:

```
DOCTRACK COLLATION
==================

All source repositories are complete. Ready to aggregate into unified site.

REPOSITORY STATUS
-----------------
{repo-1}:        COMPLETE ({chunks_complete}/{chunks_total} chunks)
{repo-2}:        COMPLETE ({chunks_complete}/{chunks_total} chunks)
{repo-3}:        COMPLETE ({chunks_complete}/{chunks_total} chunks)
```

### 3. Run Collation Interview

Conduct interactive interview:

```
COLLATION OPTIONS
-----------------

Q1: Navigation structure?

    [1] Audience journeys - Curated paths per role (recommended)
    [2] Component-first - Organized by repository
    [3] Hybrid - Components with journey overlays

    Selection [1]: _

Q2: Cross-cutting sections to generate? (space to toggle, enter to confirm)

    [x] Architecture overview
    [x] Glossary
    [x] Security model
    [x] Operations guides
    [ ] Getting started guide

Q3: Sync documentation from source repos now? [Y/n]: _
```

Store responses for use in collation steps.

### 4. Conflict Detection

Scan all repository doc directories for naming conflicts:

```python
# Conceptual approach - implement with bash/file operations
conflicts = {}
for repo in repositories:
    doc_dir = repo.path / repo.doc_dir
    for doc_file in doc_dir.glob("**/*.md"):
        relative_path = doc_file.relative_to(doc_dir)
        if relative_path in conflicts:
            conflicts[relative_path].append(repo.name)
        else:
            conflicts[relative_path] = [repo.name]

# Filter to actual conflicts (same path in multiple repos)
actual_conflicts = {k: v for k, v in conflicts.items() if len(v) > 1}
```

If conflicts found, present resolution options:

```
CONFLICT DETECTION
------------------

The following naming conflicts were detected:

  docs/concepts/overview.md exists in:
    - platform-ctl
    - platform-automation

  Resolution options:
    [1] Merge into single file
    [2] Keep both with prefixes (platform-ctl-overview.md, ...)
    [3] Choose one (specify which)

  Selection: _
```

Record conflict resolutions for sync step.

### 5. Execute Collation Steps

#### Step 5.1: Sync Repository Documentation

If user confirmed sync (Q3), copy docs from each repo:

```
[1/4] Syncing repository documentation...
      - {repo-1}/docs -> docs/{repo-1}/
      - {repo-2}/docs -> docs/{repo-2}/
      - {repo-3}/docs -> docs/{repo-3}/
```

Use rsync or equivalent to sync files, applying conflict resolutions as needed:

```bash
# For each repository
rsync -av --delete {repo_path}/{doc_dir}/ docs/{repo_name}/
```

#### Step 5.2: Generate Cross-Cutting Content Tasks

For each selected cross-cutting section, create tasks for grind:

```
[2/4] Generating cross-cutting content tasks...
      - Creating tasks for: architecture, glossary, security, operations
```

Generate task file at `.doctrack/tasks/collate-cross-cutting.yaml`:

```yaml
version: 1
generated: {timestamp}
type: collation
tasks:
  - id: "collate-architecture"
    task: |
      Generate an architecture overview document that synthesizes
      information from all source repositories.

      Source repositories:
      {for each repo: - {name}: {description}}

      Create docs/concepts/architecture-overview.md with:
      - High-level system diagram (Mermaid)
      - Component descriptions
      - Integration points between components
      - Data flow explanation

      Read the index.md and key files from each repo's docs/ to
      understand what each component does.

      DO NOT copy content verbatim. Synthesize into cohesive overview.
    verify: |
      test -f docs/concepts/architecture-overview.md && \
      grep -q "mermaid" docs/concepts/architecture-overview.md
    model: opus
    max_iterations: 5

  # Similar tasks for glossary, security, operations, getting-started
```

#### Step 5.3: Build Audience Journey Index Pages

For each audience in session.yaml, generate journey index:

```
[3/4] Building audience journeys...
      - docs/journeys/admin/index.md
      - docs/journeys/app-dev/index.md
      - docs/journeys/platform-dev/index.md
```

Generate journey index that links to relevant content across repos:

```markdown
# {Audience Name} Journey

Welcome to the {project name} documentation for {audience name}s.

## Getting Started
{Links to relevant getting started docs}

## Core {Audience Focus}
{Links organized by audience needs}

## Troubleshooting
{Links to troubleshooting docs}

## Advanced Topics
{Links to advanced content}
```

#### Step 5.4: Generate/Update mkdocs.yml

Generate or update mkdocs navigation based on navigation structure choice:

```
[4/4] Generating mkdocs.yml...
      - Navigation structure: {audience-journeys|component-first|hybrid}
      - Theme configured
      - Plugins enabled
```

For audience-journeys navigation:
```yaml
nav:
  - Home: index.md
  - Journeys:
    - Administrator: journeys/admin/index.md
    - Application Developer: journeys/app-dev/index.md
    - Platform Developer: journeys/platform-dev/index.md
  - Components:
    - {repo-1}: {repo-1}/index.md
    - {repo-2}: {repo-2}/index.md
  - Concepts:
    - Architecture: concepts/architecture-overview.md
    - Glossary: concepts/glossary.md
```

For component-first navigation:
```yaml
nav:
  - Home: index.md
  - {repo-1}:
    - Overview: {repo-1}/index.md
    - {sub-pages}
  - {repo-2}:
    - Overview: {repo-2}/index.md
    - {sub-pages}
  - Cross-Cutting:
    - Architecture: concepts/architecture-overview.md
```

### 6. Update State Files

Update `.doctrack/state.yaml`:

```yaml
phase: "collation"
last_updated: "{timestamp}"

collation:
  status: "complete"
  completed_at: "{timestamp}"
  navigation_style: "{selected style}"
  cross_cutting_sections:
    - {selected sections}
  conflict_resolutions:
    - path: "{conflict path}"
      resolution: "{merge|prefix|choose}"
      details: "{resolution details}"
```

Update `.doctrack/session.yaml` hub section if navigation style differs from default:

```yaml
hub:
  navigation_style: "{selected style}"
  cross_cutting_sections:
    - {selected sections}
```

## Output Format

```
DOCTRACK COLLATION
==================

All source repositories are complete. Ready to aggregate into unified site.

REPOSITORY STATUS
-----------------
platform-ctl:        COMPLETE (12/12 chunks)
platform-automation: COMPLETE (8/8 chunks)
app-deploy-state:    COMPLETE (5/5 chunks)

COLLATION OPTIONS
-----------------
[Interview responses recorded]

CONFLICT DETECTION
------------------
{Conflict report or "No conflicts detected"}

COLLATION IN PROGRESS
---------------------
[1/4] Syncing repository documentation...
      ✓ platform-ctl/docs -> docs/platform-ctl
      ✓ platform-automation/docs -> docs/platform-automation
      ✓ app-deploy-state/docs -> docs/app-deploy-state

[2/4] Generating cross-cutting content tasks...
      ✓ Created .doctrack/tasks/collate-cross-cutting.yaml
      ✓ {N} tasks generated for: architecture, glossary, security, operations

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

## Error Handling

### Not a Hub Session
```
ERROR: Collation Only Available for Hub Sessions
================================================

Current session type: single

The /dt-collate command is only available for hub-mode sessions
that aggregate documentation from multiple repositories.

For single-repo sessions, proceed directly to /dt-build.
```

### Repositories Not Complete
```
ERROR: Repositories Not Complete
================================

The following repositories are not yet complete:

  platform-automation: IN PROGRESS (5/8 chunks)
  app-deploy-state:    PENDING (0/5 chunks)

All repositories must be complete before collation.

Run /dt-status to see current progress.
Run /dt-next to get recommended next action.
```

### Wrong Phase
```
ERROR: Invalid Phase for Collation
==================================

Current phase: collation

Collation has already been performed for this project.
To re-run collation, first reset the phase in .doctrack/state.yaml.

To proceed, run /dt-build to generate the site.
```

## Critical Instructions

- If `~/.local/state/doctrack/current` is missing: Error and suggest `/dt-use <path>` or `/dt-sessions`
- ALWAYS verify all prerequisites before proceeding
- ALWAYS show repository status table before interview
- For navigation structure, default to "audience-journeys" if user presses enter
- For cross-cutting sections, default to architecture and glossary selected
- Sync is opt-out (default yes) - only skip if user explicitly declines
- Conflict detection MUST happen before sync
- If conflicts found, MUST present resolution options
- Generate task file for cross-cutting content - don't generate content directly
- Update BOTH state.yaml AND session.yaml hub section with selections
- Phase must transition: documentation -> collation
- Provide clear next steps: run grind for cross-cutting tasks, then /dt-build

All state changes MUST be persisted to .doctrack/ YAML files.
