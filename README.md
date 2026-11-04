# Tuxbull Ansible Role Template

A reusable scaffold for building **OS-aware Ansible roles** with deterministic
task selection. It is based on many years of productive enterprise operations,
team implementation work, role reviews, and recurring maintenance lessons.

The template replaces repeated per-role OS-detection, path-selection, and
conditional-include boilerplate with one documented dispatcher contract. That
reduces duplicate code, gives reviewers one familiar structure, improves the
overview of platform-specific behavior, and makes a new role easier to enter,
review, test, and maintain.

It is a **starting point for new roles**, not a service role. Copy it into a new
role repository, replace demo content, and keep the dispatcher unless you intend
to change the contract for every derived role.

Dispatcher logic upgrades are deliberately centralized in exactly two files:

```text
tasks/main.yml
tasks/include-file.yml
```

Align those two files with the current template when upgrading derived-role
logic; do not copy OS-selection changes into each numbered task file.

## What this template solves

Without a common pattern, roles often accumulate duplicated checks such as:

```yaml
when: ansible_os_family == 'RedHat'
when: ansible_distribution_major_version | int == 9
```

spread through otherwise unrelated tasks. That makes reviews harder and causes
drift between roles. This template centralizes platform selection:

- one logical numbered task step has one selected implementation per host;
- most-specific OS/version directories override broader fallbacks;
- generic work lives in `tasks/shared/`;
- RHEL-compatible behavior is shared through `tasks/rhelAll/`;
- task ordering is visible from two-digit filename prefixes;
- reviewers can compare the same directory layout across roles.

## Important limitation

The dispatcher selects static files discovered from the role task tree. For a
logical basename, it chooses the first matching platform implementation.

> **`tasks/main.yml` cannot conditionally include arbitrary task files.**

Do not add `include_tasks` with `when:` conditions to `tasks/main.yml` in an
attempt to make the dispatcher additive. Use the folder-selection matrix below
instead. If independent steps must both run, give them different basenames. If
a feature is optional but valid on every supported platform, use a documented
variable inside a selected task file.

This limitation is deliberate: it keeps execution deterministic and prevents
implicit interactions between a shared fallback and an OS-specific replacement.

## Start a role

1. Create a repository named `ansible.<role_name>` from this template.
2. Update `meta/main.yml` for the final Tuxbull Galaxy identity, description,
   supported platforms, and tags.
3. Update `tests/test.yml` to use `tuxbull.<role_name>`.
4. Replace demo variables in `defaults/main.yml` and `vars/main.yml`.
5. Replace or remove all demo numbered task files.
6. Replace `handlers/main.yml` with real handlers or an intentionally empty
   handler file.
7. Place static files in `files/` and Jinja templates in `templates/`.
8. Rewrite the role README with its actual purpose, variables, OS support,
   upgrade/rollback considerations, and validation commands.
9. Run the installed-role harness in [tests/README.md](tests/README.md).

Keep `.gitstay` only for empty platform directories that must remain in Git.

## Dispatcher contract

These files define the contract:

```text
tasks/main.yml
tasks/include-file.yml
```

`tasks/main.yml` discovers all numbered task basenames, sorts them, and sends
each one to `tasks/include-file.yml`. The include file selects the first existing
implementation in this order:

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

`rhelAll` covers AlmaLinux, Rocky Linux, and Red Hat Enterprise Linux. The
`shared` directory is the global fallback.

### Example selection

For `20_install.yml` on Rocky Linux 9.3, selection is:

```text
tasks/Rocky-9.3/20_install.yml
tasks/Rocky-9/20_install.yml
tasks/Rocky/20_install.yml
tasks/rhelAll-9.3/20_install.yml
tasks/rhelAll-9/20_install.yml
tasks/rhelAll/20_install.yml
tasks/RedHat/20_install.yml
tasks/shared/20_install.yml
```

The first existing file wins.

## Placement and maintenance rules

| Location | Put here |
|---|---|
| `tasks/shared/` | Platform-neutral validation, templating, content, and generic service work. No OS selectors, package-manager commands, or OS-specific paths. |
| `tasks/rhelAll/` | Behavior common to AlmaLinux, Rocky, and RHEL. |
| `tasks/<Distribution>/` | Behavior common to all supported versions of one distribution. |
| `tasks/<Distribution>-<major>/` | Behavior specific to one distribution major release. |
| `tasks/<Distribution>-<full-version>/` | Narrow workaround for one exact version. |
| `tasks/<ansible_os_family>/` | A true OS-family fallback, such as `Debian/`. |

Use two-digit filenames to express order:

```text
00_validate.yml
10_install.yml
20_configure.yml
30_service.yml
```

### Shadowing, not inheritance

Files with the same basename are alternatives, not additive layers:

```text
tasks/shared/20_configure.yml
tasks/Ubuntu/20_configure.yml
```

An Ubuntu host runs only `tasks/Ubuntu/20_configure.yml`. If common work must
also run, give it a different basename:

```text
tasks/shared/15_common_content.yml
tasks/Ubuntu/20_configure.yml
```

## Files and directories

| Path | Purpose |
|---|---|
| `defaults/main.yml` | User-overridable variables. Document each public value. |
| `vars/main.yml` | Fixed internal maps and constants. |
| `handlers/main.yml` | Handlers notified by role tasks. |
| `files/` | Static content copied unchanged. |
| `templates/` | Jinja2 templates. |
| `tests/test.yml` | Installed-role smoke test. |
| `tests/README.md` | Harness and validation instructions. |

## Requirements

- Ansible 2.9 or later.
- Fact gathering enabled before the role runs.
- Secrets supplied through Ansible Vault or an approved secret manager; never
  commit credentials, certificates, or private keys in a role.

## Repository skill

The Tuxbull-aligned reusable workflow is versioned in this repository:

```text
skills/ansible-role-template/SKILL.md
```

It documents the dispatcher contract, migration path, task placement, basename
shadowing, installed-role validation, and the deliberate limitation on
conditional includes from `tasks/main.yml`.

## Validation

Run the harness from [tests/README.md](tests/README.md). The dispatcher is
intentionally tested as an installed role below a `roles/` directory, matching
normal local Ansible, Galaxy, AWX, and AAP use.

At minimum:

```bash
ANSIBLE_ROLES_PATH=roles ansible-playbook /path/to/ansible.template/tests/test.yml \
  -i localhost, -c local --syntax-check
ANSIBLE_ROLES_PATH=roles ansible-playbook /path/to/ansible.template/tests/test.yml \
  -i localhost, -c local
```

A real derived role also needs platform syntax coverage, live QA where
authorized, idempotency verification, and independent service/application
checks.

## License

GPL-3.0. See [LICENSE](LICENSE).
