# pihole-pod

Pi-hole DNS sinkhole running on Red Hat UBI (core-runtime) as a Podman quadlet with an Unbound recursive resolver sidecar.

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

This image carries the following deviations from [pi-hole/docker-pi-hole](https://github.com/pi-hole/docker-pi-hole). Upstream scripts are otherwise unmodified.

### Modified scripts

| Script | Change |
|--------|--------|
| `bash_functions.sh` | `start_cron()` rewritten for busybox crond (replaces Alpine crond). `fix_capabilities()` uses hardcoded `/usr/bin/pihole-FTL` path instead of `$(which pihole-FTL)` (`which` is not in the runtime image). |
| `start.sh` | `pihole updatechecker` replaced with a static file copy from build-time `/etc/pihole.versions.build`. Calls to `set_uid_gid` and `install_additional_packages` removed. `install_logrotate` call added. |
| `pihole-FTL-prestart.sh` | Forked from upstream. Added `chown root:root /etc/pihole/logrotate` to fix ownership after the `install_logrotate` startup function recreates the file. |
| `crontab.txt` | `pihole updatechecker` cron entry removed (versions are frozen at build time). |

### Removed features

| Feature | Upstream mechanism | Why removed |
|---------|-------------------|-------------|
| Update checker | `pihole updatechecker` at startup and daily cron | No git at runtime; versions captured at build time from upstream repos and the FTL binary |
| PADD dashboard | Downloaded at build time | Not included |

### Added

| Addition | Purpose |
|----------|---------|
| `install_logrotate()` in `bash_functions.sh` | Re-installs logrotate config from `/etc/.pihole` on startup to survive `/etc/pihole` volume mounts |
| Build-time version file | `/etc/pihole.versions.build` generated in builder stage (CORE/WEB from git, FTL from binary) and copied to `/etc/pihole/versions` at startup |
