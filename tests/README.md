# Test harness

Test the template as an installed role. The dispatcher deliberately discovers
the role task directory through normal local, Galaxy, AWX, and AAP role paths;
a direct checkout invocation is not adequate evidence.

## Local harness

Use a real directory copy rather than a symlink. Some Ansible installations
resolve a symlink target basename, which can make the dispatcher search for the
wrong role directory.

```bash
HARNESS=$(mktemp -d)
mkdir -p "$HARNESS/roles"
cp -a /path/to/ansible.template "$HARNESS/roles/tuxbull.template"
cd "$HARNESS"

ANSIBLE_ROLES_PATH=roles ansible-playbook \
  /path/to/ansible.template/tests/test.yml \
  -i localhost, -c local --syntax-check

ANSIBLE_ROLES_PATH=roles ansible-playbook \
  /path/to/ansible.template/tests/test.yml \
  -i localhost, -c local
```

Expected result for the unmodified template:

```text
failed=0
```

Warnings about unavailable AWX/system role paths are acceptable if the
`tuxbull.template` harness directory is found and the play ends with `failed=0`.

## Derived roles

After copying the template:

1. Replace `tuxbull.template` in `tests/test.yml` with the final Galaxy role
   identity.
2. Install the derived role beneath `roles/<role-basename>/` in the harness.
3. Add tests for every supported operating-system family/version path.
4. Verify that files in `tasks/shared/` contain no OS selectors or
   platform-specific paths.
5. Run a live, authorized test and a second idempotency converge for real roles.
6. Independently verify the service/application behavior; an Ansible recap is
   not sufficient proof.

## Dispatcher model

The dispatcher selects one static implementation per logical numbered basename.
Use distinct basenames for independent work, and use the same basename only for
intentional OS-specific replacement. Keep `tasks/main.yml` and
`tasks/include-file.yml` unchanged in derived roles; dispatcher changes require
a reviewed template PR first.
