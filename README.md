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
nohup env CLAWDEX_WORKSPACE_ROOT="$PWD" node "$(npm root -g)/clawdex-mobile/scripts/codespaces-bootstrap.js" > .bridge-bootstrap.log 2>&1 < /dev/null &
```

The bootstrap:

- prepares `.env.secure` for `BRIDGE_NETWORK_MODE=codespaces`
- enables GitHub bearer auth for the current Codespace
- starts the bridge in the background
- re-applies public visibility to bridge ports when possible

The template intentionally does not install Rust anymore. The published
`clawdex-mobile` package already ships a prebuilt Linux bridge binary, so
Codespace startup does not need a toolchain install or local bridge compile.
The template also avoids a custom devcontainer image and feature build. It lets
Codespaces use GitHub's managed default image, which already has the standard
developer tools needed for bootstrap.

## Prebuilds

Prebuilds are optional for this template. The runtime package install is small
enough that GitHub's prebuild image creation and upload can be slower than a
normal Codespace create. If prebuilds are enabled, keep only one retained
prebuild version and avoid unnecessary target regions.

Codespaces waits for `updateContentCommand`, not the post-start bridge bootstrap. That keeps the install inside the prebuild while allowing the editor and app to open as soon as the Codespace is ready. Bootstrap logs live at `.bridge-bootstrap.log`, bridge logs live at `.bridge.log`, and runtime state files are `.bridge.pid` and `.env.secure`.

## Ports

- `8787`: Clawdex bridge
- `8788`: browser preview proxy

The mobile app should connect to the forwarded HTTPS URL:

```text
https://<codespace>-8787.app.github.dev
```
