# Couch to 2.5K Trainer

A single-page interval running trainer: pick a week, hit start, follow the
audio-cued run/walk timer. No build step, no dependencies, works offline.

## Files

| File | Purpose |
| --- | --- |
| `index.html` | The whole app (HTML + CSS + JS, self-contained). |
| `manifest.webmanifest` | Makes it installable ("Add to Home Screen"). |
| `sw.js` | Service worker — offline caching. |
| `icon.svg` | Browser tab / favicon (scalable). |
| `icon-192.png`, `icon-512.png` | PWA / Android install icons (maskable). |
| `apple-touch-icon.png` | iOS home-screen icon (180×180). |
| `_headers` | Cloudflare Pages caching rules. |
| `wrangler.toml` | Only needed for the Worker deploy option. |

## Run locally

Service workers and the Wake Lock API need a secure context, so open it over
`http://localhost`, not `file://`:

```sh
cd runapp
python3 -m http.server 8080   # then visit http://localhost:8080
```

## Deploy to Cloudflare

### Option A — Pages (recommended for a static site)

Fastest path, no account-side build config:

```sh
cd runapp
npx wrangler pages deploy . --project-name couch-to-2-5k
```

Or via the dashboard: **Workers & Pages → Create → Pages → Upload assets**,
drag in the `runapp` folder, done. Either way you get a
`https://couch-to-2-5k.pages.dev` URL (add a custom domain under the project's
**Custom domains** tab).

### Option B — Worker with static assets

Only worth it if you plan to add server-side routes later. `wrangler.toml` is
already set up to serve the folder from the edge:

```sh
cd runapp
npx wrangler deploy
```

### Option C — Auto-deploy from GitHub (recommended)

This folder is its own standalone GitHub repo (app files at the root), connected
to Cloudflare Pages' native Git integration. No CI secrets, no runner — Cloudflare
builds on every push and gives each pull request a preview URL.

**Push it to GitHub** (once a repo exists on GitHub):

```sh
cd runapp
git remote add origin git@github.com:<you>/couch-to-2-5k.git
git push -u origin main
```

**Connect Cloudflare Pages:** dashboard → **Workers & Pages → Create → Pages →
Connect to Git** → pick the repo, then set:

| Setting | Value |
| --- | --- |
| Framework preset | None |
| Build command | *(leave empty)* |
| Build output directory | `/` |

Save and deploy. Pushes to `main` publish to production; PRs get preview URLs.
Add a custom domain under the project's **Custom domains** tab.

> `wrangler.toml` here is only for the manual Worker path (Option B) and is
> unused by Pages. If the Git connection ever errors on it, either delete it or
> add `pages_build_output_dir = "."` to it.

## Mobile / real-world notes

- **Keep the screen on.** The app requests a [Screen Wake Lock](https://developer.mozilla.org/docs/Web/API/Screen_Wake_Lock_API)
  when a workout is running (supported on Chrome/Edge/Android and recent iOS
  Safari). Where it isn't granted, the app tells you to keep the screen on
  yourself — a locked screen freezes browser timers.
- **Timing is drift-proof.** The timer is computed from timestamps, so if the
  tab does get throttled or backgrounded and then returns, it re-syncs to the
  correct time instead of losing seconds.
- **Audio caveat (honest):** beeps are generated with the Web Audio API and fire
  only while the page is actually running in the foreground. No browser can play
  a scheduled web beep from a fully backgrounded/locked tab, so pair the wake
  lock with an armband or keep the phone awake and visible during intervals.
- **Installable + offline:** once loaded, it caches itself and runs with no
  signal. Bump `CACHE` in `sw.js` whenever you change a file so clients update.
- Selected week and completed-workout checkmarks persist in `localStorage`.
