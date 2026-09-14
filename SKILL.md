---
name: deb
description: The x-cmd APT (Debian) repository — pure APT data on GitHub, mirrored verbatim to https://www.x-cmd.com/deb/. Use when navigating this repo's structure, identifying which file to edit for a given task, or understanding the sync model.
metadata: type=apt-repository, scope=deb-stable-main, refresh=daily-mirror, signed=no
---

# x-cmd/deb — repo organization reference

> A structural map of this repository. Not a how-to-install
> guide (that's in `README.md` / `README.cn.md` / `docs/0-install.*`).
> This file is for an agent or maintainer coming back to the
> repo and asking "where do I edit X?".

## What this repo is

A canonical Debian archive (APT repository) for the x-cmd
package. The contents are pure APT data — `pool/`, `dists/`,
the `Release` file, the per-arch `Packages*` indices — plus
the human-facing metadata (`x-cmd.list`, `README*.md`, `docs/`)
that goes alongside an APT repo.

The whole tree is mirrored verbatim to
<https://www.x-cmd.com/deb/> on a daily cadence. `apt` reads
only `dists/` and `pool/`; humans see the rest.

## Top-level layout

```
.
├── pool/                       # APT payloads (the .deb files)
├── dists/                      # APT indices (Release, Packages*)
├── x-cmd.list                  # ready-to-paste sources.list snippet
├── README.md                   # English README (GitHub renders)
├── README.cn.md                # Chinese README (GitHub renders)
├── SKILL.md                    # ← this file: structural reference
├── CONTRIBUTING.md             # maintainer runbook (release, signing, incidents)
├── index.html                  # SPA shell for humans visiting x-cmd.com/deb/
└── docs/                       # user-facing bilingual docs (see below)
```

## Where to edit what

| If you want to… | Edit |
| --- | --- |
| Add a new `.deb` version | `pool/main/x/x-cmd/x-cmd_<ver>_all.deb` (drop in), then regenerate indices (see CONTRIBUTING.md) |
| Update the sources-list snippet | `x-cmd.list` |
| Refresh the per-arch `Packages` files | run the docker command in `CONTRIBUTING.md` → cuts `dists/stable/main/binary-*/Packages{,.gz}` and `dists/stable/Release` |
| Change the user-facing install doc | `docs/0-install.{md,cn.md,en.md,llms.md,faq.yml}` |
| Add a new docs topic | a new `docs/<n>-<topic>.{md,cn.md,en.md,llms.md,faq.yml}` set |
| Update English README | `README.md` |
| Update Chinese README | `README.cn.md` |
| Update the landing page | `index.html` (currently a static SPA shell) |
| Change the GPG key, signing, or `[trusted=yes]` policy | `CONTRIBUTING.md` (runbook) + `docs/2-security.*` (user-facing) |
| Fix a broken release | `CONTRIBUTING.md` → "Incident playbook" |

## Conventions

This repo mirrors the `x-cmd/cve` repo's conventions. The
short version:

- **Numbered prefix on docs topics** — `0-install`,
  `1-packaging`, `2-security`, `3-stats`. Lower numbers lead.
- **Five file variants per topic** —
  `.md` (default = English), `.cn.md` (Chinese), `.en.md`
  (explicit English), `.llms.md` (LLM-friendly structured),
  `.faq.yml` (FAQ structured).
- **YAML frontmatter** on every `.md` and `.llms.md`:
  `x-title`, `x-desc`, `x-sidebar`, `x-keywords`, `x-json-ld`
  (TechArticle schema).
- **Embedded page body starts with `#`** for top-level docs
  (the H1 is the page title; H2+ below).
- **Bilingual parity** — every English doc has a Chinese
  counterpart with the same filename stem. If you add a new
  section, add both.
- **No build step** for `docs/` — the markdown files ARE the
  deliverable. A separate doc-rendering pipeline (out of this
  repo) consumes them.

## `docs/` layout

```
docs/
├── 0-install.{md, cn.md, en.md, llms.md, faq.yml}    # how to install x-cmd
├── 1-packaging.{md, cn.md, en.md, llms.md, faq.yml}  # how to package + host APT
├── 2-security.{md, cn.md, en.md, llms.md, faq.yml}   # [trusted=yes] + signing plan
├── 3-stats.{md, cn.md, en.md, llms.md, faq.yml}      # current repo snapshot
└── assets/                                           # images, badges (reserved)
```

Each numbered topic covers one user question:

- `0` — "how do I install x-cmd?"
- `1` — "how do I host my own APT repo?"
- `2` — "is this safe to use in production?"
- `3` — "what does this repo currently publish?"

## Sync model

```
github.com/x-cmd/deb  (this repo, source of truth)
        │
        │  daily mirror (read-only sync)
        ▼
https://www.x-cmd.com/deb/  (CDN — what apt + humans hit)
```

- This repo's `pool/`, `dists/`, `x-cmd.list`, `docs/`,
  `*.md`, `index.html` are mirrored verbatim.
- No build step on the CDN side — files land as committed.
- The CDN is the canonical source for `apt` users; this repo
  is the canonical source for "what we promised to publish."
- Diff this repo against the live mirror to diagnose stale
  content.

## Tools you might need

- `dpkg-deb` — to inspect a `.deb`'s control archive
  (`dpkg-deb -I foo.deb`).
- `dpkg-scanpackages` + `apt-ftparchive` — to regenerate
  indices. **Not on macOS by default** — use the `debian:latest`
  container block in `CONTRIBUTING.md`.
- `gpg` — only relevant once signing goes live.
- `git`, `find`, `sha256sum` — everywhere.

## When in doubt

- **The maintainer wants to cut a release** → `CONTRIBUTING.md`.
- **The user wants to install x-cmd** → `docs/0-install.md`.
- **Someone asks about signing/security** → `docs/2-security.md`.
- **Someone asks "what's in this repo"** → `docs/3-stats.md`.
- **Someone asks "how do I package my own"** → `docs/1-packaging.md`.

## Related repos

- `x-cmd/release` — upstream x-cmd build pipeline; this is
  where the `.deb` is produced before being mirrored here.
- `x-cmd/cve` — sibling repo; same `docs/` + frontmatter
  convention.
- `x-cmd/rpm` — companion RPM repository →
  `https://www.x-cmd.com/rpm/`.
- `x-cmd/explorer` — the SPA webapp intended to serve as the
  human-facing landing experience at `x-cmd.com/deb/`.
