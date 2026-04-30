# agent-deck-releases

Release artifacts for [Agent Deck](https://github.com/zerry-lab/agent-deck) — a Tauri desktop app for managing multiple Claude Code and Codex CLI accounts on macOS.

The source repository is private. Built `.app` bundles are published here so they can be distributed via Homebrew tap (`zerry-lab/tap`).

## What is it

Capture the OAuth credentials of any account you've already logged into via `claude` or `codex` CLI, save it under a memorable alias, and switch the "active" account in one click — no re-running `/login`. Includes:

- Per-account usage gauges (5h + weekly) that match what claude.ai / chatgpt.com show
- Auto-rotation when one account hits a threshold (e.g. switch off the 95% Pro account to a backup)
- Auto-capture watchdog that mirrors any account you log into externally
- Encrypted vault export / import for moving accounts between machines
- Optional **Remote Server mode** (v0.3.0+) — point multiple Macs at a self-hosted `ad-server`; the server holds the vault and coordinates leases so two Macs can't fight over the same alias

## Install

Apple Silicon, macOS Sonoma or later.

```bash
brew tap zerry-lab/tap
brew install --cask agent-deck
```

If macOS Gatekeeper blocks the first launch (the app is ad-hoc signed, not Developer ID notarized), drop the quarantine attribute:

```bash
xattr -dr com.apple.quarantine "/Applications/Agent Deck.app"
```

or reinstall bypassing quarantine:

```bash
HOMEBREW_CASK_OPTS=--no-quarantine brew reinstall --cask agent-deck
```

## What it stores

All app state lives under `~/.agent-deck/`:

- `vault/{claude,codex}/<alias>/` — captured account snapshots (OAuth bundles in cleartext; same trust model as the macOS Keychain item these mirror)
- `rotation.json`, `rotation_state.json` — auto-rotation config + history
- `usage_history/`, `usage_latest/` — usage trend per (provider, alias)
- `health.json` — last "Check now" result per account
- `daemon.log`, `daemon.err` — LaunchAgent stdout/stderr (rotation daemon)
- `remote.json`, `machine.json` — only present if Remote Server mode is enabled

The macOS Keychain item that `claude-code` writes (`Claude Code-credentials`) is what gets overwritten when you click Switch — that's the actual auth surface. `claude` re-reads it at the start of every turn, so switches take effect on the next message without needing to restart `claude`.

## CLI

The bundle ships an `ad` binary inside the `.app`. Click **Settings → CLI → Install CLI to PATH** to symlink it onto your `$PATH`. Then from any terminal:

```bash
ad list                       # captured accounts
ad current                    # which alias is active per provider
ad switch claude work         # switch active alias
ad usage claude work          # fetch live usage
ad capture claude work        # snapshot the currently logged-in claude account as "work"
ad rotate --once              # run one rotation pass
ad health                     # check all accounts
ad export ./vault.json        # passphrase-encrypted backup
ad import ./vault.json        # restore
```

## Auto-rotation daemon

Settings → Daemon → Install registers a launchd LaunchAgent (`com.cyj.agent-deck.rotate`) that runs every 60 seconds, checks usage, and switches to the next account in your priority list when the active one crosses the configured threshold. Manual switches (the in-app button or `ad switch`) are honored for 30 minutes before the daemon will auto-rotate again, so it never fights you.

## Remote Server mode (v0.3.0+)

Optional — point the app at a self-hosted `ad-server` and the vault, switch logic, usage polling, rotation, and health all flow through HTTPS instead of the local disk. Useful when you have two or three Macs that should share the same set of accounts without each running its own keepalive (which would race over the OAuth refresh-token).

Settings → **Remote Server (self-host)** → enter the URL + bearer token (minted on the server with `ad-server token mint`) → Activate. A blue banner confirms Remote mode is active and the local rotation daemon is auto-disabled. Disable to fall back to local mode at any time.

The server is a separate Rust binary (`ad-server`) that you run yourself — there's no hosted service. Setup guide and systemd unit live in the source repo's `docs/server-deployment.md`.

## Uninstall

`brew uninstall --cask agent-deck` cleans up the app, the launchd LaunchAgent, and any CLI shim our installer placed on `$PATH`. Captured accounts and config (`~/.agent-deck/`) are intentionally preserved so reinstalling brings them back. To wipe everything including the vault:

```bash
brew uninstall --cask --zap agent-deck
```

If you uninstalled by dragging the .app to Trash instead of through brew, the LaunchAgent will keep firing every minute against a missing binary. To clean up manually:

```bash
launchctl unload ~/Library/LaunchAgents/com.cyj.agent-deck.rotate.plist
rm ~/Library/LaunchAgents/com.cyj.agent-deck.rotate.plist
rm -f /opt/homebrew/bin/ad /usr/local/bin/ad   # CLI shim, if it was installed
```

## Issues / requests

File issues at https://github.com/zerry-lab/agent-deck/issues (issues are mirrored from the private source repo).
