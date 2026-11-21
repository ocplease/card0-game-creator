# Game commands (3D games)

```sh
card0 game push [dir] [--submit] [--yes]
card0 game status [game|dir]
card0 game submit [game|dir] [--yes]
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
`status` and `submit` also take a `games/<slug>/` folder (its `game.json`
`workId`).

## Push (publish as the user)

`game push [dir]` sends a `games/<slug>/` folder to Card0's game worker with
the signed-in user's token. The worker bundles `rules.ts` and runs the
platform checks (playouts, leak check, scenarios) in its sandbox, then saves:

- a new `workId` → the user's new **draft** (3D table);
- the user's draft or private game → updated in place;
- the user's **published** game → the user's **edit copy** (the published game is untouched until the copy is submitted and approved).

Details come from `game.json` and `rules.md`; decks and cards by key (images
generated in the app are kept; only full `https://` image URLs in game.json
are sent). It writes a `workId` into a `game.json` that has none, and selects
the game it wrote to for the image commands below. Output: `workId`,
`sourceWorkId`, `editCopy`, `created`, `status`, `build`, `checks`. Failed
checks print the report on stderr and exit nonzero (a `failed` build is
recorded; the game itself is unchanged). A 403 means the signed-in user
doesn't own the game.

## Submit for review

`game submit [game|dir]` sends a 3D game to review once its build passed the
checks: a draft becomes `pending approval`; a published game's edit copy
replaces it and goes to review, like Edit → Publish in the app. An admin's
approval makes it public. Only when the user asks; noninteractive shells
need `--yes`. `game push --submit --yes` does both.

## Cover

`game image generate [game]` generates textless 3:4 cover art from the saved
game metadata for 50 credits on success and saves it. It takes no prompt and
works only while the game is a **draft** (push the 3D game first: a new
game is a draft; a published one gets an edit copy). Inspect the current cover before retrying an
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
