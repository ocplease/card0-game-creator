# Best practices for Card0 3D games

Learned building 瞎掰王, 一夜狼人 and 双面卧底 on the 3D table. Each rule
comes from a real mistake.

## Make the next step obvious without text

1. **The top of the table is the clock.** While turns simply run, leave `banner` out: the table shows just the clock and the seconds left. Whose move it is shows on the table (`speaking`, an `info` mark); what to do shows in the glowing seats and cards, the action buttons and their tap `prompt`s ("点你怀疑的人").
2. **A banner only for news**, one short line, said once, that the table can't show: the narrator at night ("🌙 预言家请睁眼……"), what just happened to this player ("小白 给了你一张 FLIP 3：点一位玩家"), what a reveal means, a round's result, a new round ("第 2 轮 / 共 3 轮"). Never a standing "轮到你：…" line or a rules reminder.
3. **No phase track, no goal chip.** Leave `phase` and `goal` out: they crowd the top of the table, and in live rooms the stream covers the goal corner. A player's role or word is their own card on the table (hold to look); scores go in `marks`.
4. **One tap where possible.** Use `pick` on seats, cards and the middle row instead of a list of buttons. If only one move exists and it's a tap, the table starts it. Avoid menus of 20 labelled actions.
5. **Suggestions for every text action**, built from what this seat knows: the 全知者 gets the real meaning, a 瞎掰者 gets openings for a believable story ("这是古代的一种……"), a werewolf gets believable lies, a seer gets their actual finding phrased as a claim.
6. **Every game finishes.** A fixed number of rounds or one decisive vote; every waiting step has a timer; a timeout does something sensible (skip the speech: "（没想好，跳过了）"; close the vote with the votes in).
7. **Results say why.** `result.summary` and log lines name who won and why ("投出了狼人 小白，村庄获胜！"); `reveal` opens every secret.

## Hidden information

8. **Build each player's table from what they know, not from the truth.** In 一夜狼人, a player whose card was stolen still sees their old role, as at a real table. Keep per-seat beliefs as secrets and update them on peeks and swaps.
9. **Waiting must not reveal roles.**
   - Night steps last a fixed time even when the role is in the middle or its holder acted at once.
   - If one role doesn't vote, the one player never shown "已投票" is that role. Let them vote, and don't count the vote (瞎掰王: the 全知者 votes too).
   - Speaking order rotates by round, not by role.
10. **Secrets the players must never see stay out of card text.** Answers, meanings and word lists go in a JSON the rules import. Card names and descriptions are public in "In the box".
11. **Private actions are private events.** `to: [seat]` for peeks, steals and swaps, so only that player sees the animation and the line.
12. **Run the leak check** (`npm run game -- check`) after every change to state or view.

## The table and animations

13. **Match the real gesture.** Looking at a card is `peek` (the character picks it up, its back to the others), a swap is `swap`, a reveal is `flip`, a vote is `vote` (everyone points at once, and keeps pointing).
14. **Speech is an event**: `speech: { seat, text }` gives the bubble in that character's voice and the chat line. Don't add a separate speech UI or chat.
15. **Night**: `mood: "night"` and `asleep` on every seat that has its eyes closed. Your own seat asleep closes your eyes, and the banner becomes the narrator's line ("🌙 预言家请睁眼……"). Fellow werewolves are awake together and see each other.
16. **Text cards for words and answers** (`title` + `text`). A word card in the middle row (`center`) shows everyone this round's word.
17. **`announceCards: false`** when lines name roles or cards all the time (a narrator calling "预言家请睁眼"); otherwise every such line pops a card up big.

## AI players

18. **Two kinds of AI player.** In live rooms a language model plays each AI seat: it reads `rules.md`, its own view and the events, and writes its own words. In the dev table and balance runs `bot.ts` plays: free and deterministic.
19. **Give the language model what a person would know.** Put private records in the view (`myNotes` in 一夜狼人: what you saw at night). Keep `rules.md` complete: goal, each phase, scoring, winning, tips.
20. **Bots must play like average humans.** A bot that reads data a player can't see is an oracle and makes one side win every game. Give it a skill knob (`SKILL = 0.4`: chance of spotting the truth) and keep it there. A bot that copies obvious hints (always believes the first "预言家" claim) distorts balance the other way.
21. **Bots must not give themselves away.** In 瞎掰王 bots first chose the longest explanation as the truth, so the 全知者 was obvious. Truth-tellers retell briefly; bluffers use prepared fakes (`words.json` "fakes").

## Balance

22. **Simulate before tuning by feel**: `sim --games 300` for every player count. Team games: each team's win rate per player count. Score games: each seat's share of wins (first-player advantage shows up), shared wins, winning margin.
23. **Read a game before changing rules** (`--log 1`): decide whether the rules or the bot cause the imbalance.
24. **Tune data first**: role mixes per player count (`setups.json`), scores, round count, timers. Change rules only when data can't fix it.
25. **Ties need a rule** that rewards the game's core skill (瞎掰王: the player who fooled the most wins a tie).

## Content and language

26. **One game per language.** All visible text (banner, labels, prompts, log lines, rules.md, card text) in the game's `language`. Keep code identifiers in English.
27. **Content in data, rules in code.** Word lists, role mixes and card text can be tuned without touching `rules.ts`, and the AI studio exposes them as `rules/<name>.ts` in an edit copy.
28. **Words and facts must be real.** For a word/trivia game every entry needs a correct meaning; check them.

## Engineering

29. **Never `Math.random` or `Date`**: `ctx.rng` only, so games replay.
30. **`legal` lists every concrete move**, and `apply` throws for anything out of phase or turn: the server only accepts legal moves, timers arrive with `seat === null`.
31. **Scenarios for every rule that matters**, asserting on `view` (what players see), including refusals and timeouts.
32. **Don't write UI, 3D or networking code for one game.** If a gesture is missing, add it to the platform (TableFx + TableScene) so every game gets it.
