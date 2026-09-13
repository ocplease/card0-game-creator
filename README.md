# card0-game-creator

Agent skills for creating, editing, validating, and submitting playable card0 games.

## Install

Install Node.js 20.9 or newer and the card0 CLI:

```sh
npm install -g @card0/cli
card0 login
```

Copy all three folders in `skills/` into your agent's skill directory (for Codex, `~/.codex/skills/`). Keep them alongside each other so relative references resolve.

- `card0-game`: routes game authoring requests.
- `card0-game-create`: tutorial-to-game workflow, artwork, localization, and setup.
- `card0-cli`: command guidance for existing and new games.

## Image generation

All generated images use card0 CLI commands and Card0 credits: **50 credits per successful image**, including samples and revisions. Uploading supplied or reused images does not generate art. No standalone image-provider skill is included.

Requires a CLI release exposing `card0 game image generate` and `card0 card image generate`; verify both with `--help`. If the npm release lacks them, these skills require the updated CLI before generation can proceed. Do not use a direct-provider fallback.

The backend saves generated images automatically. Generated backs apply to the whole deck. Read the image pipeline reference before generating images or estimating costs.
