# The 3D table (what `view` returns)

`view(projected, seat, ctx)` returns `{ ...projected, table }`. The 3D table,
the HUD and the AI characters are drawn from `table` alone. Build it per seat
from what **that seat** knows. Source of truth: `TableLayout` in
`packages/card0-game-engine/src/contract.ts`.

## Cards

```ts
{ id: "p2", card: "seer", back: "roles", public: false, note: "你查看过" }
```

- `card`: a component card key (from game.json), or `null` for a card this seat can't see (drawn as `back`'s deck back).
- `public: true`: face up for everyone. `public: false` with a known `card`: only this seat knows it. It lies face down and they can hold it to see its face.
- **Text cards** (written during play, or a word card): `{ id, card: null, back: "cards", title: "王不留行", text: "全知者：一味中药……" }`. Leave `text` out for a slip this seat may not read. Text cards don't need images. A word or two prints big; long text shrinks to fit.
- Hold to look: players hold any card to see it big, exactly as the table draws it. You don't do anything for this.

## Seats

```ts
seats[i] = {
  board: [card, …],       // in front of the player, up to 6 slots, slot = index
  hand: [card, …],        // in their hand (others see backs and a count)
  items: [card, …],       // items by the avatar (a gun, a token): kind "item" in game.json
  marks: [{ icon: "⭐", label: "3 分", tone: "ok" }],   // chips on the name tag: danger | warn | info | ok
  status: "已投票",         // short status on the name tag
  speaking: true,         // spotlight: has the floor now
  said: "一种古代的乐器",    // their line this round (kept in state for AI players; shown as a bubble when spoken)
  asleep: true,           // head down (night). Your own seat asleep closes your eyes: night overlay with the banner as narrator
  out: true,              // dimmed
}
```

## The rest of the table

| Field | Use |
|---|---|
| `banner` | **What happens now and what this player should do**, rewritten per phase and per seat ("轮到你解释「葳蕤」" vs "小白 正在解释……"). |
| `goal` | `{ text, icon, card, tone: "good"/"evil"/"neutral" }`: this player's aim in a few words with their role/word card. Changes as the game goes. |
| `phase` | `{ steps: [{ key, label, icon }], current, round?, rounds? }`: the flow as a track. `current: ""` when over. |
| `center` | Middle row: center cards, a public word card, answers to vote on. Pick with `on: "center"`. |
| `piles` | `{ id, label, back, count, card? }` stacks in the middle (deck, the role list of this game). |
| `mood` | `"night"` dims the room. |
| `votes` | `[{ from, to, card? }]` once public: every character points at its choice, and keeps pointing. |
| `arrows` | Persistent arrows between seats (who aims at whom). |
| `announceCards` | `false` when log lines name cards all the time (a narrator calling roles); otherwise a line naming a card pops it up big. |
| `actions[type]` | How an action type is offered (below). |
| `fx[eventType]` | Which animation an event plays (below). |

## Actions: buttons and taps

```ts
actions: {
  vote: { label: "投票", icon: "🗳", tone: "danger", pick: [{ arg: "target", on: "seat", prompt: "点你要投出的人" }] },
  swap: { label: "换两人的牌", icon: "🔀", pick: [{ arg: "a", on: "seat", prompt: "点第一个人" }, { arg: "b", on: "seat", prompt: "再点第二个人" }] },
  peekCenter: { label: "偷看中间", icon: "👀", pick: [{ arg: "card", on: "center", prompt: "点中间一张" }] },
  explain: { label: "解释", icon: "💬", tone: "primary" },   // input action: opens the speech box
}
```

- `pick.on`: `"seat"` (tap a player; arg = seat), `"board"` (a card in someone's area; arg = slot, `seatArg` = that seat), `"myBoard"` (my own area), `"center"` (arg = index in `center`).
- The legal actions grouped by type become buttons; picks narrow them down tap by tap. When only one move is possible and it's a tap, tapping starts by itself.
- Actions with `input` open the speech box (prompt, quick-pick suggestions, character limit).

## Animations (`fx`)

Map an event type to a gesture; field names point into `event.data`
(seats; `-1` = the middle row; slots):

| kind | gesture | fields |
|---|---|---|
| `peek` | `from` picks up a card of `to` (slot) and looks at it, its back to the others | from, to, slot |
| `swap` | two cards change places | from, fromSlot, to, toSlot |
| `flip` | the card at (`at`, slot) turns face up for everyone | at, slot |
| `point` | `from` points at `to` | from, to |
| `vote` | everyone points at their vote (`table.votes`) at once | |
| `shoot` / `aim` | a gun fires at / points at `to` | from, to |
| `shield` / `pulse` | a shield / a glow at `at` | at |
| `dice` | `at` rolls dice; `value` names the field with the roll | at, value |
| `drink` | `at` raises a cup and drinks | at |

```ts
fx: {
  peeked: { kind: "peek", from: "seat", to: "where", slot: "slot" },
  drank: { kind: "swap", from: "a", to: "where", toSlot: "slot" },
  votesShown: { kind: "vote" },
  truth: { kind: "flip", at: "seat", slot: "slot" },
}
```

Pick the kind that matches the real gesture at a physical table. Send private
actions' events `to: [seat]` so only that player sees them happen.

## Speech

An event with `speech: { seat, text }` shows a bubble over that character in
their voice and posts the line in the live chat as theirs. Use it for every
spoken turn (explanations, claims, accusations); don't build your own chat or
speech UI.

## New gestures

If the game needs a gesture the table doesn't have, add a kind to `TableFx`
in `packages/card0-game-engine/src/contract.ts` and play it in
`lib/table3d/TableScene.js` (`playFx`), then every game can use it. Do this
in a separate change to the platform, and say so in your report.

## The game's own props and moves (`animations.json`)

Props in front of a seat (`seats[i].props`, names) and the moves acted out
with them (`chore` fx, `value` = a move name) belong to the game, as data:

```json
{
  "version": 1,
  "models": { "potato": "assets/potato.glb", "knife": "assets/knife.glb", "peel": "assets/peel.glb" },
  "props": { "potato": { "model": "potato", "at": [-0.17, 0.02], "height": 0.036 } },
  "moves": { "peel": { "duration": 2.8, "steps": [
    { "lift": "potato", "time": 0.4 },
    { "spin": "potato", "turns": 2.5, "at": 0.4, "time": 1.9 },
    { "hold": "knife", "at": 0.4, "time": 1.9 },
    { "drop": "peel", "count": 4, "at": 0.6, "time": 1.6 },
    { "return": "potato", "at": 2.35 }
  ] } }
}
```

- Models: GLB, everything embedded, no compression extensions; ≤ 4 MB and 60k triangles each, 250k in all, ≤ 24. Export them from `art/art.js` (`window.CARD0_ART.models`).
- Props: `model`, `at: [x, z]` (metres; x to the seat's right, z toward the middle), `height`, `turn`, `tilt` (lays a standing model down; `lift … upright` stands it up), `scale`. ≤ 8.
- Steps (each starts at `at`, default when the previous ends, and lasts `time`): `lift`, `spin`, `wobble`, `unfold` (named nodes), `hold` (a model in a hand), `drop`, `rise`, `toss` … `into`, `give` (to the event's `to` seat, `wait`), `take`, `knock`, `return`, `body` (jump, stand, stretch, scratch, wave, look, nod, shake, reach, point), `emoji`. ≤ 40 moves, ≤ 24 steps, ≤ 6 s each. Lifted props go back by themselves.
- `propActions` maps a prop name to the action a tap takes (`mine`, `theirs`).
- Names the game doesn't define fall back to the built-in bowl / potato / corn and body moves.
- `npm run game -- check <slug>` validates it like the server; see `games/ahoukiller/animations.json`.
