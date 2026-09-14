---
x-title: Repository stats and what the repo publishes
x-desc: The exact set of files, sizes, and checksums this APT repo publishes at any given moment, plus the suite/component/architecture matrix and the GitHub-stars proxy for popularity.
x-sidebar: Stats
x-keywords: APT, stats, package version, architecture, checksums, Release, Packages, binary-all, binary-amd64, binary-arm64, binary-armhf, binary-i386
x-json-ld:
  '@context': https://schema.org
  '@graph':
    - '@type': TechArticle
      headline: 'Repository stats and what the repo publishes'
      inLanguage: 'en'
      about: 'Repository metadata, sizes, and checksums'
---

# Repository stats and what the repo publishes

> The canonical, exact-pinned snapshot of what's in this repo
> right now. Sizes and checksums are read straight from the
> committed files; if they change, this page changes with them.

## Suite / component / architecture matrix

| Suite | Component | Architectures | Source letter | Origin / Label |
| --- | --- | --- | --- | --- |
| `stable` | `main` | `amd64`, `arm64`, `armhf`, `i386`, `all` | `x` | x-cmd / x-cmd |

A single package (`x-cmd`) is indexed once per architecture
because it's `Architecture: all`. The same payload shows up
under every `binary-<arch>/Packages*` index.

## Package snapshot

| Field | Value |
| --- | --- |
| Package | `x-cmd` |
| Version | `0.9.9` |
| Architecture | `all` |
| Maintainer | Li Junhao <l@x-cmd.com> |
| Filename | `pool/main/x/x-cmd/x-cmd_0.9.9_all.deb` |
| Size | 4,360,540 bytes (~4.16 MB) |
| MD5 | `ad1fc276deb13675071ab29ac1f8e3d6` |
| SHA-1 | `d8fedbd604ce48ac24dae3acea9987cc8081cee1` |
| SHA-256 | `0e0e24f1d67bf69a3f748f8b1c75e091381292a81ebe87b4f2290859dc40a3a9` |
| Homepage | <https://www.x-cmd.com> |
| Description | Ultimate Integration of POSIX SHELL AWK. A super weapon built for the super engineer |

## Index files (`dists/stable/`)

Every file here carries an MD5 / SHA-1 / SHA-256 / SHA-512
checksum in `dists/stable/Release`. Sizes are bytes.

| File | Size |
| --- | --- |
| `dists/stable/Release` | 4,735 |
| `dists/stable/main/binary-all/Packages` | 438 |
| `dists/stable/main/binary-all/Packages.gz` | 354 |
| `dists/stable/main/binary-amd64/Packages` | 438 |
| `dists/stable/main/binary-amd64/Packages.gz` | 354 |
| `dists/stable/main/binary-arm64/Packages` | 438 |
| `dists/stable/main/binary-arm64/Packages.gz` | 354 |
| `dists/stable/main/binary-armhf/Packages` | 438 |
| `dists/stable/main/binary-armhf/Packages.gz` | 354 |
| `dists/stable/main/binary-i386/Packages` | 438 |
| `dists/stable/main/binary-i386/Packages.gz` | 354 |

The five per-architecture `Packages` files are byte-identical
because `x-cmd` is `Architecture: all` — APT fetches only the
one matching the host arch but they're all available if a
mirror wants to fetch all of them.

## Payload (`pool/`)

| File | Size |
| --- | --- |
| `pool/main/x/x-cmd/x-cmd_0.9.9_all.deb` | 4,360,540 |

One binary, mirrored to all architectures.

## Repository metadata (`dists/stable/Release` excerpt)

```
Origin: x-cmd
Label: x-cmd
Suite: stable
Codename: stable
Components: main
Architectures: amd64 arm64 armhf i386 all
Description: x-cmd APT repository
Date: Mon, 22 Jun 2026 20:52:23 +0000
```

## Popularity (proxy)

This repo itself doesn't track download counts (it's a static
mirror with no analytics layer). For a proxy of adoption, see
the upstream `x-cmd` project:

| Channel | URL |
| --- | --- |
| GitHub stars | <https://github.com/x-cmd/x-cmd> |
| GitHub releases | <https://github.com/x-cmd/release> |
| Companion RPM repo | <https://github.com/x-cmd/rpm> |
| Main site | <https://www.x-cmd.com> |

## Update cadence

- The repo is published when a new x-cmd release ships
  upstream.
- The CDN mirror refreshes daily from this GitHub repo.
- No CI rebuild on a fixed schedule — there's nothing to
  rebuild unless a release lands.

## How to fetch the raw values

```sh
# Suite metadata + all checksums
curl -fsSL https://www.x-cmd.com/deb/dists/stable/Release

# Per-architecture package index
curl -fsSL https://www.x-cmd.com/deb/dists/stable/main/binary-amd64/Packages

# The package itself
curl -fsSLO https://www.x-cmd.com/deb/pool/main/x/x-cmd/x-cmd_0.9.9_all.deb
```

All three are static files served from the same CDN as the
landing page.

## Next steps

- Want to know how those files get generated? See
  [Packaging](./1-packaging.md).
- Want to know what the security model promises today? See
  [Security](./2-security.md).

Source: [`dists/stable/Release`](../../dists/stable/Release),
[`dists/stable/main/binary-amd64/Packages`](../../dists/stable/main/binary-amd64/Packages),
and [`pool/main/x/x-cmd/x-cmd_0.9.9_all.deb`](../../pool/main/x/x-cmd/x-cmd_0.9.9_all.deb)
as of the commit pinned in `git log`.
