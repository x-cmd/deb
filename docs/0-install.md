---
x-title: Install x-cmd via APT
x-desc: Add the official x-cmd APT repository on Debian or Ubuntu, install the x-cmd package, and verify with one command.
x-sidebar: Install
x-keywords: x-cmd, APT, Debian, Ubuntu, install, sources.list, trusted=yes
x-json-ld:
  '@context': https://schema.org
  '@graph':
    - '@type': TechArticle
      headline: 'Install x-cmd via APT'
      inLanguage: 'en'
      about: 'APT installation of x-cmd on Debian and Ubuntu'
---

# Install x-cmd via APT

> The official x-cmd APT repository, served at
> <https://www.x-cmd.com/deb/>. One-line install on any Debian or
> Ubuntu system — no GPG key required today (the repo is unsigned,
> hence `[trusted=yes]`).

This page walks you through adding the repository, refreshing
your package index, and installing `x-cmd`. The deep dives on
pages 1..3 explain how the `.deb` is built, how to verify the
package, and what the repo publishes in terms of metadata.

## Quick install

The fastest path — one command, no editor:

```sh
echo "deb [trusted=yes] https://www.x-cmd.com/deb/ stable main" \
  | sudo tee /etc/apt/sources.list.d/x-cmd.list
sudo apt update && sudo apt install -y x-cmd
```

Verify:

```sh
x --version
```

You should see a `0.9.9` (or newer) version string. Done.

## Step-by-step

If you'd rather see every step:

```sh
# 1. Add the repository (unsigned for now → [trusted=yes])
sudo install -m 0644 \
  <(curl -fsSL https://www.x-cmd.com/deb/x-cmd.list) \
  /etc/apt/sources.list.d/x-cmd.list

# 2. Install x-cmd
sudo apt update
sudo apt install -y x-cmd
```

`x-cmd.list` is the canonical sources-list snippet that this
repo publishes alongside the packages. Piping it through
`install -m 0644` lands it at the standard APT location with
the right permissions.

## What gets installed

| Item | Value |
| --- | --- |
| Package | `x-cmd` |
| Version | `0.9.9` (or newer) |
| Architecture | `all` (architecture-independent shell code) |
| Maintainer | Li Junhao <l@x-cmd.com> |
| Homepage | <https://www.x-cmd.com> |
| Filename | `pool/main/x/x-cmd/x-cmd_0.9.9_all.deb` |
| Size | ~4.16 MB |
| SHA-256 | `0e0e24f1d67bf69a3f748f8b1c75e091381292a81ebe87b4f2290859dc40a3a9` |

The package is `Architecture: all`, so the same artifact is
indexed once and served across every architecture APT knows about.

## Why `[trusted=yes]`

The repository is currently **unsigned** — there is no GPG key
under `etc/apt/trusted.gpg.d/` and the `Release` file carries no
`Signed-By` directive. This is a known gap (see
[Security](./2-security.md) for the plan to add an `InRelease`
signature once the key is published).

`[trusted=yes]` is APT's opt-in escape hatch for unsigned
repositories. It tells `apt` to skip the signature check for
this specific source. Drop the option — and add the key — once
signing is live.

## Verify the package manually

If you don't want to add the repo, you can also fetch and
install the `.deb` directly:

```sh
curl -fsSLO https://www.x-cmd.com/deb/pool/main/x/x-cmd/x-cmd_0.9.9_all.deb
echo "0e0e24f1d67bf69a3f748f8b1c75e091381292a81ebe87b4f2290859dc40a3a9  x-cmd_0.9.9_all.deb" | sha256sum -c -
sudo apt install ./x-cmd_0.9.9_all.deb
```

This works without modifying your sources.list at all — useful
on locked-down CI runners or ephemeral containers.

## Next steps

- Want to package your own `.deb` for distribution via APT?
  See [Packaging](./1-packaging.md).
- Want to understand what "unsigned" means in practice and
  what's coming? See [Security](./2-security.md).
- Curious about what the repo currently publishes (versions,
  checksums, architectures)? See [Stats](./3-stats.md).

Source: [`README.md`](../../README.md).
