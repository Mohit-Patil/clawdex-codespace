# clawdex-codespace

Minimal GitHub Codespaces template repo for Clawdex.

This template no longer vendors the Clawdex scripts or Rust bridge source. The
Codespace installs the published `clawdex-mobile` npm package during
`postCreateCommand`, then runs the packaged Codespaces bootstrap from there.
The template is currently pinned to the `internal` npm dist-tag for testing.

## What Happens In Codespaces

On create/resume, the devcontainer runs:

```bash
npm install -g clawdex-mobile@internal @openai/codex
CLAWDEX_WORKSPACE_ROOT="$PWD" node "$(npm root -g)/clawdex-mobile/scripts/codespaces-bootstrap.js" --prepare-only
CLAWDEX_WORKSPACE_ROOT="$PWD" node "$(npm root -g)/clawdex-mobile/scripts/codespaces-bootstrap.js"
```

`--prepare-only` front-loads the slow first-boot work during Codespace creation:

- prepares `.env.secure` for `BRIDGE_NETWORK_MODE=codespaces`
- ensures `codex` is available
- prebuilds the bridge binary from the installed package without starting it

The normal bootstrap then:

- enables GitHub bearer auth for the current Codespace
- starts the bridge in the background
- re-applies public visibility to bridge ports when possible

Bridge logs live at `.bridge.log` in the repo root. Runtime state files are `.bridge.pid` and `.env.secure`.

## Ports

- `8787`: Clawdex bridge
- `8788`: browser preview proxy

The mobile app should connect to the forwarded HTTPS URL:

```text
https://<codespace>-8787.app.github.dev
```
