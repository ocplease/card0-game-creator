# Card commands (art)

```sh
card0 card list --deck <deck>
card0 card show <card>
card0 card image generate <card> [--face front|back] [--prompt "Complete face description"]
card0 card image list <card> [--face front|back]
card0 card image upload <card> ./art.png [--face front|back] [--mode full_face]
card0 card image delete <card> <image-id> [--yes]
```

- `card image generate` costs 50 credits on success and saves and selects the result. Without `--prompt` it uses the saved card metadata; a custom prompt is at most 2,000 characters. `--face back` applies the generated back to the whole deck.
- `card image upload` takes a local JPEG, PNG, WebP or GIF (at most 5 MB), converts it to WebP below 50 KB (front) or 20 KB (back) and selects it. A face holds at most 8 images.
- `card image list` shows variant ids with `face`, `isDefault` and `isSelected`. `card image delete` removes one non-default variant permanently; the default face is protected.

Card names, descriptions and quantities come from the game's `game.json`;
change them there and republish, not with the CLI.

See [image pipeline](../references/image-pipeline.md) before generating or retrying.
