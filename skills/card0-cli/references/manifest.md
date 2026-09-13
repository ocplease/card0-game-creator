# Manifest contract

Use `schemaVersion: 1`:

```json
{
  "schemaVersion": 1,
  "game": {
    "name": "Game name",
    "description": "A description long enough to validate.",
    "imageUrl": "",
    "language": "en",
    "rules": "Rules",
    "minPlayers": 2,
    "maxPlayers": 8,
    "theme": "",
    "setting": "",
    "mechanic": "",
    "vibe": "",
    "categoryTags": ["Party", "Family"],
    "mechanicTags": ["Bluffing", "Push-your-luck"],
    "estimatedMinutes": 20
  },
  "decks": [{
    "name": "Main Deck",
    "imageUrl": "",
    "cards": [{
      "name": "Card name",
      "description": "Front description",
      "backDescription": "",
      "imageUrl": "",
      "style": "default",
      "type": "normal",
      "quantity": 1
    }]
  }]
}
```

Names are trimmed. Game name length is 3–100; description 16–10,000; rules at most 8,000. Language is `en`, `ja`, `zh-Hans`, or `other`. Player counts are integers 1–10 with minimum no greater than maximum. Deck names are nonempty and unique.

Structured metadata is optional in a manifest draft but required for submission: `categoryTags` is 1–3 of Party, Family, Strategy, Educational, Puzzle, Social Deduction, Role-playing, Trivia, Kids, Casual; `mechanicTags` is 1–3 of Drafting, Bluffing, Set Collection, Memory, Storytelling, Push-your-luck, Cooperative, Dexterity, Matching, Resource Management; `estimatedMinutes` is an integer 1–480. Validation warns when they are missing and errors on invalid values.

Card name is required and at most 100; normal-card description is required; back description is at most 140; image URL at most 255; style/type at most 50; type is `normal` or `number`; quantity is an integer 1–32767. Submission also requires at least one deck containing a card.

Validate locally with `card0 game validate --file <path>`. Remote creation validates again and inserts atomically.

## Automatic setup

New submissions require `game.setup` with `version: 1` and exact player-count `variants`. Each variant declares component `pools` and ordered dealing/placement `steps`; do not save a fixed dealt snapshot. Decks/cards accept stable `key` fields, defaulting to their original names. Preserve these keys on renames. See [automatic game setup](setup.md) for the full recipe contract, destinations, and examples. Validate every supported player count and preserve meaningful player choices. Tokens and coins must be represented as cards in decks.
