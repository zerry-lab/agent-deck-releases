# agent-deck-releases

Release artifacts for [Agent Deck](https://github.com/zerry-lab/agent-deck) — a Tauri desktop app for managing multiple Claude Code and Codex CLI accounts on macOS.

The source repository is private. Built `.app` bundles are published here so they can be distributed via Homebrew tap (`zerry-lab/tap`).

## Install

Apple Silicon, macOS Sonoma or later.

```bash
brew tap zerry-lab/tap
brew install --cask agent-deck
```

If macOS Gatekeeper blocks the first launch (the app is ad-hoc signed, not Developer ID notarized):

```bash
brew reinstall --cask --no-quarantine agent-deck
```
