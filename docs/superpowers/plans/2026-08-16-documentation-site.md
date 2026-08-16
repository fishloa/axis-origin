# Comprehensive Documentation Site Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Publish a two-audience documentation site (operator guide + contributor guide, both mdBook-rendered) plus a generated rustdoc API reference, all served from one GitHub Pages deployment, built by CI on every push to `main`.

**Architecture:** Hand-written markdown pages under `docs/src/`, organized into `operator/` and `contributor/` sections, built by mdBook into a static site; `cargo doc` output is nested under `site/api/` in the same publish artifact. A new `docs` job in the existing `.github/workflows/ci.yml` builds and deploys via the official `actions/upload-pages-artifact` + `actions/deploy-pages` actions.

**Tech Stack:** mdBook (Rust static site generator), `cargo doc` (rustdoc), GitHub Actions, GitHub Pages.

## Global Constraints

- Docs job triggers on push to `main` only — not PRs, not tags (spec: CI/deployment).
- rustdoc build is host-only (`cargo doc` with default features — no `device` feature; that needs the ACAP SDK container) (spec: CI/deployment step 3).
- No JS-based site tooling (spec: Approach, Out of scope).
- No versioned docs / no version switcher (spec: Out of scope).
- `mdbook build` and `cargo doc` must both fail the job on error (broken `SUMMARY.md` link, broken intra-doc link) — a broken site must never deploy (spec: Verification).
- Enabling GitHub Pages (Settings → Pages → Source: GitHub Actions) is a manual, non-scriptable repo-admin step — flag it, don't attempt it (spec: CI/deployment).
- Rust toolchain is 1.97 (already bumped in `rust-toolchain.toml`/`Cargo.toml`/`ci.yml` in a prior change) — reuse that pin, don't reintroduce 1.86.

---

## File Structure

```
docs/
  book.toml                          # new — mdBook config
  src/
    SUMMARY.md                       # new — nav tree
    index.md                         # new — landing page
    operator/
      installing.md                  # new
      supported-devices.md           # new
      configuration.md               # new
      troubleshooting.md             # new
      security.md                    # new
    contributor/
      architecture.md                # new
      module-map.md                  # new
      building.md                    # new
      releasing.md                   # new
      testing.md                     # new
  superpowers/                       # existing — specs/plans, untouched
.github/workflows/ci.yml             # modify — add `docs` job
.gitignore                           # modify — ignore docs/book/ (mdbook output)
README.md                            # modify — shrink to quickstart + links
```

---

### Task 1: mdBook scaffold

**Files:**
- Create: `docs/book.toml`
- Create: `docs/src/SUMMARY.md`
- Create: `docs/src/index.md` (stub, replaced fully in Task 2)
- Create: `docs/src/operator/installing.md` … `docs/src/contributor/testing.md` (10 stub files, one line each, replaced in Tasks 3–12)
- Modify: `.gitignore`

**Interfaces:**
- Produces: the `docs/` directory tree every later task writes real content into. `SUMMARY.md` fixes the exact filenames every later task must use.

- [ ] **Step 1: Create `docs/book.toml`**

```toml
[book]
title = "axis-origin"
authors = ["Alex Fishlock"]
src = "src"

[output.html]
git-repository-url = "https://github.com/fishloa/axis-origin"
```

- [ ] **Step 2: Create `docs/src/SUMMARY.md`**

```markdown
# Summary

[Introduction](index.md)

# Operator Guide

- [Installing](operator/installing.md)
- [Supported Devices](operator/supported-devices.md)
- [Configuration](operator/configuration.md)
- [Troubleshooting](operator/troubleshooting.md)
- [Security](operator/security.md)

# Contributor Guide

- [Architecture](contributor/architecture.md)
- [Module Map](contributor/module-map.md)
- [Building](contributor/building.md)
- [Releasing](contributor/releasing.md)
- [Testing](contributor/testing.md)
```

- [ ] **Step 3: Create the 11 stub content files**

Each stub is a single top-level heading matching its `SUMMARY.md` title, e.g. `docs/src/index.md`:

```markdown
# Introduction

Stub — replaced in Task 2.
```

Create the same one-line-body stub (heading + "Stub — replaced in Task N.") for:
- `docs/src/operator/installing.md` → `# Installing` (Task 3)
- `docs/src/operator/supported-devices.md` → `# Supported Devices` (Task 4)
- `docs/src/operator/configuration.md` → `# Configuration` (Task 5)
- `docs/src/operator/troubleshooting.md` → `# Troubleshooting` (Task 6)
- `docs/src/operator/security.md` → `# Security` (Task 7)
- `docs/src/contributor/architecture.md` → `# Architecture` (Task 8)
- `docs/src/contributor/module-map.md` → `# Module Map` (Task 9)
- `docs/src/contributor/building.md` → `# Building` (Task 10)
- `docs/src/contributor/releasing.md` → `# Releasing` (Task 11)
- `docs/src/contributor/testing.md` → `# Testing` (Task 12)

- [ ] **Step 4: Ignore mdBook's build output**

Add to `.gitignore`:

```
/docs/book
```

- [ ] **Step 5: Install mdBook locally and verify the skeleton builds**

Run: `cargo install mdbook --locked` (if not already installed), then `mdbook build docs`
Expected: exits 0, produces `docs/book/index.html` and the 10 other pages with no warnings about missing files.

- [ ] **Step 6: Commit**

```bash
git add docs/book.toml docs/src .gitignore
git commit -m "docs: scaffold mdBook site skeleton"
```

---

### Task 2: `index.md` landing page

**Files:**
- Modify: `docs/src/index.md`

**Interfaces:**
- Consumes: nothing (top-level page).
- Produces: nothing later tasks depend on programmatically — links to other pages must use the exact paths from Task 1's `SUMMARY.md`.

- [ ] **Step 1: Write the landing page**

```markdown
# axis-origin

An **Axis ACAP application** that captures the camera's hardware-encoded
H.264/H.265 stream via **VDO** and serves **LL-HLS on the camera** — no
restream hop. It reuses the [`multimux`](https://crates.io/crates/multimux)
library (the same LL-HLS segmenter + RAM rolling window + axum origin +
blocking-reload that the standalone `multimux-cli` server uses), fed by a
`VdoIngestSession` instead of RTSP.

Target SoCs: **ARTPEC-6 / 7 / 8 / 9**; **H.264** on all of them, **H.265** on
ARTPEC-7/8/9 only. See [Supported Devices](operator/supported-devices.md)
for the full firmware/architecture matrix.

## Two guides, one project

- **[Operator Guide](operator/installing.md)** — installing the `.eap` on a
  camera, supported device matrix, configuration reference, troubleshooting
  playback, and the reverse-proxy security model.
- **[Contributor Guide](contributor/architecture.md)** — how the capture
  pipeline works, the module map, the host/device build split, the release
  process, and how the project is tested.

## API reference

Generated rustdoc for the host-testable library (`convert`, `admin`, `error`
modules) is published alongside this guide at [`/api/`](api/index.html).

## Source

[github.com/fishloa/axis-origin](https://github.com/fishloa/axis-origin) —
MIT OR Apache-2.0.
```

- [ ] **Step 2: Verify it builds and the internal links resolve**

Run: `mdbook build docs`
Expected: exits 0. mdBook does not validate cross-page links by default, so also manually confirm every `.md` path referenced above exists (it does, from Task 1's stubs) — `ls docs/src/operator/installing.md docs/src/operator/supported-devices.md docs/src/contributor/architecture.md`.

- [ ] **Step 3: Commit**

```bash
git add docs/src/index.md
git commit -m "docs: write landing page"
```

---

### Task 3: `operator/installing.md`

**Files:**
- Modify: `docs/src/operator/installing.md`

**Interfaces:**
- Consumes: nothing.

- [ ] **Step 1: Write the page**

```markdown
# Installing

Grab the `.eap` for your camera's firmware and architecture from the
[GitHub Releases page](https://github.com/fishloa/axis-origin/releases) —
see [Supported Devices](supported-devices.md) for which file matches your
camera. Each release also ships a `SHA256SUMS` file to verify the download.

## Option 1: web UI

On the camera's web interface: **Apps → Add app**, upload the `.eap` file,
then start it from the Apps list.

## Option 2: `cargo-acap-sdk` CLI

Install the installer tool once:

```bash
cargo install --locked --git https://github.com/AxisCommunications/acap-rs \
  --rev 8e58acb8f0617253ad21fb71ac319fea19454a38 cargo-acap-sdk
```

Then, with the `.eap` file downloaded locally:

```bash
export AXIS_DEVICE_IP=<camera-ip> AXIS_DEVICE_USER=root AXIS_DEVICE_PASS=<pw>
cargo-acap-sdk install --host "$AXIS_DEVICE_IP" --user root --pass "$AXIS_DEVICE_PASS"
cargo-acap-sdk start   --host "$AXIS_DEVICE_IP" --user root --pass "$AXIS_DEVICE_PASS"
```

`install` uploads the `.eap` over VAPIX; `start` launches it.

## Verify the install

1. The app shows **running** in the camera's Apps list.
2. `curl -u <user>:<pw> https://<cam>/local/axisorigin/hls/cam/media.m3u8`
   returns an LL-HLS media playlist (look for `#EXT-X-PART`,
   `#EXT-X-PART-INF`, `#EXT-X-SERVER-CONTROL`).
3. The admin settings page loads at `https://<cam>/local/axisorigin/`.

If any of these fail, see [Troubleshooting](troubleshooting.md).
```

- [ ] **Step 2: Verify it builds**

Run: `mdbook build docs`
Expected: exits 0.

- [ ] **Step 3: Commit**

```bash
git add docs/src/operator/installing.md
git commit -m "docs: write operator installing guide"
```

---

### Task 4: `operator/supported-devices.md`

**Files:**
- Modify: `docs/src/operator/supported-devices.md`

**Interfaces:**
- Consumes: nothing. Content sourced from `.github/workflows/ci.yml`'s `eap` job matrix and `README.md`'s SoC/codec line.

- [ ] **Step 1: Write the page**

```markdown
# Supported Devices

`axis-origin` targets Axis cameras built on the **ARTPEC-6, 7, 8, and 9**
SoCs. Every release ships four `.eap` packages — one per (firmware,
architecture) pair:

| Firmware | SDK version | Architecture | `.eap` filename suffix |
|---|---|---|---|
| 11 | 1.15.1 | aarch64 | `fw11_aarch64` |
| 11 | 1.15.1 | armv7hf | `fw11_armv7hf` |
| 12 | 12.1.0 | aarch64 | `fw12_aarch64` |
| 12 | 12.1.0 | armv7hf | `fw12_armv7hf` |

Match your camera's CPU architecture and installed firmware major version
to pick the right file from a [release](https://github.com/fishloa/axis-origin/releases).

## Codec support

| SoC | H.264 | H.265 |
|---|---|---|
| ARTPEC-6 | yes | no |
| ARTPEC-7 | yes | yes |
| ARTPEC-8 | yes | yes |
| ARTPEC-9 | yes | yes |

Set the codec via the [`codec` config field](configuration.md) — H.265 is
only valid on ARTPEC-7/8/9; requesting it on ARTPEC-6 will not work since the
hardware encoder doesn't support it.

## Architecture, not model number

Axis camera model numbers don't map 1:1 to a SoC generation across product
lines — check your specific camera model's datasheet or `Vendor` /
`ProdNbr` VAPIX parameters for its ARTPEC generation if you're unsure which
`.eap` to install.
```

- [ ] **Step 2: Verify it builds**

Run: `mdbook build docs`
Expected: exits 0.

- [ ] **Step 3: Commit**

```bash
git add docs/src/operator/supported-devices.md
git commit -m "docs: write supported devices matrix"
```

---

### Task 5: `operator/configuration.md`

**Files:**
- Modify: `docs/src/operator/configuration.md`

**Interfaces:**
- Consumes: nothing. Content sourced verbatim from `src/admin.rs`'s `Config` struct fields/defaults (lines 24-67) and `Config::validate` (lines 85-109).

- [ ] **Step 1: Write the page**

```markdown
# Configuration

Configuration is read/written through the admin HTTP API and persisted
on-device via the ACAP `axparameter` store. **Changes take effect on the
next app restart** — the running capture pipeline is not reconfigured live.

## Fields

| Field | Type | Default | Meaning |
|---|---|---|---|
| `channel` | integer | `0` | VDO channel index to capture from (single-sensor cameras use `0`). |
| `width` | integer | `1920` | Capture width, pixels. |
| `height` | integer | `1080` | Capture height, pixels. |
| `framerate` | integer | `30` | Capture frame rate, fps. |
| `codec` | string | `"h264"` | `"h264"` or `"h265"` — see [Supported Devices](supported-devices.md) for which SoCs support H.265. |
| `target_duration_secs` | float | `4.0` | LL-HLS target segment duration, seconds. |
| `part_target_ms` | integer | `500` | LL-HLS target part duration, milliseconds. |
| `window_segments` | integer | `8` | Number of segments kept in the LL-HLS media playlist window. |
| `port` | integer | `2999` | HTTP bind port (must match `manifest.json`'s `reverseProxy` target). |

## Validation

A `POST /admin/config` is rejected with `400 Bad Request` if:

- `codec` is anything other than `"h264"` or `"h265"`.
- `target_duration_secs` is `<= 0`.
- `part_target_ms` is `0`.
- `window_segments` is `0`.
- `port` is `0`.

## API

```bash
# Read the current config
curl -u <user>:<pw> https://<cam>/local/axisorigin/admin/config

# Update it (takes effect on next restart)
curl -u <user>:<pw> -X POST https://<cam>/local/axisorigin/admin/config \
  -H 'content-type: application/json' \
  -d '{"channel":0,"width":1920,"height":1080,"framerate":30,"codec":"h265","target_duration_secs":4.0,"part_target_ms":500,"window_segments":8,"port":2999}'
```

A successful `POST` returns `{"status":"ok","note":"takes effect on restart"}`.
After changing the config, restart the app (Apps list, or
`cargo-acap-sdk start`) for it to take effect.
```

- [ ] **Step 2: Verify it builds**

Run: `mdbook build docs`
Expected: exits 0.

- [ ] **Step 3: Commit**

```bash
git add docs/src/operator/configuration.md
git commit -m "docs: write configuration reference"
```

---

### Task 6: `operator/troubleshooting.md`

**Files:**
- Modify: `docs/src/operator/troubleshooting.md`

**Interfaces:**
- Consumes: `/admin/status` field names from `src/admin.rs`'s `Status` struct (lines 332-343): `running`, `current_segment`, `current_part`, `frames`, `last_error`.

- [ ] **Step 1: Write the page**

```markdown
# Troubleshooting

## Check `/admin/status` first

```bash
curl -u <user>:<pw> https://<cam>/local/axisorigin/admin/status
```

Returns:

```json
{
  "running": true,
  "current_segment": 5,
  "current_part": 2,
  "frames": 1234,
  "last_error": null
}
```

- `running: false` or a non-null `last_error` — the capture pipeline itself
  has failed; the error text describes why.
- `last_error` set but `running: true` — this can be a **stale config-store
  error** (a config load failure at boot, e.g. a broken `axparameter`
  backend) rather than a live pipeline fault. It surfaces here because
  nothing else clears it, not because the pipeline is currently unhealthy.
- `current_segment`/`current_part`/`frames` not advancing over repeated
  polls — media is not flowing even though the process is up. Check VDO
  itself (is another app already holding the channel?) before assuming
  `axis-origin` is at fault.

## Playlist 404s

The origin serves LL-HLS nested under `/hls` inside the app
(`https://<cam>/local/axisorigin/hls/...`). If your camera's reverse-proxy
strips the `apiPath` segment differently than expected, the nest prefix
seen by the app may not match. Confirm the actual proxied path reaching the
app and, if needed, this is a code-level fix — see the nest configuration
in `src/bin/axis-origin.rs` and [Building](../contributor/building.md) for
how to rebuild.

## No video / garbled video

`convert` (the VDO access-unit → `transmux::Sample` conversion) assumes VDO
delivers Annex-B framing (start codes). If a given camera/VDO version emits
a different framing, this assumption breaks. This is a code-level issue —
see [Architecture](../contributor/architecture.md) for where `VdoIngestSession`
does this conversion.

## Config changes not taking effect

Config is **not** hot-reloaded — see [Configuration](configuration.md).
`POST` the new config, then restart the app.

## Full verification checklist

For the complete on-device acceptance checklist (used when verifying a
build against real hardware), see
[Testing → Hardware verification checklist](../contributor/testing.md#hardware-verification-checklist).
```

- [ ] **Step 2: Verify it builds**

Run: `mdbook build docs`
Expected: exits 0.

- [ ] **Step 3: Commit**

```bash
git add docs/src/operator/troubleshooting.md
git commit -m "docs: write troubleshooting guide"
```

---

### Task 7: `operator/security.md`

**Files:**
- Modify: `docs/src/operator/security.md`

**Interfaces:**
- Consumes: `manifest.json`'s `reverseProxy` entries (lines 11-22): `hls` → `viewer` access, `admin` → `admin` access.

- [ ] **Step 1: Write the page**

```markdown
# Security

## Access levels

`axis-origin` registers two reverse-proxy paths with the camera's web
server, each gated by a different VAPIX access level (`manifest.json`):

| Path | `apiPath` | Access level |
|---|---|---|
| `https://<cam>/local/axisorigin/hls/...` | `hls` | `viewer` |
| `https://<cam>/local/axisorigin/admin/...` | `admin` | `admin` |

This means: any account with **viewer** rights on the camera can pull the
LL-HLS stream; only accounts with **admin** rights can read/write config or
status. Grant camera accounts accordingly — a viewer-only account cannot
read or change `axis-origin`'s configuration.

## Local-only bind

The app itself binds to `127.0.0.1:2999` (see [Configuration](configuration.md)'s
`port` field) — it is not reachable directly on the network; all access goes
through the camera's own web server and its VAPIX authentication, via the
reverse-proxy paths above.

## No live reconfiguration

Config changes only take effect on restart (see [Configuration](configuration.md)) —
there is no live-reload code path an attacker with `admin` access could use
to affect a running pipeline beyond what a restart itself does.

## Reporting a vulnerability

Use the repository's standard GitHub issue/security-advisory process:
[github.com/fishloa/axis-origin](https://github.com/fishloa/axis-origin).
```

- [ ] **Step 2: Verify it builds**

Run: `mdbook build docs`
Expected: exits 0.

- [ ] **Step 3: Commit**

```bash
git add docs/src/operator/security.md
git commit -m "docs: write security guide"
```

---

### Task 8: `contributor/architecture.md`

**Files:**
- Modify: `docs/src/contributor/architecture.md`

**Interfaces:**
- Consumes: README's Architecture section (current `README.md` lines 12-37, pre-shrink — read it before Task 14 removes it).

- [ ] **Step 1: Write the page**

```markdown
# Architecture

```text
VDO (libvdo) --coded AU + ts--> VdoIngestSession --> transmux Sample/TrackSpec
    --> media_plane::ingress::IngestDriver --> multimux::supervise_driver
    --> multimux::source::advance_route --> RouteHandle (RAM window)
    --> axum (multimux::origin nested /hls  +  admin routes)  on 127.0.0.1:2999
    <-- camera web server reverse-proxies (manifest.json) with VAPIX access levels
```

- **`src/convert.rs`** — pure (host-testable): VDO Annex-B access unit →
  `transmux::Sample`/`TrackSpec`, in-band SPS/PPS/VPS extraction (avcC / hvcC).
- **`src/vdo_source.rs`** (`device`-gated) — drives `vdo::StreamBuilder` /
  `RunningStream::next_buffer`, implements `media_plane::ingress::IngestSession`
  directly (no `Dialer`: VDO is a local capture, not a dial-out). Driven by a
  plain `IngestDriver` the caller (the bin) builds itself, not
  `multimux::pipeline::run_pipeline` (removed from multimux upstream).
- **`src/admin.rs`** — `Config` + `ConfigStore` (`axparameter` on device) +
  the `/admin/config` + `/admin/status` routes. See
  [Module Map](module-map.md) for the full breakdown.
- **`src/bin/axis-origin.rs`** (`device`-gated) — the ACAP entrypoint: loads
  config, runs the capture pipeline on a dedicated OS thread (VDO
  `next_buffer` blocks) via `multimux::supervise_driver`/
  `multimux::source::advance_route`, serves the origin + admin on
  `127.0.0.1:2999`.
- **`manifest.json`** — ACAP `reverseProxy` (`hls` = viewer, `admin` = admin)
  + `settingPage` (`html/index.html`). See
  [Security](../operator/security.md) for what the access levels mean.

See [Module Map](module-map.md) for file-by-file detail and
[Building](building.md) for the host/device split this architecture forces
on the build.
```

- [ ] **Step 2: Verify it builds**

Run: `mdbook build docs`
Expected: exits 0.

- [ ] **Step 3: Commit**

```bash
git add docs/src/contributor/architecture.md
git commit -m "docs: write contributor architecture guide"
```

---

### Task 9: `contributor/module-map.md`

**Files:**
- Modify: `docs/src/contributor/module-map.md`

**Interfaces:**
- Consumes: file line counts (`wc -l src/*.rs src/bin/*.rs`): `admin.rs` 731, `convert.rs` 393, `error.rs` 28, `lib.rs` 14, `vdo_source.rs` 530, `bin/axis-origin.rs` 365.

- [ ] **Step 1: Write the page**

```markdown
# Module Map

| File | Lines | Host-testable? | Responsibility |
|---|---|---|---|
| `src/lib.rs` | 14 | yes | Crate root — re-exports `convert`, `admin`, `error`; gates `vdo_source`/the bin behind `device`. |
| `src/error.rs` | 28 | yes | `OriginError` + `Result` alias used across the crate. |
| `src/convert.rs` | 393 | yes | Pure VDO Annex-B access unit → `transmux::Sample`/`TrackSpec` conversion, in-band SPS/PPS/VPS extraction. No `vdo` dependency. |
| `src/admin.rs` | 731 | yes (routes + `DefaultStore`); `AxParameterStore` is device-gated | `Config`, `ConfigStore` trait + `DefaultStore`/`AxParameterStore` impls, `Status`/`StatusHandle`, the `/admin/config` + `/admin/status` axum routes. |
| `src/vdo_source.rs` | 530 | no — `device`-gated | Drives `vdo::StreamBuilder`/`RunningStream::next_buffer`; implements `media_plane::ingress::IngestSession` for VDO capture. |
| `src/bin/axis-origin.rs` | 365 | no — `device`-gated, `required-features = ["device"]` | The ACAP entrypoint binary: loads config, spawns the capture pipeline thread, serves origin + admin routes. |

## Host vs device

Only `convert.rs`, `error.rs`, and `admin.rs`'s non-`AxParameterStore` code
paths compile and run on a host machine (macOS/Linux) without the `device`
feature. `vdo_source.rs` and the `axis-origin` binary require the `device`
feature, which pulls in `vdo`/`axparameter`/`acap-logging` (git-pinned,
ACAP-only crates) and only builds inside the Axis ACAP Native SDK Docker
image. See [Building](building.md).

## Tests

- `tests/convert_synthetic.rs` — integration tests against `convert.rs`
  using synthetic Annex-B input, host-only.
- `src/admin.rs`'s `#[cfg(test)] mod tests` — axum route tests using
  `tower::ServiceExt::oneshot` against `DefaultStore`/test-double
  `ConfigStore` implementations (`BrokenStore`, `StoredStore`), host-only.

See [Testing](testing.md) for how to run these and how device-only code is
verified instead.
```

- [ ] **Step 2: Verify it builds**

Run: `mdbook build docs`
Expected: exits 0.

- [ ] **Step 3: Commit**

```bash
git add docs/src/contributor/module-map.md
git commit -m "docs: write module map"
```

---

### Task 10: `contributor/building.md`

**Files:**
- Modify: `docs/src/contributor/building.md`

**Interfaces:**
- Consumes: current `Cargo.toml` dependency versions (post dependency-bump: `multimux = "0.10"`, `transmux = "0.24"`, `broadcast-common = "9.3"`, `media-plane = "0.4"`, git rev `3c513f92bf3dc2e6d1dbb1e3d5c00c0c294a6a69` for `vdo`/`axparameter`/`acap-logging`) and `rust-toolchain.toml`'s `channel = "1.97"`. **Do not copy the stale versions/rev/workflow-filename from the current `README.md`'s Build model section** — that section predates the dependency and toolchain bump and the `ci.yml` rename; this page must state the corrected values below.

- [ ] **Step 1: Write the page**

```markdown
# Building

`axis-origin` is **out of the main cargo workspace** (its own `Cargo.toml` +
`rust-toolchain.toml`, pinned to Rust **1.97**). It depends on published
`multimux 0.10` + `transmux 0.24` + `media-plane 0.4` + `broadcast-common 9.3`
(crates.io — the latter two only under the `device` feature, see
`vdo_source.rs`) and **git-pinned** acap-rs crates (`vdo`/`axparameter`/
`acap-logging` at rev `3c513f92bf3dc2e6d1dbb1e3d5c00c0c294a6a69` via a fork,
`fishloa/acap-rs`). It is **not** published to crates.io (`publish = false`)
— it is a deployable `.eap`. The fork exists for firmware-11 SDK 1.15.1
compatibility: it drops a firmware-12-only `VDO_ERROR_NO_VIDEO` diagnostic
arm so the `vdo` crate compiles against the fw11 SDK too.

## Host build (macOS/Linux, no `device` feature)

Compiles + tests the host-testable lib only (`convert`, `admin`, `error` —
see [Module Map](module-map.md)):

```bash
cargo test --locked
cargo clippy --locked -- -D warnings
cargo fmt --all --check
```

## Device build (the `.eap`)

The `device` feature pulls in the acap-rs crates and **only builds inside
the Axis ACAP Native SDK Docker** (`vdo-sys` needs `pkg-config vdo` from the
SDK sysroot — macOS cannot cross-compile ACAP). CI
(`.github/workflows/ci.yml`'s `eap` job) builds all four (firmware,
architecture) combinations on every push/PR — see
[Supported Devices](../operator/supported-devices.md) for the matrix. To
build locally in the SDK Docker:

```bash
docker run --rm -v "$PWD:/w" -w /w axisecp/acap-native-sdk:12.1.0-aarch64-ubuntu24.04 bash -lc '
  apt-get update && apt-get install -y --no-install-recommends build-essential curl ca-certificates pkg-config clang libclang-dev
  curl --proto "=https" --tlsv1.2 -sSf https://sh.rustup.rs | sh -s -- -y --default-toolchain 1.97.0 --target aarch64-unknown-linux-gnu
  . "$HOME/.cargo/env"
  # set the sysroot / PKG_CONFIG / linker env per .github/workflows/ci.yml's "Discover SDK sysroot" + "Set cross-compile env" steps
  cargo install --locked --git https://github.com/AxisCommunications/acap-rs --rev 8e58acb8f0617253ad21fb71ac319fea19454a38 cargo-acap-build
  ACAP_BUILD_IMPL=equivalent cargo-acap-build --target aarch64 -- -p axis-origin --features device
'
# -> target/acap/axis-origin_0_1_0_aarch64.eap
```

For firmware 11, swap the image tag for `1.15.1-aarch64-ubuntu22.04` (or the
`armv7hf` variant of either). `.github/workflows/ci.yml` is the
authoritative, working recipe — the container env vars it sets for the
sysroot/linker/pkg-config are required; this snippet elides them for
brevity.

See [Releasing](releasing.md) for how these builds become a published
`.eap` set, and [Testing](testing.md) for what "passing" actually means for
each build.
```

- [ ] **Step 2: Verify it builds**

Run: `mdbook build docs`
Expected: exits 0.

- [ ] **Step 3: Commit**

```bash
git add docs/src/contributor/building.md
git commit -m "docs: write building guide with corrected dependency versions"
```

---

### Task 11: `contributor/releasing.md`

**Files:**
- Modify: `docs/src/contributor/releasing.md`

**Interfaces:**
- Consumes: `.github/workflows/ci.yml`'s `release` job (triggers on `refs/tags/v*`, downloads all `axis-origin-*` artifacts, renames to `<stem>_<fw>_<arch>.eap`, generates `SHA256SUMS`, `gh release create`/`gh release upload`).

- [ ] **Step 1: Write the page**

```markdown
# Releasing

Releases are entirely CI-driven — never manually `gh release create` or
upload artifacts from a local machine. Pushing a `v*` tag is the only
trigger.

## What happens on `git push --tags`

1. The `eap` matrix job (four combinations: firmware 11/12 ×
   aarch64/armv7hf — see [Supported Devices](../operator/supported-devices.md))
   builds a `.eap` for each and uploads it as a workflow artifact.
2. The `release` job (gated on `startsWith(github.ref, 'refs/tags/v')`)
   downloads every `axis-origin-*` artifact, renames each to
   `<name>_<version>_<fw>_<arch>.eap`, generates a `SHA256SUMS` file, then
   creates (or reuses) a GitHub Release named after the tag and uploads the
   four `.eap`s + `SHA256SUMS` to it.

## Cutting a release

```bash
git tag vX.Y.Z
git push origin vX.Y.Z
```

Then watch the Actions run (`gh run watch` or the Actions tab) — the
release appears at `github.com/fishloa/axis-origin/releases/tag/vX.Y.Z`
once the `release` job completes.

## Moving a tag (fixing a bad release)

If a tag was pushed against a broken commit, move it and re-push:

```bash
git tag -f vX.Y.Z <good-commit>
git push -f origin vX.Y.Z
```

This is safe for a tag that's only minutes old with no external consumers;
for anything already depended on externally, cut a new patch tag instead of
force-moving.
```

- [ ] **Step 2: Verify it builds**

Run: `mdbook build docs`
Expected: exits 0.

- [ ] **Step 3: Commit**

```bash
git add docs/src/contributor/releasing.md
git commit -m "docs: write releasing guide"
```

---

### Task 12: `contributor/testing.md`

**Files:**
- Modify: `docs/src/contributor/testing.md`

**Interfaces:**
- Consumes: the hardware verification checklist (from the pre-shrink `README.md`'s "Hardware verification checklist" section) and the host test commands from Task 10.
- Produces: the `#hardware-verification-checklist` anchor that `operator/troubleshooting.md` (Task 6) links to — the heading text below (`## Hardware verification checklist`) must stay exactly this so mdBook's auto-generated anchor slug matches.

- [ ] **Step 1: Write the page**

```markdown
# Testing

## Host tests

The host-testable lib (`convert`, `admin`, `error` — see
[Module Map](module-map.md)) is tested on every push/PR with:

```bash
cargo test --locked
cargo clippy --locked -- -D warnings
cargo fmt --all --check
```

This covers `tests/convert_synthetic.rs` (synthetic Annex-B conversion
cases) and `src/admin.rs`'s route tests (config get/post validation,
status snapshot fields, `ConfigStore` broken/stored/unset behavior via test
doubles).

## Device build verification

The `device`-gated code (`vdo_source.rs`, the `axis-origin` binary) can only
be **compiled** on host CI (inside the ACAP SDK Docker — see
[Building](building.md)); compiling is not the same as verifying it works.
Compilation success is a prerequisite, not an acceptance criterion.

## Hardware verification checklist

The real "done" for any change touching the capture/serving path is
verified on a real ARTPEC-6/7/8/9 camera:

1. The `.eap` installs and the app shows **running** in the camera's Apps
   list.
2. `curl -u <user>:<pw> https://<cam>/local/axisorigin/hls/cam/media.m3u8`
   returns an LL-HLS media playlist (`#EXT-X-PART`, `#EXT-X-PART-INF`,
   `#EXT-X-SERVER-CONTROL`).
3. A real LL-HLS player (Safari / hls.js / `ffplay`) plays **live** video via
   that URL at low latency.
4. The admin settings page loads at `https://<cam>/local/axisorigin/` and a
   config change (e.g. `part_target_ms`) takes effect after a restart.
5. H.264 verified on all SoCs; H.265 verified additionally on
   ARTPEC-7/8/9 (see [Supported Devices](../operator/supported-devices.md)).

> **Known verify-on-device detail:** the origin is served nested under
> `/hls`; if the camera's reverse-proxy strips the `apiPath` segment before
> forwarding, the nest prefix may need adjusting in
> `src/bin/axis-origin.rs`'s `Router::nest("/hls", …)`. Confirm the actual
> proxied path on the device if the playlist 404s. Likewise confirm VDO
> delivers Annex-B (start codes) — `convert` assumes it; if a given
> camera/VDO version carries a different framing, `VdoIngestSession`'s
> `data_copy()` handling needs adjusting.

See [Troubleshooting](../operator/troubleshooting.md) for diagnosing a
failure at any of these steps via `/admin/status`.
```

- [ ] **Step 2: Verify it builds**

Run: `mdbook build docs`
Expected: exits 0.

- [ ] **Step 3: Verify the cross-link anchor from Task 6 resolves**

Run: `grep -n "^## Hardware verification checklist" docs/src/contributor/testing.md`
Expected: one match — confirms the heading text matches what
`operator/troubleshooting.md`'s `#hardware-verification-checklist` link
targets (mdBook slugifies headings to lowercase-hyphenated anchors).

- [ ] **Step 4: Commit**

```bash
git add docs/src/contributor/testing.md
git commit -m "docs: write testing guide with hardware verification checklist"
```

---

### Task 13: `docs` CI job — build + deploy to GitHub Pages

**Files:**
- Modify: `.github/workflows/ci.yml`

**Interfaces:**
- Consumes: `rust-toolchain.toml`'s `1.97` pin (already set); `docs/` (from Tasks 1-12).

- [ ] **Step 1: Add the `docs` job to `.github/workflows/ci.yml`**

Add this as a new top-level job (sibling to `host`/`eap`/`release`):

```yaml
  docs:
    name: docs (mdBook + rustdoc -> GitHub Pages)
    if: github.ref == 'refs/heads/main' && github.event_name == 'push'
    runs-on: ubuntu-latest
    permissions:
      contents: read
      pages: write
      id-token: write
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    steps:
      - uses: actions/checkout@v5

      - name: Install Rust (rustup, 1.97)
        run: |
          curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh -s -- -y \
            --default-toolchain 1.97.0 --profile minimal
          echo "$HOME/.cargo/bin" >> "$GITHUB_PATH"

      - name: Install mdBook
        run: |
          set -euo pipefail
          MDBOOK_VERSION=$(curl -fsSL https://api.github.com/repos/rust-lang/mdBook/releases/latest | grep -m1 '"tag_name"' | sed -E 's/.*"v([^"]+)".*/\1/')
          curl -fsSL "https://github.com/rust-lang/mdBook/releases/download/v${MDBOOK_VERSION}/mdbook-v${MDBOOK_VERSION}-x86_64-unknown-linux-gnu.tar.gz" \
            | tar xz -C /usr/local/bin

      - name: Build the mdBook guide
        run: mdbook build docs -d ../site

      - name: Build rustdoc (host lib only, no device feature)
        env:
          RUSTDOCFLAGS: -D warnings
        run: cargo doc --locked --no-deps -p axis-origin

      - name: Assemble rustdoc into site/api
        run: |
          mkdir -p site/api
          cp -r target/doc/* site/api/
          echo '<meta http-equiv="refresh" content="0; url=axis_origin/index.html">' > site/api/index.html

      - name: Upload Pages artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: site

      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

- [ ] **Step 2: Validate the YAML parses**

Run: `python3 -c "import yaml; yaml.safe_load(open('.github/workflows/ci.yml'))" && echo OK`
Expected: prints `OK` (catches indentation errors before pushing).

- [ ] **Step 3: Dry-run the doc build steps locally**

Run:
```bash
mdbook build docs -d /tmp/axis-origin-site
RUSTDOCFLAGS="-D warnings" cargo doc --locked --no-deps -p axis-origin
mkdir -p /tmp/axis-origin-site/api
cp -r target/doc/* /tmp/axis-origin-site/api/
ls /tmp/axis-origin-site/index.html /tmp/axis-origin-site/api/axis_origin/index.html
```
Expected: both files exist, no errors from either build command.

- [ ] **Step 4: Commit**

```bash
git add .github/workflows/ci.yml
git commit -m "ci: add docs job to build + deploy mdBook guide and rustdoc to GitHub Pages"
```

- [ ] **Step 5: Push and confirm the manual prerequisite with the user**

This job will fail with a permissions/environment error until GitHub Pages
is enabled for the repo. Tell the user: **before merging/pushing this to
`main`, a repo admin must go to Settings → Pages → Source and select
"GitHub Actions"** — this cannot be done via API/CLI in a way that fits
this plan's scope; it's a one-time manual click.

---

### Task 14: Shrink `README.md` to quickstart + links

**Files:**
- Modify: `README.md`

**Interfaces:**
- Consumes: all pages from Tasks 2-12 (links into them).

- [ ] **Step 1: Replace `README.md`'s full content**

```markdown
# axis-origin

An **Axis ACAP application** that captures the camera's hardware-encoded
H.264/H.265 stream via **VDO** and serves **LL-HLS on the camera** — no
restream hop.

Target SoCs: **ARTPEC-6 / 7 / 8 / 9**; **H.264** on all, **H.265** on
ARTPEC-7/8/9 only.

## Documentation

Full docs: **https://fishloa.github.io/axis-origin/**

- [Installing](https://fishloa.github.io/axis-origin/operator/installing.html)
- [Supported Devices](https://fishloa.github.io/axis-origin/operator/supported-devices.html)
- [Configuration](https://fishloa.github.io/axis-origin/operator/configuration.html)
- [Troubleshooting](https://fishloa.github.io/axis-origin/operator/troubleshooting.html)
- [Security](https://fishloa.github.io/axis-origin/operator/security.html)
- [Architecture](https://fishloa.github.io/axis-origin/contributor/architecture.html)
- [Building](https://fishloa.github.io/axis-origin/contributor/building.html)
- [Releasing](https://fishloa.github.io/axis-origin/contributor/releasing.html)
- [Testing](https://fishloa.github.io/axis-origin/contributor/testing.html)
- [API reference (rustdoc)](https://fishloa.github.io/axis-origin/api/)

## Quickstart

```bash
# Host build/test (no ACAP SDK needed)
cargo test --locked

# Install a released .eap on a camera
export AXIS_DEVICE_IP=<camera-ip> AXIS_DEVICE_USER=root AXIS_DEVICE_PASS=<pw>
cargo-acap-sdk install --host "$AXIS_DEVICE_IP" --user root --pass "$AXIS_DEVICE_PASS"
cargo-acap-sdk start   --host "$AXIS_DEVICE_IP" --user root --pass "$AXIS_DEVICE_PASS"
```

Grab a `.eap` from the [Releases page](https://github.com/fishloa/axis-origin/releases)
— see the [Supported Devices](https://fishloa.github.io/axis-origin/operator/supported-devices.html)
matrix for which file matches your camera.

## License

MIT OR Apache-2.0.
```

- [ ] **Step 2: Verify no other file references the removed README sections**

Run: `grep -rn "axis-origin.yml" --include="*.md" --include="*.yml" . 2>/dev/null`
Expected: no output (this was the stale workflow-filename reference the
old README carried — confirm it's gone, not just moved).

- [ ] **Step 3: Commit**

```bash
git add README.md
git commit -m "docs: shrink README to quickstart, move full docs to the mdBook site"
```

---

## Final Step: Report the manual prerequisite

After all tasks are done and pushed to `main`, remind the user that the
`docs` job's first run will fail until a repo admin enables **Settings →
Pages → Source → GitHub Actions** on `github.com/fishloa/axis-origin` —
this is the spec's documented one-time manual step, not something to work
around.
