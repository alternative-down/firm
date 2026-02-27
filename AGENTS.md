# Firm Workspace - AGENTS.md

This file documents the Firm workspace and provides reference for AI agents and team members.

## Overview

This is a **Firm workspace** — a text-based, version-controlled business management system. All data is stored as plain `.firm` files in this directory.

**Key Properties:**
- Plain text format (easy to read, edit, and diff)
- Version controlled with Git
- Queryable via CLI
- Structured with schemas and relationships
- No SaaS vendor lock-in

## File Structure

```
/firm/
├── .gitignore           # Excludes .firm-graph* files
├── schemas.firm         # Default entity type definitions
├── AGENTS.md            # This file
└── entities/            # (optional) Organized entity files
    ├── people.firm
    ├── organizations.firm
    ├── projects.firm
    └── tasks.firm
```

Firm recursively discovers all `.firm` files in the workspace.

## Entity Types & Schemas

### Default Schemas

The `schemas.firm` file defines four default entity types:

#### `person`
Represents a team member or contact.
- `name` (string, required)
- `email` (string)
- `phone` (string)

#### `organization`
Represents a company, team, or client.
- `name` (string, required)
- `email` (string)
- `urls` (list of strings)

#### `project`
Represents a work initiative.
- `name` (string, required)
- `description` (string)
- `status` (enum: planning, active, paused, completed)
- `organization_ref` (reference to organization)

#### `task`
Represents a unit of work.
- `name` (string, required)
- `description` (string)
- `is_completed` (boolean)
- `assignee_ref` (reference to person)
- `project_ref` (reference to project)
- `due_date` (datetime)

### Creating Entities

Example entity (in a `.firm` file):

```
person jane_doe {
  name = "Jane Doe"
  email = "jane@example.com"
  phone = "+55 11 99999-9999"
}

organization acme_corp {
  name = "Acme Corporation"
  email = "info@acme.com"
  urls = ["https://acme.com"]
}

project website_redesign {
  name = "Website Redesign"
  description = "Modernize the company website"
  status = "active"
  organization_ref = organization.acme_corp
}

task design_mockups {
  name = "Design mockups"
  description = "Create wireframes and visual mockups"
  is_completed = false
  assignee_ref = person.jane_doe
  project_ref = project.website_redesign
  due_date = 2025-03-15 at 17:00 UTC
}
```

## CLI Commands

### List Entities

List all entities of a type:
```bash
firm list person
firm list project
firm list task
```

### Get Entity

View a specific entity:
```bash
firm get person jane_doe
firm get project website_redesign
```

### Add Entity

Interactive:
```bash
firm add
```

Non-interactive:
```bash
firm add --type task --id new_task --field name "My Task"
```

### Query Data

Find all incomplete tasks:
```bash
firm query 'from task | where is_completed == false'
```

Find tasks assigned to Jane:
```bash
firm query 'from task | where assignee_ref == person.jane_doe'
```

Count tasks by project:
```bash
firm query 'from project | related task | count'
```

Find active projects for Acme:
```bash
firm query 'from organization | where name == "Acme Corporation" | related project | where status == "active"'
```

### Query Language

**Basic syntax:**
```
from <type> | <operation> | <operation> | ... | <aggregation>
```

**Operations:**
- `where <field> <operator> <value>` - Filter entities
- `related([degrees])` - Traverse relationships (default 1 degree)
- `order [asc|desc] <field>` - Sort results
- `limit <n>` - Limit results to N items

**Aggregations:**
- `count` - Count matching entities
- `select <field>, <field>, ...` - Extract specific fields
- `sum <field>` - Sum numeric field
- `average <field>` - Compute mean
- `median <field>` - Compute median

**Operators:** `==`, `!=`, `>`, `<`, `>=`, `<=`, `contains`

### Related Entities

Show all entities connected to one:
```bash
firm related organization acme_corp
```

## Git Workflow

**Always commit after changes:**

```bash
# Make changes via firm add or by editing .firm files
firm add --type task --id task_123 --field name "New Task"

# Commit changes
git add .
git commit -m "feat: add new task for project X"

# Push to remote
git push origin main
```

All changes are automatically tracked in Git for auditability and version control.

## Agent Integration

When using the `firm-app` skill in OpenClaw:

```bash
# List all projects
~/.openclaw/skills/firm-app/scripts/queries/list-all.sh project

# Run a query
~/.openclaw/skills/firm-app/scripts/queries/query.sh 'from task | where is_completed == false'

# Add a new entity (auto-commits and pushes)
~/.openclaw/skills/firm-app/scripts/workflows/add-entity.sh --type task --id my_task --field name "My Task"
```

## Customization

Edit `schemas.firm` to:
- Add new entity types
- Add fields to existing types
- Change field requirements or types
- Create domain-specific schemas

Example custom schema:

```
schema sprint {
  field {
    name = "number"
    type = "integer"
    required = true
  }
  
  field {
    name = "start_date"
    type = "datetime"
    required = true
  }
  
  field {
    name = "end_date"
    type = "datetime"
    required = true
  }
  
  field {
    name = "project_ref"
    type = "reference"
    required = true
  }
}
```

## Field Types

- `string` - Text
- `integer` - Whole numbers
- `float` - Decimal numbers
- `boolean` - True/false
- `currency` - Money (e.g., `5000.00 USD`)
- `datetime` - Date and time (e.g., `2025-03-15 at 17:00 UTC`)
- `date` - Date only (e.g., `2025-03-15`)
- `reference` - Link to another entity (e.g., `person.jane_doe`)
- `list` - Array of values
- `enum` - Enumerated values with allowed options
- `path` - File paths

## Resources

- **Official Docs:** https://firm.42futures.com/
- **GitHub:** https://github.com/42futures/firm
- **Query Language Reference:** https://firm.42futures.com/guide/querying.html
- **DSL Reference:** https://firm.42futures.com/reference/dsl-reference.html

## Git Configuration

This workspace is configured with:
- **User:** `Firm` (firm@alternativedown.com.br)
- **Remote:** `alternative-down/firm` on GitHub
- **Branch:** `main`
- **.gitignore:** Excludes Firm graph cache files (`.firm-graph*`)

### Syncing Changes

Push changes to keep all agents synchronized:

```bash
git add .
git commit -m "feat: describe your changes"
git push origin main
```
