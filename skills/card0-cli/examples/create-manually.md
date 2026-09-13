# Create manually

```sh
card0 game create "Tiny Debates" --description "Argue cheerful, impossible positions." --rules "Draw and defend." --min-players 3 --max-players 8
card0 deck create "Prompts"
card0 deck use "Prompts"
card0 card create "Cats run city hall" --description "Argue in favor for thirty seconds."
card0 card create "Breakfast at midnight" --description "Argue against for thirty seconds."
card0 game validate
card0 game open
```

Stop after preview unless the user explicitly asks to submit.
