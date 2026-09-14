# Contributing & maintaining x-cmd/deb

> Runbook for whoever is cutting the next release, debugging
> a sync issue, or adding GPG signing. Read this end-to-end
> once before your first release; for the next one it should
> be skimmable.

## Repo roles

| Role | Person / bot | Does what |
| --- | --- | --- |
| Maintainer | Li Junhao <l@x-cmd.com> | Reviews releases, holds the GPG key, cuts final tags |
| Release pipeline | `x-cmd/release` upstream | Builds the `.deb` artifact with pinned commit SHAs |
| This repo's sync | Daily from `main` branch | Mirrors `pool/` + `dists/` + `x-cmd.list` + `docs/` to the CDN |

## Cutting a release

1. **Build the new `.deb`** in `x-cmd/release` upstream. The
   pipeline is reproducible and emits a single
   `x-cmd_<version>_all.deb` plus a SHA256SUMS file.

2. **Compute checksums** for the new `.deb`:
   ```sh
   shasum -a 256 x-cmd_<version>_all.deb
   shasum -a 1 x-cmd_<version>_all.deb
   md5sum x-cmd_<version>_all.deb
   ```
   Save these for step 5.

3. **Drop the `.deb` into `pool/`**:
   ```sh
   cp x-cmd_<version>_all.deb pool/main/x/x-cmd/
   ```
   Decide whether to **keep** the previous version's `.deb`
   (rollback path) or **remove** it. The default is to keep
   both — APT can downgrade via `apt install x-cmd=<old>` as
   long as the index still references both.

4. **Regenerate the indices** in a Debian container (macOS
   doesn't ship `dpkg-scanpackages` or `apt-ftparchive`):
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

5. **Update `docs/3-stats.{md,cn.md,en.md,llms.md}`** with the
   new version, size, and checksums. The `docs/3-stats.faq.yml`
   answer-set rarely changes — review only.

6. **Sanity-check the diff**:
   ```sh
   git diff --stat
   git diff dists/stable/Release   # eyeball the checksum section
   ```
   Confirm:
   - One new `.deb` in `pool/`.
   - One updated `Packages` per arch.
   - `Release` checksum section reflects the new size /
     hash.
   - `docs/3-stats.*` reflects the new version.

7. **Commit + push**:
   ```sh
   git add pool/main/x/x-cmd/x-cmd_<version>_all.deb \
          dists/stable \
          docs/3-stats.* \
          docs/3-stats.faq.yml
   git commit -m "release: x-cmd <version>"
   git push
   ```

8. **Wait for the daily sync** (or trigger it manually if the
   pipeline supports it). Verify at
   `https://www.x-cmd.com/deb/pool/main/x/x-cmd/` that the new
   file is present.

9. **Sanity-check from a fresh container**:
   ```sh
   docker run --rm -it debian:latest bash -lc '
     echo "deb [trusted=yes] https://www.x-cmd.com/deb/ stable main" \
       > /etc/apt/sources.list.d/x-cmd.list
     apt update
     apt install -y x-cmd
     x --version'
   ```
   Should report `<version>`.

## Adding a new architecture

1. Add the new arch's loop iteration to step 4 above (e.g.
   `for a in amd64 arm64 armhf i386 all riscv64`).
2. Update the `APT::FTPArchive::Release::Architectures=...`
   field in the same `apt-ftparchive release` invocation.
3. Update `docs/3-stats.md` and `docs/3-stats.en.md` (the
   matrix table).
4. Update `README.md` and `README.cn.md` ("Adding a new
   architecture" hint in the Notes section).
5. Commit + push + verify from a machine of that arch.

For native-code packages, also rebuild the `.deb` for the new
arch. For `Architecture: all` packages, this whole section is
a one-time text edit.

## Adding GPG signing (the `[trusted=yes]` removal)

Pre-requisites: a GPG key generated and stored somewhere safe
(Hardware token preferred — YubiKey, Nitrokey, etc.). The
sub-key used for APT signing should be a separate signing-only
sub-key with an expiry.

1. **Export the public key** to a stable URL:
   ```sh
   gpg --export --armor <KEY-ID> > x-cmd-repo.pub.asc
   ```
   Host at e.g.
   `https://www.x-cmd.com/deb/x-cmd-repo.pub.asc` and
   `https://github.com/x-cmd/release/releases/latest`.

2. **Generate the signing material** alongside step 4 above:
   ```sh
   gpg --default-key <KEY-ID> --armor \
       --detach-sign --output dists/stable/Release.gpg \
       dists/stable/Release
   gpg --default-key <KEY-ID> --armor \
       --clear-sign --output dists/stable/InRelease \
       dists/stable/Release
   ```

3. **Drop `[trusted=yes]` from `x-cmd.list`** and switch to
   deb822-style (modern):
   ```
   Types: deb
   URIs: https://www.x-cmd.com/deb/
   Suites: stable
   Components: main
   Signed-By: /usr/share/keyrings/x-cmd-archive-keyring.gpg
   ```
   …or classic one-liner:
   ```
   deb [signed-by=/usr/share/keyrings/x-cmd-archive-keyring.gpg] \
       https://www.x-cmd.com/deb/ stable main
   ```

4. **Document the key install**:
   ```sh
   sudo install -m 0644 x-cmd-repo.pub.asc \
       /usr/share/keyrings/x-cmd-archive-keyring.gpg
   ```

5. **Update `docs/2-security.{md,cn.md,en.md,llms.md,faq.yml}`**
   — flip the "What you get today" table to all ✅ and remove
   the `[trusted=yes]` rationale.

6. **Announce** on the x-cmd blog and a GitHub release.

## Incident playbook

### "Users say `apt update` fails with checksum error"

- Check that `dists/stable/Release` and the per-arch
  `Packages*` files are byte-aligned with what's in
  `pool/main/x/x-cmd/`. Most "checksum mismatch" reports come
  from a CDN serving stale `Release` against a freshly
  re-uploaded `.deb`. Force a CDN purge.

### "GitHub repo push triggered a sync, but CDN is stale"

- The daily sync is event-driven on push; the cron is a
  backstop. Wait one cron tick, or trigger the sync workflow
  manually.

### "I uploaded a `.deb` and forgot to update the indices"

- `apt update` will not see it. Re-run step 4 above and push
  the regenerated `dists/`. The `.deb` itself doesn't move,
  so existing mirrors don't re-download it.

### "I need to yank a release"

- Move the offending `.deb` out of `pool/main/x/x-cmd/` (or
  rename it), regenerate the indices, push. `apt upgrade` will
  downgrade users who had it installed — unless you instead
  publish a fixed version and let the normal upgrade path
  handle it. Choose the destructive option only for security
  emergencies.

### "I broke the `Release` checksum section"

- Run step 4 again. The script regenerates the whole file.
  Diff before pushing.

### "Mirror operator reports missing files"

- Confirm the file exists in `pool/` in this repo (the source
  of truth). If yes, the mirror is out of sync — point them
  at the daily-sync trigger. If no, someone accidentally
  deleted a tracked file; `git restore` it and re-run step 4.

## Release cadence

- **Event-driven**: a release ships when upstream
  `x-cmd/release` cuts one.
- **Daily sync**: the CDN mirror pulls this repo once every
  24 hours.
- **No fixed CI rebuild** on a schedule — there's nothing to
  rebuild unless a release lands or an index regen is
  needed.

## Branch / tag policy

- All releases land on `main`. No long-lived branches.
- A `git tag x-cmd/<version>` may be added for traceability,
  but the canonical "what version is out" is the `Version:`
  field in `dists/stable/main/binary-all/Packages`.
