# Changelog

## Unreleased
- Fix: `--check` with `remediate` no longer fails in the re-audit.
- Fix: reject `linux_hardening_fs_extra[].mode` values that are not 4 octal digits.
- Fix: work with `gather_facts: false` (gathers the `min` subset).
- Change: `linux_hardening_fs_findings` items no longer carry `fixable` or `new_mode`.
- Fix: in audit, an item whose check fails (e.g. unknown owner) is reported as a finding instead of aborting.
- Refactor: audit and remediation share one `ansible.builtin.file` task with a symbolic mode (audit runs it in
  check mode) instead of a Jinja bit-mask comparison; `tasks/facts.yml` + `tasks/configure.yml` become `tasks/apply.yml`.
- Task names prefixed by sub-task file; CI actions pinned by commit SHA.

## 1.0.0
- Split out of the iamenr0s.linux_hardening collection; no functional change.
