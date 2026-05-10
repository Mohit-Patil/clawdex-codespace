# clawdex-codespace

Minimal GitHub Codespaces template repo for Clawdex.

This template does not vendor the Clawdex scripts or Rust bridge source. The
Codespace installs the published `clawdex-mobile` npm package during
`updateContentCommand`, then runs the packaged Codespaces bootstrap from there.
`updateContentCommand` is intentionally used instead of `postCreateCommand` so
GitHub Codespaces prebuilds can bake the package install into the restored
environment. The template is currently pinned to the `internal` npm dist-tag
for testing.

## What Happens In Codespaces

On create or prebuild refresh, the devcontainer runs:

```bash
npm install -g --no-fund --no-audit clawdex-mobile@internal @openai/codex
```

On create and on later start/resume, it runs:

```bash
env CLAWDEX_WORKSPACE_ROOT="$PWD" node "$(npm root -g)/clawdex-mobile/scripts/codespaces-bootstrap.js" > .bridge-bootstrap.log 2>&1
```

The bootstrap:

- prepares `.env.secure` for `BRIDGE_NETWORK_MODE=codespaces`
- enables GitHub bearer auth for the current Codespace
- starts the bridge in the background and waits until `/health` responds
- re-applies public visibility to bridge ports when possible

The template intentionally does not install Rust anymore. The published
`clawdex-mobile` package already ships a prebuilt Linux bridge binary, so
Codespace startup does not need a toolchain install or local bridge compile.
The template also avoids a custom devcontainer image and feature build. It lets
Codespaces use GitHub's managed default image, which already has the standard
developer tools needed for bootstrap.

## Prebuilds

Prebuilds should include the `updateContentCommand` install, so a restored
Codespace only needs to run the post-start bridge bootstrap. Keep one retained
prebuild version and avoid unnecessary target regions; prebuild creation can
still take several minutes because GitHub has to create and upload the template
image.

Codespaces waits for `postCreateCommand`, so API-created Codespaces do not report ready before the bridge bootstrap has run. The bootstrap itself starts the bridge in the background, waits until `/health` responds, and fails visibly if the bridge cannot become healthy. Bridge logs live at `.bridge.log`, and runtime state files are `.bridge.pid` and `.env.secure`.

## Ports

- `8787`: Clawdex bridge
- `8788`: browser preview proxy

The mobile app should connect to the forwarded HTTPS URL:

```text
https://<codespace>-8787.app.github.dev
```
