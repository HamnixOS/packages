# Hamnix packages

Repository for Hamnix's package manager (`hpm`). JSON index + plain
tarballs, served as static files via GitHub Pages at
[255.one](https://255.one/).

## Layout

```
packages/
├── index.json                # machine-readable package list (see schema)
├── index.html                # human-browsable view
├── README.md                 # this file
└── packages/
    ├── hamnix-hello-1.0.tar.gz
    └── ...
```

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
name: hamnix-hello
version: 1.0
arch: x86_64
description: Minimal example package — proves the hpm install pipeline works
maintainer: HamnixOS
license: ISC
depends:
homepage: https://255.one/
```

`depends` is a comma-separated list of `<name>` or `<name> >= <version>`
constraints. Empty value means no deps.

## index.json schema

```json
{
  "schema": 1,
  "repo": "HamnixOS/packages",
  "url": "https://255.one/",
  "updated": "<YYYY-MM-DD>",
  "description": "...",
  "packages": [
    {
      "name": "...",
      "version": "...",
      "arch": "x86_64",
      "url": "packages/<name>-<version>.tar.gz",
      "sha256": "<lowercase hex sha256 of the tarball>",
      "size": <bytes>,
      "description": "...",
      "depends": []
    }
  ]
}
```

`hpm` fetches `index.json`, picks packages by name/version, downloads
their tarballs, verifies SHA-256, extracts, runs `install.hamsh` if
present. No deb/rpm — flat tar + tiny metadata.

## Adding a package (manual, until CI lands)

1. Stage your package tree under `packages/<name>-<version>/`.
2. `cd packages && tar -czf <name>-<version>.tar.gz <name>-<version>/`
3. `rm -rf <name>-<version>/` (the staging dir; tarball stays).
4. `sha256sum packages/<name>-<version>.tar.gz` → copy into `index.json`.
5. Add the entry to `index.json`'s `packages` array.
6. Commit + push. GitHub Pages serves the update within a few minutes.

## Auto-build pipeline (planned)

Future CI: on push, parse `sources/<name>-<version>/build.hamsh`,
build the package, generate the tarball + update `index.json`. Source
upload only; CI handles packaging. Not implemented yet.

## Non-free pool (planned)

A separate `nonfree/` subtree for firmware blobs (iwlwifi etc.). Same
format; gated by an `hpm` flag the user opts into.

## Cross-refs

- [HamnixOS/Hamnix](https://github.com/HamnixOS/Hamnix) — the OS
- [HamnixOS/adder](https://github.com/HamnixOS/adder) — the language + compiler
