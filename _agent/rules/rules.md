---
rule_id: rule-file-standards
version: 1.2.0
authors: []
priority: high
description: >
  Defines the required location, naming conventions, metadata, and structure
  for repository rule files.
globs: "**/*.md"
status: active
last_updated: 2026-08-30
dependencies:
  requires: []
filters:
  - type: file_extension
    pattern: "\\.md$"
  - type: content
    pattern: "(?s)^---\\s*.*?\\s*---"
  - type: event
    pattern: "file_create|file_update"
---

# Rule File Standards

Rule files must be stored in the repository’s `_agent/rules/` directory and
must use the required Markdown and YAML frontmatter structure.

## Actions

### Validate file location

**Type:** reject

**Condition:**

The file path does not match:

```text
_agent/rules/**/*.md
```

**Message:**

```text
Rule files must be placed in the repository's _agent/rules/ directory or one
of its subdirectories.
```

### Validate file extension

**Type:** reject

**Condition:**

The file does not use the `.md` extension.

**Message:**

```text
Rule files must use the .md extension.
```

### Validate filename

**Type:** reject

**Condition:**

The filename is not written in kebab-case.

**Expected pattern:**

```regex
^[a-z0-9]+(?:-[a-z0-9]+)*\.md$
```

**Message:**

```text
Rule filenames must use descriptive kebab-case names, such as
code-formatting.md.
```

### Validate frontmatter

**Type:** reject

**Condition:**

The file does not begin with YAML frontmatter containing these required fields:

- `rule_id`
- `version`
- `priority`
- `description`
- `globs`
- `status`
- `last_updated`
- `dependencies`
- `filters`

**Message:**

```text
Rule files must begin with YAML frontmatter containing all required metadata
fields.
```

### Validate rule identifier

**Type:** reject

**Condition:**

`rule_id` is missing, empty, or does not follow one of these formats:

```text
simple-rule-name
system-name/component-name
```

**Expected pattern:**

```regex
^[a-z0-9]+(?:[-_][a-z0-9]+)*(?:/[a-z0-9]+(?:[-_][a-z0-9]+)*)?$
```

**Message:**

```text
rule_id must be a descriptive kebab-case identifier. Namespaced identifiers
must use the system/component format.
```

### Validate required sections

**Type:** reject

**Condition:**

The Markdown body does not contain:

```markdown
# <Rule title>

## Actions
```

**Message:**

```text
Rule files must contain a title and an Actions section.
```

## Guidance

### File location

Place every rule file under:

```text
PROJECT_ROOT/_agent/rules/
```

Subdirectories may be used to group related rules by domain or namespace:

```text
_agent/rules/security/input-validation.md
_agent/rules/formatting/markdown-style.md
_agent/rules/code-quality/style-checker.md
```

Do not place rule files in the project root, `.rules/`, `rules/`, or any other
directory.

### File naming

Use descriptive kebab-case filenames:

```text
✅ dependency-validation.md
✅ api-security.md
✅ code-quality/style-checker.md

❌ DependencyValidation.md
❌ dependency_validation.md
❌ rule1.md
```

### Frontmatter

Use this structure:

```yaml
---
rule_id: example-rule
version: 1.0.0
authors:
  - author_identifier
priority: medium
description: Describes the purpose of the rule.
globs: "**/*.ts"
status: active
last_updated: 2026-08-30
dependencies:
  requires:
    - rule_id: prerequisite-rule
      version: ">=1.0.0"
filters:
  - type: file_extension
    pattern: "\\.ts$"
  - type: event
    pattern: "file_create|file_update"
---
```

Use semantic versioning for `version` and ISO 8601 dates for
`last_updated`.

### Actions

Actions should state what happens when a condition is met.

Supported action types are:

- `suggest` — provide guidance without blocking the operation.
- `reject` — prevent the operation.
- `require` — require a specific change or artifact.
- `process` — perform a defined transformation or workflow.

Example:

```markdown
### Validate API responses

**Type:** reject

**Condition:**

An API endpoint returns an undocumented response field.

**Message:**

```text
Document all externally visible API response fields.
```
```

### Rule namespacing

Use a simple `rule_id` for independent rules:

```yaml
rule_id: security-validation
```

Use a namespaced identifier when rules belong to a coordinated system:

```yaml
rule_id: code-quality/style-checker
```

For namespaced rules:

- Place component rules beneath a matching directory.
- Use the system name as the prefix.
- Give the orchestrator rule the system name as its `rule_id`.
- List required component rules under `dependencies.requires`.

Example:

```text
_agent/rules/code-quality.md
_agent/rules/code-quality/style-checker.md
_agent/rules/code-quality/naming.md
```

```yaml
# code-quality.md
rule_id: code-quality
dependencies:
  requires:
    - rule_id: code-quality/style-checker
      version: ">=1.0.0"
    - rule_id: code-quality/naming
      version: ">=1.0.0"
```

## Minimal valid rule

```markdown
---
rule_id: example-rule
version: 1.0.0
authors: []
priority: medium
description: Ensures example files follow the required format.
globs: "**/*.example"
status: active
last_updated: 2026-08-30
dependencies:
  requires: []
filters:
  - type: file_extension
    pattern: "\\.example$"
---

# Example Rule

Ensures example files follow the required format.

## Actions

### Provide formatting guidance

**Type:** suggest

Use the repository's standard formatting conventions.
```
