# Localization Guide (en / zh-Hans / ja)

The user often asks for a "中文版" or "日本語版" of a game already built in English. This is **not** a metadata swap — you must rebuild the manifest in the new language **and** regenerate the artwork so the printed text on each card matches the target language.

## When to do it

- The user says "做一个中文版本" / "再做一个 ja 版本" / "translate to Chinese/Japanese" of a game that already exists.
- The user explicitly asks for a multi-language launch ("create in en, zh, ja from the start").

## Pipeline

1. **Translate the manifest** — name, description, rules, every card's `name` and `description`. Keep `value`, `quantity`, `type` unchanged. Use idiomatic phrasing, not word-for-word.
2. **Re-create the game** with `card0 game create` using the translated manifest. The CLI returns a new `gameId`. (Don't try to mutate the English game in place — the schema may not support that and the audit trail is cleaner if each language is its own game.)
3. **Re-generate the cover** with card0 CLI and the localized title rendered on the image.
4. **Run the sample gate in the target language**: generate one complete localized card0 CLI card per deck, show the samples to the user, and wait for explicit deck-style approval.
5. **After approval, re-generate every remaining card** with card0 CLI and the localized card names, ability text, numbers, and labels rendered directly on the image. Never add localized text with code. An "English animal" inside a "Chinese game" looks wrong.
6. **Preserve the reverse-side mapping**. Explicit game requirements take priority. Otherwise generate each deck's own textless shared back; localize content-bearing per-card backs; and use each accessory's localized front as its back.
7. **Re-upload** to the new game.
8. **Optionally submit** (usually hold off until the user reviews).

## Common game terms

| English | 中文 (zh-Hans) | 日本語 (ja) | Notes |
|---|---|---|---|
| Player | 玩家 | プレイヤー | |
| Turn | 回合 | ターン | |
| Round | 轮 | ラウンド | oasis card grid rounds |
| Score / Point | 分 / 分数 | ポイント / 点 | pick one and be consistent |
| Hand | 手牌 | 手札 | |
| Draw | 抽牌 | ドロー | |
| Discard | 弃牌 | 捨てる | |
| Deck | 牌堆 | 山札 | |
| Reveal | 翻出 | 公開する | |
| Win | 胜利 | 勝利 | |
| Lose | 失败 | 敗北 | |
| Oasis | 绿洲 | オアシス | |
| Portal | 传送门 | ポータル | |
| Positive | 正面 / 加分 | プラス | |
| Negative | 负面 / 减分 | マイナス | |
| Animal | 动物 | 動物 | |
| Desert | 沙漠 | 砂漠 | |

## Card text conventions

- **Chinese**: use the *short* form. 2-4 character names when possible. No spaces between CJK characters. Use traditional Chinese punctuation only if your game's target audience is Hong Kong / Taiwan — otherwise use full-width comma `，` and period `。`.
- **Japanese**: 2-6 character names typical for card games. No full-width spaces between kanji. Hiragana ok for flavor text. Use 「」 for quoted ability text, not " ".
- **Numbers**: render in Arabic numerals (`+1`, `+2`, `-3`). Don't translate to Chinese/Japanese digits — they're harder to read at card-game speed and card0 CLI is much better at drawing the Latin form.
- **Ability text**: keep under 30 characters per line on the card image. The card frame is small; long text gets clipped.

## Prompting card0 CLI in CJK languages

When the prompt language is English but the on-card text must be Chinese/Japanese, add an explicit instruction:

```
All {Chinese|Japanese} text on the image must be perfectly clear and correctly rendered.
Use the EXACT characters given below — do not substitute similar characters.
```

Then provide the EXACT character strings in the per-card bullet list. Avoid:
- Rare CJK characters (card0 CLI will hallucinate shapes for them)
- Mixed scripts in a single line
- Anything past U+4E00 basic CJK block unless you've tested it

If a particular character keeps garbling, swap to a synonym (e.g. 死亡追逐者蝎子 → 死神蝎子 if the original is hard).

## Layout adjustments

- CJK characters are visually denser than Latin. Reduce the on-card font size for the bottom ability text by ~20% in your prompt ("use 80% the size of the name" etc.) or the text will overflow.
- Japanese mixed kana+kanji needs slightly more horizontal space per character. Plan for shorter ability text.

## Parallelism

You may generate the one-per-deck samples for all requested languages in parallel, then show them together and wait for approval. After every relevant deck sample is approved, generate remaining images one at a time with explicit card IDs, accounting for 50 credits per image. Serialize per-language uploads (card0 image upload isn't parallel-safe at high concurrency — keep it to one upload at a time, with a small `sleep 1` between calls).

## Verification

After localizing, open the new game in the browser (or via the card0 app). Check:
- Cover image: title in the target language
- Each card: name and ability text in the target language, no garbled characters
- Card backs: each deck keeps its distinct mapped back; content-bearing per-card backs are localized; accessories use front = back unless the source game says otherwise
- Card values: still Arabic numerals
- Manifest: passes `card0 game validate --file manifest_xx.json`
- Rules: idiomatic, not machine-translated
