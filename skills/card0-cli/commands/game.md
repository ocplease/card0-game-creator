# Game commands

```sh
card0 game create "Game name" --description "At least sixteen characters"
card0 game create "Game name" --description "..." --rules "..." --category Party --category Family --mechanic-tag Bluffing --minutes 20
card0 game create --file /tmp/card0-game.json
card0 game list
card0 game show [game]
card0 game update [game] --description "Updated description"
card0 game update [game] --category Kids --minutes 15
card0 game use <game>
card0 game unuse
card0 game validate [game]
card0 game validate --file /tmp/card0-game.json
card0 game image generate [game]
card0 game image upload [game] ./cover.png
card0 game rule-image generate [game]
card0 game open [game]
card0 game submit [game]
card0 game delete [game]
```

IDs take precedence. Names must match exactly and uniquely. `game create --file` always creates a new game and Card0 generates all IDs. `game use` binds the game in the current directory and clears a deck binding that belongs to another game.

## Structured metadata

Submission requires all four metadata dimensions; set them on `game create` or `game update`:

- `--category <tag>` — repeatable, at most 3, from: Party, Family, Strategy, Educational, Puzzle, Social Deduction, Role-playing, Trivia, Kids, Casual.
- `--mechanic-tag <tag>` — repeatable, at most 3, from: Drafting, Bluffing, Set Collection, Memory, Storytelling, Push-your-luck, Cooperative, Dexterity, Matching, Resource Management. (The free-form `--mechanic <text>` flag is a separate legacy description field.)
- `--minutes <number>` — estimated play time, an integer from 1–480.

Invalid tags or out-of-range values fail the command with the accepted values listed; canonical casing is applied automatically (for example `--category party` becomes `Party`). `game show` reports the stored values as `categoryTags`, `mechanicTags`, and `estimatedMinutes`.

## Rule visual guide

`card0 game rule-image generate [game]` turns the current rules into a child-friendly 3:4 infographic (shown above the written rules in the app). Requirements and behavior:

- The game must be a draft (or private) with non-empty rules; no other generation may be running for it.
- Costs **50 credits**, charged only on success — storyboard, moderation, provider, download, upload, or finalization failures refund automatically.
- Generation takes up to a few minutes (text storyboard, then image render).
- Output is `{ workId, imageUrl, stale }`; the URL is also stored on the game as `ruleImageUrl` (see `game show`).
- Editing the rules afterwards marks the visual stale (`ruleImageStale: true` in `game show`) but keeps it visible; regenerating replaces the old image and clears the flag.

`game image upload` sets the cover for a game. Covers must be JPEG, PNG, WebP, or GIF, at most 5 MB, with a 3:4 width-to-height ratio (for example 1242x1656); other ratios are rejected. If an image needs adjusting, crop or resize it first rather than relying on the app to crop it.

Preview URL precedence is `--web-url`, `CARD0_WEB_URL`, then persisted CLI configuration. `game open` launches the create-flow editor at `/create-flow?mode=edit&workId=<id>`. Draft/private/pending previews use the browser's normal Card0 login; no CLI token appears in the URL.

Submission reruns server validation and changes the game to `pending approval`. It is the only authoring action in this CLI that requires explicit user authorization. In a noninteractive environment, use `--yes` only after that authorization. Submission validation requires the four metadata dimensions and rejects a game without them.

## Delete

`card0 game delete [game]` permanently removes a draft (or private) game: its database rows (decks, cards, image records) and its uploaded images. Pending-approval and published games are read-only and cannot be deleted. The command asks for confirmation interactively; in a noninteractive environment it requires `--yes` — like submission, treat that as needing explicit user authorization. If the deleted game was bound as the active game in the current directory, the binding is cleared.

`game image generate [game]` generates and saves textless 3:4 cover art from saved game metadata for 50 credits on success. It accepts no prompt flag. Inspect the current cover before retrying an uncertain request.
