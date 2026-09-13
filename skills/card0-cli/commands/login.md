# Login commands

Use one of these authentication flows:

```sh
card0 login web --open
card0 login web --provider google
card0 login email --email person@example.com
printf '%s' "$PASSWORD" | card0 login email --email person@example.com --password-stdin
card0 login otp send --email person@example.com
card0 login otp verify --email person@example.com --code 123456
card0 logout
```

`login web` starts a random `127.0.0.1` PKCE callback listener and prints the authorization URL. `--open` launches that URL with the operating system. Passwords must come from the hidden prompt or stdin; never put them in a command argument or log them.

Credentials and refresh tokens live in `${CARD0_CONFIG_DIR:-~/.card0}/credentials.json` with owner-only permissions. Do not copy them into a repository.
