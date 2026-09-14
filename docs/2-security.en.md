---
x-title: Security, signing, and the [trusted=yes] question
x-desc: Why this APT repo is unsigned today, what `[trusted=yes]` actually opts into, how to verify checksums manually, and the InRelease / GPG signing plan to remove the escape hatch.
x-sidebar: Security
x-keywords: APT, signing, GPG, InRelease, trusted=yes, sha256, supply chain, deb822, sources.list
x-json-ld:
  '@context': https://schema.org
  '@graph':
    - '@type': TechArticle
      headline: 'Security, signing, and the [trusted=yes] question'
      inLanguage: 'en'
      about: 'APT repository security, signing, and verification'
---

# Security, signing, and the `[trusted=yes]` question

> The honest, complete picture of what this repo promises today
> — and what it does not. If you're a security reviewer or a
> cautious sysadmin, this page is for you.

This repo is currently **unsigned**. The `[trusted=yes]` you
see in `x-cmd.list` is the explicit opt-in that acknowledges
that fact. We have a plan to add a real GPG signature and
remove the escape hatch; the section at the bottom tracks it.

## What you get today

| Property | Status |
| --- | --- |
| TLS at the CDN | ✅ (https://www.x-cmd.com/deb/) |
| Per-package SHA-256 in `Packages` | ✅ |
| Suite-level `Release` file with checksums of every index | ✅ |
| `Release.gpg` detached signature | ❌ |
| `InRelease` clear-signed `Release` | ❌ |
| GPG key under `/etc/apt/trusted.gpg.d/` | ❌ |
| Time-based key expiry + rotation policy | ❌ (no key yet) |

The first three rows are the integrity layer you can already
rely on. The bottom four rows are what `[trusted=yes]` is
silently disabling.

## What `[trusted=yes]` actually opts into

APT's signature check protects against two things:

1. **Network MITM**: a TLS-stripping attacker rewriting the
   `.deb` you fetch.
2. **Repository compromise**: the operator (or someone who
   takes over their account) pushing a `.deb` you didn't
   intend to install.

`[trusted=yes]` disables **both** for this specific source.
The `https://` URL still gives you TLS, so the MITM vector is
covered. The operator-trust vector is the one you are now
trusting on operator reputation alone — the same trust model
as `pip install` from PyPI without hashes, or
`curl | bash` from a random README.

For x-cmd specifically: the repo is on GitHub (you can audit
every commit), mirrored to `www.x-cmd.com` via the standard
read-only pipeline, and the `.deb` is built deterministically
from the same source as the public x-cmd release artifacts.
You can verify the SHA-256 against the GitHub release's
checksum file. That's the practical safety net today.

## How to verify without adding the repo

```sh
URL="https://www.x-cmd.com/deb/pool/main/x/x-cmd/x-cmd_0.9.9_all.deb"
EXPECTED="0e0e24f1d67bf69a3f748f8b1c75e091381292a81ebe87b4f2290859dc40a3a9"

curl -fsSLO "$URL"
echo "$EXPECTED  x-cmd_0.9.9_all.deb" | sha256sum -c -
```

`OK` means the bytes you fetched match what the repo claimed
in `dists/stable/main/binary-all/Packages`. Compare the
expected hash against a second channel (the GitHub release
page, a Twitter announcement, a friend) for full confidence.

For maximum rigor, fetch `dists/stable/Release` and verify the
SHA-256 it carries for `pool/main/x/x-cmd/x-cmd_0.9.9_all.deb`
against the file you actually downloaded:

```sh
curl -fsSL https://www.x-cmd.com/deb/dists/stable/Release
# SHA256 section lists the expected hash for the .deb
```

## The signing plan

When the operator publishes a GPG key, this is what changes:

1. The key (long-term, with a sub-key for APT) is published
   at a stable URL and announced on the x-cmd blog + GitHub
   release.
2. The CI pipeline that builds the repo emits both
   `dists/stable/Release` and `dists/stable/InRelease`
   (clear-signed) plus `dists/stable/Release.gpg`
   (detached).
3. The `sources.list` line drops `[trusted=yes]` and gains a
   `Signed-By:` directive (deb822-style) or the key is
   installed under `/etc/apt/keyrings/x-cmd.gpg` (classic
   style).
4. `apt update` then refuses to install from this source if
   the signature is missing, invalid, or made by an
   unexpected key — the same protection as any other signed
   APT source.

Until that work is published, **do not** blindly trust this
repo on production fleets you can't easily audit. Use it on
dev machines and pin the version.

## Supply-chain risks we **don't** mitigate today

- **Account takeover of `x-cmd/deb`**: a compromised GitHub
  token could publish a malicious `.deb`. Mitigation: GPG
  signature on `Release` makes any such push require the
  operator's signing key, not just any commit access.
- **Mirror compromise**: `www.x-cmd.com/deb/` is a static
  mirror; if it's swapped before the TLS layer, you'd get the
  attacker's bytes. Mitigation: same GPG signature catches
  it.
- **Replay / downgrade**: an attacker with TLS-MITM could
  serve you an older, vulnerable `.deb`. Mitigation: APT's
  date check on `Release` plus the key signature catches
  this once signing is live.

## Source provenance

The `.deb` is built from the upstream x-cmd release pipeline
([`x-cmd/release`](https://github.com/x-cmd/release)) and
mirrored verbatim into this repo. The build is reproducible
to the extent that the upstream pipeline is — for shell-only
content with pinned commit SHAs that is generally a yes.

## Next steps

- Want to know what's currently being published? See
  [Stats](./3-stats.en.md).
- Want to audit the package metadata yourself? See
  [Packaging](./1-packaging.en.md) for the index layout.

Source: this repo's [`README.md`](../../README.md) and the
upstream APT signing documentation.
