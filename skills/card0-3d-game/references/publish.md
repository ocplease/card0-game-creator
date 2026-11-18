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

## Publish

```bash
npm run game -- publish <slug>                 # status "published"
npm run game -- publish <slug> --status draft  # a draft first (for cover/card art)
```

This builds, re-runs the checks and writes `games/<slug>/.build/publish.sql`,
one statement that:
- creates or updates the game row (`runtime = 'engine'`, so it opens on the 3D table), its tags and minutes;
- creates or updates the decks and cards by key, keeping images already generated in the app unless game.json sets them;
- saves a new build (revision n+1) with the bundled rules, the studio-editable source and the check report, and points the game at it.

The SQL must run with owner rights on Card0's database (a maintainer's
`psql`, or the Supabase SQL tool with project-owner access). If you don't have
that access, hand over the file; never ask the user for service-role keys or
passwords. When passing the SQL through a tool that re-transcribes text,
verify the stored result: compare `md5(rules_js)` and the per-file md5 of
`rules_source` against the local `.build/build.json`. Unicode escapes can
silently change in transit. `--lean` stores a rules-only bundle and a
pointer to the repository instead of the source (smaller, but an edit copy
then has nothing to edit); avoid it unless the SQL is too big to send.

## Cover and card art (optional)

The 3D table draws every card itself: a name band, a letter and the
description, or for text cards the title and text. Art is optional.

When the user wants art, use the card0 CLI (`card0-cli` skill; 50 credits per
generated image, state the estimate first) while the game is a **draft**:

1. Publish with `--status draft`.
2. `card0 game image generate <workId>`: the cover (drafts only).
3. `card0 card image generate <cardId> --face front --prompt "…"`: the table draws a card's image edge to edge as the whole card face, so ask for a complete card with its exact title and text in the game's language. One sample per deck first; show it and get approval before the rest.
4. `card0 card image generate <cardId> --face back --prompt "…"`: one textless back per deck (applies to the whole deck).
5. Re-run `npm run game -- publish <slug>` (status published). Images generated in the app are kept.

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
