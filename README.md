# Tuxbull Ansible Role Template

The starting point for new OS-aware Ansible roles. It captures recurring
enterprise implementation, team-review, and maintenance practice in one
predictable structure: reduce duplicated platform checks, make task selection
obvious, and give the next maintainer a simple way into the role.

## The protected dispatcher core

These are the soul of this template:

```text
tasks/main.yml
tasks/include-file.yml
```

**Do not change either file in a derived role.** Any dispatcher correction or
new dispatch capability must be proposed, reviewed, and merged as a PR against
`Tuxbull-Open-Source/ansible.template` first. Derived roles then align these two
files with the merged template version.

Keeping that rule gives every role the same well-tested selection behavior. It
also means reviewers can focus on the role's actual task files rather than
re-auditing a private dispatcher variant in every repository.

## Why use this template

Without a common layout, roles tend to repeat distribution checks and path
selection inside unrelated tasks. That makes code harder to review and creates
drift between roles.

This template provides:

- numbered task steps, so execution order is visible;
- one OS-aware selection rule for each task step;
- shared RHEL-family behavior through `rhelAll`;
- a clear location for generic versus platform-specific logic;
- a consistent role layout across the Tuxbull namespace;
- an installed-role test harness that resembles Galaxy, AWX, and AAP use.

## Create a role

1. Copy this repository into `ansible.<role_name>`.
2. Set the final Galaxy identity in `meta/main.yml`:

   ```yaml
   namespace: tuxbull
   role_name: <role_name>
   ```

3. Change `tests/test.yml` to use `tuxbull.<role_name>`.
4. Replace demo variables, task files, handlers, and documentation with the
   real role behavior.
5. Keep the two dispatcher-core files unchanged.
6. Run the installed-role harness in [tests/README.md](tests/README.md).

Use `.gitstay` only for intentionally empty platform folders.

## How task selection works

The dispatcher finds every numbered task basename, sorts the names, and selects
one implementation for each basename using this order:

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

`rhelAll` applies to AlmaLinux, Rocky Linux, and Red Hat Enterprise Linux.
`tasks/shared/` is the fallback for platform-neutral work.

For example, a Rocky 9.3 host selecting `20_install.yml` checks:

```text
Rocky-9.3 → Rocky-9 → Rocky → rhelAll-9.3 → rhelAll-9 → rhelAll → RedHat → shared
```

The first existing file is the selected implementation.

## Put tasks in the right place

| Location | Use it for |
|---|---|
| `tasks/shared/` | Work valid on every supported platform: validation, generic templating, content, or service operations. |
| `tasks/rhelAll/` | Behavior common to AlmaLinux, Rocky, and RHEL. |
| `tasks/<Distribution>/` | Behavior common to every supported version of one distribution. |
| `tasks/<Distribution>-<major>/` | One distribution major version. |
| `tasks/<Distribution>-<full-version>/` | A narrow exact-version workaround. |
| `tasks/<ansible_os_family>/` | A genuine OS-family fallback, for example `Debian/`. |

Keep `tasks/shared/` free of distribution checks, package-manager modules, and
platform-specific paths. Put platform behavior in a platform directory instead.

Use two-digit prefixes for order:

```text
00_validate.yml
10_install.yml
20_configure.yml
30_service.yml
```

## Model common and platform-specific work

A basename is one task step, not a stack of files. If both of these exist:

```text
tasks/shared/20_configure.yml
tasks/Ubuntu/20_configure.yml
```

Ubuntu uses its platform-specific `20_configure.yml`. The shared file is the
fallback, not an additional layer.

When both common and platform-specific work are required, use two steps:

```text
tasks/shared/15_common_content.yml
tasks/Ubuntu/20_configure.yml
```

This keeps the execution plan visible from filenames and avoids hidden ordering
between generic and OS-specific work.

## Repository layout

| Path | Purpose |
|---|---|
| `defaults/main.yml` | User-overridable variables; document every public variable. |
| `vars/main.yml` | Fixed internal maps and constants. |
| `handlers/main.yml` | Handlers notified by task files. |
| `files/` | Static files copied unchanged. |
| `templates/` | Jinja2 templates. |
| `tests/test.yml` | Installed-role smoke playbook. |
| `tests/README.md` | Harness and validation instructions. |
| `skills/ansible-role-template/SKILL.md` | Reusable workflow for agents and maintainers. |

## Requirements

- Ansible 2.9 or later.
- Fact gathering enabled before the role runs.
- Secrets supplied through Ansible Vault or an approved secret manager. Never
  commit credentials, certificates, or private keys.

## Validate the template

Use the installed-role harness in [tests/README.md](tests/README.md). A direct
checkout run does not prove that discovery works through normal role paths.

```bash
ANSIBLE_ROLES_PATH=roles ansible-playbook /path/to/ansible.template/tests/test.yml \
  -i localhost, -c local --syntax-check
ANSIBLE_ROLES_PATH=roles ansible-playbook /path/to/ansible.template/tests/test.yml \
  -i localhost, -c local
```

A real role also needs OS-path coverage, authorized live QA where appropriate,
a second idempotency converge, and independent verification of its service or
application behavior.

## License

GPL-3.0. See [LICENSE](LICENSE).
