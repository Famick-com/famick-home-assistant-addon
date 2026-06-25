# Changelog

All notable changes to the Famick Home Assistant add-on are documented here.
The format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and the version numbers follow [Semantic Versioning](https://semver.org/).

## [Unreleased]

## [0.1.12]

### Changed

- Rebuilds the add-on on the canonical image (`1.0.0-beta.13`), which adds
  first-class server-platform detection, store integrations that expose
  product features over client credentials without an OAuth account link, and
  location-aware product lookups (Kroger 3.0.0). No add-on source changed.

## [0.1.11]

### Security

- Patches a high-severity .NET denial-of-service in a transitive dependency
  of the passkey/WebAuthn library (`Microsoft.Bcl.Memory`, CVE-2026-26127),
  pinned to 9.0.14 in the canonical image (`1.0.0-beta.12`). This rebuilds
  the add-on on the patched base. No add-on source changed.

## [0.1.10]

### Changed

- First-run setup is smoother under the add-on: the **public URL** field is
  now optional (the web UI never needs it — it only matters for external /
  mobile clients), and the **timezone** field is pre-filled from the
  server's detected zone, which Home Assistant sets to your home's zone,
  instead of defaulting to UTC. Fixed in the canonical image
  (`1.0.0-beta.11`); this rebuilds the add-on on it.

## [0.1.9]

### Fixed

- Admin-only actions returned "forbidden" under Ingress: the server setup
  wizard's Next button and the Settings → Users page (which showed no
  users). Under Ingress every request was authenticated from the HA
  identity headers, whose principal carried no role claim, so role-gated
  endpoints rejected even the admin user. The server now prefers the
  signed-in JWT (which carries roles) when the client presents one, falling
  back to the header identity only for the SSO handshake. Fixed in the
  canonical image (`1.0.0-beta.10`); this rebuilds the add-on on it.

## [0.1.8]

### Added

- Home Assistant single sign-on now works end-to-end: opening the add-on
  signs you in automatically as your HA user instead of showing a login
  page. The server already trusted Supervisor's `X-Remote-User-*` headers;
  this adds the client side — the SPA exchanges that trusted identity for a
  normal session on startup. Fixed in the canonical image (`1.0.0-beta.9`);
  this release rebuilds the add-on on that base. No add-on source changed.

## [0.1.7]

### Fixed

- After the UI loaded inside the HA sidebar it immediately redirected to a
  404 (the HA-root `/login`) and couldn't navigate. The Blazor app used
  root-absolute paths (`/login`, `/settings`, …), which under Ingress
  resolve against the HA host root instead of the add-on's ingress prefix,
  so every navigation escaped the add-on. Fixed in the canonical image
  (`1.0.0-beta.8`): in-app navigation is now base-relative so it resolves
  against the ingress base path. This release rebuilds the add-on on that
  base. No add-on source changed.

## [0.1.6]

### Fixed

- The add-on UI hung on the "Loading application…" splash (with a broken
  logo) inside the HA sidebar. The Blazor WASM shell ships a static
  `<base href="/">`, so under Ingress — served from
  `/api/hassio_ingress/<token>/` — the browser resolved `_framework/*` and
  `_content/*` against the HA root, which Supervisor doesn't route to the
  add-on, so the app never booted. Fixed in the canonical image
  (`1.0.0-beta.7`) with middleware that rewrites the shell's `<base href>`
  to the Ingress prefix; this release rebuilds the add-on on that base. No
  add-on source changed.

## [0.1.5]

### Fixed

- On arm64 hosts (Raspberry Pi) the scheduler service hot-looped with
  `/usr/local/bin/supercronic: cannot execute binary file: Exec format
  error`. The binary comes from the canonical base image
  (`mtherienfamick/homemanagement`), whose Dockerfile had the same
  `ARG TARGETARCH=amd64` default-shadowing bug — the arm64 base baked in an
  x86_64 supercronic. Fixed in the base image (republished as
  `1.0.0-beta.6`); this release just rebuilds the add-on on top of the
  corrected base. No add-on source changed.

## [0.1.4]

### Fixed

- First-boot Postgres init aborted with `CREATE DATABASE cannot be executed
  from a function`, stopping the container. The `cont-init.d/01-postgres`
  script wrapped `CREATE DATABASE famick` in a `DO $$ ... $$` block, which
  Postgres forbids (it can't run inside a function/transaction context). The
  database is now created idempotently via psql's `\gexec`, which runs the
  statement at the top level. The role creation (which is allowed in a DO
  block) is unchanged.

## [0.1.3]

### Fixed

- The `aarch64` image bundled an x86_64 `s6-overlay`, so on arm64 hosts
  (Raspberry Pi 4/5) `/init` failed with `s6-overlay-suexec: Exec format
  error` and the add-on never started. Cause: the Dockerfile declared
  `ARG TARGETARCH=amd64`, and the default value shadows buildx's
  per-platform value, so the aarch64 build leg still resolved `amd64` and
  downloaded the wrong s6-overlay. The default is removed (with a
  `dpkg --print-architecture` fallback for plain `docker build`), so each
  arch now gets the matching s6-overlay binary. The amd64 image was
  unaffected.

## [0.1.2]

### Fixed

- First-boot bootstrap was failing on v0.1.0 and v0.1.1 because the
  `/app/scripts/bootstrap.sh` it expected wasn't yet shipped in the
  canonical Famick image. v0.1.2 inlines the bootstrap logic directly
  into `cont-init.d/02-famick-bootstrap`, removing the canonical-image
  dependency entirely. The add-on now completes its first start without
  any manual intervention.

## [0.1.1]

### Changed

- Install now pulls a prebuilt image from ghcr.io instead of building
  locally on the HA host. Download takes ~30 seconds versus ~15 minutes
  for the v0.1.0 local-build path. No behavior change inside the
  container.

## [0.1.0]

Initial release.

### Added

- Bundled single-container build of Famick Home Management for Home Assistant
  OS / Supervised, with Postgres 16 and s6-overlay process supervision.
- HA Ingress + single sign-on: opening the add-on from the HA sidebar
  signs you in as your HA user automatically. Existing Famick users whose
  email local part matches the HA username are linked rather than recreated.
- Multi-arch support: `amd64` (x86 HA hosts) and `aarch64` (Raspberry Pi 4/5).
- All state lives in Supervisor's `/data` mount — Home Assistant's built-in
  backup captures it whole.

### Known limitations

- HA Container / Core installs cannot run add-ons. Use the
  [docker-compose deployment strategy](https://github.com/Famick-com/FamickHomeManagement/tree/main/self-hosted/docker-compose)
  instead.
- First install builds the image on your HA host; expect ~15 minutes on a
  Raspberry Pi 4. A future release will publish prebuilt images so install
  is a quick download.
