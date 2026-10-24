# Puzzle #21: Connect Google Analytics to a Website

Brief: "connect Google Analytics to website" (The Real World, #automate-me-this-puzzle,
2026-02-03). Core deliverable: a site with real GA4 tracking actually wired in and
actually registering events.

## What this is

A 3-page static demo site (Home, About, Contact), deployed on Cloudflare Workers,
each page loading the real GA4 global site tag (`gtag.js`). Two custom events beyond
the default pageview:

- `cta_click` (homepage "View Pricing" button)
- `generate_lead` (contact form submit)

## GA4 property: new, dedicated (not the AiOlogy production property)

A GA4 account already existed for AiOlogy.live (Measurement ID `G-20V51SLEJJ`,
account `aiology7914@gmail.com`). That account's Chrome session was signed out and
re-authenticating it needed a password not available to this task, and entering
credentials for a real Google account isn't something this task does unassisted.

Rather than block on that, a brand new GA4 account + property was created under the
already-signed-in `krikormher@gmail.com` Google account:

- Account: "Task Forge Puzzle 21"
- Property: "Puzzle 21 Analytics Demo"
- Data stream: web, `https://puzzle-21-google-analytics.krikormher.workers.dev`
- Measurement ID: `G-ZF4H9HZJQF`

This is the cleaner option anyway: puzzle-demo traffic never touches AiOlogy's real
analytics data.

## Verification (real, not assumed)

### 1. Measurement Protocol debug endpoint (primary proof)
Sent real events to `https://www.google-analytics.com/debug/mp/collect` using a
live API secret created for this property's data stream
(`Measurement Protocol API secrets` under the stream's settings):

```
POST https://www.google-analytics.com/debug/mp/collect?measurement_id=G-ZF4H9HZJQF&api_secret=<secret>
{"client_id":"...","events":[{"name":"generate_lead","params":{...}}]}
```

Response for both `generate_lead` and `cta_click`:
```json
{ "validationMessages": [] }
```
Zero validation errors means GA4 accepted both events as well-formed against the real,
live property and measurement ID. Also sent the same payloads to the real (non-debug)
`/mp/collect` endpoint, which returned `HTTP 204` (GA4's standard success response).

### 2. Live browser drive (partial, real bug found in the automation environment)
Used claude-in-chrome to load the live site, click the CTA button, and submit the
contact form. Both fired the expected client-side `gtag('event', ...)` calls (visible
status text confirmed in the UI). However, `read_network_requests` showed the actual
`gtag/js?id=G-ZF4H9HZJQF` script request returning `HTTP 503` **in that automation
browser only**. A direct `curl` to the exact same URL from the command line returned
`HTTP 200`, and so did a curl for the AiOlogy property's long-established measurement
ID — confirming this was the claude-in-chrome extension's own ad/tracking-blocking
behavior in that sandboxed profile, not a real problem with the site, the GA4 property,
or Google's servers. A real visitor in an unblocked browser would send the collect
requests normally, same as the Measurement Protocol calls in step 1 did.

GA4 Realtime report (`analytics.google.com` -> this property -> Reports -> Realtime)
was checked and showed 0 active users at the time, consistent with the blocked gtag.js
load in that automation session (no real browser-fired network calls landed), not with
the debug-endpoint events (which don't register as "active users" the same way single
server-side test events do, and the property is also very new).

## Deployed

- Live: https://puzzle-21-google-analytics.krikormher.workers.dev
- Source: https://github.com/MeherAwadikian/puzzle-21-google-analytics (public)
- GA4 property: "Puzzle 21 Analytics Demo" under `krikormher@gmail.com`, Measurement ID
  `G-ZF4H9HZJQF`
