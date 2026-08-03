# pihole-pod

Pi-hole DNS sinkhole running on RHHI `core-runtime` with an RHHI `unbound` recursive resolver sidecar as a Podman quadlet.

Upstream components pinned by release tag in the Containerfile:

- [pi-hole/pi-hole](https://github.com/pi-hole/pi-hole) (core scripts)
- [pi-hole/web](https://github.com/pi-hole/web) (admin UI)
- [pi-hole/FTL](https://github.com/pi-hole/FTL) (DNS engine)

## Install

```bash
sudo podman quadlet install pihole-quadlet

sudo systemctl daemon-reload

sudo systemctl start pihole
```

## Changes from upstream pi-hole

This image carries the following changes from [pi-hole/docker-pi-hole](https://github.com/pi-hole/docker-pi-hole) to support the migration from Alpine to  the `core-runtime` base.
Minimal changes to the upstream components are made to support the new runtime environment.
Some [additional packages are installed](https://github.com/nzwulfin/pihole-pod/blob/7aea76c602d1849a6c64c823394058ff43cb7f04/pihole/Containerfile#L87) in the final stage and some [convenience symlinks created](https://github.com/nzwulfin/pihole-pod/blob/7aea76c602d1849a6c64c823394058ff43cb7f04/pihole/Containerfile#L108).

### Build-time versioning instead of runtime checks

| Script | Change |
|--------|--------|
| `/etc/pihole.versions.build` | Added — captures CORE/WEB/FTL versions at build |
| `start.sh` | `pihole updatechecker` replaced with static copy from build-time `/etc/pihole.versions.build` |
| `crontab.txt` | `pihole updatechecker` entry removed |

### Fix log rotation 

| Script | Change |
|--------|--------|
| `bash_functions.sh` | `install_logrotate()` added — re-installs logrotate config on startup to survive `/etc/pihole` volume mounts |
| `pihole-FTL-prestart.sh`** | `chown root:root /etc/pihole/logrotate` added to fix ownership after startup reinstall |

### Changes to support `core-runtime` environment

| Script | Change |
|--------|--------|
| `bash_functions.sh` | `start_cron()` rewritten for busybox crond |
| `bash_functions.sh` | `fix_capabilities()` uses `busybox which` to locate `pihole-FTL` |
| `bash_functions.sh` | `sed` → `busybox sed` |
| `start.sh` | `set_uid_gid` and `install_additional_packages` removed; `sed` → `busybox sed`; `killall` → `busybox killall` |
| `gravity.sh`** | `sudo -u pihole test` → `test` (no privilege separation in container) |
| `pihole-FTL-prestart.sh`** | Numeric UID/GID references replaced with named `pihole` user; `find` → `busybox find` |
** upstream component

[PADD](https://github.com/pi-hole/PADD) not used, dropped from upstream Containerfile
