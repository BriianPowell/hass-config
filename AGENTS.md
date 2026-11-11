# hass-config

Home Assistant configuration (YAML), running in the two-node k3s cluster managed in
`k3s-collective`. The live config is a git clone of this repo on the cluster's Home
Assistant volume; changes reach it by pushing here and running `hass_deploy`.

## Verify before you finish

Run these and fix failures before reporting a task done. Report which ones you ran.

- `pre-commit run --all-files`: YAML/JSON syntax, merge markers, private keys,
  secrets scan, forbidden local-only files. Needs no running Home Assistant. (Verified.)
- `hass_deploy -c`: runs Home Assistant's own `check_config` against the live volume
  and pulls nothing. It checks what is deployed, not your uncommitted edits, so run it
  after the maintainer pushes. Needs ssh to the node and `kubectl`. (Unverified here.)

CI (`.github/workflows/ci.yaml`) runs pre-commit and `check_config` on every push and pull request,
against the Home Assistant version in `k3s-collective/apps/base/homeassistant/deployment.yaml`, with
placeholder secrets and `hass-oidc-auth` at the pinned `OIDC_VERSION`. There are no unit tests. Do
not invent a test command.

## Commands

- `hass_deploy [-n] [-y]`: fish function in the `nix-config` repo. Fetches on the node,
  fast-forward pulls, runs `check_config`, rolls back if it fails, then asks before
  restarting. `-n` never restarts. Do not run it without being asked; it restarts a
  production service.

## Layout

- `configuration.yaml`: entry point; almost everything is an `!include` from here.
- `includes/`: one file per integration (`mqtt.yaml`, `prometheus.yaml`, ...).
  `includes/templates/` holds template sensors, merged in numeric order (`10_`, `20_`, ...).
- `automations/`: hand-written automations, every file merged into one list
  (`automation manual:`). Give each one an `id:` so it gets traces.
- `automations.yaml`, `scenes.yaml`, `scripts.yaml` (root): written by the Home Assistant UI
  editors and git-ignored (HA backups cover them). `configuration.yaml` includes them, so a fresh
  clone needs them to exist: `echo '[]' > automations.yaml; touch scenes.yaml scripts.yaml`.
- `ui-lovelace.yaml` and `lovelace/`: the YAML dashboard and card fragments.
- `www/`: custom card JavaScript and icons, served at `/local/`; registered under
  `lovelace.resources` in `configuration.yaml`. Treat as vendored.
- `themes/`, `blueprints/`: themes and stock blueprints.

## Conventions

- Credentials come from `secrets.yaml` (git-ignored) via `!secret`. Never read, print or
  commit it, and never put a token in a tracked file.
- HTTP and trusted-proxy settings are no longer YAML; Home Assistant stores them in
  `.storage`. Do not add an `http:` block (it is ignored).
- OIDC sign-in needs the `hass-oidc-auth` integration, installed from HACS into
  `custom_components/` (git-ignored). A change under `includes/auth_oidc.yaml` can lock
  the maintainer out: show it before suggesting a restart.
- Restart needed: integration config (`auth_oidc`, `mqtt`, `homekit`, ...). Reloadable in
  the UI without a restart: automations, scripts, scenes, templates, dashboards, themes.
- Runtime files (`.storage/`, `home-assistant.*`, `.cloud`, `.HA_VERSION`) are not in
  the repo; the pre-commit hook rejects them if force-added.
- Do not commit or push unless asked.
