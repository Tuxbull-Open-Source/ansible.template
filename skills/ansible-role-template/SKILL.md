---
name: ansible-role-template
description: Build OS-aware roles from a deterministic dispatcher.
version: 1.0.0
author: Chris Ruettimann (joe-speedboat), Hermes Agent
license: GPL-3.0-or-later
platforms: [linux]
metadata:
  hermes:
    tags: [Ansible, roles, templates, operating-systems, automation]
    related_skills: []
---

# Ansible Role Template

Use this skill when creating, refactoring, or reviewing reusable Ansible roles
based on `Tuxbull-Open-Source/ansible.template`. The template reflects recurring
enterprise implementation, review, and maintenance practice: one deterministic
OS-aware dispatcher reduces duplicated conditional code, improves role overview,
and gives maintainers a simpler, predictable entry point.

This is a role scaffold, not a deployable service role. Replace its demo content
when creating a real role. This skill covers source and test design; it does not
authorize managed-target changes.

## When to Use

- Create a new Tuxbull Ansible role from this repository.
- Refactor a role that needs OS/version-specific task selection.
- Review a role using `tasks/main.yml` and `tasks/include-file.yml`.
- Diagnose task-basename shadowing or platform-fallback behavior.

Do not use for one-off playbooks without reusable role structure, or to apply a
role to managed targets without the applicable authorization and change-safety
workflow.

## Template Contract

The template is the starting point for new roles. Dispatcher upgrades for
derived roles are centralized in exactly:

```text
tasks/main.yml
tasks/include-file.yml
```

Align those files with the current template when intentionally upgrading dispatch
behavior; do not duplicate dispatcher changes throughout numbered task files.

The dispatcher discovers numbered task basenames, sorts them, and selects one
implementation per basename. Its fallback order is:

```text
tasks/<distribution>-<full-version>/<task>.yml
tasks/<distribution>-<major-version>/<task>.yml
tasks/<distribution>/<task>.yml
tasks/rhelAll-<full-version>/<task>.yml
tasks/rhelAll-<major-version>/<task>.yml
tasks/rhelAll/<task>.yml
tasks/<ansible_os_family>/<task>.yml
tasks/shared/<task>.yml
```

`rhelAll` covers AlmaLinux, Rocky Linux, and Red Hat Enterprise Linux.
`tasks/shared/` is the global fallback.

## Deliberate Limitation

`tasks/main.yml` cannot conditionally include arbitrary task files. It discovers
static numbered basenames, then `tasks/include-file.yml` selects one first-match
implementation per basename.

Do not add conditional `include_tasks` calls to `tasks/main.yml` to make the
dispatcher additive. Use distinct basenames for independent work. Use the same
basename across folders only for deliberate OS-specific replacement.

## Task Placement

| Location | Use for |
|---|---|
| `tasks/shared/` | Platform-neutral validation, templating, content, or generic service work. |
| `tasks/rhelAll/` | Behavior common to AlmaLinux, Rocky, and RHEL. |
| `tasks/<Distribution>/` | Behavior common to one distribution. |
| `tasks/<Distribution>-<major>/` | One distribution major release. |
| `tasks/<Distribution>-<full-version>/` | A narrow exact-version workaround. |
| `tasks/<ansible_os_family>/` | A genuine OS-family fallback such as Debian. |

`tasks/shared/` must not contain OS selectors, package-manager modules,
distribution names, or platform-specific paths.

Use two-digit ordered basenames:

```text
00_validate.yml
10_install.yml
20_configure.yml
30_service.yml
```

## Shadowing Rule

Files with the same basename are alternatives, not additive layers:

```text
tasks/shared/20_configure.yml
tasks/Ubuntu/20_configure.yml
```

Ubuntu runs only `tasks/Ubuntu/20_configure.yml`. If platform-neutral work must
also run, assign it a different basename:

```text
tasks/shared/15_common_content.yml
tasks/Ubuntu/20_configure.yml
```

## Procedure

1. Inspect `README.md`, `AGENTS.md`, `tasks/main.yml`, and
   `tasks/include-file.yml`. Completion: dispatcher contract and repository
   identity are known.
2. Map each intended task to shared, RHEL-compatible, distribution, major, or
   exact-version scope. Completion: every task has one intentional location.
3. Use FQCN Ansible modules, stable permissions, and independent controls for
   package, configuration, and service lifecycle behavior. Completion: no secret
   or platform selector enters a shared task file.
4. Update metadata, role tests, documentation, and examples to the final
   `tuxbull.<role_name>` identity. Completion: no legacy namespace remains.
5. Test as an installed role under a real `roles/<role-name>/` directory;
   symlinks can resolve to an unexpected basename. Completion: syntax and smoke
   harnesses pass.
6. Run YAML parsing, `git diff --check`, and a focused secret scan. Completion:
   every changed file is accounted for and no sensitive value is committed.

## Validation

```bash
python3 - <<'PY'
from pathlib import Path
import yaml
for path in Path('.').rglob('*.yml'):
    list(yaml.safe_load_all(path.read_text()))
print('YAML parse passed')
PY

ANSIBLE_ROLES_PATH=roles ansible-playbook tests/test.yml \
  -i localhost, -c local --syntax-check
ANSIBLE_ROLES_PATH=roles ansible-playbook tests/test.yml \
  -i localhost, -c local
git diff --check
```

A real derived role also needs authorized live QA, independent service checks,
and a second idempotency converge.

## Pitfalls

- Do not expect shared and OS-specific files with the same basename to both run.
- Do not use OS conditions in `tasks/shared/`; split the task by platform.
- Do not test only a checkout path; test a copied installed-role harness.
- Do not assume syntax success proves package, service, or application behavior.
- Do not put credentials, certificates, private keys, or generated test output
  in the role repository.
