# Repository instructions

This repository is the canonical Tuxbull scaffold for OS-aware Ansible roles.
It provides a deterministic task dispatcher, not a deployable service role.

## Dispatcher contract

This template is the starting point for new roles. Keep its dispatcher logic
aligned with the current template through exactly two files:

```text
tasks/main.yml
tasks/include-file.yml
```

Do not duplicate dispatcher upgrades throughout numbered task files.

- Preserve `tasks/main.yml` and `tasks/include-file.yml` unless changing the
  template contract deliberately and documenting the migration impact.
- The dispatcher selects one static file per numbered basename through the
  platform fallback matrix. It is replacement/shadowing behavior, not additive
  inheritance.
- Do not add conditional `include_tasks` calls to `tasks/main.yml`. Conditional
  arbitrary includes from that dispatcher are intentionally unsupported.
- Put platform-neutral tasks in `tasks/shared/`; do not use OS selectors,
  package-manager commands, or platform-specific paths there.
- Give independent steps distinct two-digit basenames. Use the same basename
  across folders only for deliberate OS-specific replacement.

## Tuxbull migration rules

- New roles use `tuxbull.<role_name>` in Galaxy metadata, tests, README examples,
  and release documentation.
- Keep credentials, tokens, certificates, private keys, inventories, and
  generated test output out of Git.
- Treat this template as a source scaffold; replace all demo variables, tasks,
  handlers, metadata, and documentation before publishing a real role.

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

Use the installed-role harness in `tests/README.md`; a direct checkout run does
not prove dispatcher compatibility with Galaxy/AWX/AAP role paths.
