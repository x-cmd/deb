---
name: 2-security
description: APT repository security model for x-cmd/deb. Status of GPG signing, what `[trusted=yes]` opts into, manual checksum verification recipe, and the InRelease signing roadmap.
type: explanation
---

# Core Content

security_status:

  tls_at_cdn: true
  per_package_sha256_in_packages: true
  release_file_with_index_checksums: true
  release_gpg_detached: false
  inrelease_clear_signed: false
  gpg_key_published: false
  key_rotation_policy: false
  trusted_yes_required: true

what_trusted_yes_disables:

  - id: network-mitm
    description: |
      Without TLS-only signature checks, a network MITM with
      TLS-strip would normally be caught — `https://` URL still
      covers this.
    mitigated_by_tls: true

  - id: repo-compromise
    description: |
      A compromised repo operator (or compromised GitHub token)
      pushing a malicious `.deb`. Without a signature, only
      operator reputation stands between you and the attack.
    mitigated_by_tls: false

verification_recipes:

  - id: sha256-manual
    description: |
      Download the .deb, verify against the expected SHA-256
      from a second channel (GitHub release page, etc.).
    command: |
      URL="https://www.x-cmd.com/deb/pool/main/x/x-cmd/x-cmd_0.9.9_all.deb"
      EXPECTED="0e0e24f1d67bf69a3f748f8b1c75e091381292a81ebe87b4f2290859dc40a3a9"
      curl -fsSLO "$URL"
      echo "$EXPECTED  x-cmd_0.9.9_all.deb" | sha256sum -c -

  - id: release-file-fetch
    description: |
      Fetch the suite-level `Release` file and read its
      `SHA256:` section to get the expected hash for the
      `.deb`.
    command: curl -fsSL https://www.x-cmd.com/deb/dists/stable/Release

signing_roadmap:

  - step: 1
    title: Publish GPG key
    description: |
      Long-term key + APT sub-key, published at a stable URL,
      announced on the x-cmd blog and a GitHub release.

  - step: 2
    title: Emit InRelease + Release.gpg
    description: |
      The CI that builds the repo emits `dists/stable/Release`,
      `dists/stable/InRelease` (clear-signed), and
      `dists/stable/Release.gpg` (detached).

  - step: 3
    title: Update sources.list
    description: |
      Drop `[trusted=yes]`; add `Signed-By:` (deb822-style) or
      install the key under `/etc/apt/keyrings/x-cmd.gpg`
      (classic style).

  - step: 4
    title: Enforce
    description: |
      `apt update` then refuses unsigned / invalid-signature
      sources — same protection as any signed APT repo.

## Key Information

supply_chain_risks_not_mitigated_today:

  - id: account-takeover
    description: |
      A compromised GitHub token with push access to
      `x-cmd/deb` could publish a malicious `.deb`.
    mitigation_after_signing: true

  - id: mirror-compromise
    description: |
      `www.x-cmd.com/deb/` is a static mirror; if swapped
      before TLS, you'd get attacker's bytes.
    mitigation_after_signing: true

  - id: replay-downgrade
    description: |
      TLS-MITM attacker serves an older, vulnerable `.deb`.
    mitigation_after_signing: true

practical_safety_net_today:

  - repo_on_github_for_audit
  - verbatim_mirror_via_readonly_pipeline
  - deterministic_build_from_same_source_as_public_release_artifacts
  - cross_channel_sha256_verification_via_github_release

## Use Cases

use_cases:

  - Dev-machine install: fine as-is, `[trusted=yes]` is
    documented.
  - Production fleet with audit: wait for InRelease signing
    OR pin the version AND verify SHA-256 from a second
    channel.
  - CI ephemeral container: skip the APT source, fetch +
    verify the .deb directly.

## Related Resources

related:

  repo: https://github.com/x-cmd/deb
  upstream: https://github.com/x-cmd/release
  apt_signing_docs: "https://wiki.debian.org/SecureApt"
  deb822_signed_by: "https://manpages.debian.org/bookworm/apt/sources.list.5.en.html"

## Summary

This APT repo is currently unsigned. The integrity layer you
can rely on today is TLS at the CDN, per-package SHA-256 in
`Packages`, and suite-level `Release` with checksums of every
index. `[trusted=yes]` disables APT's signature check for this
specific source; it does not disable TLS. The supply-chain
risks not mitigated today are account takeover, mirror
compromise, and replay/downgrade — all caught once GPG signing
goes live. Until then, pin the version and verify SHA-256 from
a second channel (e.g., GitHub release page).
