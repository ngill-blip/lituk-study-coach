# LITUK Study Coach — rules for AI agents (READ BEFORE TOUCHING ANYTHING)

This app is small but easy to break in non-obvious ways. Past agents have broken it
three times by doing the "obvious" thing. Follow these rules exactly.

## The system in one paragraph

A single static `index.html` (plain HTML + vanilla JS, **no build step**) is hosted on
GitHub Pages. It calls a tiny Node proxy on Render, which holds the Groq API key and
forwards chat requests to Groq. The browser never talks to Groq directly.

- Frontend repo: `ngill-blip/lituk-study-coach` → live at https://ngill-blip.github.io/lituk-study-coach/
- Proxy repo: `ngill-blip/lituk-proxy` → live at https://lituk-proxy.onrender.com (Render service `lituk-proxy`)

## RULE 1 — GitHub Pages serves the `gh-pages` branch, NOT `main`

Any change to the live website MUST be committed to the **`gh-pages`** branch.
Editing `main` will appear to succeed but will NOT change the live site.
After committing, the Pages rebuild takes ~30–90s; hard-refresh before judging.

## RULE 2 — Do NOT rebuild or re-bundle the frontend

`index.html` is self-contained and hand-written. There is no `src/` to compile.
Do not run `npm run build`, do not introduce Vite/React, do not replace it with a
compiled bundle. Edit `index.html` directly.

## RULE 3 — The frontend must call the proxy at its FULL URL

Correct:  `const PROXY = "https://lituk-proxy.onrender.com/claude";`
WRONG:    `"/api/claude"` or any relative path.
A relative path resolves to the GitHub Pages host, which has no backend, and returns
**HTTP 405**. This is the single most common way this app gets broken.

## RULE 4 — The Groq API key lives ONLY in Render, never in code

The proxy reads `process.env.GROQ_API_KEY`. If you rotate the key in the Groq console,
you MUST paste the new value into Render → service `lituk-proxy` → Environment, then
deploy. The key is shown only once at creation — capture it immediately.
Never hardcode a key in `server.js` or the frontend.

## RULE 5 — Keep the proxy contract stable

The proxy exposes `POST /claude`.
Request body: `{ system: string, messages: [{role, content}], max_tokens?: number }`
Response: `{ content: [{ type: "text", text: string }] }`  (Anthropic-style shape)
CORS is `*`. It must keep handling `OPTIONS` preflight.
If you change the route or response shape, you must update `index.html` to match.

## Known quirks (not bugs — do not "fix")

- Render free tier sleeps after ~15 min idle, so the first request can take ~50s.
  The UI already shows a "server waking up" hint. This is expected.
- The model is `llama-3.1-8b-instant` (a current Groq model). Changing it is optional.

## How to test after any change

1. Confirm the served file is yours:
   `https://ngill-blip.github.io/lituk-study-coach/index.html` should contain
   `lituk-proxy.onrender.com` and NOT `/api/claude`.
2. Load the site, enter a profile name, click "Get a Random Fact".
3. A fact + memory hook + quiz should appear within a few seconds (allow ~50s on a cold start).
4. If you see "Proxy error 405": the frontend is calling a relative path or the wrong
   branch was deployed (see Rules 1 and 3).
5. If you see "Proxy error 401": the Groq key in Render is wrong (see Rule 4).

## Current working version

A known-good copy of `index.html` is kept as `lituk-index-WORKING-2026-06-02.html`
in this folder, and tagged in the repo. To restore, copy it back over `index.html`
on the `gh-pages` branch and commit.
