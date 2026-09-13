# Account commands

Check the active account before editing private games when identity is uncertain:

```sh
card0 account info
```

The command returns the authenticated user ID, email, and providers as JSON. An authentication failure means the agent should ask the user to choose a login flow; it must not seek database keys.
