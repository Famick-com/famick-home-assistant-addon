# Changelog

All notable changes to the Famick Home Assistant add-on are documented here.
The format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and the version numbers follow [Semantic Versioning](https://semver.org/).

## [Unreleased]

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
