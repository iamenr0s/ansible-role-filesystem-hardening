[![Molecule](https://github.com/iamenr0s/ansible-role-filesystem-hardening/actions/workflows/molecule.yml/badge.svg)](https://github.com/iamenr0s/ansible-role-filesystem-hardening/actions/workflows/molecule.yml) ![Ansible Role](https://img.shields.io/ansible/role/d/iamenr0s/filesystem_hardening) [![CodeFactor](https://www.codefactor.io/repository/github/iamenr0s/ansible-role-filesystem-hardening/badge)](https://www.codefactor.io/repository/github/iamenr0s/ansible-role-filesystem-hardening)

# Ansible Role: Filesystem Hardening

This role audits and hardens the ownership and permissions of core system files on Debian/Ubuntu, RHEL/AlmaLinux/RockyLinux and Fedora servers. Every item has a *maximum* mode: a stricter mode is kept, so remediation only removes permission bits and never loosens anything.

Features
--------
- Audit mode by default: read-only, and reports every deviation. Changes need `linux_hardening_confirm_remediation: true`.
- Never adds permission bits; a file that is already stricter than required is left alone.
- Distro-aware: shadow files are `0000 root:root` on EL/Fedora and `0640 root:shadow` on Debian/Ubuntu.
- Only touches files that exist; missing paths and symlinks are skipped, nothing is created.
- Extend or trim the list with `linux_hardening_fs_extra` / `linux_hardening_fs_exclude`.
- The only role in the `iamenr0s.linux_hardening` collection that owns core system file permissions; mount options belong to `mount_hardening`.

## What it checks
Our own selection (not a verified CIS mapping):

| Path | Maximum mode | Owner |
|---|---|---|
| `/etc/passwd`, `/etc/group` and their `-` backups | `0644` | `root:root` |
| `/etc/shadow`, `/etc/gshadow` and their `-` backups | `0000` (EL/Fedora), `0640` (Debian/Ubuntu) | `root:root` / `root:shadow` |
| `/etc/issue`, `/etc/issue.net` | `0644` | `root:root` |
| `/etc/crontab` | `0600` | `root:root` |
| `/etc/cron.{hourly,daily,weekly,monthly,d}` | `0700` | `root:root` |

Exact list: `vars/main.yml`. Add items with `linux_hardening_fs_extra` (`path, mode, owner, group`), skip paths with `linux_hardening_fs_exclude`.

## Variable names
Variables use the collection-wide `linux_hardening_*` prefix (shared with sibling roles) rather than `filesystem_hardening_*`. Full list: `defaults/main.yml` and `meta/argument_specs.yml`.

## Modes
`audit` (default) is read-only and sets `linux_hardening_fs_findings` (`path, expected, actual, ...`). `remediate` needs `linux_hardening_confirm_remediation: true`.

## Safety
- Never adds permission bits. Ownership is set to the listed owner:group, which can break a site that deliberately runs a daemon as another owner of one of these files (exclude the path).
- Only files that exist are touched; nothing is created.

## Example
```yaml
- hosts: all
  become: true
  roles:
    - role: iamenr0s.filesystem_hardening
      vars:
        linux_hardening_mode: remediate
        linux_hardening_confirm_remediation: true
        linux_hardening_fs_exclude: [/etc/issue]
```

## Verification
Molecule `default` (loosened files fixed, a stricter file kept, wrong owner fixed, idempotence, re-audit), `audit` (reports, changes nothing), `refuse`. Tested: AlmaLinux 9 and Debian 13. CI matrix: AlmaLinux 8–10, Rocky 8–10, Fedora 42–44, Debian 12–13, Ubuntu 22.04/24.04.

## Rollback
No automatic rollback or backup; to revert, `chmod`/`chown` the reported files back (the audit output shows the previous values).

## Known gaps
- No sticky-bit, world-writable-file, unowned-file or SUID/SGID scans (they need a filesystem walk and are not fixed safely automatically).
- `sshd_config`, bootloader and sudoers permissions belong to their own roles. CIS IDs not mapped.

CI & Release (maintainers)
----------------------------
A single workflow (`.github/workflows/molecule.yml`) runs lint and the full Molecule distro matrix on pushes to `main`, PRs, and `v*` tags. On `v*` tags, a `release` job publishes to Ansible Galaxy after all tests pass.

One-time setup:

```bash
# Galaxy publishing key (environment-scoped, get it from galaxy.ansible.com/ui/token)
gh secret set GALAXY_API_KEY --repo iamenr0s/ansible-role-filesystem-hardening

# Code scanning notifications (Slack webhook URL; for Discord append /slack to the webhook URL)
gh secret set SECURITY_ALERT_WEBHOOK --repo iamenr0s/ansible-role-filesystem-hardening
```

`.github/workflows/code-scanning-notify.yml` polls the code-scanning API every 6 hours and posts new or updated open alerts to that webhook (GitHub Actions cannot trigger on `code_scanning_alert` directly).

To release: tag a commit `vX.Y.Z` and push the tag — CI gates the Galaxy publish.

## Contributing

Contributions are welcome — see [CONTRIBUTING.md](CONTRIBUTING.md) for the
local pipeline commands and pull request checklist. This project follows the
[Contributor Covenant Code of Conduct](CODE_OF_CONDUCT.md).

## Security

See [SECURITY.md](SECURITY.md) — GitHub private vulnerability reporting, no
public issues for security bugs.

## Changelog

See [CHANGELOG.md](CHANGELOG.md) for version history and release notes.

## License

This project is licensed under the [MIT License](LICENSE).

## Author Information

Author: iamenr0s

Galaxy: `iamenr0s.filesystem_hardening`
