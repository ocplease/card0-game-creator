---
name: card0-game
description: |
  Entry point for all Card0 game work: creating a new game, changing an existing one, balancing it, publishing it, and its cover or card art. Card0 games are played on the 3D table with 3D AI players; this skill routes to the right sub-skill.

  Trigger on any request about Card0 games: "make a card0 game", "做一个 card0 游戏", "turn this party game / video into a card0 game", "change the rules / words / roles of game X", "the bots always win in X", "publish game X", "generate a cover / card art for X".
---

# card0-game: dispatch

Every Card0 game is a 3D game: rules written as code, played on the 3D table
by people and 3D AI characters, with the room, chat, AI host, animations and
HUD supplied by the platform. There is no separate 2D card-kit workflow any
more; don't build games as manifests of printed cards.

| Task | Sub-skill |
|---|---|
| Create a new game (from a description, a rulebook or a video) | `card0-3d-game` |
| Change, fix, rebalance or republish an existing 3D game | `card0-3d-game` |
| Log in, check the account, generate or upload a cover / card art, open a game in the browser | `card0-cli` |

`card0-3d-game` drives a whole build itself and tells you when to use
`card0-cli` (art, account id). Once dispatched to it, follow it through; don't
re-dispatch per stage.
