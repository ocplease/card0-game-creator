---
name: card0-3d-game
license: MIT
description: |
  Create, test, balance and publish a Card0 game on the 3D table: rules as code (TypeScript), played by people and 3D AI characters in a live room. Also for changing an existing 3D game (rules, words, roles, balance, look of the table).

  Trigger on: "make a card0 game", "做一个 card0 游戏", "turn this party game / board game / video into a card0 game", "add a game like 一夜狼人 / 瞎掰王 / Codenames to card0", "the bots always win in game X", "change the rules / words / roles of game X", "publish game X".

  Expected output: a game folder games/<slug>/ in the card0 repository whose rules pass `npm run game -- check`, whose balance report (BALANCE.md) is reasonable, that plays end to end on the dev table, and, when the user asks, is published to Card0.
metadata:
  version: "2.0.0"
  category: game-authoring
---

# Card0 3D games

A Card0 game is **rules + content**. The platform already provides, for every
game: the 3D room and table, dealing/flipping/holding cards, 8 AI characters
(looks, motions, voices, personalities, chat), the lobby, live chat, AI host,
identity check, restart/play again, results, the HUD (turn clock, news banner,
action buttons, tap-to-pick, speech box, night overlay), a
balance simulator and a hidden-information leak check.

You write five things in one folder:

| File | What it is |
|---|---|
| `game.json` | name, players, language, tags, workId, and the cards & items (decks) |
| `rules.md` | how to play in prose: the lobby shows it and the live AI players read it |
| `rules.ts` | the rules: `defineGame({ meta, setup, legal, apply, waitingFor, result, view })` |
| `scenarios.ts` | scripted checks of the rules |
| `bot.ts` | a simple AI player for the dev table and balance runs |
| `*.json` | data the rules import (word lists, role mixes per player count) |

Never write UI code, 3D code, networking, lobby or chat for a game. If the
game seems to need something the table can't show, see "New gestures" in
[references/table-layout.md](references/table-layout.md).

## Prerequisites

- A checkout of the card0 repository (`ocplease/card0`) with `games/`, the
  engine (`packages/card0-game-engine`) and `npm install` done. Read
  `games/README.md` there; it matches this skill.
- Worked examples to copy from, in that repository:
  - `games/xiabai/` (瞎掰王): free-text bluffing, text cards, scoring over rounds.
  - `games/werewolf/` (一夜狼人): night steps, per-player knowledge, swaps, simultaneous vote, role mixes per player count.
  - `games/crossfire/` (双面卧底): hidden teams, items (guns), reactions.
  Start from whichever is closest. Read its `rules.ts` fully before writing yours.
- No repository? People can also build a 3D game in the Card0 app: the create
  flow's AI studio writes the same `rules.ts` / `scenarios.ts` and saves tested
  builds. New games in the app are always 3D games.

## Workflow

Do the stages in order. Each has a "done" line; don't move on before it holds.

### 1. Design on one page

Write down, in the game's language:
- the goal for each side or player, and how the game ends (a fixed number of rounds or one decisive vote; every game must finish);
- the phases in order (e.g. 夜晚 → 发言 → 投票) and, for each, **who acts, what they choose, and how long they get**;
- what each player knows and when (hidden roles, a secret word, a peeked card);
- the components: role cards, word cards, items, and the data the rules need but players must not see (answers, meanings);
- player counts and the role mix per count.

If the user's rules leave a gap (ties, a player who never acts, a role with
nothing to do at 3 players), decide it, write the decision down, and tell the
user. If the user corrects the design, rebuild from their version; don't
patch the old one.

**Done:** you can say, for every phase, what each seat sees and taps.

### 2. Create the folder

`games/<slug>/` with `game.json` first (see
[references/publish.md](references/publish.md) for the fields and the allowed
tags), then `rules.md` (short sections: 目标 / 每一轮 / 计分 / 胜负 / 小技巧 or
Goal / Each round / Scoring / Winning / Tips). Add the game to
`games/index.js` so the dev table can load it.

### 3. Write rules.ts

Read [references/engine-contract.md](references/engine-contract.md) (the five
functions, secrets, timers, events, free-text actions) and
[references/table-layout.md](references/table-layout.md) (what `view` returns
for the 3D table). Then apply every rule in
[references/best-practices.md](references/best-practices.md). The most
important ones:

1. The top of the table is the clock. Whose move it is shows on the table (spotlight, a turn mark); what to do shows in the glowing seats and cards, the buttons and the tap prompts. A banner only for news the table can't show; no phase track, no goal chip. A newcomer never needs `rules.md` to know their next move.
2. Every choice is a tap or a suggestion away; text actions come with `suggestions` built from what this seat knows.
3. Every waiting step has a timer (`timer` in `apply`, `meta.startTimer` for the first step).
4. Secrets are `secret(value, [seats])`; the table each player sees is built from what they know, not from the truth.
5. Waiting must not give anyone away: if a role skips a step, everyone else learns who it is.

**Done:** `npm run game -- check <slug>` reports no crashes, stuck games, refused moves or leaks.

### 4. Scenarios

Write `scenarios.ts`: one scenario per rule that matters (deal and who sees
what, turn order, refusals, every scoring case, ties, timeouts, the end). Force
a deal by editing `g.state()` when a scenario needs specific roles. Check
`view` output, not only state: that's what players see.

**Done:** all scenarios pass in `check`.

### 5. A simple AI player (bot.ts)

Write a policy that plays like an average human, not like an oracle and not at
random: truth-tellers retell the truth briefly, bluffers use believable lies,
voters find the truth with a fixed chance (e.g. `SKILL = 0.4`) that you can
tune. Bots that read data a real player can't see must be limited on purpose.
See `games/xiabai/bot.ts` and `games/werewolf/bot.ts`.

Live rooms don't use bot.ts: there, language-model AI players read
`rules.md` and their own view and write their own words. So `rules.md` must
be complete and the view must carry what a player needs to reason (e.g.
`myNotes` with their private night record).

### 6. Play it and balance it

```bash
npm run web
# then open http://localhost:8081/dev/table?game=<slug>&players=6
npm run game -- check <slug>
npm run game -- sim <slug> --games 300
npm run game -- sim <slug> --games 1 --log 1
```

- Dev table: you play one seat, the other seats run `bot.ts`. No server, login or credits. The panel switches seat (play every role's screen), changes player count, speeds the clock up to 8×, skips a timer, restarts, and shows the referee log (private events included). `&seed=x` deals the same game every time.
- Play at least one full game per distinct role at the smallest and largest player count. Look at it as a newcomer: is the next move obvious without reading anything? Does every animation match what happened?
- `sim` writes `BALANCE.md`: team win rates, or each seat's share of wins for score games. If one side wins far more than intended, read one game with `--log 1` and decide whether the **rules** or the **bot** causes it (bots copying obvious hints, a role that always wins a tie), then fix that one. Tune data (role mixes, timers, scores) before code.
- Headless check: Playwright is available; screenshot the dev table at each phase and look at the screenshots.

**Done:** you played full games on the dev table, `check` is clean, and `BALANCE.md` is within the intended range (say what that range is).

### 7. Publish (only when the user asks)

Follow [references/publish.md](references/publish.md): sign in with the
card0 CLI as the user (`card0 login web --open`), then `card0 game push
games/<slug>`. The game worker checks the rules in its sandbox and saves them
as the user's draft (or the user's edit copy of their published game);
`card0 game submit` sends it to review when the user asks. No database keys
or SQL: never ask the user for them.

Then verify on the real app: the game opens on the 3D table (not the old 2D
table), the lobby shows `rules.md`, AI players join and talk, and a full game
reaches the results screen.

**Done:** game id, build revision, what you verified live, and what you could not.

## Changing an existing 3D game

- In the repository: edit `games/<slug>/`, run `check` and `sim` again, and `card0 game push` (it goes to the user's edit copy of the published game), then `card0 game submit` when the user asks.
- In the app: **Edit** on a published 3D game makes an edit copy that opens in the AI studio with the rules, scenarios and data (`rules/<name>.ts`). Publishing the copy replaces the game and keeps its builds and results. A copy needs a tested build before it can be published.
- Words, roles per player count and timers live in data/constants: change those before touching game logic.

## Report

End with: what the game is (one paragraph), files created, `check` and
scenario results, the balance numbers and what you changed because of them,
what you played on the dev table, and, if published, the game id, build
revision and the live checks done or not done. Say plainly what is untested.
