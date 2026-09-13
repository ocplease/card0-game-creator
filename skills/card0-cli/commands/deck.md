# Deck commands

```sh
card0 deck create "Main Deck" [--game <game>]
card0 deck list [--game <game>]
card0 deck show <deck> [--game <game>]
card0 deck update <deck> --name "New name" [--game <game>]
card0 deck use <deck> [--game <game>]
card0 deck unuse
```

Create/select a game before resolving a deck by name. Explicit `--game` wins over the directory game binding. Deck names must be nonempty and unique within one game.
