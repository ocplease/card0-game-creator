# Card commands

```sh
card0 card create "Question" --description "What would you choose?" [--deck <deck>] [--quantity 1]
card0 card import ./cards.json [--deck <deck>]
card0 card import - [--deck <deck>]
card0 card number-range 1 20 [--deck <deck>]
card0 card list [--deck <deck>]
card0 card show <card>
card0 card update <card> --back-description "Back text"
card0 card image generate <card> [--face front|back] [--prompt "Complete face description"]
card0 card image list <card> [--face front|back]
card0 card image upload <card> ./art.png [--face front|back] [--mode artwork|full_face]
card0 card image delete <card> <image-id> [--yes]
```

Card import accepts either a JSON array or `{ "cards": [...] }`. Normal cards require a front description. Number ranges are inclusive. Use explicit `--deck` to override the directory binding.

Card fields are `name`, `description`, `backDescription`, `imageUrl`, `style`, `type`, and `quantity`. V1 types are `normal` and `number`.

`card image upload` accepts a local JPEG, PNG, WebP, or GIF (at most 5 MB) for the card's `--face` (default `front`). Before upload, the CLI converts it to WebP and progressively reduces its dimensions and quality until a front image is below 50 KB or a back image is below 20 KB. Only that processed image is stored in Card0's Supabase storage and selected in the app's card image gallery. `--mode` (default `full_face`) sets the render mode. A card face holds at most 8 images; the upload replaces the currently selected image for that face.

Use `card image list` to obtain the exact image variant IDs and inspect `face`, `isDefault`, and `isSelected`. `card image delete` permanently removes one non-default variant and its unreferenced storage object. It requires interactive confirmation, or `--yes` for agents and scripts. The original code-rendered default is protected; deleting a selected variant automatically selects the default face as the fallback.

Generation costs 50 credits on success and saves/selects the result. Omitting `--prompt` uses saved metadata; custom prompts are limited to 2,000 characters. Generated backs apply to the entire deck. See [image pipeline](../../card0-game-create/references/image-pipeline.md) before generation or retry.
