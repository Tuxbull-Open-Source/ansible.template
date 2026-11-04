# Changelog

## [1.5.1] - 2026-11-04

### Changed

- Align Galaxy metadata, smoke tests, documentation, and maintainer guidance
  with the `tuxbull` namespace and Tuxbull-Open-Source workflow.
- Add the repository-versioned `skills/ansible-role-template/SKILL.md`.
- Establish `tasks/main.yml` and `tasks/include-file.yml` as the protected
  dispatcher core: update them only through a reviewed template PR, then align
  derived roles from the merged template.
- Rewrite adoption and maintenance documentation around task placement,
  shadowing, installed-role validation, and the dispatcher migration path.

### Compatibility

- Dispatcher runtime behavior is unchanged: one static implementation is
  selected for each numbered basename through the existing OS/version fallback
  order.

## [1.5.0] - 2026-10-27

Published release notes:

- Added the `/workspace/ansible` roles path for Hermes Kubernetes-agent QA.
- Rewrote and aligned documentation for current operating systems.
- Began agent-assisted documentation work.

## [1.4] - 2025-01-26

Published release notes:

- Removed the `ansible_python_interpreter` handling for EL 6–8 after their
  Python support reached end of life.

## [1.3] - 2024-09-17

Published release notes:

- Added dynamic `task_dir` variables to support role nesting.

## [1.2] - 2023-05-17

Published release notes:

- Fixed `ansible.cfg` handling for Rocky Linux 9.

## [1.1] - 2022-10-18

Published release notes:

- Added the RHEL-clone distribution grouping that later evolved into the
  `rhelAll` task-dispatch family.

## [1.0] - 2022-03-20

First published release.

Published compatibility notes:

- Tested with Ansible 2.11 and AWX 18.0.
