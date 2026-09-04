---
rule_id: ansible-best-practices-rule
version: 1.1.0
authors:
  - Cline AI Agent
priority: high
description: |
  Defines the comprehensive set of mandatory rules and guidelines for AI agents operating on Ansible repositories. 
  This rule ensures all Ansible interactions prioritize security, idempotency, and adherence to modern Ansible best practices.
globs: "**/playbooks/*.yml" # Applies to all Ansible playbooks
status: active
last_updated: 2026-09-03
dependencies:
  requires: []
filters:
  - type: content
    pattern: "ansible"
---

# Rule Title: AI Agent Rules for Ansible Best Practices

The agent must follow these rules when interacting with any Ansible repository. The primary goal is to ensure that all changes are correct, idempotent, secure, and maintainable.

## Actions

### Action: Prioritize Inspection and Minimal Change
**Type:** require

#### Step: Repository Discovery and Analysis
**Thinking Approach:**
> Technical Mentor: Before any code is written, a thorough assessment of the existing infrastructure setup must occur.
**Instructions:**
The agent must first identify all critical components of the repository (inventory files, roles, playbooks, collections, `ansible.cfg`, etc.). Key areas include determining the active inventory, primary entry playbook, expected Ansible version, and existing naming/directory conventions. Never assume behavior; inspect first.

#### Step: Scope Adherence
**Thinking Approach:**
> Minimalist Engineer: Limit changes strictly to what is necessary to achieve the required outcome.
**Instructions:**
Changes must be the smallest possible unit to satisfy the request. Avoid refactoring unrelated code, modifying production configurations unnecessarily, or upgrading dependencies unless explicitly directed.

### Action: Module Over Shell Execution
**Type:** require

#### Step: Module Preference
**Thinking Approach:**
> Core Architect: Favor specialized, built-in Ansible modules for all operations.
**Instructions:**
Always prefer a dedicated Ansible module (e.g., `ansible.builtin.package`, `ansible.builtin.file`) over generic modules like `ansible.builtin.shell` or `ansible.builtin.command`. Use shell modules only when no dedicated module can perform the operation.

#### Step: Idempotency Enforcement
**Thinking Approach:**
> Reliability Engineer: Ensure that running the process multiple times yields the same result without unintended 'changes'.
**Instructions:**
Every task must be idempotent where possible. If non-idempotent command execution is required (e.g., using `command`), justify it by explaining why a module is insufficient, and utilize `creates`/`removes` or `changed_when`/`failed_when` appropriately.

## Conceptual Approaches Used

### Thinking Approach: Security and Compliance Guardian
**Thinking Approach:**
> Security Architect: Treat all infrastructure changes as high-risk security events.
**Instructions:**
1. **Secrets Handling:** Never hard-code sensitive data (passwords, tokens, keys). Use Ansible Vault, environment variables, or external secret management, and always use `no_log: true`.
2. **Permissions:** Always explicitly define ownership (`owner`, `group`) and file modes (`mode`) for configuration files to prevent relying on system defaults.
3. **Validation:** Changes must be verifiable using `--check` and `--diff` flags. Validation via `ansible-lint` and `yamllint` is mandatory.

### Thinking Approach: Structured Development Model
**Thinking Approach:**
> System Architect: Design rules to ensure separation of concerns and clean project structure.
**Instructions:**
1. **Variable Usage:** Use role defaults (`defaults/main.yml`) for configurable behavior and reserved `vars/main.yml` only for non-overridable values.
2. **Handlers/Templates:** Use `handlers` for service restarts/reloads only when configuration changes. Use Jinja2 templates for all configuration files requiring dynamic content.
3. **Role Structure:** Adhere strictly to the standard role directory structure (`defaults`, `handlers`, `tasks`, etc.).

## Examples

### Example 1: Module Preference (Module > Shell)
**Input:**
A task attempting to ensure a directory exists using the shell:
```markdown
- name: Create directory
  ansible.builtin.command: mkdir -p /etc/app/conf
```

**Output:**
The preferred, idempotent, and modular approach:
```markdown
- name: Create application configuration directory
  ansible.builtin.file:
    path: /etc/app/conf
    state: directory
    owner: root
    group: root
    mode: "0755"
```

### Example 2: Handling Credentials
**Input:**
A task exposing a secret:
```markdown
- name: Debug password
  ansible.builtin.debug:
    var: sensitive_password
```

**Output:**
The secure pattern utilizing logging suppression:
```markdown
- name: Configure credentials
  ansible.builtin.some_module:
    username: "{{ app_username }}"
    password: "{{ app_password }}"
  no_log: true
```
