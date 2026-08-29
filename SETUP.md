# GXNG DEPT — deploy & tracking setup

This folder is a complete, deployable site: the landing page, a QR tag that
scans back to it, and a small first-party visitor tracker with a private
dashboard. Everything here runs on Cloudflare's free tier — no credit card
required to start, no domain purchase required to start.

Three pieces:

- `index.html` — the landing page, with the tracker wired in
- `functions/api/track.js` and `functions/api/dashboard.js` — the backend
  (Cloudflare Pages Functions — no separate server to manage)
- `dashboard.html` — your private "who's interested in what" view

## 1. Create a free Cloudflare account

Go to dash.cloudflare.com and sign up (email + password). No card needed
for what's below.

## 2. Deploy this folder as a Pages project

In the Cloudflare dashboard: **Workers & Pages → Create → Pages → Upload
assets**. Give the project a name (e.g. `gxngdept`) and drag this whole
folder in — `index.html`, the `functions` folder, and `dashboard.html`
together. Cloudflare auto-detects the `functions` folder and wires up the
`/api/track` and `/api/dashboard` routes for you.

When it finishes, you'll get a URL like `https://gxngdept.pages.dev`.
That's your real site now.

## 3. Create the KV namespace (where events get stored)

**Workers & Pages → KV → Create namespace.** Name it something like
`gxng-tracking`.

Then bind it to your Pages project: open your Pages project → **Settings
→ Functions → KV namespace bindings → Add binding**. Set the variable
name to exactly `GXNG_KV` and point it at the namespace you just created.
Do this for both **Production** and **Preview**.

## 4. Set your dashboard password

Same Settings screen → **Environment variables → Add variable**. Name it
`DASH_KEY`, set the value to a password only you know, and mark it
**Encrypt** (so it's stored as a secret, not plain text). Add it for both
Production and Preview.

This is the password `dashboard.html` will ask for.

## 5. Redeploy so the bindings take effect

Bindings and environment variables only apply to deploys made after you
set them. Go to your project's **Deployments** tab and hit **Retry
deployment** on the latest one (or just re-upload the folder). This step
is easy to miss and is the #1 reason `/api/track` will 500 right after
setup.

## 6. Point the page at its real URL

Open `index.html`, find this near the top of the closing `<script>` block:

```js
var SITE_URL = 'https://gxngdept.pages.dev';
```

Replace it with your actual `*.pages.dev` URL from step 2 (or your custom
domain, once you add one). This is what the QR code encodes and what
tags QR scans as `?src=qr_tag` so they show up separately on the
dashboard. Re-upload the folder after changing it.

## 7. Check it's working

- Visit `https://your-site.pages.dev/api/track` directly — it should
  return `{"ok":true,"kvBound":true}`. If `kvBound` is `false`, the KV
  binding in step 3 didn't take, or you skipped the redeploy in step 5.
- Visit the landing page itself, click around (a product card, a hero
  button), then open `https://your-site.pages.dev/dashboard.html`, enter
  your `DASH_KEY`, and you should see those clicks show up.

`dashboard.html` isn't linked from the site's navigation — it's only
reachable if you go to that URL directly and know the password. Don't
post that URL publicly.

## Optional: a custom domain

Once you're happy with the `*.pages.dev` link, **Workers & Pages → your
project → Custom domains** lets you point `gxngdept.com` (or whatever you
buy) at it. Update `SITE_URL` in `index.html` to match and redeploy.

## What this does and doesn't do

It sets a first-party cookie (`gxng_vid`) so a return visit is
recognizable, and logs five things: page views, which product cards get
looked at, which get clicked, which of the two hero buttons gets pressed,
and who signs up on the "Get Put On" ticket. That's it — no third-party
trackers, no ad pixels, no cross-site tracking. The footer already says
the site uses first-party cookies; if you want a fuller privacy policy
page linked from it before you launch, that's worth writing (or asking
me to draft) once the site's live.

Counters aren't perfectly exact under heavy simultaneous traffic (two
visitors clicking the same thing in the same instant can occasionally
undercount by one) — a non-issue at boutique-drop scale. If GXNG DEPT
ever needs airtight, high-volume analytics, that's the point to move
counters to a Cloudflare Durable Object or a real analytics platform —
worth a follow-up then, not now.
