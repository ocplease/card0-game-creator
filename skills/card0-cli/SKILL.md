---
name: card0-cli
description: "Use the card0 CLI around a Card0 3D game, signed in as the user: log in, read the account's user id, push a games/<slug>/ folder (checked and saved as the user's draft or edit copy), check its status, submit it for review, list its decks and cards, generate or upload the cover and card art (paid, 50 credits each), inspect and remove images, and open a game in the browser. The game folder itself is written with the card0-3d-game skill."
---

# Card0 CLI (account, art, preview)

Card0 games are 3D games: rules as code, written in a `games/<slug>/`
folder with the `card0-3d-game` skill. This CLI publishes them as the
signed-in user (`card0 game push` / `card0 game submit`) and covers what is
done from a terminal around them. Don't use it to build a game out of decks and
cards: `card0 game create`, `card import`, `card number-range`, manifests and
setup recipes belong to the retired 2D card-kit workflow.

Never query or change Card0's database with this skill, and never ask for
service-role credentials. In Card0's code a game is also called a **work**.

## Setup

See [scripts/install.md](scripts/install.md), [commands/login.md](commands/login.md)
and [commands/account.md](commands/account.md). `card0 account info` prints the
logged-in user's id: that is the `creatorId` in a game's `game.json`.

## Publish a 3D game (as the user)

```sh
card0 game push games/<slug>             # checked in Card0's sandbox, saved as the user's draft / edit copy
card0 game status games/<slug>           # status, current build, open edit copy
card0 game submit games/<slug> --yes     # to review, only when the user asks
```

See [commands/game.md](commands/game.md). The user's login decides whose
game it is; there are no database keys or SQL in this flow.

## Find a game's cards

```sh
card0 game list
card0 game show <gameId>
card0 deck list --game <gameId>
card0 card list --deck <deckId>
```

Decks and cards come from the game's `game.json` (by key) when it is
published; their ids are what the image commands take.

## Art

Read [references/image-pipeline.md](references/image-pipeline.md) before
generating anything: costs, the draft-only rule, sample approval, and how the
3D table draws card images. Commands: [commands/game.md](commands/game.md),
[commands/card.md](commands/card.md).

## Preview

`card0 game open <gameId>` opens the game in the browser (the normal Card0
login applies).

## Rules for agents

- Successful commands print JSON on stdout; errors go to stderr with a nonzero exit code. See [references/output-and-context.md](references/output-and-context.md).
- Treat `card0 --help` and subcommand help as authoritative when they differ from this skill.
- Only `draft` and `private` games can be changed through the CLI (that includes generating art).
- Generating images costs credits: state the estimate and stay within what the user authorized.
- `card0 game delete` and `card0 card image delete` are permanent: only on the user's explicit request.
