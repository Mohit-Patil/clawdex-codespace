# clawdex-codespace

Minimal GitHub Codespaces template repo for Clawdex.

This template does not vendor the Clawdex scripts or Rust bridge source. The
Codespace installs the published `clawdex-mobile` npm package during
`updateContentCommand`, then runs the packaged Codespaces bootstrap from there.
That makes the package install part of GitHub Codespaces prebuilds instead of
doing it after every new Codespace is created. The template is currently pinned
to the `internal` npm dist-tag for testing.

## What Happens In Codespaces

On create/prebuild, the devcontainer runs:

```bash
npm install -g --no-fund --no-audit clawdex-mobile@internal @openai/codex
```

On start/resume, it runs:

```bash
CLAWDEX_WORKSPACE_ROOT="$PWD" node "$(npm root -g)/clawdex-mobile/scripts/codespaces-bootstrap.js"
```

The bootstrap:

- prepares `.env.secure` for `BRIDGE_NETWORK_MODE=codespaces`
- enables GitHub bearer auth for the current Codespace
- starts the bridge in the background
- re-applies public visibility to bridge ports when possible

The template intentionally does not install Rust anymore. The published
`clawdex-mobile` package already ships a prebuilt Linux bridge binary, so
Codespace startup does not need a toolchain install or local bridge compile.
The template also relies on the default GitHub Codespaces image instead of a
custom image plus devcontainer features, which avoids extra feature image pulls
during creation.

## Prebuilds

Enable a prebuild for the `main` branch in the repository's Codespaces settings.
Use the default prebuild trigger or a schedule that matches internal package
testing. Keep only one retained prebuild version unless you need rollback
coverage.

Bridge logs live at `.bridge.log` in the repo root. Runtime state files are `.bridge.pid` and `.env.secure`.

## Ports

- `8787`: Clawdex bridge
- `8788`: browser preview proxy

The mobile app should connect to the forwarded HTTPS URL:

```text
https://<codespace>-8787.app.github.dev
```
