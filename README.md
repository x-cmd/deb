# x-cmd APT Repository

> `https://www.x-cmd.com/deb/` — a flat APT repository that hosts the official
> **x-cmd** Debian package.

This repository is published verbatim under `https://www.x-cmd.com/deb/`. The
root holds the `.deb` artifacts plus the `Packages` / `Packages.gz` indices and
a `Release` file, forming a standard *flat* APT repository consumable directly
by `apt`.

The package is built and released upstream at
[x-cmd/release](https://github.com/x-cmd/release); this repo only mirrors the
`.deb` artifacts and maintains the repository indices.

The package is architecture-independent (`Architecture: all`), so a single flat
index serves every architecture — no `dists/` / per-arch split is needed.

---

## Install (Debian / Ubuntu)

```sh
# 1. Add the repository (unsigned for now → [trusted=yes])
sudo install -m 0644 \
  <(curl -fsSL https://www.x-cmd.com/deb/x-cmd.list) \
  /etc/apt/sources.list.d/x-cmd.list

# 2. Install x-cmd
sudo apt update
sudo apt install -y x-cmd
```

Or, as a one-liner:

```sh
echo "deb [trusted=yes] https://www.x-cmd.com/deb/ ./" \
  | sudo tee /etc/apt/sources.list.d/x-cmd.list
sudo apt update && sudo apt install -y x-cmd
```

Verify:

```sh
x --version
```

---

## Repository layout

```
.
├── x-cmd_0.9.9_all.deb     # hosted package(s)
├── Packages                # package index (dpkg-scanpackages)
├── Packages.gz             # gzip-compressed index (what apt fetches)
├── Release                 # per-repo Release with checksums (apt-ftparchive)
├── x-cmd.list              # ready-to-use sources.list entry
└── README.md
```

This is a **flat** repository: the sources entry uses `./` as the suite, so
`apt` fetches `Packages`/`Release` from the repository root and follows each
`Filename:` (e.g. `./x-cmd_0.9.9_all.deb`).

---

## Adding a new release

1. Drop the new `.deb` at the repository root (optionally remove superseded
   ones). Regenerate the indices with `dpkg-scanpackages`; the `Release` file
   with `apt-ftparchive` (both need Debian tooling — run them through a
   container on macOS):

   ```sh
   docker run --rm -v "$PWD:/repo" debian:latest bash -lc '
     export DEBIAN_FRONTEND=noninteractive
     apt-get update -qq
     apt-get install -y -qq --no-install-recommends dpkg-dev apt-utils >/dev/null
     cd /repo
     dpkg-scanpackages --multiversion . /dev/null > Packages
     gzip -9c Packages > Packages.gz
     rm -f Release
     apt-ftparchive release \
       -o APT::FTPArchive::Release::Origin=x-cmd \
       -o APT::FTPArchive::Release::Label=x-cmd \
       -o APT::FTPArchive::Release::Suite=stable \
       -o APT::FTPArchive::Release::Codename=stable \
       -o APT::FTPArchive::Release::Architectures=all \
       -o APT::FTPArchive::Release::Components=main \
       -o APT::FTPArchive::Release::Description="x-cmd APT repository" \
       . > /tmp/Release && cp /tmp/Release Release'
   ```

   `Release` is emitted outside the scanned tree so it carries no
   self-referential checksum entry.

2. Commit the new `.deb`, `Packages`, `Packages.gz`, and `Release`, and push.
   The site refreshes from this branch.

---

## Notes

- Repository and packages are currently **unsigned**, hence `[trusted=yes]`.
  Switch to a signed `InRelease` (drop `[trusted=yes]`) once a GPG key is
  published.
- Build provenance and source live in
  [x-cmd/release](https://github.com/x-cmd/release).
- Companion RPM repository: [x-cmd/rpm](https://github.com/x-cmd/rpm) →
  `https://www.x-cmd.com/rpm/`.
