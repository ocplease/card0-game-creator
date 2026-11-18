# Game commands (3D games)

```sh
card0 game list
card0 game show [game]
card0 game use <game>          # bind the game to this directory (.card0/context.json)
card0 game unuse
card0 game image generate [game]
card0 game image upload [game] ./cover.png
card0 game rule-image generate [game]
card0 game rule-image list [game]
card0 game open [game]
card0 game delete [game]
```

IDs take precedence over names; names must match exactly and uniquely.

## Cover

`game image generate [game]` generates textless 3:4 cover art from the saved
game metadata for 50 credits on success and saves it. It takes no prompt and
works only while the game is a **draft** (publish the 3D game with
`--status draft` first). Inspect the current cover before retrying an
uncertain request: the first one may have succeeded.

`game image upload` sets a supplied cover: JPEG, PNG, WebP or GIF, at most
5 MB, exactly 3:4 (e.g. 1242×1656). Crop it first; other ratios are rejected.

## Rule visual guide

`game rule-image generate [game]` turns the saved rules into a 3:4
infographic shown above the written rules (50 credits on success, refunded on
failure, takes a few minutes, drafts only). Generate it after the rules text
is final; editing the rules marks it stale (`ruleImageStale: true` in
`game show`).

## Delete

`game delete [game]` permanently removes a draft or private game and its
images. Only on the user's explicit request; in a noninteractive shell it
needs `--yes`.
