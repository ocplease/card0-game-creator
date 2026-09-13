# Output, errors, and context

Successful authoring commands write one JSON value to stdout. Treat stderr as human diagnostics and inspect the process exit code. API errors have a stable shape:

```json
{"ok":false,"error":{"code":"DECK_NAME_EXISTS","message":"A deck with this name already exists.","details":{}}}
```

Validation returns `valid`, `errors`, and `warnings`; each error has `path`, `code`, and `message`. Validation failures use a nonzero exit status even though the validation document is valid JSON.

`card0 game use` and `card0 deck use` write `.card0/context.json` in the current directory. Add `.card0/` to `.gitignore`. Explicit flags override bindings. IDs win over names; exact duplicate names require an ID.

Only `draft` and `private` games can be changed. `pending approval` and `published` games are read-only in CLI v1.
