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
npm run codespaces:bootstrap
```

That bootstrap:

- prepares `.env.secure` for `BRIDGE_NETWORK_MODE=codespaces`
- enables GitHub bearer auth for the current Codespace
- installs `codex` if needed
- starts the bridge in the background
- re-applies public visibility to bridge ports when possible

## Ports

- `8787`: Clawdex bridge
- `8788`: browser preview proxy

The mobile app should connect to the forwarded HTTPS URL:

```text
https://<codespace>-8787.app.github.dev
```

## Local Commands

```bash
npm run codespaces:bootstrap
npm run secure:bridge
```
