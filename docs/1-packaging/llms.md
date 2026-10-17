---
name: 1-packaging
description: End-to-end recipe for building a Debian package and hosting an APT repository in the canonical pool/ + dists/ layout. Includes dpkg-deb, dpkg-scanpackages, apt-ftparchive commands.
type: tutorial
---

# Core Content

core_steps:

  - name: layout-source
    description: |
      Lay out a directory that mirrors the post-install filesystem
      plus a `DEBIAN/` folder for metadata (`control` is required;
      `postinst`, `prerm`, `conffiles` optional).

  - name: write-control
    description: |
      Write `DEBIAN/control` with Package, Version, Architecture,
      Maintainer, Description. Add Depends/Recommends/Section/Priority
      as needed. See `man 5 deb-control`.

  - name: build-deb
    description: Run `dpkg-deb --build` to produce a `.deb` ar archive.
    command: dpkg-deb --build --root-owner=group myapp-1.2.3

  - name: place-in-pool
    description: |
      Move the .deb into `pool/<component>/<first-letter>/<source-name>/`.
      Example for `myapp`: `pool/main/m/myapp/myapp_1.2.3_all.deb`.

  - name: regenerate-indices
    description: |
      Inside a Debian container, run `dpkg-scanpackages` for each
      architecture and `apt-ftparchive release` to write `Release`.
    command: |
      docker run --rm -v "$PWD:/repo" debian:latest bash -lc '
        export DEBIAN_FRONTEND=noninteractive
        apt-get update -qq
        apt-get install -y -qq --no-install-recommends dpkg-dev apt-utils >/dev/null
        cd /repo
        for a in amd64 arm64 armhf i386 all; do
          out="dists/stable/main/binary-$a"; mkdir -p "$out"
          dpkg-scanpackages --multiversion --arch "$a" pool /dev/null > "$out/Packages"
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

  - name: serve-static
    description: |
      `pool/` + `dists/` are pure static files. Serve from any
      HTTP server (GitHub Pages, Cloudflare R2, nginx).

  - name: add-sources-snippet
    description: |
      Add a `myapp.list` snippet at the repo root so humans can
      copy-paste it into `/etc/apt/sources.list.d/`.

## Key Information

package_layout:

  pool_path_template: "pool/<component>/<first-letter>/<source-name>/<filename>"
  example_pool_path: "pool/main/m/myapp/myapp_1.2.3_all.deb"
  example_dists_path: "dists/stable/main/binary-all/Packages"

deb_format:

  type: ar archive
  contents:
    - control.tar.gz   # metadata
    - data.tar.gz      # payload
    - debian-binary    # literal "2.0\n"

required_tools:

  - dpkg-deb           # build
  - dpkg-dev           # dpkg-scanpackages
  - apt-utils          # apt-ftparchive
  note: |
    None of these ship on macOS by default. Run them inside a
    `debian:latest` container, mounted with the repo as `/repo`.

## Use Cases

use_cases:

  - Ship a shell-only CLI to Debian/Ubuntu users without going
    through `ftp-master.debian.org`
  - Internal corporate package distribution
  - Pre-release / beta channels for a project
  - Hosting multiple versions side-by-side (`--multiversion`)
    for rollback

## Anti-patterns

avoid_when:

  - You want the package IN Debian/Ubuntu official — go through
    `ftp-master.debian.org` or find a sponsor.
  - You ship native code for many architectures — use `sbuild`
    + Launchpad-style pipelines instead.
  - You need reproducibility / SBOMs — add `dpkg-buildpackage`
    + a CI matrix.

## Related Resources

related:

  man_deb_control: "man 5 deb-control"
  man_dpkg_deb: "man 1 dpkg-deb"
  debian_handbook_chapter: "https://www.debian.org/doc/manuals/developers-reference/ch05.en.html"
  reference_repo: "https://github.com/x-cmd/deb"

## Summary

End-to-end recipe: build a `.deb` with `dpkg-deb --build`,
place it in `pool/<component>/<first-letter>/<source>/`,
regenerate `Packages` per arch with `dpkg-scanpackages` and
the suite-level `Release` with `apt-ftparchive release`,
serve the tree from any static HTTP server. The same playbook
this repo (`x-cmd/deb`) uses to publish itself.
