# Create from a manifest

Write the manifest to a temporary or neutral project location, then:

```sh
card0 game validate --file /tmp/tiny-debates.json
card0 game create --file /tmp/tiny-debates.json
card0 game show
card0 game open
```

Creation is atomic and always creates new Card0 IDs; it does not replace an existing game.
