# Flow Monitor — install as a standalone Android app

No app store, no re-uploading. You host these files **once**, then install to your
home screen. After that it opens like any other app: own icon, full screen, no
browser bar.

## Step 1 — put the files online (once)

**Option A — GitHub Pages (free, permanent, yours)**
1. Create a free GitHub account → **New repository** → name it e.g. `flowmonitor`, make it Public.
2. Upload every file in this folder (`index.html`, `manifest.json`, `sw.js`, `icons/`).
   Keep the folder structure — `icons/` must stay a folder.
3. Repo **Settings → Pages → Source: main branch / root → Save**.
4. After ~1 minute you get `https://<your-name>.github.io/flowmonitor/`.

**Option B — Netlify Drop (fastest, no account needed to start)**
Go to `app.netlify.com/drop` and drag this **whole folder** in. You get an https link
immediately.

Either way the link is permanent — you never upload again unless you want to update.

## Step 2 — install it as an app
Open your link in **Chrome on Android** → menu **⋮** → **Install app**
(or "Add to Home Screen"). Done — it now lives in your app drawer.

The install prompt needs https, which both hosts give you. It will not work from a
`file://` link, which is why opening the raw HTML from Downloads shows a blank screen.

## What each tab does
- **🎯 Signals** — graded setups (A/B/C confluence) with 4h alignment; tap for entry, stop, 1R–5R targets and position size.
- **🌐 Market** — *every* coin listed on the connected exchange: price, 24h move, volume, funding. Search any coin; tap ★ to add it to live streaming (max 12).
- **📰 News** — headlines (highlighted when they name a coin you follow), plus market context: total market cap, Fear & Greed, BTC/ETH dominance, trending searches.
- **🌊 Flow** — aggressive taker delta, buy/sell bias, z-score, session CVD.
- **📖 Book** — order-book ladder, depth bands ±0.5–10%, imbalance, large resting orders.
- **💼 Pos** — your open positions: live P&L, R-multiple, portfolio heat, and a stop-vs-liquidation check.
- **⚡ Liq** — live forced liquidations and cascade alerts.
- **🇰🇷 KR** — new Upbit/Bithumb listings (usually blocked in a browser; see below).

## Honest limits
- **Scan all, stream some.** Price/volume/funding covers every listed coin. Live order-flow streaming is limited to 12 watchlist coins — no browser can hold 500+ sockets.
- **Exchange in/outflows.** What's shown is *derivatives* flow (funding, taker flow, liquidations, open interest) — the thing that actually moves perp prices. True exchange-wallet netflow and whale wallet tracking come from on-chain providers (CryptoQuant, Glassnode, Nansen, Arkham) and require a paid key.
- **Korea listings & news** may show "blocked in browser" — those APIs refuse browser requests (CORS). Only a server-side version can read them reliably.
- **Alerts fire only while the app is open.** Android suspends background web apps. For true 24/7 alerts, run the Python version with Telegram.
- Nothing here places orders, and your positions never leave your phone.

## Updating later
Re-upload `index.html` to the same repo/site. The app picks it up on next launch
(the service worker fetches network-first, so you always get the newest version).
