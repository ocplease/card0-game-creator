# Output, errors, and context

Successful authoring commands write one JSON value to stdout. Treat stderr as human diagnostics and inspect the process exit code. API errors have a stable shape:

```json
{"ok":false,"error":{"code":"DECK_NAME_EXISTS","message":"A deck with this name already exists.","details":{}}}
```

`card0 game use` writes `.card0/context.json` in the current directory. Add `.card0/` to `.gitignore`. Explicit flags override bindings. IDs win over names; exact duplicate names require an ID.

Only `draft` and `private` games can be changed. `pending approval` and `published` games are read-only in the CLI: publish a 3D game as a draft first when it needs art.
