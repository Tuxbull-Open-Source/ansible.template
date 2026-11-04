# Changelog

## [1.0.0] - 2026-11-04

### Changed

- Align template metadata, smoke tests, and documentation with the `tuxbull`
  Galaxy namespace and Tuxbull-Open-Source maintenance model.
- Document this repository as the starting point for new roles and centralize
  derived-role dispatcher upgrades in `tasks/main.yml` and
  `tasks/include-file.yml`.
- Document the dispatcher contract, basename shadowing, folder-selection matrix,
  installed-role test harness, and the explicit limitation on conditional task
  includes from `tasks/main.yml`.
- Add repository maintainer guidance in `AGENTS.md`.

### Compatibility

- The dispatcher behavior is unchanged: it selects one static task file for each
  numbered basename using the documented OS/version fallback order.
