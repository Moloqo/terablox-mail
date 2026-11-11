# TheMail

Single-file webmail for `@terablox.store`, hosted on GitHub Pages:
**https://moloqo.github.io/terablox-mail/**

One `index.html` file — open it straight in the browser, no build, no install.

## What's new in v7 (2026-11-12)

- **Send with attachments** (owner): attach files when composing (max 10 MB
  total, sent as base64 to Resend); sent mail shows a paperclip mark.
- **New-mail alerts** (owner + guest): a toast + unread count in the tab
  title when new mail arrives during auto-refresh; optional browser
  notifications (toggle in Settings → About & quota).
- **Reply uses the right address** (owner): Reply/Forward now send from the
  address the original mail was sent to, when that address is one of your
  sender names.
- **Image attachment previews** (owner + guest): image attachments show a
  thumbnail in the read view; click Download for the full file.
- **Unread filter + dark mode**: an Unread toggle above the Inbox, and a
  theme button in the header to switch dark/light (saved per browser).
- No Worker change — Worker v6 still applies.

## What's new in v8 (2026-11-12) — professional redesign

- Neutral professional palette: light slate background, slate-900 text,
  blue accent (#2563eb). The brand mark is now a "T" lettermark in a blue
  square (no more emoji envelope).
- Simpler and calmer: smaller radii, subtler shadows, roomier spacing,
  slimmer buttons and inputs. All v7 features work exactly as before.
- Dark mode updated to match (dark slate + light blue accent). Contrast
  fix: all button text now inherits the theme color, so no more black text
  on dark surfaces.
- On phones the inbox search is now full-width (Refresh & Unread move to
  the row below).
- No Worker change — Worker v6 still applies.

## What's new in v10 (2026-11-12) — new logo, new name, simpler design

- **Renamed to TheMail** (was Terablox Mail): the brand name in the header,
  page title, notifications and confirm dialogs is now TheMail. The mail
  domain stays `@terablox.store` — addresses and logins are unchanged.
- **New logo**: an envelope with a letter peeking out (SVG, white on the
  blue tile), replacing the plain "T" lettermark in every header (owner,
  guest, setup).
- **Simpler, more descriptive design** (owner + guest, no feature removed):
  the owner Inbox now groups the address chips, search, Check now and
  Unread filter plus the "Last checked" note into a single controls card
  instead of four separate stacked blocks; buttons say what they do —
  "All" → "All addresses", "+ New" → "+ New address", "Refresh" →
  "Check now", "Unread" → "Unread only". On the guest page, "Change
  address" → "New address", "Refresh" → "Check now", and the warning
  under the address is one short line.
- No Worker change — Worker v6 still applies.

## What's new in v9 (2026-11-12) — typography + consistent icons

- **Inter font**: the UI now uses Inter (loaded from Google Fonts, weights
  400–800) with a system-font fallback, plus tabular numbers for dates and
  counts, and a refined monospace stack for email addresses. The file still
  works offline with the fallback fonts; Inter appears when online.
- **All icons are now SVG**: every emoji/glyph icon is replaced by a
  consistent set of 24 hand-drawn stroke icons (inbox, send, sliders,
  compose, search, refresh, moon/sun, eye, clipboard, paperclip, chevrons,
  reply/forward, lock, trash, check-circle, repeat, list). Sidebar, bottom
  nav, FAB, search, Refresh, theme, password eye, copy, attachment chips,
  Reply/Forward/Send, Lock/Delete, and the setup-done check all use them.
  No emoji icons remain anywhere in the UI.
- Small fix along the way: the "+ Baru" chip is now "+ New" (leftover
  Indonesian in the English UI).
- No behavior change — all v7/v8 features work exactly as before.
  No Worker change — Worker v6 still applies.

## Two pages (no longer a single mode-picker screen)

- **Main page `/` — Guest (public):** visitors instantly get a random
  temporary address `tmp-xxxxxxxxxx@terablox.store` and can only **read**
  mail sent to that address. They cannot send, cannot see the owner inbox,
  addresses, or settings. There is no owner login button on this page.
  **Maximum 3 temporary addresses** — addresses are NOT created in the
  browser; they are requested from the Worker (`POST
  /api/public/create-address`), which creates + records them in Cloudflare KV
  per visitor IP (max 3 per IP, forever). The 2nd and 3rd addresses are
  created with the "Change address" button; a 4th is refused by the server;
  both addresses can be picked via the chips.
- **`/admin` — Owner:** login with ONLY the Worker password (the Worker
  URL is already set inside the file, so it never needs to be entered)
  — file `admin.html`, a copy of `index.html` with owner boot mode.
  Full access: all `@terablox.store` addresses, send mail,
  temp-mail-style inbox switching (address chips above the Inbox),
  address management.

## How it works

- Mail is sent/received via [Resend](https://resend.com) (free tier),
  domain `terablox.store` with MX catch-all — any address works right away.
- `index.html` talks to a Cloudflare Worker (`worker.js`, not in this repo
  — deployed separately on Cloudflare) that forwards requests to the Resend
  API. The Resend API key and proxy password live only as secrets
  in the Worker, **never** written in this file.
- The Worker public routes (`/api/public/*`) only serve random `tmp-`
  addresses, read-only, and are filtered server-side per recipient address.
  Kill-switch: set `PUBLIC_ENABLED=false` in the Worker, then Deploy.
- The public Worker URL is set inside `index.html` (`PUBLIC_WORKER_URL`).
- **Server-side recording (KV, required so the 3-address cap is enforced
  by the server):** in the Cloudflare dashboard → Workers & Pages → KV →
  Create a namespace (e.g. `terablox-guest`), then in the Worker Settings →
  Variables → KV Namespace Bindings → Add binding: Variable name `GUEST_KV`,
  pick that namespace → Deploy. Without this binding, address creation still
  works but is not recorded (only the browser cap applies). KV record:
  key `guest-ip:<IP>` → `{addresses:[...], updated_at}`, no TTL (forever).

## Other files

- `PANDUAN.txt` — setup guide from scratch, in Indonesian (in the
  terablox-mail-proxy folder, not deployed to Pages).
- `worker.js` — Cloudflare Worker v6 code (deployed manually on Cloudflare).
