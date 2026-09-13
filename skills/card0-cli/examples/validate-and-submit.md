# Validate and submit

```sh
card0 game validate
card0 game show
```

Show the validation result and draft to the user. Only after the user explicitly authorizes approval submission:

```sh
card0 game submit
```

In a noninteractive agent process, the authorized equivalent is `card0 game submit --yes`. A repeated submit of an already-pending game is idempotent.
