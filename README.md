# Hamnix packages

Repository for Hamnix's package manager (`hpm`). JSON index + plain
tarballs, served as static files via GitHub Pages at
[255.one](https://255.one/).

## Layout

Top-level directories under the repo root are **channels** mirroring
Debian's `main` / `contrib` / `non-free` / `non-free-firmware` split.
Each channel has its own `index.json` listing the tarballs under that
channel's `packages/` subdir.

```
255.one/
├── index.html                       # human-browsable view (channel index)
├── README.md                        # this file
├── main/                            # default channel — first-party / free
│   ├── index.json                   # machine-readable package list
│   └── packages/
│       ├── hamnix-base-1.0.0.tar.gz
│       ├── hamnix-init-1.0.0.tar.gz
│       └── ...
├── non-free/                        # opt-in: non-DFSG-free software
│   └── index.json                   # placeholder ({"packages":[]})
└── non-free-firmware/               # opt-in: binary firmware blobs
    └── index.json                   # placeholder
```

`hpm` reads `/etc/hpm/channels` to decide which channels to fetch on
`hpm refresh`, then merges every channel's index into one local
database. Each package entry carries a `"channel"` field so the
install path fetches the tarball from the right channel directory.

```
hpm channels                         # list enabled channels
hpm enable non-free-firmware         # subscribe to a channel
hpm disable non-free-firmware        # unsubscribe
hpm refresh                          # fetch every enabled channel
hpm install hamnix-base              # install (resolves deps across channels)
```

Default install subscribes to `main` only.

## Package format

A package is a gzipped tar containing a single top-level directory
`<name>-<version>/`. Inside:

| File             | Required | Purpose |
|------------------|----------|---------|
| `PKGINFO`        | yes      | Key:value metadata. Required keys: `name`, `version`, `arch`, `description`. Optional: `depends`, `maintainer`, `license`, `homepage`. |
| `files/`         | yes      | Tree of files to install. Path inside `files/` mirrors path on target (e.g. `files/bin/foo` lands at `/bin/foo`). |
| `install.hamsh`  | no       | Post-install hook, plain hamsh. Run after `files/` is staged. |
| `remove.hamsh`   | no       | Pre-removal hook. |

Example PKGINFO:

```
name: hamnix-init
version: 1.0.0
arch: x86_64
description: Hamnix init (PID 1 shim + /etc/rc.boot + identity)
maintainer: HamnixOS
license: ISC
depends:
homepage: https://255.one/
```

`depends` is a comma-separated list of `<name>` or `<name> >= <version>`
constraints. Empty value means no deps.

## index.json schema (per-channel)

```json
{
  "schema": 1,
  "repo": "HamnixOS/packages",
  "channel": "main",
  "url": "https://255.one/main/",
  "updated": "<YYYY-MM-DD>",
  "description": "...",
  "packages": [
    {
      "name": "...",
      "version": "...",
      "arch": "x86_64",
      "channel": "main",
      "url": "packages/<name>-<version>.tar.gz",
      "sha256": "<lowercase hex sha256 of the tarball>",
      "size": <bytes>,
      "description": "...",
      "depends": [],
      "target": "#hamnix-system"
    }
  ]
}
```

`hpm` fetches every enabled channel's `index.json`, merges them in
memory, picks packages by name/version, downloads their tarballs from
`<channel>/<url>`, verifies SHA-256, extracts, runs `install.hamsh`
if present. No deb/rpm — flat tar + tiny metadata.

## Channels

| Channel              | Status      | Description |
|----------------------|-------------|-------------|
| `main`               | active      | First-party / free software. `hamnix-base` metapackage + all components (init/hamsh/coreutils/net/sshd/hpm/fs/drivers/installer/bootloader) + `linux-debian-12`. Enabled by default on a fresh install. |
| `non-free`           | placeholder | DFSG-non-free software whose source/redistribution is restricted. Empty today. |
| `non-free-firmware`  | placeholder | Binary firmware blobs (wifi, GPU microcode, camera ISP). Empty today; future drivers ship their firmware here. |
| `contrib`            | reserved    | Free software depending on non-free components. Not auto-created. |

A future installer step can opt the user into `non-free-firmware` at
first-boot if it detects hardware that needs it (iwlwifi, ath11k,
nouveau-firmware, etc.).

## Adding a package (manual, until CI lands)

1. Stage your package tree under `<channel>/packages/<name>-<version>/`.
2. `cd <channel>/packages && tar -czf <name>-<version>.tar.gz <name>-<version>/`
3. `rm -rf <name>-<version>/` (the staging dir; tarball stays).
4. `sha256sum <channel>/packages/<name>-<version>.tar.gz` → copy into
   `<channel>/index.json`.
5. Add the entry to `<channel>/index.json`'s `packages` array (don't
   forget the `"channel": "<channel>"` field on every entry).
6. Commit + push. GitHub Pages serves the update within a few minutes.

Hamnix's own `scripts/build_packages.py` automates this for the
`main` channel; the output lands under `build/packages/main/` and
gets copied here on release.

## Cross-refs

- [HamnixOS/Hamnix](https://github.com/HamnixOS/Hamnix) — the OS
- [HamnixOS/adder](https://github.com/HamnixOS/adder) — the language + compiler
