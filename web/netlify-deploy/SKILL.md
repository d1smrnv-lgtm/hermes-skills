---
name: netlify-deploy
description: "Deploy a plain HTML/CSS/JS site to Netlify (no build step)."
version: 1.0.0
author: Hermes Agent
license: MIT
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [deploy, netlify, static-site, web, frontend]
    related_skills: [blocked-page-recovery]
---

# netlify-deploy

Deploy a static frontend site (HTML/CSS/JS, no build step) to Netlify: scaffold the files, verify every asset locally (all HTTP 200), then production deploy.

## When to Use This Skill

Use this skill when the user asks to publish a static site (HTML/CSS/JS, no build step) to Netlify, or to put any frontend project on the platform with a plain `publish`/`netlify.toml` setup.

Do not use it for a bundled app (React/Vue/etc. with a real build pipeline) — that needs its own build step and `package.json`; ask the user for the project config first.

## Standing rules

- Reply in Russian to this user, and keep site copy simple and readable.
- Build the site first, verify every asset locally (all HTTP 200), then deploy. Never deploy before the local check.
- Authenticate explicitly: `netlify login` (interactive browser) is preferred. Never rely on a token that came out masked/abbreviated from a config file — it returns HTTP 401 and fails the deploy.
- If a step cannot be completed (missing credentials, permissions), state the exact missing piece and stop. Do not emit a canned refusal like "not processed" or invent tool results.
- The site is done only when the deploy URL prints from the deploy output and loads in a browser.

## Procedure

1. Scaffold the project folder. Add the drop-in files:
   - `index.html` — page shell, links `styles.css` and `script.js`.
   - `styles.css` — page styling.
   - `script.js` — interactivity; wrap DOM-ready code in an IIFE or `DOMContentLoaded`.
   - `netlify.toml` — `publish = "."`, plus a 200 rewrite `/* → /index.html`.
2. Verify locally: run `python3 -m http.server <port>` in the project folder, then curl each asset and require HTTP 200.
3. Authenticate and confirm it stuck. After auth, run `netlify status` and require it to print the user (Name/Email) and a `Current project`. If there is no `Current project`, create one: `netlify sites:create --name <site-name> --json` (it auto-links to the folder you run it in).
   - Interactive (preferred): `netlify login` (browser OAuth).
   - Token: `export NETLIFY_AUTH_TOKEN=<full-token>`. A token containing a literal `...` inside is broken — verify it first by requesting `https://api.netlify.com/api/v1/user` and requiring HTTP 200.
4. Create or link the site. New site, auto-linked: `netlify sites:create --name <site-name> --json`. Existing project on the account: `netlify link` (or `netlify init`), then re-run `netlify status` until `Current project` matches.
   - Note: `sites:create --name` appends a random suffix to the given name, e.g. `ai-explorer` → `ai-explorer-591`; read the returned `url` for the deploy target.
5. Deploy: `cd <project> && netlify deploy --prod --dir=. --json`. Copy the printed `url`.
6. Open the URL once and confirm pages/scripts/CSS load before reporting done.

## Completed example

- Site scaffolded for this user: `C:\Users\smirndv2\ai-site` (index.html, styles.css, script.js, netlify.toml). Static, Russian copy, no build step. Was blocked only by an expired/masked auth token — fully verified locally.

## References

- `references/dev-check.md` — local 200 verification snapshot.
- `references/auth.md` — auth options and the token-masking pitfall.
