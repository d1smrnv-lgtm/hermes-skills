# Local verification snapshot

Run this before any Netlify deploy to prove the site is self-contained (no build step, no missing assets):

    cd <project>
    python3 -m http.server 8123
    for f in index.html styles.css script.js; do
      curl -s -o /dev/null -w "%{http_code} $f\n" http://localhost:8123/$f
    done

All three lines must print `200`. Anything else means a missing asset or a wrong path.

Watch out:
- `netlify.toml` rewrites `/* → /index.html` must not break the root URL.
- `script.js` must not rely on relative imports that only exist after a real build.
