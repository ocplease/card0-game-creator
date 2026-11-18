# The rules contract (rules.ts)

Source of truth: `packages/card0-game-engine/src/contract.ts` in the card0
repository. Import everything from `"@card0/game-engine"`.

```ts
import {
  defineGame, secret, learn, reveal, peekValue, copiesOf, cardByKey,
  type Action, type Secret, type TableCard, type TableLayout, type TableSeat,
} from "@card0/game-engine";
import WORDS from "./words.json";        // data the rules need, players never see

export default defineGame<State>({
  meta: { name: "…", minPlayers: 3, maxPlayers: 8, startTimer: { afterMs: 60_000, action: { type: "timeoutSpeak" } } },
  setup(ctx)                          // -> initial state
  legal(state, seat, ctx)             // -> Action[]: every concrete move this seat may make now ([] if none)
  apply(state, seat, action, ctx)     // -> { state, events?, timer? }
  waitingFor(state, ctx)              // -> seats the game waits for (several = simultaneous)
  result(state, ctx)                  // -> null while playing, then { winners, winningTeam?, summary, scores?, reveal? }
  view(projected, seat, ctx)          // -> { ...projected, table: TableLayout }
});
```

## Context

`ctx.players` (`{ seat, id, name, bot? }[]`; seat = index), `ctx.components`
(game.json decks: `{ key, name, cards: [{ key, kind, name, description, back, quantity, image, backImage }] }`),
`ctx.rng`, `ctx.options`. `view` gets the same without `rng`.

- Randomness only from `ctx.rng`: `rng()` float, `rng.int(n)`, `rng.pick(list)`, `rng.shuffle(list)`. Never `Math.random` or `Date`: games must replay exactly.
- Deal from components, not hard-coded names: `copiesOf(components, "roles")` expands quantities; `cardByKey(components, "seer").name` gives the display name. Then renaming a card in game.json needs no code change.
- Data players must never see (answers, meanings, word lists) goes in a JSON the rules import. Card text is public ("In the box" shows every card).

## State and secrets

State is plain JSON. Wrap hidden information:

| Call | Meaning |
|---|---|
| `secret(value, [seats])` | a value only those seats know (`[]` = nobody yet) |
| `learn(s, seat)` | one more seat knows it (a peek, an investigation) |
| `reveal(s)` | everyone knows it from now on |
| `peekValue(s)` | the raw value, inside rules code (never in `view` for other seats' secrets) |

Before `view` runs, the platform projects the state for that seat: secrets
the seat knows become their plain value; the others become `{ $hidden: true }`.
So in `view`, test `typeof v.knower === "number"` (known) rather than reading
`.v`. A view cannot leak what it never receives, and `check` runs a leak test
that looks for any secret value in views of seats that don't know it.

Per-player knowledge that changes during play (who a player *thinks* holds
which card, their private notes) is itself a secret per seat:
`beliefs: Secret<Record<number, Role>>[]` with `secret({...}, [seat])`.

## Actions

```ts
{ type: "vote", args: { target: 2 }, label: "投 小白" }
```

- `legal` lists **every** concrete action (one per target/slot). The server only accepts actions from this list, so `apply` can trust its args, but still check the phase and seat and `throw new Error("现在不能…")` for anything else.
- `seat` in `apply` is `null` for timer actions.
- Free text: an action with `input` asks the player for words:

```ts
{ type: "explain", label: "解释", input: { arg: "text", max: 80, prompt: "编一个解释", suggestions: ["…", "…"] } }
```

  `legal` lists it **without** the text arg; the request carries it (`action.args.text`), capped and tidied by the platform. People get a text box with the suggestions as quick picks; live AI players write their own words; bots pass `text`.

## Outcome: events and timers

```ts
return {
  state: s,
  events: [
    { type: "said", data: { seat }, speech: { seat, text } },       // bubble in their voice + chat line from them
    { type: "peeked", to: [seat], data: { seat, where: t, slot: 0 }, text: "小白 的牌是：狼人" }, // private
    { type: "voteOpen", text: "都说完了！投票吧。" },                    // public log line
  ],
  timer: { afterMs: 60_000, action: { type: "closeVote" } },        // replaces the pending timer; null cancels
};
```

- `to: [seats]` sends an event only to them (private animations and lines). Omitted = everyone.
- `text` is a log line in the game's language. `speech` makes a spoken line: the room waits for people to read it before AI players continue.
- `data` holds the fields the event's animation reads (see table-layout.md, `fx`).
- `timer` omitted keeps the current timer; set one at every step that waits for someone. The first step's timer is `meta.startTimer` (it starts after the identity check).

## Result

```ts
{ winners: [0, 3], winningTeam: "village", summary: "投出了狼人 小白，村庄获胜！", scores: { "0": 5 }, reveal: { … } }
```

`winners` drives the results screen and the room's win/lose for each player.
`reveal` is shown at the end with every secret opened.

## scenarios.ts

```ts
import { scenario } from "@card0/game-engine";
export default [
  scenario("only the 全知者 sees the meaning", async (t) => {
    const g = await t.start(5, "seed-name");
    const st = g.state();                         // raw state: secrets as { $secret, v, k }
    const view = await g.view(1);                 // what seat 1 sees, table included
    await g.act(seat, { type: "explain", args: { text: "…" } });
    await g.refuse(other, { type: "explain", args: { text: "抢话" } });   // must be refused
    await g.timeout();                            // fire the pending timer
    t.ok(cond, "message"); t.equal(actual, expected, "message");
    t.ok(g.result());
  }),
];
```

Also `g.legal(seat)`, `g.waitingFor()`, `g.session.eventsFor(seat)`. Force a
deal by assigning `g.state().cards[i].v = "werewolf"` (and the matching
`initial`/`beliefs`) right after `t.start`.

## bot.ts

```ts
import { definePolicy, type SimGame } from "@card0/game-engine";
export default definePolicy(({ seat, view, legal, events, rng, memory, components }) => {
  const speak = legal.find((a) => a.type === "speak");
  if (speak) return { action: speak, text: "我是村民" };
  return { index: rng.int(legal.length) };
});
/** Optional extra columns for BALANCE.md. */
export function tally(game: SimGame) { return { "winning score": … }; }
```

`memory` persists for one seat through one game. `events` are the events this
seat received (what people said, private results).
