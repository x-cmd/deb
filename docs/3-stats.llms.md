---
name: 3-stats
description: Snapshot of what x-cmd/deb currently publishes — suite/component/architecture matrix, package metadata, index file sizes, and the popularity proxy (upstream GitHub stars).
type: snapshot
---

# Core Content

suite_component_arch:

  suite: stable
  component: main
  architectures:
    - amd64
    - arm64
    - armhf
    - i386
    - all
  source_letter: x
  origin: x-cmd
  label: x-cmd

package_snapshot:

  name: x-cmd
  version: 0.9.9
  architecture: all
  maintainer: Li Junhao <l@x-cmd.com>
  homepage: https://www.x-cmd.com
  description: Ultimate Integration of POSIX SHELL AWK. A super weapon built for the super engineer
  filename: pool/main/x/x-cmd/x-cmd_0.9.9_all.deb
  size_bytes: 4360540
  size_human: ~4.16 MB
  md5: ad1fc276deb13675071ab29ac1f8e3d6
  sha1: d8fedbd604ce48ac24dae3acea9987cc8081cee1
  sha256: 0e0e24f1d67bf69a3f748f8b1c75e091381292a81ebe87b4f2290859dc40a3a9

index_files:

  - path: dists/stable/Release
    size_bytes: 4735
    checksum_algorithms: [md5, sha1, sha256, sha512]
  - path: dists/stable/main/binary-all/Packages
    size_bytes: 438
  - path: dists/stable/main/binary-all/Packages.gz
    size_bytes: 354
  - path: dists/stable/main/binary-amd64/Packages
    size_bytes: 438
  - path: dists/stable/main/binary-amd64/Packages.gz
    size_bytes: 354
  - path: dists/stable/main/binary-arm64/Packages
    size_bytes: 438
  - path: dists/stable/main/binary-arm64/Packages.gz
    size_bytes: 354
  - path: dists/stable/main/binary-armhf/Packages
    size_bytes: 438
  - path: dists/stable/main/binary-armhf/Packages.gz
    size_bytes: 354
  - path: dists/stable/main/binary-i386/Packages
    size_bytes: 438
  - path: dists/stable/main/binary-i386/Packages.gz
    size_bytes: 354

payload:

  - path: pool/main/x/x-cmd/x-cmd_0.9.9_all.deb
    size_bytes: 4360540

## Key Information

release_excerpt: |
  Origin: x-cmd
  Label: x-cmd
  Suite: stable
  Codename: stable
  Components: main
  Architectures: amd64 arm64 armhf i386 all
  Description: x-cmd APT repository

popularity_proxy:

  - github_stars: https://github.com/x-cmd/x-cmd
  - github_releases: https://github.com/x-cmd/release
  - companion_rpm: https://github.com/x-cmd/rpm
  - main_site: https://www.x-cmd.com

  note: |
    This repo itself doesn't track download counts (static
    mirror, no analytics). The upstream x-cmd project is the
    adoption proxy.

update_cadence:

  trigger: new x-cmd release ships upstream
  cdn_refresh: daily
  rebuild_schedule: none (event-driven only)

## Use Cases

use_cases:

  - Auditing the current repo state before installing
  - Diffing two snapshots to see what changed between releases
  - Sanity-checking that a mirror matches the source repo
  - Confirming checksums before a manual install

## Common Checks

commands:

  fetch_release:
    description: Get suite-level metadata + all checksums.
    command: curl -fsSL https://www.x-cmd.com/deb/dists/stable/Release

  fetch_packages_amd64:
    description: Get the per-arch Packages index.
    command: curl -fsSL https://www.x-cmd.com/deb/dists/stable/main/binary-amd64/Packages

  fetch_deb:
    description: Download the .deb.
    command: curl -fsSLO https://www.x-cmd.com/deb/pool/main/x/x-cmd/x-cmd_0.9.9_all.deb

  verify_sha256:
    description: Verify the .deb against an expected hash.
    command: |
      echo "0e0e24f1d67bf69a3f748f8b1c75e091381292a81ebe87b4f2290859dc40a3a9  x-cmd_0.9.9_all.deb" | sha256sum -c -

## Related Resources

related:

  upstream_repo: https://github.com/x-cmd/deb
  cdn_root: https://www.x-cmd.com/deb/

## Summary

Snapshot of what `x-cmd/deb` publishes today: one suite
(`stable`), one component (`main`), five architecture indices
(`amd64`, `arm64`, `armhf`, `i386`, `all`) which are
byte-identical because `x-cmd` is `Architecture: all`. The
single package is `x-cmd 0.9.9`, ~4.16 MB. Update cadence is
event-driven (a release lands) plus a daily CDN sync from
this GitHub repo; no fixed CI rebuild schedule.
