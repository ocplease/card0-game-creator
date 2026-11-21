# game.json, publishing, art and edit copies

## game.json

```json
{
  "slug": "xiabai",
  "workId": "19954868-450a-4c3b-a702-72e6f7c82565",
  "creatorId": "<the owner's Card0 user id>",
  "name": "瞎掰王",
  "description": "One or two sentences for the game list (at least 16 characters).",
  "language": "zh-Hans",
  "minPlayers": 3,
  "maxPlayers": 8,
  "estimatedMinutes": 15,
  "categoryTags": ["Party", "Trivia"],
  "mechanicTags": ["Bluffing", "Storytelling"],
  "sourceFiles": ["words.json"],
  "decks": [
    { "key": "cards", "name": "词卡", "cards": [
      { "key": "word", "name": "词卡", "description": "Public text of the card.", "back": "瞎掰王", "quantity": 8 }
    ] }
  ]
}
```

- `language`: exactly `en`, `ja`, `zh-Hans` or `other`.
- `categoryTags`: 1–3 of Party, Family, Strategy, Educational, Puzzle, Social Deduction, Role-playing, Trivia, Kids, Casual.
- `mechanicTags`: 1–3 of Drafting, Bluffing, Set Collection, Memory, Storytelling, Push-your-luck, Cooperative, Dexterity, Matching, Resource Management. Anything else is rejected by the database; `check` validates it.
- `workId`: a new uuid (`uuidgen`) set once before the first publish and never changed. It is the game's id; republishing updates the same game.
- `creatorId`: the Card0 user who owns the game (`card0 account info` prints the logged-in user's id).
- `decks[].cards[]`: `key` (stable; the rules use it), `name`, `description` (public), `back` (text on the back), `quantity`, optional `kind: "item"` for physical pieces (guns, tokens), optional `image` / `backImage` URLs. Keys must stay stable: renaming a card is a `name` change, not a `key` change.
- `sourceFiles`: data JSON files the rules import. They are stored with the build as `rules/<name>.ts` so an edit copy can change them in the AI studio.
- Same-name check: before choosing a name, make sure no other Card0 game already uses it (an old 2D game with the same name confuses players and testers).

## Publish (signed in as the user: `card0 game push`)

Publishing runs as the **user who is signed in to the card0 CLI**. No database
keys, no SQL: the user's login decides whose game it is, and Card0's game
worker does the rest server-side.

```bash
card0 login web --open                 # once; the user signs in in the browser
card0 account info                     # the user id: put it in game.json as creatorId
card0 game push games/<slug>           # check + save: the user's draft (or edit copy)
card0 game status games/<slug>         # status, current build, open edit copy
card0 game submit games/<slug> --yes   # send to review (only when the user asks)
```

Run `card0` from the card0 checkout with `npm run card0 -- <args>` if the
global CLI is older than 0.4 (`card0 --version`).

What `card0 game push` does:
- Reads `game.json` (details and decks), `rules.md` (the lobby's rules text), `rules.ts`, `scenarios.ts`, `notes.md` and the `sourceFiles` data (sent as `rules/<name>.ts`, the AI studio's layout). `bot.ts` and `BALANCE.md` stay in the repository. If `game.json` has no `workId`, it writes a new one there once: keep it.
- Sends them with the user's access token to the game worker, which bundles the rules and runs the platform checks (random playouts at every player count, the hidden-information leak check, your scenarios) in its sandbox, the same checks as the AI studio.
- Where it writes:
  - a new `workId` → a new **draft** owned by the user, played on the 3D table;
  - the user's draft or private game → updated in place;
  - the user's **published** game → the user's **edit copy** of it (made on the first push, reused after). The published game does not change until the copy is submitted and approved.
- Saves the decks and cards by key (images already generated in the app are kept unless `game.json` sets full `https://` image URLs; site paths like `/media/…` are not sent) and a new build: `tested` if the checks pass. If they fail, nothing changes in the game, a `failed` build is recorded, the report is printed on stderr and the command exits nonzero: fix and push again.
- Prints JSON: `workId` (the draft or the edit copy it wrote to), `sourceWorkId` (the published game, for an edit copy), `editCopy`, `created`, `status`, `build` (`revision`, `status`), `checks`.

Going public: `card0 game submit` (or `push --submit`) sends the game to
**review**: a draft becomes `pending approval`; an edit copy replaces the
published game and goes to review, exactly like Edit → Publish in the app.
An admin's approval makes it public. It needs a build that passed the checks.
Submit only when the user asks.

A push is refused (403) when the signed-in user doesn't own the game: check
`card0 account info` against `creatorId`. Never ask for service-role keys,
database passwords or SQL access to get around it.

### Maintainers only: publish.sql

`npm run game -- publish <slug>` still writes `games/<slug>/.build/publish.sql`
(one statement that creates or updates the game, its cards and a published
build). It needs owner rights on Card0's database and skips review, so it is
for Card0's own maintainers, not for agents working for a user. When passing
the SQL through a tool that re-transcribes text, verify the stored
`md5(rules_js)` and per-file `rules_source` against `.build/build.json`.

## Cover and card art (optional)

The 3D table draws every card itself: a name band, a letter and the
description, or for text cards the title and text. Art is optional.

When the user wants art, use the card0 CLI (`card0-cli` skill; 50 credits per
generated image, state the estimate first) while the game is a **draft**
(right after `card0 game push`; the push selects the game for these commands):

1. `card0 game push games/<slug>`.
2. `card0 game image generate <workId>`: the cover (drafts only).
3. `card0 card image generate <cardId> --face front --prompt "…"`: the table draws a card's image edge to edge as the whole card face, so ask for a complete card with its exact title and text in the game's language. One sample per deck first; show it and get approval before the rest.
4. `card0 card image generate <cardId> --face back --prompt "…"`: one textless back per deck (applies to the whole deck).
5. Push again after any change (images generated in the app are kept), then `card0 game submit` when the user asks.

Text cards (words, answers) never need art.

## After publishing: verify live

- The game opens on the **3D table** (if it shows the old 2D table, `runtime` or `current_build_id` is wrong).
- The lobby shows `rules.md`; AI players join, act and talk (their words appear as bubbles and chat lines).
- A full game reaches the results screen; restart works.
- Report what you couldn't verify (for example, no signed-in browser).

## Edit copies (changing a published game in the app)

- **Edit** on a published 3D game makes an edit copy that keeps `runtime = 'engine'` and the current build. It opens in the AI studio with `rules.ts`, `scenarios.ts`, data as `rules/<name>.ts` and `notes.md`.
- The studio saves new builds on the copy; their revision numbers continue after the game's own builds.
- A copy can be published only once it has a tested build. Publishing replaces the game and keeps all its builds, finished-game results and rooms.
- Changes made in the studio are not copied back to `games/<slug>/` in the repository. If both are used, the repository is the source of truth: bring studio changes back by hand before the next `publish`.
