---
x-title: Build your own .deb and host it in an APT repo
x-desc: How to assemble a Debian package from a shell project, regenerate the Packages / Release indices, and host the result as an APT repository in the canonical pool/ + dists/ layout.
x-sidebar: Packaging
x-keywords: deb, dpkg-deb, dpkg-scanpackages, apt-ftparchive, Debian package, APT repository, pool, dists, control file
x-json-ld:
  '@context': https://schema.org
  '@graph':
    - '@type': TechArticle
      headline: 'Build your own .deb and host it in an APT repo'
      inLanguage: 'en'
      about: 'Building a Debian package and hosting an APT repository'
---

# Build your own .deb and host it in an APT repo

> A minimal end-to-end recipe: turn a directory of files into a
> signed-or-unsigned `.deb`, regenerate the APT indices with
> `dpkg-scanpackages` and `apt-ftparchive`, and serve the result
> from any static CDN. Same playbook this repo uses.

If you just want to ship shell scripts, completions, and a
manpage to Debian/Ubuntu users without going through the
official archive, this is the shortest path.

## What a `.deb` actually is

A `.deb` is an `ar` archive containing three files:

```
deb_extract/
├── control.tar.gz   # metadata: package name, deps, scripts
├── data.tar.gz      # the actual files to install
└── debian-binary    # the literal string "2.0\n"
```

You don't have to build that by hand. `dpkg-deb -b` does it.

## 1. Lay out the source tree

```
myapp-1.2.3/
├── DEBIAN/
│   ├── control       # required
│   ├── conffiles     # optional
│   ├── postinst      # optional
│   ├── prerm         # optional
│   └── ...
└── usr/
    ├── bin/myapp
    ├── share/man/man1/myapp.1
    └── share/myapp/...
```

The `DEBIAN/` directory holds metadata; the rest mirrors the
filesystem as it should look post-install.

## 2. Write `DEBIAN/control`

The single most important file. Minimum viable version:

```
Package: myapp
Version: 1.2.3
Architecture: all
Maintainer: Your Name <you@example.com>
Description: One-line short description
 One longer description (indented, any number of lines)
```

Add fields as needed: `Depends`, `Recommends`, `Section`,
`Priority`, `Homepage`. Run `man 5 deb-control` for the full
schema.

## 3. Build the `.deb`

```sh
dpkg-deb --build --root-owner=group myapp-1.2.3
# → myapp-1.2.3.deb
```

`--root-owner=group` is the modern way to set ownership without
needing root; for shell-only packages the `root:root` default
is fine.

## 4. Drop it into a `pool/`

APT's canonical layout is `pool/<component>/<first-letter>/<source-name>/<file>`:

```
repo/
├── pool/main/m/myapp/myapp_1.2.3_all.deb
├── dists/stable/Release
└── dists/stable/main/binary-all/Packages*
```

The path under `pool/` is by **source package name, not
binary**, and the leading letter is the first letter of that
name. For `myapp` → `pool/main/m/myapp/`.

## 5. Regenerate the indices

`dpkg-scanpackages` walks `pool/` and produces the per-arch
`Packages` file; `apt-ftparchive release` produces the
suite-level `Release` file with checksums of every index. Run
both inside a Debian container (macOS doesn't ship these
tools):

```sh
docker run --rm -v "$PWD:/repo" debian:latest bash -lc '
  export DEBIAN_FRONTEND=noninteractive
  apt-get update -qq
  apt-get install -y -qq --no-install-recommends dpkg-dev apt-utils >/dev/null
  cd /repo

  for a in amd64 arm64 armhf i386 all; do
    out="dists/stable/main/binary-$a"; mkdir -p "$out"
    dpkg-scanpackages --multiversion --arch "$a" pool /dev/null \
      > "$out/Packages"
    gzip -9c "$out/Packages" > "$out/Packages.gz"
  done

  rm -f dists/stable/Release
  apt-ftparchive release \
    -o APT::FTPArchive::Release::Origin=myorg \
    -o APT::FTPArchive::Release::Label=myorg \
    -o APT::FTPArchive::Release::Suite=stable \
    -o APT::FTPArchive::Release::Codename=stable \
    -o APT::FTPArchive::Release::Architectures="amd64 arm64 armhf i386 all" \
    -o APT::FTPArchive::Release::Components=main \
    -o APT::FTPArchive::Release::Description="My APT repository" \
    dists/stable > /tmp/Release && cp /tmp/Release dists/stable/Release
'
```

Three things worth noting:

- `Release` is emitted **outside** the scanned tree and copied
  in, so it carries no self-referential checksum entry.
- Pass `--multiversion` so old `.deb` files in `pool/` are
  retained in the index (lets you roll back).
- Repeat for every architecture you publish. The `all`
  architecture is a special case — it lists every
  architecture-independent package alongside the per-arch
  indices.

## 6. Serve it

`pool/` + `dists/` are pure static files. Any HTTP server will
do — GitHub Pages, Cloudflare R2, an nginx box, whatever. Users
add a `sources.list` line:

```
deb [trusted=yes] https://example.com/deb/ stable main
```

…or `InRelease` once you've set up GPG signing (see
[Security](./2-security.en.md)).

## 7. Add a `x-cmd.list` snippet

A ready-to-paste sources-list snippet in the repo root is
friendly to humans landing on your repo page:

```
# myapp APT repository  (suite: stable, component: main)
#   curl -fsSL https://example.com/deb/myapp.list \
#     | sudo tee /etc/apt/sources.list.d/myapp.list
#   sudo apt update && sudo apt install -y myapp
deb [trusted=yes] https://example.com/deb/ stable main
```

This is the exact pattern this repo uses — see
[`x-cmd.list`](../../x-cmd.list).

## When NOT to do this

- **You want the package in Debian or Ubuntu proper.** Go
  through `ftp-master.debian.org` or a sponsor instead; the
  self-hosted route is for projects that don't want that
  overhead.
- **You have many architectures of native code.** Use
  `sbuild` + a Launchpad-style pipeline; the manual recipe
  above is for the shell-only / `Architecture: all` case.
- **You need reproducibility / SBOMs.** Add
  `dpkg-buildpackage` + a CI matrix; the recipe here gives you
  a binary, not a reproducible source-to-binary path.

## Next steps

- Want to add GPG signing to the resulting `Release`?
  See [Security](./2-security.en.md).
- Want to publish a one-liner mirror alongside the repo?
  See [Stats](./3-stats.en.md) for the canonical layout this
  repo exposes.

Source: this repo's [`README.md`](../../README.md) and the
upstream Debian handbook chapter on APT repositories.
