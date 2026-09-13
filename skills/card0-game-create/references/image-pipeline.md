# Image generation through card0

Use only the authenticated card0 CLI for generating or regenerating covers, fronts, backs, accessories, and rule visuals. Do not call an image provider, another image-generation tool/skill, or a local generation script. Card0 handles provider credentials, storage, selection, credit reservation, and refunds.

## Prerequisites and cost

Install `@card0/cli`, authenticate with `card0 login`, and check `card0 account info`. Verify that `card0 game image generate --help` and `card0 card image generate --help` exist; update the CLI if missing. Create and validate the game and cards before generating art.

Each successful generated image costs **50 credits**. Estimate `50 × (covers + unique fronts + unique backs + rule visuals + revisions)` across all languages. Samples count toward the final fronts when reused. State the estimate before generation and honor the user's authorized budget. Insufficient credits stop generation; never bypass billing with direct provider calls. Failed server generations are refunded; after a timeout or uncertain response, inspect the saved cover or card image list before retrying, because the first request may have succeeded and another generation costs again.

## Generate and inspect

```sh
card0 game image generate <gameId>
card0 card image generate <cardId> --face front --prompt "One complete portrait card, exact title, full rules, values, and approved visual style..."
card0 card image generate <representativeCardId> --face back --prompt "Textless ornamental back matching this deck..."
card0 game rule-image generate <gameId>
card0 card image list <cardId>
```

These commands save images remotely; do not re-upload the result to the same face. Record returned IDs and URLs and inspect the actual images. Card generation defaults to an automatic prompt from saved metadata when `--prompt` is omitted. A custom prompt must contain 1–2,000 characters. No batch/count/size/reference-image flags are supported. Generate one unique design per call, sequentially.

Covers use saved game metadata and produce textless 3:4 art. Rule visuals use the saved rules. Neither accepts a custom prompt. Complete card fronts must include exact title, full description, numbers, icons, labels, and layout in the target language, except explicitly text-free designs. Use the layout guidance in [card-layout.md](card-layout.md); dimensions in prompts are design targets, not guaranteed output settings. Do not draw or overlay visible content with code.

For each deck, generate one representative finished front, inspect it, show it to the user, and obtain art-direction approval before the remaining fronts or backs. Reuse accepted samples. Each rejected sample's regeneration is another paid call.

## Shared backs and special reverses

`--face back` generates a **shared deck back**, automatically applying it to sibling cards. Generate it once per deck and inspect sibling image lists afterward; propagation can fail independently of generation. Never use this command for an individual answer, clue, or special reverse in a multi-card deck.

For a card-specific reverse, generate the reverse artwork as a front on a temporary card in a separate temporary deck within the game, using an explicit reverse-side prompt. This is still paid card0 generation. Download the returned image URL and use `card0 card image upload <targetCardId> <file> --face back --mode full_face` to apply it only to the intended card. Keep temporary components out of the final playable manifest; if the installed CLI cannot remove them, report this limitation and finish cleanup through the supported game editor before submission. Do not submit a game with temporary components.

Accessories normally reuse their generated front for the back: download the saved front and upload it to that card's back. Reuse textless shared backs across language variants in the same way. Uploading an existing image performs no generation and incurs no generation charge. Never generate artwork externally and disguise it as an upload.

## Existing image uploads

```sh
card0 card image upload <cardId> ./existing-image.jpg --face back --mode full_face
card0 game image upload <gameId> ./existing-cover.jpg
card0 game rule-image upload <gameId> ./page-01.jpg ./page-02.jpg
```

Card uploads accept files up to 5 MB and automatically compress them to WebP below 50 KB for fronts or 20 KB for backs. Covers must be 3:4 and at most 5 MB. Rule uploads compress to JPEG below 100 KB and append in order. Mechanical conversion/compression is allowed; inspect text after compression. Record every mapping and verify the saved selected images before submission.
