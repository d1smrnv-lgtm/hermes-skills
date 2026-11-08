# Auth options for Netlify deploys

## Options
1. Interactive (preferred): `netlify login` — opens a browser OAuth flow and stores a session. Keep the session; no token needed.
2. Token: set `NETLIFY_AUTH_TOKEN` to a full, unexpired token, then export it in the same terminal that runs `netlify deploy`.

## The token-masking pitfall
A token, config file, or secret whose value contains a literal `...` inside (for example `eyJhbG...NIn0..`) is truncated or masked — the server rejects it with HTTP 401.

Verify the token first:

    curl -s -o /dev/null -w "%{http_code}\n" -H "Authorization: Bearer $NETLIFY_AUTH_TOKEN" https://api.netlify.com/api/v1/user

Expect `200`. Do not deploy until this passes.

## Deprecated / removed
- `netlify whoami` was removed from newer CLI versions. It prints "not a netlify command" and is not a valid auth check.
