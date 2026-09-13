---
name: card0-cli
description: "Author Card0 games through the card0 CLI: authenticate, create or import games, manage decks and cards, validate drafts, preview them, and submit them for approval. Use whenever a user asks an agent to create, edit, inspect, validate, preview, or submit Card0 content with the CLI."
---

# Card0 CLI

## Card0 product model — read before planning

Create a playable tabletop component kit, not a custom rules engine. Players handle communication, private consultation, moving/covering cards and rules adjudication as at a physical table. Auto setup handles initial preparation only. Read [../card0-game-create/references/tabletop-model.md](../card0-game-create/references/tabletop-model.md) before deciding what to build or declaring a game unsupported. Ordinary manual play is not a rules deviation.


Use `card0` for every Card0 authoring operation. Never query or mutate Card0's Supabase tables directly and never request service-role credentials.

Card0 calls a complete authored experience a **game**; older app/database code may call it a **work**. A game contains named **decks**, and decks contain **cards**. Cards can be normal front/back cards or number cards and may have a quantity.

## Image generation and credits

All image generation must use the card0 CLI and charge Card0 credits. Each successful cover, card face, shared deck back, or rule visual costs 50 credits. Read [image-pipeline.md](../card0-game-create/references/image-pipeline.md) for commands, cost estimates, sample approval, deck-wide back behavior, and safe retries. Never call providers or other image-generation tools directly. Upload commands are only for user-supplied or previously generated images.

## Workflow

1. Confirm the executable with `card0 --help`. Treat it and subcommand help as authoritative when this skill differs from an installed CLI version.
2. Authenticate only when needed. See [commands/login.md](commands/login.md) and [commands/account.md](commands/account.md).
3. Create or select a game, then create or select a deck. See [commands/game.md](commands/game.md) and [commands/deck.md](commands/deck.md).
4. Add and update cards. See [commands/card.md](commands/card.md).
5. Set the structured metadata (categories, mechanics, estimated minutes) on every game—submission requires all of them. `card0 game rule-image generate` can also produce a 50-credit visual guide for the rules when the user asks for one. See [commands/game.md](commands/game.md).
6. Run validation and fix every error before presenting the draft as complete.
7. Use `card0 game open` when the user wants a browser preview.
8. Run `card0 game submit` only after the user explicitly authorizes submission. Creating or validating a draft is not submission authorization.
9. Run `card0 game delete` only after the user explicitly asks to delete a game. Deletion is unrecoverable (rows and images), so "clean up" or "fix this game" is not delete authorization — confirm first. Only draft/private games can be deleted.
10. Use `card0 card image list` before deleting a card image so the exact variant ID and selected/default state are visible. Run `card0 card image delete` only for images the user clearly asked to remove; it is permanent and the default face is protected.

Successful commands emit JSON to stdout. Progress and errors use stderr. Inspect exit status before trusting output. Directory bindings are stored in `.card0/context.json`; explicit `--game` and `--deck` flags override them. See [references/output-and-context.md](references/output-and-context.md).

For bulk work, prefer a schema-versioned manifest and validate it before remote creation. Put generated manifests in a temporary directory or a neutral user project location—never in Card0's built-in game/source directories. See [references/manifest.md](references/manifest.md).

Use these walkthroughs when useful:

- [examples/create-manually.md](examples/create-manually.md)
- [examples/create-from-manifest.md](examples/create-from-manifest.md)
- [examples/validate-and-submit.md](examples/validate-and-submit.md)

For auto setup, read [references/setup.md](references/setup.md). Define and test each deck’s playable composition for every supported player count before random dealing; surplus roles must be excluded from the session, not left in draw piles.
