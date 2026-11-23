# Changelog

## Unreleased
- Fix: `--check` with `remediate` no longer fails in the re-audit.
- Fix: reject `linux_hardening_fs_extra[].mode` values that are not 4 octal digits.
- Fix: work with `gather_facts: false` (gathers the `min` subset).
- Change: `linux_hardening_fs_findings` items no longer carry the unused `fixable` key.
- Task names prefixed by sub-task file; CI actions pinned by commit SHA.

## 1.0.0
- Split out of the iamenr0s.linux_hardening collection; no functional change.
