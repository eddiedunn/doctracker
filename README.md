# doctrack

Track documentation work across multi-repository ecosystems.

## What It Does

- Inventories source code across multiple repos
- Identifies documentation needs (not audits - creates from source)
- Creates actionable tasks
- Tracks progress
- Generates automation-ready task files

## Core Principle

**Source code is truth.**

This is NOT a documentation audit system. We don't compare against
existing docs. We read source code, understand what it does, and
create documentation from that understanding.

Existing docs may be wrong, outdated, or misleading. We ignore them.

## Quick Start

1. Create doctrack.yaml (see example)
2. Run /init
3. Run /status to see repos
4. Run /continue to get next action
5. Follow the 7-step process per repo

## The 7-Step Process

For each repository:

1. `/chunk {repo}` - Inventory source, create chunk list
2. `/review {chunk}` - Read source, identify doc needs
3. `/task {chunk}` - Create documentation tasks
4. `/generate {repo}` - Compile into tasks.yaml
5. `grind batch {repo}-tasks.yaml` - Execute tasks
6. Human review
7. `/complete {chunk}` - Verify and mark done

## Commands

| Command | Purpose |
|---------|---------|
| /init | Initialize from doctrack.yaml |
| /status | Show progress dashboard |
| /continue | Get next action |
| /chunk {repo} | Inventory source files |
| /review {chunk} | Review chunk, identify needs |
| /task {chunk} | Create documentation tasks |
| /generate {repo} | Generate tasks.yaml |
| /complete {chunk} | Mark chunk complete |

## Configuration

See doctrack.yaml.example for configuration options.

## State Tracking

Progress is tracked in STATE.yaml. This file can be committed
to git to preserve progress across sessions.
