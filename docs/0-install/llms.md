---
name: 0-install
description: Install x-cmd via the official APT repository on Debian and Ubuntu. One-line install, verification, and the `[trusted=yes]` rationale for the unsigned repo.
type: how-to
---

# Core Content

core_steps:

  - name: add-apt-source
    description: |
      Write `deb [trusted=yes] https://www.x-cmd.com/deb/ stable main`
      into `/etc/apt/sources.list.d/x-cmd.list` with mode 0644.
    command: |
      sudo install -m 0644 \
        <(curl -fsSL https://www.x-cmd.com/deb/x-cmd.list) \
        /etc/apt/sources.list.d/x-cmd.list

  - name: refresh-and-install
    description: Run `apt update` then `apt install -y x-cmd`.
    command: |
      sudo apt update && sudo apt install -y x-cmd

  - name: verify
    description: Confirm x-cmd is on PATH and reports the expected version.
    command: x --version
    expected_output: "0.9.9 or newer"

  - name: manual-install-alternative
    description: |
      Skip APT and install the `.deb` directly. Useful on locked-down
      CI runners or ephemeral containers that can't add sources.
    command: |
      curl -fsSLO https://www.x-cmd.com/deb/pool/main/x/x-cmd/x-cmd_0.9.9_all.deb
      echo "0e0e24f1d67bf69a3f748f8b1c75e091381292a81ebe87b4f2290859dc40a3a9  x-cmd_0.9.9_all.deb" | sha256sum -c -
      sudo apt install ./x-cmd_0.9.9_all.deb

## Key Information

package_metadata:

  name: x-cmd
  version: 0.9.9
  architecture: all
  maintainer: Li Junhao <l@x-cmd.com>
  homepage: https://www.x-cmd.com
  filename: pool/main/x/x-cmd/x-cmd_0.9.9_all.deb
  size_bytes: 4360540
  size_human: ~4.16 MB
  sha256: 0e0e24f1d67bf69a3f748f8b1c75e091381292a81ebe87b4f2290859dc40a3a9
  md5: ad1fc276deb13675071ab29ac1f8e3d6
  sha1: d8fedbd604ce48ac24dae3acea9987cc8081cee1

repository_metadata:

  origin: x-cmd
  label: x-cmd
  suite: stable
  codename: stable
  components: [main]
  architectures: [amd64, arm64, armhf, i386, all]
  signed: false
  signed_with: null
  trusted_option_required: "[trusted=yes]"

## Use Cases

use_cases:

  - One-line install on a fresh Debian/Ubuntu machine
  - Install on CI runners or ephemeral containers
  - Manual install via direct `.deb` fetch (no sources.list change)
  - Adding the repo to a fleet managed by Ansible / Puppet / Chef

## Common Gotchas

gotchas:

  - id: unsigned-warning
    description: |
      apt may print "The following packages cannot be authenticated"
      on first install. This is expected for an unsigned repo.
      `[trusted=yes]` in the sources.list line suppresses it for
      THIS source only.

  - id: arch-independent
    description: |
      Package is `Architecture: all` so the same artifact is served
      on every architecture. Don't try to fetch per-arch packages —
      there is only one `.deb`.

  - id: pinned-version
    description: |
      If a newer x-cmd version is published, `apt upgrade` will
      pick it up automatically once the repo is refreshed. Pin
      the version in `/etc/apt/preferences.d/x-cmd` if you need
      to freeze.

## Related Resources

related:

  repo: https://github.com/x-cmd/deb
  website: https://www.x-cmd.com/
  cdn: https://www.x-cmd.com/deb/
  manual_install: https://www.x-cmd.com/deb/pool/main/x/x-cmd/x-cmd_0.9.9_all.deb

## Summary

One-line install on Debian/Ubuntu via the official x-cmd APT repo
at `https://www.x-cmd.com/deb/`. The repo is currently unsigned,
hence `[trusted=yes]` in the sources.list line; drop the option
once GPG signing is published. The package is `Architecture: all`
(architecture-independent shell code), indexed once and served
across all architectures. Manual install via direct `.deb` fetch
is also supported for CI/containers.
