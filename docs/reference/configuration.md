# Configuration Reference

This reference documents all configuration files and options used by doctrack.

## Overview

doctrack uses two main configuration files:

1. **`session.yaml`** - Project-level configuration defining repositories, audiences, and documentation standards
2. **`audience-templates.yaml`** - Reusable audience templates for common documentation personas

Additionally, doctrack maintains:
- **`state.yaml`** - Progress tracking for the current session
- **`sessions.yaml`** - Global registry of all doctrack sessions

## session.yaml

The `session.yaml` file is created in `.doctrack/session.yaml` when you initialize a new session with `/dt-init`. It defines the complete configuration for a documentation project.

### File Location

```
{project-root}/.doctrack/session.yaml
```

### Full Structure

```yaml
version: 1
created: "2025-12-07T10:30:00Z"
type: single  # or "hub"

project:
  name: "Project Name"
  description: "Brief project description"
  output_path: "./site"
  mkdocs: true

repositories:
  - name: repo-name
    path: /path/to/repo
    content_type: python
    priority: high
    prefix: PREFIX
    source_dirs:
      - src/
      - lib/
    doc_dir: docs/

audiences:
  - id: developer
    name: Developer
    description: "Developers using the system"
    checklist:
      - "API reference complete"
      - "Examples provided"

standards:
  docstring_style: numpy
  diagram_format: mermaid
  require_examples: true

hub:
  navigation_style: audience-journeys
  sync_direction: pull
  cross_cutting_sections:
    - "Architecture Overview"
    - "Getting Started"
```

### Root-Level Fields

#### version
- **Type:** `integer`
- **Required:** Yes
- **Value:** `1`
- **Description:** Schema version for forward compatibility

#### created
- **Type:** `string` (ISO 8601 datetime)
- **Required:** No
- **Example:** `"2025-12-07T10:30:00Z"`
- **Description:** Timestamp when the session was created

#### type
- **Type:** `string`
- **Required:** Yes
- **Values:** `"single"` or `"hub"`
- **Description:** Session mode determining workflow and output structure
  - `single` - Document one repository or monorepo
  - `hub` - Aggregate documentation from multiple distributed repositories

### project Object

Configuration for the overall documentation project.

#### project.name
- **Type:** `string`
- **Required:** Yes
- **Example:** `"Jenkins Platform Docs"`
- **Description:** Human-readable project name displayed in outputs and status reports

#### project.description
- **Type:** `string`
- **Required:** No
- **Example:** `"Documentation for Jenkins microservices platform"`
- **Description:** Brief description of the documentation project

#### project.output_path
- **Type:** `string`
- **Required:** No
- **Default:** `"./site"`
- **Example:** `"docs/"` (single mode) or `".doctrack/output/"` (hub mode)
- **Description:** Directory where final documentation is written

#### project.mkdocs
- **Type:** `boolean`
- **Required:** No
- **Default:** `true`
- **Description:** Whether to generate MkDocs-compatible output

### repositories Array

List of repositories to document. Must contain at least one repository.

#### repository.name
- **Type:** `string`
- **Required:** Yes
- **Example:** `"api-server"`
- **Description:** Unique identifier for this repository within the session

#### repository.path
- **Type:** `string`
- **Required:** Yes
- **Example:** `"."` (single mode) or `"/Users/me/repos/api-server"` (hub mode)
- **Description:** Absolute or relative path to the repository
  - Single mode: Use `"."` for current directory
  - Hub mode: Use absolute paths to external repositories

#### repository.content_type
- **Type:** `string`
- **Required:** Yes
- **Values:** `"python"`, `"ansible"`, `"json-schema"`, `"yaml-config"`, `"mixed"`
- **Description:** Primary content type of the repository, affects documentation strategies

#### repository.priority
- **Type:** `string`
- **Required:** No
- **Default:** `"medium"`
- **Values:** `"high"`, `"medium"`, `"low"`
- **Description:** Documentation priority for workflow guidance in hub mode

#### repository.prefix
- **Type:** `string`
- **Required:** Yes
- **Pattern:** `^[A-Z][A-Z0-9]*$`
- **Example:** `"API"`, `"AUTH"`, `"USER"`
- **Description:** Uppercase prefix for chunk IDs (e.g., `API-AUTH-001`)

#### repository.source_dirs
- **Type:** `array` of `string`
- **Required:** No
- **Example:** `["src/", "lib/"]`
- **Description:** Directories containing source code to document

#### repository.doc_dir
- **Type:** `string`
- **Required:** No
- **Default:** `"docs/"`
- **Description:** Directory where generated documentation is written (relative to repository root)

### audiences Array

List of documentation audiences. Must contain at least one audience.

#### audience.id
- **Type:** `string`
- **Required:** Yes
- **Pattern:** `^[a-z][a-z0-9_]*$`
- **Example:** `"developer"`, `"administrator"`, `"end_user"`
- **Description:** Unique lowercase identifier for the audience

#### audience.name
- **Type:** `string`
- **Required:** Yes
- **Example:** `"Application Developer"`
- **Description:** Human-readable display name

#### audience.description
- **Type:** `string`
- **Required:** No
- **Example:** `"Developers integrating with the API"`
- **Description:** Brief description of who this audience is

#### audience.checklist
- **Type:** `array` of `string`
- **Required:** Yes
- **Minimum items:** 1
- **Example:**
  ```yaml
  checklist:
    - "API reference complete"
    - "Code examples provided"
    - "Error handling documented"
  ```
- **Description:** Requirements for documenting to this audience

### standards Object

Documentation conventions and quality standards.

#### standards.docstring_style
- **Type:** `string`
- **Required:** No
- **Default:** `"numpy"`
- **Values:** `"numpy"`, `"google"`, `"sphinx"`
- **Description:** Python docstring format convention

#### standards.diagram_format
- **Type:** `string`
- **Required:** No
- **Default:** `"mermaid"`
- **Description:** Format for diagrams in documentation

#### standards.require_examples
- **Type:** `boolean`
- **Required:** No
- **Default:** `true`
- **Description:** Whether code examples are required in documentation

### hub Object (Hub Mode Only)

Configuration specific to hub mode. Only used when `type: hub`.

#### hub.navigation_style
- **Type:** `string`
- **Required:** No
- **Default:** `"audience-journeys"`
- **Values:** `"audience-journeys"`, `"component-first"`, `"hybrid"`
- **Description:** How to organize the final documentation site
  - `audience-journeys` - Organize by audience needs
  - `component-first` - Organize by repository/component
  - `hybrid` - Mixed organization

#### hub.sync_direction
- **Type:** `string`
- **Required:** No
- **Default:** `"pull"`
- **Values:** `"pull"`
- **Description:** How documentation is synchronized (currently only `"pull"` is supported)

#### hub.cross_cutting_sections
- **Type:** `array` of `string`
- **Required:** No
- **Example:**
  ```yaml
  cross_cutting_sections:
    - "Architecture Overview"
    - "Getting Started"
    - "Security"
  ```
- **Description:** Top-level documentation sections that span multiple repositories

## Examples

### Single Mode Example

Complete configuration for documenting a single Python API server:

```yaml
version: 1
created: "2025-12-07T10:30:00Z"
type: single

project:
  name: "My API Server"
  description: "REST API for user management"
  output_path: docs/
  mkdocs: true

repositories:
  - name: my-api-server
    path: .
    content_type: python
    priority: high
    prefix: API
    source_dirs:
      - src/
    doc_dir: docs/

audiences:
  - id: developer
    name: Application Developer
    description: "Developers integrating with the API"
    checklist:
      - "API reference with parameters and return values"
      - "Code examples for common use cases"
      - "Error handling documented"
      - "Authentication flow explained"

  - id: administrator
    name: Administrator
    description: "Operations team deploying and maintaining the service"
    checklist:
      - "Installation and setup documented"
      - "Configuration options explained"
      - "Operational procedures included"
      - "Troubleshooting guides provided"

standards:
  docstring_style: google
  diagram_format: mermaid
  require_examples: true
```

### Hub Mode Example

Complete configuration for aggregating documentation from multiple microservices:

```yaml
version: 1
created: "2025-12-07T10:30:00Z"
type: hub

project:
  name: "Platform Documentation Hub"
  description: "Unified documentation for microservices platform"
  output_path: .doctrack/output/
  mkdocs: true

repositories:
  - name: api-server
    path: /Users/me/repos/api-server
    content_type: python
    priority: high
    prefix: API
    source_dirs:
      - src/
    doc_dir: docs/

  - name: auth-service
    path: /Users/me/repos/auth-service
    content_type: python
    priority: high
    prefix: AUTH
    source_dirs:
      - src/auth/
    doc_dir: docs/

  - name: user-service
    path: /Users/me/repos/user-service
    content_type: python
    priority: medium
    prefix: USER
    source_dirs:
      - src/
    doc_dir: docs/

  - name: ansible-automation
    path: /Users/me/repos/automation
    content_type: ansible
    priority: low
    prefix: AUTO
    source_dirs:
      - playbooks/
      - roles/
    doc_dir: docs/

audiences:
  - id: application_developer
    name: Application Developer
    description: "Developers integrating with the platform"
    checklist:
      - "API reference complete for all services"
      - "Integration examples provided"
      - "Authentication and authorization explained"
      - "Error codes documented"

  - id: platform_developer
    name: Platform Developer
    description: "Developers contributing to platform services"
    checklist:
      - "Architecture documented"
      - "Development setup guide"
      - "Contributing guidelines"
      - "Extension points explained"

  - id: administrator
    name: Administrator
    description: "Operations team managing the platform"
    checklist:
      - "Deployment procedures"
      - "Configuration reference"
      - "Troubleshooting guides"
      - "Security and RBAC documentation"

standards:
  docstring_style: numpy
  diagram_format: mermaid
  require_examples: true

hub:
  navigation_style: audience-journeys
  sync_direction: pull
  cross_cutting_sections:
    - "Platform Architecture"
    - "Getting Started"
    - "Security & Authentication"
    - "Deployment Guide"
```

## audience-templates.yaml

The `audience-templates.yaml` file provides reusable audience definitions based on common documentation personas. It's located in the doctrack root directory and referenced during session initialization.

### File Location

```
{doctrack-root}/audience-templates.yaml
```

### Structure

```yaml
version: 1
templates:
  template_id:
    name: "Display Name"
    description: "Brief description of this audience"
    checklist:
      - "Checklist item 1"
      - "Checklist item 2"
      - "Checklist item 3"
```

### Fields

#### version
- **Type:** `integer`
- **Value:** `1`
- **Description:** Template file schema version

#### templates
- **Type:** `object`
- **Description:** Map of template IDs to template definitions

#### template_id
- **Type:** `string`
- **Example:** `administrator`, `application_developer`
- **Description:** Unique identifier for the template (snake_case)

#### template.name
- **Type:** `string`
- **Example:** `"Administrator"`
- **Description:** Human-readable name for this audience

#### template.description
- **Type:** `string`
- **Example:** `"Operations, deployment, configuration"`
- **Description:** Brief description of the audience's role

#### template.checklist
- **Type:** `array` of `string`
- **Description:** Standard documentation requirements for this audience

### Default Templates

doctrack includes these standard templates:

#### administrator
For operations teams managing deployments and infrastructure.

**Checklist:**
- Installation and setup documented
- Configuration options explained
- Operational procedures (backup, restore, upgrade)
- Troubleshooting guides
- CLI/TUI usage with examples
- Security (RBAC, secrets) explained

#### application_developer
For developers integrating with or consuming the system.

**Checklist:**
- Integration points documented
- API reference complete
- Configuration format with schema
- Error messages and handling explained
- Working examples provided

#### platform_developer
For developers contributing to or extending the system.

**Checklist:**
- Architecture documented
- All public APIs have docstrings
- Extension points explained
- Development setup guide
- Contributing guidelines

#### end_user
For non-technical users of the system.

**Checklist:**
- Getting started guide
- Feature walkthroughs
- FAQ
- UI/UX documentation

### Example

Complete `audience-templates.yaml`:

```yaml
version: 1
templates:
  administrator:
    name: Administrator
    description: "Operations, deployment, configuration"
    checklist:
      - "Installation and setup documented"
      - "Configuration options explained"
      - "Operational procedures (backup, restore, upgrade)"
      - "Troubleshooting guides"
      - "CLI/TUI usage with examples"
      - "Security (RBAC, secrets) explained"

  application_developer:
    name: Application Developer
    description: "Integration, APIs, consuming the system"
    checklist:
      - "Integration points documented"
      - "API reference complete"
      - "Configuration format with schema"
      - "Error messages and handling explained"
      - "Working examples provided"

  platform_developer:
    name: Platform Developer
    description: "Contributing, extending, building on"
    checklist:
      - "Architecture documented"
      - "All public APIs have docstrings"
      - "Extension points explained"
      - "Development setup guide"
      - "Contributing guidelines"

  end_user:
    name: End User
    description: "Non-technical users of the system"
    checklist:
      - "Getting started guide"
      - "Feature walkthroughs"
      - "FAQ"
      - "UI/UX documentation"
```

### Custom Templates

You can add custom audience templates to match your organization's needs:

```yaml
version: 1
templates:
  # ... standard templates ...

  data_scientist:
    name: Data Scientist
    description: "Analytics and ML practitioners"
    checklist:
      - "Data pipeline documentation"
      - "Model API reference"
      - "Feature engineering guides"
      - "Example notebooks"
      - "Performance characteristics"

  security_engineer:
    name: Security Engineer
    description: "Security review and compliance"
    checklist:
      - "Authentication mechanisms"
      - "Authorization model (RBAC)"
      - "Secrets management"
      - "Audit logging"
      - "Compliance considerations"
```

## Related Files

### state.yaml

While not a configuration file, `state.yaml` tracks documentation progress and is automatically managed by doctrack.

**Location:** `{project-root}/.doctrack/state.yaml`

**Example:**
```yaml
version: 1
created: "2025-12-07T10:00:00Z"
last_updated: "2025-12-07T14:30:00Z"

summary:
  total_repos: 1
  pending: 0
  in_progress: 1
  complete: 0
  current_focus: my-api-server

repositories:
  my-api-server:
    status: in_progress
    current_step: 3
    step_name: "Create Tasks"
    chunks_total: 15
    chunks_reviewed: 15
    chunks_tasked: 10
    chunks_complete: 5
    current_chunk: API-AUTH-003
    tracking_file: /path/to/CHUNK_TRACKING.md
    started: "2025-12-07T10:00:00Z"
    completed: null
    notes: |
      Focused on authentication and user management modules

step_definitions:
  1: "Chunk Repository"
  2: "Review Chunks"
  3: "Create Tasks"
  4: "Generate Tasks YAML"
  5: "Execute Tasks"
  6: "Verify Results"
  7: "Mark Complete"
```

### sessions.yaml

Global registry of all doctrack sessions.

**Location:** `~/.local/state/doctrack/sessions.yaml`

**Example:**
```yaml
version: 1
sessions:
  - path: /Users/gdunn6/code/my-api-server
    name: "My API Server"
    type: single
    last_accessed: "2025-12-07T14:30:00Z"
    status: in_progress

  - path: /Users/gdunn6/code/platform-docs-hub
    name: "Platform Documentation Hub"
    type: hub
    last_accessed: "2025-12-06T10:00:00Z"
    status: complete
```

## Schema Validation

doctrack includes a JSON Schema for validating `session.yaml` files.

**Schema location:** `{doctrack-root}/config.schema.json`

The schema enforces:
- Required fields
- Valid enum values
- Pattern matching for IDs and prefixes
- Minimum array lengths
- Type constraints

## See Also

- [Session Management](../concepts/session-management.md) - Understanding doctrack's session architecture
- [Single vs Hub Mode](../concepts/single-vs-hub.md) - Choosing the right mode for your project
- [Commands Reference](./commands.md) - All available commands
