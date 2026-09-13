# Automatic game setup

Setup recipes automate initial preparation only. Card0 is a tabletop component kit: players perform the game actions and adjudicate rules. Missing automatic rule enforcement is not a playability blocker. Read [the tabletop model](../../card0-game-create/references/tabletop-model.md) before evaluating limitations; independent word cards and a preprinted positional key do not require correlated generation.

Games store `game.setup`, a recipe rather than a saved deal. The server runs it on Start and Restart for the actual occupied seats, saves one authoritative state, and leaves reconnects unchanged. Existing games with no recipe retain manual setup. New submissions require a recipe. Setup does not automate player decisions such as drafting.

## Recipe v1

```json
{
  "version": 1,
  "variants": {
    "2": {
      "pools": [{ "deck": "main" }, { "deck": "coins" }],
      "steps": [
        { "deck": "main", "target": "each_hand", "count": 5, "selection": "random", "face": "down" },
        { "deck": "coins", "target": "each_player_board", "count": 1, "slots": [0], "selection": "fixed", "face": "up" },
        { "deck": "main", "target": "public", "count": 2, "positions": [{ "row": 0, "column": 0 }, { "row": 0, "column": 1 }], "selection": "random", "face": "up" }
      ]
    }
  }
}
```

Supply an exact variant for each count from `game.minPlayers` through `game.maxPlayers` (1–10). A variant has `pools` and ordered `steps`. There are no formulas or executable scripts.

Decks and cards accept an optional stable `key`; it defaults to the original name when imported. Keys must be unique (cards within their deck). Recipes reference these keys, never usernames or dealt card IDs. Export preserves keys. Keep keys unchanged when translating or renaming display names. At runtime the server resolves them to the deck/card UUIDs for this work and deals distinct instances of each copy.

A pool selects all copies in a deck unless it specifies `cards`, for example:

```json
{ "deck": "roles", "cards": [{ "card": "wolf", "count": 1 }, { "card": "town", "count": 3 }] }
```

A pool may also declare `includeDecks: ["other-deck-key"]` to mix additional source decks into its primary draw/discard pile. Each source belongs to only one pool and card keys must be unique across merged decks. `backDescription: "neutral shared back"` overrides back text for all pieces in that pool without changing authored cards.

This is a four-player mix. Other player-count variants supply their own exact mix. Unlisted decks and unlisted cards in restricted pools are excluded from the session. Unused selected pieces remain in shuffled draw piles. Coins/tokens must exist as cards in decks; component descriptions alone do not create pieces. Recipes support up to 10,000 component copies per game.

For hand targets, optional `ownerFace: "down"` starts the hand hidden even from its owner (default: up). Restart clears local face overrides. This follows the existing trust-based private-flip model; it is not server-side redaction.

Every step specifies `deck`, positive `count`, `selection` (`random` or `fixed`), and `face` (`up` or `down`). Optional `card` selects a specific card key. Deal specific cards before unfiltered steps from that pool so an earlier random draw cannot consume a later required card. Selection is without replacement. `fixed` uses stable instance-ID order; `random` chooses a fresh shuffled sample and may occasionally repeat a prior result.

| Target | Meaning | Additional fields |
| --- | --- | --- |
| `each_hand` | `count` pieces to every seated player's hand | None |
| `each_player_board` | `count` pieces to every player's board | `slots`: one slot 0–5 per piece |
| `player_hand` | Pieces to one occupied player | `player`: zero-based occupied-seat index |
| `player_board` | Pieces to one occupied player's board | `player` and `slots` |
| `public` | Pieces to the public board | `positions`: one `{row,column}` (0–9) per piece |

Occupied-seat indices follow ascending seat numbers: for seats 1 and 3, `player: 1` means seat 3. Each player slot holds one piece; repeated public positions form stacks in step order. The face flag follows existing hand/board rendering, not a new access-control boundary.

Use explicit pools and `steps: []` when no initial placement is prescribed. This keeps the selected decks available and begins at the first player decision. Do not use an empty recipe to bypass preparation required by the game rules. Conditional correlated choices (for example selecting a solution and only compatible clues) are not supported by v1 recipes.

## Authoring and validation

- AI creation accepts `artifact.setup` through creation and metadata tools and retains it through subsequent turns, save, and Undo. Setup is optional, including at submission and readiness checks; validate it when provided.
- CLI manifests include `game.setup` and optional deck/card `key` fields.
- `card0 game update <game> --setup recipe.json` updates an existing draft/private game's recipe through the authoring API.
- `card0 game validate --file manifest.json` checks all advertised variants. `card0 game validate <game>` validates a saved game; submission also allows games without setup.
- Database validation also runs when saving a recipe, changing its player range, and submitting. Start and Restart validate available instances and placements inside the existing room transaction. Failures roll back state, revision, and event together.

Use `target: "discard"` to place setup pieces in their deck’s discard pile. It requires no positions; `face: "down"` preserves a hidden set-aside card.

## Playable setup for every player count

Before dealing, define the exact playable composition of **every deck** for each supported player count. Select only the card types and copy counts allowed for that session; shuffle and deal from that selected pool. Do not deal N random roles from the full inventory when the rules require a particular role mix. Excluded copies must be absent from runtime state and all draw/discard piles, while the authored inventory remains intact.

Check required roles, unique host counts, team ratios, resource supply and information dependencies against the rules. If the rules do not specify a composition, document a reasoned proposal; do not describe it as empirically balanced without playtesting. Preserve reserves and draw decks needed later by the rules—“used” includes the whole session, not just the initial hand.

For Card0 recipes, express player-specific composition with `variants[n].pools[].cards: [{card: "stable-key", count: k}]`; omit excluded card types (do not use count 0). A pool without `cards` includes the entire authored deck and needs a gameplay reason. For a role deck fully dealt at setup, selected counts must sum to the player count and leave no extra roles in draw/discard. Randomize assignment after selecting the composition.

Validate every supported count on Start and Restart: exact role/card histograms across the entire runtime state, excluded copies absent, required roles present, hand/public/board counts, legitimate reserves, hidden faces, conservation and no duplicates. Randomness checks alone do not establish playability or balance. Verify the game ID and publication status used by the actual play entry point; a draft passing tests does not enable its published source.
