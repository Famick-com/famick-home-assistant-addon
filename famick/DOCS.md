# Famick Home Management

Famick Home Management is a self-hosted household management application:
inventory and stock tracking, equipment and vehicle maintenance, recipes
and meal planning, contacts, chores, todos, and shopping lists. It's a
single sign-on companion to your Home Assistant install — open it from
the sidebar and you're already signed in as your HA user.

## How sign-on works

When you open the add-on from the HA sidebar, Home Assistant's Supervisor
authenticates you at the edge and forwards your identity (`X-Remote-User-Id`,
`X-Remote-User-Name`, `X-Remote-User-Display-Name`) to Famick. The first
HA user to open the add-on becomes the Famick admin; subsequent HA users
get the Editor role.

If you already have a Famick account on the mobile app whose email local
part matches your HA username (for example, you log into the mobile app as
`alice@example.com` and your HA username is `alice`), the add-on links the
two automatically — you see the same data in both places.

## Where your data lives

Everything mutable is in Supervisor's `/data` mount:

- `/data/postgres/` — the bundled Postgres 16 cluster
- `/data/keys/jwt-rsa.pem` — the RSA key Famick uses to sign session JWTs
  (generated on first boot, persists across restarts)
- `/data/config/tenant-id` — the per-install household identifier
- `/data/uploads/` — files you've attached to equipment, contacts, etc.
- `/data/dataprotection/` — ASP.NET data protection keys

## Backing up

Home Assistant's built-in **Settings → Backups** captures `/data` for every
installed add-on. A full backup of Famick is included in any HA backup.
Restoring from backup restores the database, the JWT key, and any uploaded
files — sessions issued before the backup remain valid.

## Configuration

The add-on has one optional setting:

| Option | Default | Description |
|---|---|---|
| `log_level` | `info` | Verbosity for the add-on's startup and supervision messages. Famick's own application log level is unaffected and follows the app's defaults. |

## Updating

When a new version is published, the Home Assistant Add-on Store offers
an in-place upgrade. The Famick database keeps its existing schema —
migrations are applied automatically on the next start. If the upgrade
fails for any reason, restoring the most recent HA backup brings the
previous version back exactly as it was.

## Troubleshooting

### The add-on log shows `/app/scripts/bootstrap.sh missing`

The add-on relies on the canonical `mtherienfamick/homemanagement:latest`
image containing a bootstrap script that ships from v0.1.0 onward. If you
see this error, the underlying image is older than the wrapper expects.
Wait for the next image release, or pin to an older add-on version.

### Postgres won't start

Check the add-on log for the `[01-postgres]` lines. The most common cause
is filesystem corruption in `/data/postgres/`. Restoring a recent HA
backup is the cleanest recovery path; deleting `/data/postgres/`
manually will let the add-on re-initialize an empty database on next boot
(you lose all data).

### The Famick UI loads but every link 404s

Make sure you accessed it via the HA sidebar (Ingress), not by typing a
direct port URL. The add-on intentionally doesn't expose a port; URLs
are generated assuming the Ingress prefix.

## Limitations

- HA Container and HA Core installs can't run add-ons. Use the
  [docker-compose deployment strategy](https://github.com/Famick-com/FamickHomeManagement/tree/main/self-hosted/docker-compose)
  on those hosts.
- The bundled Postgres is sized for a single household. If you want to
  share an external Postgres with other workloads, watch for a future
  "Famick (external Postgres)" variant.

## Support

- Issues: <https://github.com/Famick-com/famick-home-assistant-addon/issues>
- Famick application source: <https://github.com/Famick-com/FamickHomeManagement>
