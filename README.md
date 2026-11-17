# ❄️ WoW Browser Portal — Keldurn

A self-hostable **portal page for [Keldurn](https://keldurn.com)**, the free vanilla-era (1.12.1-compatible) MMORPG that runs in a browser tab. Push this repo to GitHub and you get:

1. **Your own GitHub Pages link** — `https://<your-username>.github.io/wow-browser-portal/` — a personalized entry point to the realm instead of the official URL.
2. **A second copy served on port 5500 from a GitHub Codespace** — the page runs "locally" in a cloud dev container and reaches your browser through a forwarded port (Live Server style, with auto-reload).

Everything is static — no build step, no dependencies, no database.

---

## How the portal presents the game (read this first)

The portal has a **smart embed** engine with three modes (see the `CONFIG` block at the top of the `<script>` section in `index.html`):

| `EMBED_MODE` | Behaviour |
| --- | --- |
| `"auto"` *(default)* | Embed the game full-screen in an iframe **unless** the game host is known to refuse framing, in which case the portal falls back to a clean direct-launch screen ("Launch in new tab ↗ / Take over this tab"). Blocked frames are also detected at runtime. |
| `"always"` | Always attempt the iframe, with stuck-detection and fallback. |
| `"never"` | Pure launcher page — never embeds. |

**Why Keldurn launches directly by default:** `play.keldurn.com` sends `X-Frame-Options: DENY` plus CSP `frame-ancestors 'none'` (verified 2026-11-17). Browsers refuse to render such pages inside an iframe from *any* other origin — on GitHub Pages, on port 5500, anywhere. So for Keldurn the portal shows its direct-launch screen instead of a black frame. Your link still works; it simply hands you over to the realm with one click.

Want the *true* in-page experience? Point the portal at the other browser realm, **Lasthavens** (WotLK 3.3.5a), which sends no framing restrictions and embeds beautifully — see [Re-pointing the portal](#re-pointing-the-portal).

---

## Repository structure

```
wow-browser-portal/
├── index.html                     # The portal page (WotLK ice theme, snowfall, smart embed)
├── assets/favicon.svg             # Snowflake favicon
├── scripts/serve.js               # Zero-dependency Node static server (port 5500)
├── .devcontainer/devcontainer.json # Codespaces: Node 20 + Live Server + port 5500 forwarded
├── .github/workflows/deploy.yml   # Auto-deploys the site to GitHub Pages on every push
├── package.json                   # npm start -> node scripts/serve.js
├── .nojekyll                      # Lets Pages serve files as-is (also useful in branch mode)
├── .gitignore
└── LICENSE                        # MIT
```

---

## 1) Publish your own GitHub Pages link

1. Create a new repo on GitHub (name it `wow-browser-portal` or anything you like — the name becomes the URL slug).
2. Push this folder:

   ```bash
   cd wow-browser-portal
   git init -b main          # skip if the folder already contains .git
   git add -A
   git commit -m "Keldurn browser portal"
   git remote add origin https://github.com/<your-username>/wow-browser-portal.git
   git push -u origin main
   ```

   *(No command line? On the repo page press `.` to open github.dev, or use **Add file → Upload files** and drop the folder contents in.)*

3. Enable Pages: repo **Settings → Pages → Build and deployment → Source: GitHub Actions**. The included workflow (`.github/workflows/deploy.yml`) already deployed on your first push; redeploys happen automatically on every push, or manually via the workflow's **Run workflow** button.
4. Your portal is live at:

   ```
   https://<your-username>.github.io/wow-browser-portal/
   ```

The page banner shows where it is being served from (GitHub Pages, Codespaces, or a local server).

---

## 2) Host it on port 5500 from a GitHub Codespace

This gives you a second, locally-hosted copy on a different port — handy for editing, testing config changes with live reload, or just having a private entry point.

1. On your repo page: green **Code** button → **Codespaces** tab → **Create codespace on main**.
2. The container builds in about a minute (Node 20 + the **Live Server** extension pre-installed, port **5500** pre-forwarded and labeled "Portal").
3. Serve the page — either way works, pick one:

   **Option A — Live Server (live reload):**
   In the Explorer, right-click `index.html` → **Open with Live Server**. It serves on port 5500 and auto-refreshes the browser tab whenever you edit a file.

   **Option B — the bundled Node server:**
   In the terminal:

   ```bash
   npm start          # or: node scripts/serve.js
   ```

4. Codespaces detects port 5500 and pops up **"Open in browser"** (or open the **Ports** panel → port `5500` → the globe icon). Your URL looks like:

   ```
   https://<codespace-name>-5500.app.github.dev/
   ```

   That's your portal, hosted inside the codespace, on port 5500, reachable from your normal browser.

5. Sharing: the forwarded port is **Private** by default (only you). Right-click port `5500` in the Ports panel → **Port visibility → Public** if you want others to reach it.

> **Note:** A codespace shuts down when idle. Nothing is lost — your characters live on the Keldurn realm (not in the codespace), and your code is in the repo. Restart the codespace and run Live Server / `npm start` again.

---

## 3) Run it on your own machine (no Codespaces)

Any of these, from the repo folder:

```bash
npm start                       # Node server on port 5500
node scripts/serve.js --port 8080   # any port
python3 -m http.server 5500     # Python alternative
```

or open `index.html` directly from disk — the page works from `file://` too. If you use VS Code's Live Server extension locally, it defaults to port 5500 and works as-is.

---

## Configuration

Edit the `CONFIG` block at the top of the `<script>` section in `index.html`:

| Constant | Default | What it does |
| --- | --- | --- |
| `GAME_URL` | `https://play.keldurn.com` | Where *Enter the Realm* sends people. |
| `GAME_HOME` | `https://keldurn.com` | The "Official site" link. |
| `EMBED_MODE` | `"auto"` | `auto` / `always` / `never` — see table above. |
| `KNOWN_NO_EMBED` | `["play.keldurn.com", "keldurn.com"]` | Hosts that measured as refusing iframes; `auto` skips the iframe for them. Remove entries if a host starts allowing framing. |

### Re-pointing the portal

To make the portal an in-page, full-screen gateway to **Lasthavens** (WotLK 3.3.5a browser realm — no framing restrictions, so the smart embed actually renders the game inside your page):

```js
const GAME_URL = "https://lasthavens.com";
const GAME_HOME = "https://lasthavens.com";
const EMBED_MODE = "auto";   // embeds for real, with automatic stuck detection
```

### Changing the port

- **Node server:** `npm start` uses 5500; override with `PORT=5173 npm start` or `node scripts/serve.js --port 5173`.
- **Live Server:** `.devcontainer/devcontainer.json` sets `liveServer.settings.port` (5500) and `forwardPorts` — change both together.
- **Prefer 5173 (Vite-style)?** `PORT=5173 npm start` and add `"forwardPorts": [5173]` in the devcontainer. Port 5500 is the default because it is Live Server's native port.

---

## Troubleshooting

| Symptom | Cause & fix |
| --- | --- |
| Fallback screen appears instead of the game in the frame | Expected for Keldurn — the game server sends `X-Frame-Options: DENY`. Use **Launch in new tab** or **Take over this tab**. |
| Frame shows *"refused to connect"* | Same cause — the browser blocked the framed page. The portal usually catches this first and shows its own fallback. |
| First launch takes a long time | Normal. The browser downloads the game client into its cache on first visit; later visits are fast. |
| No sound | Browsers block audio until you interact. Click once inside the game. |
| Port 5500 already in use | Run `node scripts/serve.js --port 5501` (or any free port). |
| Port 5500 not in the Codespaces Ports panel | Open the Ports panel → **Forward a Port** → type `5500`. |
| Friends can't open the forwarded-port URL | Set port 5500 visibility to **Public** in the Ports panel. |
| Workflow didn't run | Pages Source must be **GitHub Actions** (Settings → Pages). Check the Actions tab for errors. |

---

## Disclaimer

This is an **unofficial fan portal**. It is not affiliated with, endorsed by, or connected to Blizzard Entertainment or the Keldurn project. The portal is a single static page — it does not host, mirror, or proxy any game files; the game itself is loaded from (or launched into) its official site. Never reuse your Battle.net password on third-party sites or private servers, and remember that unofficial realms can disappear at any time.

Licensed under the [MIT License](LICENSE).
