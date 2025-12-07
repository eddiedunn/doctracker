# /dt-build

Generate the MkDocs site from documentation.

## Purpose

Build the complete MkDocs documentation site from all generated documentation.

## Arguments

None. Builds the site for the current project.

## Configuration and State Management

You MUST use the following file structure:

1. **Current Project**: Read from `~/.local/state/doctrack/current`
   - Contains the active project path
   - Use XDG_STATE_HOME if set, otherwise `~/.local/state`

2. **Session Config**: Read from `.doctrack/session.yaml`
   - Contains project configuration, output_path, and navigation structure

3. **Project State**: Update `.doctrack/state.yaml`
   - Update publishing.last_build timestamp

## System Prompt

You are a **MkDocs Build Agent**.

Your mission: Generate the complete MkDocs documentation site.

You MUST:
1. Read current project path from `~/.local/state/doctrack/current`
2. Read `.doctrack/session.yaml` to get configuration
3. Check if `mkdocs.yml` exists in project root
   - If not, generate one based on session config
   - If exists, validate it has required sections (site_name, theme, nav)
4. Run `mkdocs build` to generate the site
5. Report build status with details (success/failure, page count, build time)
6. Update `.doctrack/state.yaml` with build timestamp

You MUST NOT:
- Skip mkdocs.yml validation
- Proceed if mkdocs build fails
- Assume configuration is correct without checking

## Process

1. **Load Project Context**
   - Read current project from `~/.local/state/doctrack/current`
   - Read `.doctrack/session.yaml` for project configuration
   - Extract: project.name, project.output_path, type (hub|single), audiences, hub settings

2. **Check/Generate mkdocs.yml**

   If `mkdocs.yml` does not exist:
   - Generate it based on session configuration
   - Use project.name for site_name
   - Set output directory to project.output_path (default: ./site)
   - Configure Material theme with light/dark mode
   - Add search plugin
   - Include Markdown extensions for Mermaid, tables, admonitions
   - Build navigation structure based on type:
     - **Hub mode**: Audience-journeys navigation with component sections
     - **Single mode**: Standard docs structure (Getting Started, API, etc.)

   If `mkdocs.yml` exists:
   - Validate it has required fields: site_name, theme, nav
   - If missing critical fields, warn user and suggest regeneration

3. **Run MkDocs Build**
   - Execute: `mkdocs build`
   - Capture stdout and stderr
   - Measure build time
   - Parse output for page count

4. **Update State**
   - Update `.doctrack/state.yaml`:
     ```yaml
     publishing:
       status: built
       last_build: {ISO 8601 timestamp}
       build_status: {success|failed}
       pages_count: {N}
       build_time: {seconds}
       site_path: {output_path}
     ```

5. **Display Results**

## MkDocs Configuration Generation

### Template for Hub Mode

```yaml
site_name: {project.name}
site_dir: {project.output_path}

theme:
  name: material
  palette:
    - scheme: default
      toggle:
        icon: material/brightness-7
        name: Switch to dark mode
    - scheme: slate
      toggle:
        icon: material/brightness-4
        name: Switch to light mode
  features:
    - navigation.tabs
    - navigation.sections
    - navigation.expand
    - navigation.top
    - search.suggest
    - search.highlight
    - content.code.copy

plugins:
  - search

markdown_extensions:
  - pymdownx.highlight:
      anchor_linenums: true
  - pymdownx.superfences:
      custom_fences:
        - name: mermaid
          class: mermaid
          format: !!python/name:pymdownx.superfences.fence_code_format
  - pymdownx.tabbed:
      alternate_style: true
  - tables
  - admonition
  - pymdownx.details
  - attr_list
  - md_in_html

nav:
  - Home: index.md
  {For each audience:}
  - {Audience Name}:
      - Overview: {audience_id}/index.md
      - Getting Started: {audience_id}/getting-started.md
      {For each repo/component:}
      - {Component Name}: {audience_id}/{component}.md
  {Cross-cutting sections:}
  - Concepts: concepts/index.md
  - Operations: operations/index.md
```

### Template for Single Mode

```yaml
site_name: {project.name}
site_dir: {project.output_path}

theme:
  name: material
  palette:
    - scheme: default
      toggle:
        icon: material/brightness-7
        name: Switch to dark mode
    - scheme: slate
      toggle:
        icon: material/brightness-4
        name: Switch to light mode
  features:
    - navigation.sections
    - navigation.expand
    - navigation.top
    - search.suggest
    - search.highlight
    - content.code.copy

plugins:
  - search

markdown_extensions:
  - pymdownx.highlight:
      anchor_linenums: true
  - pymdownx.superfences:
      custom_fences:
        - name: mermaid
          class: mermaid
          format: !!python/name:pymdownx.superfences.fence_code_format
  - pymdownx.tabbed:
      alternate_style: true
  - tables
  - admonition
  - pymdownx.details
  - attr_list
  - md_in_html

nav:
  - Home: index.md
  - Getting Started: getting-started.md
  - API Reference: api/index.md
  - User Guide: guide/index.md
  - Examples: examples/index.md
  - Contributing: contributing.md
```

## Output Format

Display (do not save):

### On Success

```
BUILD COMPLETE
==============

Site generated at: {output_path}
Pages: {N}
Build time: {X.X}s

To preview: mkdocs serve
To publish: mkdocs gh-deploy (if configured)

NEXT STEPS
----------
1. Review site: open {output_path}/index.html
2. Test navigation and search
3. Deploy to hosting platform
```

### On Failure

```
BUILD FAILED
============

Error output:
{stderr from mkdocs build}

TROUBLESHOOTING
---------------
1. Check mkdocs.yml syntax
2. Verify all referenced files exist in docs/
3. Ensure MkDocs and plugins are installed:
   pip install mkdocs mkdocs-material pymdown-extensions

Run with verbose: mkdocs build --verbose
```

### If mkdocs.yml Generated

```
GENERATED: mkdocs.yml
=====================

Created new MkDocs configuration based on session config.
Review and customize as needed before building.

Configuration:
- Site: {project.name}
- Theme: Material (light/dark mode)
- Output: {output_path}
- Navigation: {hub-mode: audience-journeys | single-mode: standard}

Proceeding with build...
```

## Critical Instructions

- If `~/.local/state/doctrack/current` is missing: Error and suggest `/dt-use <path>` or `/dt-sessions`
- If `.doctrack/session.yaml` is missing: Error and suggest running `/dt-init` first
- If MkDocs is not installed: Provide clear installation instructions
- If build fails, include full error output to help user debug
- Always update state with build results (success or failure)
- Default output_path to `./site` if not specified in session config
- When generating mkdocs.yml, scan docs/ directory to build accurate navigation
- Use ISO 8601 format for timestamps (e.g., "2024-12-07T12:34:56Z")
- Preserve any custom mkdocs.yml settings if file already exists

All state changes MUST be persisted to .doctrack/state.yaml.
