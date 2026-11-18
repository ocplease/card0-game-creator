# Art for a 3D game

Art is optional. The 3D table draws every card by itself: the deck colour, a
name band, a large first letter and the description; text cards (words,
answers) print their title and text. Add art when the user wants it.

## How the table uses images

- A card's front image is drawn **edge to edge as the whole card face** (3:4). So a generated front must be a complete card: illustration plus the exact card name and text in the game's language. The table adds nothing on top.
- A deck's back image is drawn edge to edge on every face-down card of that deck. Keep backs textless and language-neutral.
- Hold-to-look, the identity check and the results screen show the same picture, enlarged.
- Text cards are drawn from their text; they never use images.

## Cost and authorization

Every generated image costs **50 credits** on success (cover, card front,
deck back, rule visual, each revision). Estimate
`50 × (cover + unique fronts + deck backs + rule visual + expected revisions)`,
tell the user, and stay within what they authorized. Cards with quantity > 1
share one image. Failed generations are refunded; after a timeout, check
`card0 card image list` / `card0 game show` before retrying, because the first
request may have succeeded.

Use only the card0 CLI for generation. Don't call image providers or other
generation tools, and don't draw or overlay card text with code.

## Order

1. The game must be a **draft** (`npm run game -- publish <slug> --status draft`). Published games are read-only to the CLI.
2. `card0 game image generate <gameId>`: cover.
3. Per deck, one sample front: `card0 card image generate <cardId> --face front --prompt "One complete portrait 3:4 card. <style>. Exact title '<name>', text '<description>' in <language>. Illustration: <scene>. No watermark."` Inspect it, show it to the user, and wait for approval of the style before the remaining fronts.
4. Remaining unique fronts, one call each, repeating the approved style.
5. One back per deck: `card0 card image generate <oneCardOfTheDeck> --face back --prompt "Textless ornamental back, <emblem>, matching <style>"`. It applies to the whole deck; check a sibling card's image list afterwards.
6. Republish as published: `npm run game -- publish <slug>`. Images generated in the app are kept unless game.json sets `image`/`backImage`.

## Supplied images

The user's own files go up with `card0 card image upload` / `card0 game image upload`
(no generation charge). Covers must be exactly 3:4.

If the backend rejects image input (`400 Model only support text input`), don't
read image files; verify through image lists and leave visual review to the user.
