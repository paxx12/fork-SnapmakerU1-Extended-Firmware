---
title: Firmware Upgrade Channels
---

# Firmware Upgrade Channels

Controls how the stock `unisrv` firmware-update check behaves, via the
`upgrade.firmware` setting in `extended/extended2.cfg`.

Stock firmware periodically asks `https://id.snapmaker.com/api/device/firmware/latest`
whether a new build is available. This project can redirect that check to a
community-run mirror, or disable it outright.

## Channels

```ini
[upgrade]
firmware: snapmaker   # or: stable, beta, none
```

- **snapmaker** (default) — stock behaviour, unmodified. The device talks
  to `id.snapmaker.com` as it always has.
- **stable** — redirects the check to a Cloudflare Pages mirror tracking the
  latest tagged [GitHub release](https://github.com/paxx12/SnapmakerU1/releases)
  of this project.
- **beta** — same mirror, but tracks whichever of a release or pre-release
  is newest.
- **none** — the update check fails immediately without making any network
  request at all, so the device never contacts any update host.

## What is being sent

When `stable` or `beta` is selected, `unisrv`'s update-check request is
rewritten to the mirror with the following query parameters:

| Parameter | Source | Example |
|-----------|--------|---------|
| `channel` | the selected value (`stable`/`beta`) | `stable` |
| `version` | `/etc/VERSION` — stock firmware version | `1.5.1` |
| `build_version` | `/etc/BUILD_VERSION` — this project's `git describe` string | `0.9.0-paxx12-1-gabcdef0` |
| `build_profile` | `/etc/BUILD_PROFILE` — the build profile used | `extended` |

The device's own `Authorization: Bearer` token is stripped before the
request is sent, so no Snapmaker account credentials reach the mirror.
Nothing else from the request is forwarded — no printer serial number,
network info, or usage data.

When `none` is selected, no request is made at all: the check is failed
locally before any network I/O happens.

See [`overlays/firmware-extended/40-feature-upgrade-firmware`](https://github.com/paxx12/SnapmakerU1/tree/main/overlays/firmware-extended/40-feature-upgrade-firmware)
for the implementation.
