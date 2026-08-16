# Comprehensive documentation site — design

## Goal

`axis-origin` currently documents itself only through `README.md` (dev-focused
build/deploy instructions) and module-level rustdoc comments in `src/*.rs`
(thorough, but never rendered anywhere — the crate is `publish = false`, so
there's no docs.rs). There is no operator-facing install/config/troubleshooting
guide, and no rendered API reference. This spec covers building both, as a
single published site.

## Audience

Two audiences, roughly equal weight:

- **Operators/integrators** — people installing the `.eap` on Axis cameras
  across the supported SoC/firmware matrix, configuring it, and diagnosing
  playback problems.
- **Contributors/maintainers** — people changing `axis-origin` itself: the
  VDO→HLS pipeline, the host/device build split, the release process.

## Approach

**mdBook** for hand-written guide content (Rust-native, single static binary,
no JS toolchain — matches the project's all-Rust footprint) + **`cargo doc`**
for the generated API reference, published together as one **GitHub Pages**
site.

Rejected alternatives:
- Docusaurus/other JS site generators — pulls in a whole npm/bun+React
  toolchain for a project that currently has zero JS build tooling.
- Plain markdown + GitHub's default file viewer, no site — works, but no
  cross-page nav/search/rendering for the operator guide, and no path to
  publish the rustdoc API reference at all.

## Structure

```
docs/
  book.toml
  src/
    SUMMARY.md
    index.md                   landing page — what axis-origin is
    operator/
      installing.md            .eap install: cargo-acap-sdk + web UI paths
      supported-devices.md     ARTPEC-6/7/8/9 x fw11/fw12 matrix, H.264 vs H.265
      configuration.md         admin API: Config fields, VAPIX access levels
      troubleshooting.md       playback checklist (from README) + /admin/status fields
      security.md              viewer vs admin reverseProxy access levels
    contributor/
      architecture.md          VDO -> IngestSession -> multimux -> axum pipeline
      module-map.md            convert.rs / vdo_source.rs / admin.rs / bin, host vs device
      building.md              host test loop vs SDK Docker .eap build
      releasing.md             tag v* -> CI eap matrix -> GH Release (from ci.yml)
      testing.md               host-testable lib, convert_synthetic.rs, hardware checklist
```

Content for each page is sourced from what already exists in `README.md`,
module rustdoc comments, `CHANGELOG.md`, and `.github/workflows/ci.yml` —
this is a restructuring/expansion pass, not new invented content, except
where a page (e.g. `troubleshooting.md`, `security.md`) doesn't have an
existing equivalent and needs to be written from the current code/config
behavior.

## CI / deployment

New `docs` job in the existing `.github/workflows/ci.yml`, alongside `host`/
`eap`/`release` (keeps one workflow file, matches the current pattern):

- Trigger: push to `main` only. Not on PRs (avoids publishing from unmerged
  branches) and not on tags (the release job already handles tags).
- Steps:
  1. Install mdBook from a pinned GitHub release binary (`curl -L` the
     `mdbook-<ver>-x86_64-unknown-linux-gnu.tar.gz` asset + extract to
     `$GITHUB_PATH`) — faster and more reproducible in CI than
     `cargo install mdbook` (avoids a multi-minute from-source compile on
     every run).
  2. `mdbook build docs -d ../site`
  3. `cargo doc --no-deps --no-default-features -p axis-origin` — host lib
     only (`convert`, `admin`, `error`); the `device` feature needs the ACAP
     SDK container's `vdo` sysroot, so device-only API docs are out of scope
     for this pass.
  4. Copy `target/doc` → `site/api/`.
  5. Publish `site/` via `actions/upload-pages-artifact` +
     `actions/deploy-pages` (official GH Pages actions — `GITHUB_TOKEN` +
     Pages OIDC, no PAT/third-party action).
- **Manual one-time step** (repo settings, not scriptable): enable Pages in
  Settings → Pages → Source: GitHub Actions. Flagged to the user — must be
  done by a repo admin in the GitHub UI before the first deploy succeeds.

## Verification

- `mdbook build` fails the job on a broken `SUMMARY.md` link or malformed
  markdown — a broken site never deploys.
- `cargo doc --no-deps -D warnings` (deny broken intra-doc links) fails the
  job on a bad rustdoc link.
- Local dry run (`mdbook build docs`) before committing the initial content,
  to catch structural issues before CI does.
- Not covered by this pass: visual QA of the rendered site (spot-check after
  first deploy is a manual step, not automated).

## README changes

`README.md` shrinks to: what the project is, a quickstart (install +
hardware checklist summary), and links into the book for everything else
(architecture, build model, full hardware checklist, config reference).
Avoids maintaining the same architecture/build prose in two places.
`CHANGELOG.md` is untouched.

## Out of scope

- Versioned docs (multiple releases side-by-side) — single `main`-tracked
  site only, no version switcher.
- Device-feature rustdoc (needs the SDK container; host-only API docs only).
- Any JS-based site tooling.
