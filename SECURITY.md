# Security review — Flow Monitor

Audit of `flowmonitor-app/` (index.html, sw.js, manifest.json, icons/).
Method: static review of every external-data path, network destination, storage
write and execution primitive, plus payload testing of the fixes.

---

## Summary

| | |
|---|---|
| Findings | **1 real vulnerability class** (XSS via unescaped API text) — **fixed** |
| Data leaving your device | **None.** Every network call is a read-only public `GET`. Zero POST/PUT, no analytics, no beacons. |
| Credentials in code | **None.** No API keys, no secrets, no wallet material. |
| Trading capability | **None.** No order placement, no signing, no HMAC, no exchange auth. It cannot move your money. |
| Code execution primitives | **None.** No `eval`, no `new Function`, no `document.write`, no remote scripts. |

---

## Finding 1 — Cross-site scripting via untrusted API text (was: HIGH) — FIXED

**What it was.** Text from third-party APIs was injected straight into the DOM via
`innerHTML` with no escaping:

- news headlines, source names and categories (CryptoCompare)
- the headline link `href` (a `javascript:` URL would have run on tap)
- trending coin names/symbols (CoinGecko)
- exchange symbols, including inside an inline `onclick="watchToggle('...')"`
- Upbit/Bithumb market names

**Why it mattered here.** No cookies or API keys exist to steal, but injected script
could read `localStorage` (your positions and settings) and — worse for a trading
tool — **silently rewrite displayed numbers**, e.g. showing a different stop or
entry than the one calculated. That is a realistic path to a real loss.

**Fix applied.**
- `esc()` HTML-escapes `& < > " ' \`` on every externally-sourced string.
- `safeUrl()` allows only `http:`/`https:` — `javascript:`, `data:`, `vbscript:` become `#`.
- `safeSym()` strips exchange symbols to `[A-Za-z0-9]`, max 20 chars.
- The inline `onclick` carrying API data was replaced with a `data-` attribute plus a
  bound listener, removing the attribute-injection surface entirely.
- `watchToggle()` now rejects any symbol not present in live ticker data.
- Link hardened to `rel="noopener noreferrer"`.

**Verified** against `<img onerror>`, `</span><script>`, `"><svg onload>`,
quote-breaking `'); …; //`, and template-literal `` `+alert(1)+` `` payloads — all
neutralised; scheme tests and symbol-injection tests pass; functional regression
suite still green.

## Finding 2 — No Content-Security-Policy (was: MEDIUM) — FIXED

Added a strict CSP with `default-src 'none'` and an explicit `connect-src`
allowlist naming only the exchange, news and context endpoints. Practical effect:
**even if a future XSS slipped through, it could not send your data anywhere** —
the browser blocks connections to any host not on the list. Also set
`base-uri 'none'` and `form-action 'none'`.

Caveat: the policy allows `'unsafe-inline'` for script because the whole app is one
inline `<script>`. Splitting it into an external file would let that be dropped —
a worthwhile hardening if this ever grows.

## Finding 3 — Service worker scope — OK, no change needed

`sw.js` caches **same-origin shell files only** (`if (url.origin !== self.location.origin) return;`)
and is network-first. Exchange and news responses are never cached, so the app
cannot serve you stale prices from disk — important for a market tool.

---

## What the app sends and stores

**Outbound (all read-only GET / WebSocket subscribe):**
`fapi.binance.com`, `api.bybit.com`, `www.okx.com`, `wss://fstream.binance.com`,
`wss://stream.bybit.com`, `wss://ws.okx.com`, `api.upbit.com`, `api.bithumb.com`,
`min-api.cryptocompare.com`, `api.coingecko.com`, `api.alternative.me`,
plus `fonts.googleapis.com` / `fonts.gstatic.com` for fonts.

**Stored locally** (`localStorage` key `flowmon`, plaintext, on your device only):
exchange choice, equity, risk %, ATR multiplier, coin list, alert settings, and your
positions (symbol, side, entry, size, leverage, stop, target). Never transmitted.

---

## Residual risks (accepted, not bugs)

1. **Your equity and positions sit in plaintext localStorage.** Anyone with your
   unlocked phone, or any other page on the same origin, can read them. Don't host
   this on a domain you share with untrusted content. Clearing browser data wipes it.
2. **Third-party data is trusted for correctness, not safety.** Escaping stops code
   execution; it can't stop a wrong price. If an exchange returns bad data, the app
   displays bad data. Your exchange is always the authority before you act.
3. **Google Fonts sees your IP** on load. Self-host the two font files to remove it.
4. **Whoever hosts the files can change them.** Prefer a host you control (your own
   GitHub Pages repo) over a public drop service, and re-check the file after updates.
5. **Liquidation prices are estimates** (0.5% maintenance margin, isolated). Treat
   the exchange's own liquidation figure as authoritative.
6. **Alerts stop when the app is closed** — Android suspends background web apps.
   Not a security flaw, but don't rely on it as a safety net for open positions.

## If you ever add API keys — read this first

The current design's biggest safety property is that it **holds no credentials**.
If you later connect an exchange key:
- never put a key in this HTML file — anyone with the URL could read it;
- use a **read-only** key with withdrawals disabled and IP-restricted;
- keep it server-side (the Python version), never in the browser.
