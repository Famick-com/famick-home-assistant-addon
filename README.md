# Famick — Home Assistant Add-on

Self-hosted household management for Home Assistant — inventory, equipment,
vehicles, recipes, chores, contacts — installed as an add-on, accessed
through HA's sidebar with single sign-on to your HA user.

This repository is a thin Home Assistant Supervisor wrapper around the
canonical Famick self-hosted Docker image
(`mtherienfamick/homemanagement`). It contains no Famick application
source: just the add-on manifest, a Dockerfile that adds Postgres 16
and s6-overlay on top of the canonical image, the s6 service definitions
that bring up the three processes (Postgres, Famick web, scheduler), and
the cont-init scripts that bootstrap them on first boot.

The application lives at <https://github.com/Famick-com/FamickHomeManagement>
and is licensed under the Elastic License 2.0. This wrapper repo is
licensed under the same terms — see [`LICENSE`](LICENSE).

## Install

Add this repository to your Home Assistant Supervisor:

1. Open **Settings → Add-ons → Add-on Store**
2. Click the menu in the top-right → **Repositories**
3. Paste `https://github.com/Famick-com/famick-home-assistant-addon` and
   click **Add**
4. **Famick Home Management** appears in the store under the Famick
   section — click **Install**

First install builds the multi-stage image on your HA host. On a
Raspberry Pi 4 expect ~15 minutes; on an x86 host ~5 minutes.

## What you get

- Famick UI in the HA sidebar via Ingress — no port exposure, no
  separate login, accessed at the same URL your HA is on.
- Single sign-on: the HA user clicking through the sidebar is recognized
  as a Famick user automatically. Existing Famick users whose email
  local part matches the HA username are linked rather than recreated.
- Bundled Postgres 16: state lives in Supervisor's persistent `/data`
  volume; HA backups capture it whole.

## Requirements

- Home Assistant OS or Home Assistant Supervised
- ~600 MB disk for the image, plus growing space for `/data`
- amd64 (most NAS / mini-PCs) or aarch64 (Raspberry Pi 4 / 5)
- Container or Core HA installs cannot run add-ons — use the
  [docker-compose strategy](https://github.com/Famick-com/FamickHomeManagement/tree/main/self-hosted/docker-compose)
  instead.

## License

Elastic License 2.0 — see [`LICENSE`](LICENSE).

The Famick name and logo are trademarks of Mike Therien.
