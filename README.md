# x-cmd APT Repository

> `https://www.x-cmd.com/deb/` — a standard Debian archive (APT repository)
> hosting the official **x-cmd** package.

This repository is published verbatim under `https://www.x-cmd.com/deb/` using
the canonical Debian archive layout (`pool/` + `dists/`), so it is consumable
directly by `apt` for any listed architecture.

The package is built and released upstream at
[x-cmd/release](https://github.com/x-cmd/release); this repo only mirrors the
`.deb` artifacts and maintains the repository indices. The package is
architecture-independent (`Architecture: all`), so it is indexed once and
served across all architectures.

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
echo "deb [trusted=yes] https://www.x-cmd.com/deb/ stable main" \
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
├── pool/
│   └── main/x/x-cmd/x-cmd_0.9.9_all.deb   # the hosted package(s)
├── dists/
│   └── stable/                              # suite = stable, component = main
│       ├── main/
│       │   ├── binary-amd64/Packages(.gz)
│       │   ├── binary-arm64/Packages(.gz)
│       │   ├── binary-armhf/Packages(.gz)
│       │   ├── binary-i386/Packages(.gz)
│       │   └── binary-all/Packages(.gz)
│       └── Release                          # checksums of every index file
├── x-cmd.list                               # ready-to-use sources.list entry
└── README.md
```

`apt` fetches `dists/stable/Release`, then the `binary-<arch>/Packages.gz` for
the host architecture (and `binary-all`). The `Architecture: all` package is
listed in every per-arch index, so the same repository serves `amd64`, `arm64`,
`armhf` and `i386`.

---

## Adding a new release

1. Drop the new `.deb` into `pool/main/x/x-cmd/` (optionally remove superseded
   ones). Regenerate the indices with `dpkg-scanpackages`; the `Release` file
   with `apt-ftparchive` (both need Debian tooling — run them through a
   container on macOS):

   ```sh
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
       -o APT::FTPArchive::Release::Origin=x-cmd \
       -o APT::FTPArchive::Release::Label=x-cmd \
       -o APT::FTPArchive::Release::Suite=stable \
       -o APT::FTPArchive::Release::Codename=stable \
       -o APT::FTPArchive::Release::Architectures="amd64 arm64 armhf i386 all" \
       -o APT::FTPArchive::Release::Components=main \
       -o APT::FTPArchive::Release::Description="x-cmd APT repository" \
       dists/stable > /tmp/Release && cp /tmp/Release dists/stable/Release'
   ```

   `Release` is emitted outside the scanned tree so it carries no
   self-referential checksum entry.

2. Commit the new `.deb`, the `dists/stable/main/binary-*/Packages*` files, and
   `dists/stable/Release`, then push. The site refreshes from this branch.

---

## Notes

- Repository and packages are currently **unsigned**, hence `[trusted=yes]`.
  Switch to a signed `InRelease` (drop `[trusted=yes]`) once a GPG key is
  published.
- Served architectures: `amd64`, `arm64`, `armhf`, `i386`, `all`. Add a new
  architecture by adding a `binary-<arch>/` index and listing it in the
  `Release` `Architectures` field.
- Build provenance and source live in
  [x-cmd/release](https://github.com/x-cmd/release).
- Companion RPM repository: [x-cmd/rpm](https://github.com/x-cmd/rpm) →
  `https://www.x-cmd.com/rpm/`.
