---
name: card0-game
description: |
  Umbrella skill for all card0 game work — creating AND modifying games. Route the task to the right sub-skill: build a new game end-to-end from a YouTube tutorial video to card0-game-create; edit, inspect, validate, preview, or submit an EXISTING game to card0-cli; ALL image generation (covers, card art, card backs) to card0-cli.

  Trigger on any request about card0 games: "make this video a card0 game", "turn this video into a card0 game", "fix the cards in game X", "update game X's rules", "validate my manifest", "generate card art", "submit game Y", "make a 中文版 / 日本語版".
---

# card0-game — dispatch

## Card0 product model — read before planning

Create a playable tabletop component kit, not a custom rules engine. Players handle communication, private consultation, moving/covering cards and rules adjudication as at a physical table. Auto setup handles initial preparation only. Read [../card0-game-create/references/tabletop-model.md](../card0-game-create/references/tabletop-model.md) before deciding what to build or declaring a game unsupported. Ordinary manual play is not a rules deviation.


This is the entry point for card0 game authoring. Dispatch to exactly one sub-skill based on the task; the sub-skills are installed alongside this one — invoke them by name with the Skill tool.

| Task | Sub-skill |
|---|---|
| Build a new game end-to-end from a YouTube video (Stages 1-10, incl. multi-language builds) | `card0-game-create` |
| Edit, inspect, validate, preview, or submit an existing game (metadata, decks, cards, rule images) | `card0-cli` |
| Generate any image — cover, card fronts, card backs, rule visuals | `card0-cli` |

Notes on how the sub-skills interlock:

- `card0-game-create` drives a complete build itself and already instructs when to use the card0 CLI for draft creation, paid image generation, validation, and submission. Once dispatched to it, follow it exactly — do not re-dispatch per stage.
- For edit/modify/submit requests on a game that already exists, go straight to `card0-cli`. Do NOT rebuild from scratch.
- Image-only requests (regenerate a cover, make a card back) go straight to `card0-cli`.

## Card-back policy (summary)

Card backs must be textless and language-neutral so one back serves every language variant. Reuse a back from the shared library only when its theme/art style/palette genuinely matches THIS game — when running inside the workbench, the workbench prompt gives the library path (default `<workbench-root>/assets/card-backs/`). If nothing matches, generate ONE theme-matched textless back and donate it to the library. Full ruling: `card0-game-create` Stage 5.
