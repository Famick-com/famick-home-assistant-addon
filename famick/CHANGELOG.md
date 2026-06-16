# Changelog

All notable changes to the Famick Home Assistant add-on are documented here.
The format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and the version numbers follow [Semantic Versioning](https://semver.org/).

## [Unreleased]

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
