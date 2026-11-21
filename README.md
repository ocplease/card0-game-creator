# card0-game-creator

Agent skills for creating, testing, balancing and publishing Card0 games.
Every Card0 game is a **3D game**: rules written as code (TypeScript), played on
Card0's 3D table by people and 3D AI characters. The platform supplies the
room, table, animations, AI players, chat, AI host, lobby, restart and results;
a game supplies rules and content.

## Skills

Copy all three folders in `skills/` into your agent's skill directory (for
Codex, `~/.codex/skills/`) and keep them side by side.

- `card0-game`: entry point; routes a request to the right skill.
- `card0-3d-game`: the whole workflow for a game: design, `game.json`, `rules.ts`, scenarios, a simple AI player, dev-table play, balance simulation, publishing, edit copies. Includes the engine contract, the 3D table layout reference and the best practices.
- `card0-cli`: login, account id, pushing and submitting 3D games, and paid cover / card art through the card0 CLI.

## Requirements

- A checkout of the card0 repository (`ocplease/card0`) with `npm install` done: the engine, the `games/` folder, `npm run game` and the dev table (`/dev/table`) live there. Its `games/README.md` is the same guide in short.
- Node.js 20.9+ and the card0 CLI (`npm install -g @card0/cli`, or `npm run card0 -- <args>` in the card0 checkout), signed in as the user (`card0 login web --open`). Publishing (`card0 game push`, then `card0 game submit`) runs as that user: the game worker checks the rules and saves them as the user's draft or edit copy; no database keys. Each generated image costs 50 Card0 credits.

The previous 2D card-kit workflow (manifests of printed cards, auto-setup
recipes, `card0 game create --file`) has been retired.
