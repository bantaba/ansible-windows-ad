# AI Agent Rules for Ansible

## 1. Purpose

This document defines the rules and guidelines an AI agent must follow when working with this Ansible repository.

The agent must prioritize:

1. Correctness
2. Idempotency
3. Security
4. Maintainability
5. Minimal and focused changes
6. Successful validation before considering work complete

Do not make assumptions about infrastructure behavior when the repository contains existing conventions or configuration that can be inspected.

---

## 2. Repository Discovery

Before making changes, inspect the repository structure and identify:

* `ansible.cfg`
* Inventory files
* `group_vars/`
* `host_vars/`
* `roles/`
* `playbooks/`
* `collections/`
* `requirements.yml`
* `requirements.yaml`
* `vault` files
* Templates
* Handlers
* Tasks
* Molecule configuration
* CI/CD configuration
* Documentation

First determine:

* Which inventory is used
* Which playbook is the entry point
* Which roles are involved
* Which Ansible version is expected
* Which collections are required
* Existing naming and directory conventions
* Existing testing and linting commands

Do not introduce a new repository structure if an existing structure already provides an appropriate solution.

---

## 3. Change Scope

Make the smallest change necessary to satisfy the requested task.

Do not:

* Refactor unrelated code
* Rename existing variables without a reason
* Reformat unrelated files
* Upgrade dependencies unless explicitly requested
* Change unrelated roles or playbooks
* Remove existing functionality
* Modify production configuration unnecessarily

If a broader change is required, explain why before making it.

---

## 4. Ansible Best Practices

Follow modern Ansible practices.

Prefer:

* Fully qualified collection names (FQCN)
* Idempotent modules
* Variables instead of hard-coded values
* Handlers for service restarts/reloads
* Templates for generated configuration
* `notify` instead of restarting services unnecessarily
* Tags where the repository already uses them
* Role defaults for configurable behavior
* Appropriate variable precedence
* Explicit task names

Example:

```yaml
- name: Ensure nginx is installed
  ansible.builtin.package:
    name: nginx
    state: present
```

Prefer this over shell or command execution when an Ansible module can perform the same operation.

---

## 5. Idempotency

Every task must be idempotent whenever possible.

Running the same playbook multiple times should not continually report changes.

Avoid patterns such as:

```yaml
- name: Run command
  ansible.builtin.command: some-command
```

when an appropriate Ansible module exists.

If `command` or `shell` is genuinely required:

* Explain why it is required
* Use `creates` or `removes` where appropriate
* Use `changed_when` when necessary
* Use `failed_when` when necessary
* Avoid unnecessary command execution

Do not use `changed_when: false` merely to hide incorrect task behavior.

---

## 6. Modules Over Shell

Always prefer an Ansible module over `ansible.builtin.shell` or `ansible.builtin.command`.

Preferred order:

1. Dedicated Ansible module
2. `ansible.builtin.command`
3. `ansible.builtin.shell`

Use shell only when the operation cannot reasonably be performed with an existing module.

---

## 7. Variables

Use meaningful and descriptive variable names.

Follow the repository's existing naming convention.

Avoid unnecessarily generic names such as:

```yaml
name:
config:
enabled:
path:
user:
```

Prefer namespaced role variables where appropriate:

```yaml
nginx_listen_port:
nginx_worker_processes:
nginx_config_path:
```

Do not hard-code values that are expected to vary between environments.

Use defaults for configurable role behavior:

```yaml
# defaults/main.yml

myapp_enabled: true
myapp_port: 8080
```

---

## 8. Variable Precedence

Understand Ansible variable precedence before changing where a variable is defined.

Do not move variables between:

* `defaults`
* `vars`
* `group_vars`
* `host_vars`
* inventory
* playbook variables

unless there is a clear reason.

Prefer `defaults/main.yml` for variables that users are expected to override.

Use `vars/main.yml` only for values that should generally not be overridden.

---

## 9. Secrets and Sensitive Data

Never hard-code secrets.

Do not commit:

* Passwords
* API keys
* Tokens
* Private keys
* Cloud credentials
* SSH private keys
* Database credentials
* Encryption keys

Use the repository's existing secret-management approach, such as:

* Ansible Vault
* Environment variables
* External secret management
* CI/CD secret injection

Never print secrets in task output.

Avoid:

```yaml
- ansible.builtin.debug:
    var: password
```

Use `no_log: true` when sensitive values could appear in task output:

```yaml
- name: Configure credentials
  ansible.builtin.some_module:
    username: "{{ app_username }}"
    password: "{{ app_password }}"
  no_log: true
```

---

## 10. Templates

Use Jinja2 templates for configuration files that require dynamic values.

Templates should:

* Be readable
* Use descriptive variables
* Avoid unnecessary logic
* Preserve valid configuration syntax
* Avoid embedding secrets directly

Prefer:

```jinja2
listen {{ nginx_listen_port }};
```

over hard-coded configuration when the value is intended to be configurable.

---

## 11. Handlers

Use handlers for operations that should happen only when configuration changes.

Examples:

* Restart service
* Reload service
* Reload daemon
* Restart application

Prefer:

```yaml
notify: Restart nginx
```

with:

```yaml
handlers:
  - name: Restart nginx
    ansible.builtin.service:
      name: nginx
      state: restarted
```

Do not restart services on every playbook run unless explicitly required.

---

## 12. Task Naming

Every task should have a clear, descriptive name.

Good:

```yaml
- name: Install required nginx packages
```

Bad:

```yaml
- name: Install packages
```

Task names should describe the intended outcome rather than the implementation.

---

## 13. Conditionals

Use clear and readable conditionals.

Prefer:

```yaml
when: nginx_enabled
```

Avoid unnecessary Jinja expressions inside `when`:

```yaml
when: "{{ nginx_enabled }}"
```

Do not compare booleans unnecessarily:

```yaml
when: nginx_enabled == true
```

Prefer:

```yaml
when: nginx_enabled
```

---

## 14. Loops

Use loops instead of duplicating similar tasks.

Prefer:

```yaml
- name: Install required packages
  ansible.builtin.package:
    name: "{{ item }}"
    state: present
  loop: "{{ required_packages }}"
```

Avoid unnecessarily complex nested loops.

Use meaningful loop variables when nesting is required:

```yaml
loop_control:
  loop_var: package_name
```

---

## 15. Error Handling

Do not hide failures unless there is a deliberate reason.

Avoid broad usage of:

```yaml
ignore_errors: true
```

If failure is expected and should be handled, use:

* `failed_when`
* `changed_when`
* `block`
* `rescue`
* `always`

Make error handling explicit.

---

## 16. Privilege Escalation

Use privilege escalation only where required.

Prefer:

```yaml
become: true
```

at the appropriate play, role, or task level rather than unnecessarily applying it globally.

Do not disable privilege-related security controls simply to make a task pass.

---

## 17. Files and Permissions

When creating files or directories, explicitly consider:

* Owner
* Group
* Mode
* SELinux context where applicable
* Parent directory
* Whether the resource should be created or replaced

Avoid relying on system defaults for security-sensitive files.

Example:

```yaml
- name: Create application configuration directory
  ansible.builtin.file:
    path: /etc/myapp
    state: directory
    owner: root
    group: root
    mode: "0755"
```

Use quoted file modes such as `"0644"` and `"0755"`.

---

## 18. Service Management

Use Ansible service modules instead of shell commands.

Prefer:

```yaml
ansible.builtin.service:
  name: nginx
  state: started
  enabled: true
```

Do not use:

```yaml
shell: systemctl start nginx
```

unless there is a documented reason.

---

## 19. Package Management

Use the appropriate package module:

```yaml
ansible.builtin.package:
```

when cross-platform behavior is desired.

Use platform-specific package modules only when their functionality is required.

Avoid installing packages using shell commands.

---

## 20. Collections

Use FQCN for modules:

```yaml
ansible.builtin.copy
ansible.builtin.template
ansible.builtin.service
ansible.builtin.package
ansible.builtin.file
ansible.builtin.command
ansible.builtin.shell
```

If a third-party collection is required:

1. Check whether it already exists in `requirements.yml`
2. Follow the repository's versioning convention
3. Do not silently add dependency upgrades
4. Prefer pinned or appropriately constrained versions according to project conventions

---

## 21. Compatibility

Respect the Ansible and target-platform versions supported by the repository.

Before using a newer module parameter or feature:

* Check the project's Ansible version
* Check collection compatibility
* Check operating-system compatibility
* Follow existing compatibility patterns

Do not introduce functionality that requires a newer Ansible version without explicitly identifying the compatibility impact.

---

## 22. Security

Treat infrastructure changes as security-sensitive.

Pay particular attention to:

* File permissions
* User and group ownership
* SSH configuration
* Sudo configuration
* Firewall rules
* TLS configuration
* Credentials
* Secrets
* Privilege escalation
* Service exposure
* Network listeners

Do not weaken security controls simply to make a deployment succeed.

Never disable:

* TLS verification
* Host key checking
* Authentication
* Firewall protections

unless explicitly requested and the implications are documented.

---

## 23. Destructive Operations

Be extremely cautious with destructive actions.

Before introducing tasks that can:

* Delete files
* Remove packages
* Remove users
* Drop databases
* Destroy cloud resources
* Modify firewall rules
* Overwrite important configuration
* Restart critical services

verify that the requested behavior actually requires the destructive operation.

Do not add destructive behavior as a workaround for a failed task.

---

## 24. Check Mode and Diff

When practical, changes should work correctly with:

```bash
ansible-playbook --check
```

and:

```bash
ansible-playbook --diff
```

Avoid implementations that unnecessarily prevent check mode or diff functionality.

If a task cannot support check mode correctly, handle it explicitly rather than pretending it does.

---

## 25. Validation

After making changes, validate them.

Use the repository's existing validation process first.

Typical checks may include:

```bash
ansible-lint
```

```bash
yamllint .
```

```bash
ansible-playbook --syntax-check <playbook.yml>
```

```bash
ansible-playbook --check <playbook.yml>
```

If Molecule is configured:

```bash
molecule test
```

Do not claim that tests passed unless they were actually executed successfully.

If validation cannot be run, clearly state what could not be validated and why.

---

## 26. YAML Formatting

Follow existing YAML formatting.

General rules:

* Use 2-space indentation
* Do not use tabs
* Quote file modes
* Keep lines reasonably readable
* Use valid YAML
* Avoid unnecessary quoting
* Maintain consistent list formatting

Example:

```yaml
- name: Configure application
  ansible.builtin.template:
    src: app.conf.j2
    dest: /etc/myapp/app.conf
    owner: root
    group: root
    mode: "0644"
  notify: Restart myapp
```

---

## 27. Comments

Comments should explain **why**, not simply restate **what** the task does.

Good:

```yaml
# The application requires this directory to exist before the service starts.
```

Avoid:

```yaml
# Create directory
```

Do not add comments unnecessarily.

---

## 28. Role Structure

When working with roles, follow the standard structure where applicable:

```text
roles/
└── myrole/
    ├── defaults/
    │   └── main.yml
    ├── handlers/
    │   └── main.yml
    ├── tasks/
    │   └── main.yml
    ├── templates/
    ├── files/
    ├── vars/
    │   └── main.yml
    ├── meta/
    │   └── main.yml
    └── molecule/
```

Do not create unnecessary directories or files.

---

## 29. Playbooks

Playbooks should be readable and focused.

Prefer:

```yaml
- name: Configure web servers
  hosts: webservers
  become: true

  roles:
    - nginx
```

Avoid putting large amounts of implementation logic directly into a top-level playbook when that logic belongs in a role.

---

## 30. Testing Changes

When modifying a role:

1. Identify existing tests
2. Update or add tests when behavior changes
3. Test both expected and important edge-case behavior
4. Run the narrowest relevant test first
5. Run broader validation when practical

For example:

```text
Role change
    ↓
Role-specific test
    ↓
ansible-lint
    ↓
Syntax check
    ↓
Molecule/integration tests
```

---

## 31. Existing Conventions

Repository conventions take precedence over generic recommendations.

Before introducing a new pattern, inspect existing code.

If the repository consistently uses:

* A particular variable naming style
* Specific role layouts
* Specific tags
* Specific testing commands
* Specific collection versions
* Specific task patterns

follow those conventions unless they conflict with security or correctness.

---

## 32. AI Agent Behavior

The AI agent must:

* Inspect before modifying
* Understand the existing implementation
* Make minimal changes
* Reuse existing variables and roles where appropriate
* Prefer existing modules over custom shell commands
* Preserve backward compatibility where possible
* Validate changes
* Report validation results accurately
* Explain significant assumptions
* Ask for clarification when requirements are ambiguous and the wrong interpretation could cause infrastructure impact

The AI agent must not:

* Invent infrastructure requirements
* Invent credentials
* Invent inventory hosts
* Invent variable values when they affect production behavior
* Delete unrelated code
* Perform unnecessary refactoring
* Upgrade dependencies without authorization
* Disable security controls to bypass failures
* Claim tests passed when they were not run

---

## 33. Change Summary

When completing a task, provide a concise summary containing:

### Changes

What was changed.

### Files

Which files were modified or added.

### Validation

Which checks were executed and their results.

### Assumptions

Any important assumptions made.

### Potential Risks

Any deployment or operational risks introduced by the change.

Example:

```text
Changes:
- Added nginx configuration template
- Added configurable listen port
- Added handler for nginx reload

Files:
- roles/nginx/defaults/main.yml
- roles/nginx/tasks/main.yml
- roles/nginx/templates/nginx.conf.j2
- roles/nginx/handlers/main.yml

Validation:
- ansible-lint: passed
- ansible-playbook --syntax-check: passed
- molecule test: passed

Assumptions:
- nginx is managed by systemd

Potential Risks:
- Changing the listen port may require firewall updates
```

---

## 34. Definition of Done

An Ansible change is considered complete only when:

* The requested behavior is implemented
* The implementation follows repository conventions
* Tasks are idempotent where practical
* Security has been considered
* Secrets are not exposed
* Appropriate modules are used
* Handlers are used for service reload/restart behavior where appropriate
* Relevant tests or validation have been run
* No unrelated files have been changed unnecessarily
* The final changes are clearly summarized
