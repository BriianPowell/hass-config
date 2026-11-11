# Home Assistant

My Home Assistant configuration, running in a two-node k3s cluster
([k3s-collective](https://github.com/BriianPowell/k3s-collective)). Sign-in goes through Keycloak
via [hass-oidc-auth](https://github.com/christiaangoossens/hass-oidc-auth).

## How a change ships

1. Edit here and run `pre-commit run --all-files`.
2. Push to `main`. CI runs the same hooks plus Home Assistant's `check_config`, against the version
   the cluster runs, with placeholder secrets.
3. Run `hass_deploy` (a fish function in
   [nix-config](https://github.com/BriianPowell/nix-config)). It pulls on the node, checks the
   config there, rolls back if the check fails, and asks before restarting.

## What is not in git

- `secrets.yaml`: real values for every `!secret`.
- `automations.yaml`, `scenes.yaml`, `scripts.yaml`: written by the UI editors. Hand-written
  automations go in `automations/` instead.
- `.storage/` and `custom_components/`: integrations, users and HACS installs, covered by Home
  Assistant backups.

See [AGENTS.md](AGENTS.md) for the layout and conventions.
