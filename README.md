# clawdex-codespace

Minimal GitHub Codespaces template repo for Clawdex.

Purpose:

- create a user-owned fork quickly
- start a Codespace with the right devcontainer bootstrap
- run the Clawdex Rust bridge in Codespaces without pulling the full mobile app repo

## What Happens In Codespaces

On create/resume, the devcontainer runs:

```bash
npm install --include=dev
npm run codespaces:bootstrap -- --prepare-only
npm run codespaces:bootstrap
```

`--prepare-only` front-loads the slow first-boot work during Codespace creation:

- prepares `.env.secure` for `BRIDGE_NETWORK_MODE=codespaces`
- installs `codex` if needed
- prebuilds the Rust bridge binary without starting it

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

## Local Commands

```bash
npm run codespaces:bootstrap -- --prepare-only
npm run codespaces:bootstrap
npm run secure:bridge
```
