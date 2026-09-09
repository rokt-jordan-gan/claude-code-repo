# Rokt SDK+ `selectPlacements` test page

A single-file dummy site for testing a Rokt page setup. It initializes the
**Rokt Ecommerce Web SDK (SDK+)** and fires one `selectPlacements` call with
`email` as the only attribute.

Pre-filled values:

| Field | Value |
| --- | --- |
| JS Web API key | `us2-5d8607c00a70614ea6d1e01686ee528d` |
| Rokt domain | `https://apps.rokt-api.com` |
| `identifier` | `testsurvey.stg.rokt.conf` |
| `attributes.email` | `jordan.rokt5@gmail.com` |
| `isDevelopmentMode` | `true` |

## Run it

Serve over HTTP — don't open it with `file://`, the SDK needs a real origin:

```bash
python3 -m http.server 8000
# or: npx serve .
```

Open <http://localhost:8000/>, press **Run selectPlacements**, and watch the log
panel (everything is mirrored to the browser console too).

Query-string overrides:

```
http://localhost:8000/?email=someone@example.com&identifier=testsurvey.stg.rokt.conf&dev=0&autorun=1
```

Supported params: `apiKey`, `roktDomain`, `identifier`, `email`, `dev`, `tyj`, `autorun`.

## What the page does

1. Sets `window.mParticle.config` with `isDevelopmentMode` and an `identifyRequest`
   carrying the email, then runs the SDK+ loader snippet from the
   [Initialize section of the docs](https://docs.rokt.com/integration-guides/ecommerce/sdk/web/#initialize)
   with the API key — which loads `https://apps.rokt-api.com/js/v2/<API_KEY>/app.js`
   (falling back to `apps.roktecommerce.com` if that fails).
2. Waits for `mParticle.ready()`.
3. Calls:

```js
const selection = await mParticle.Rokt.selectPlacements({
  identifier: 'testsurvey.stg.rokt.conf',
  attributes: {
    email: 'jordan.rokt5@gmail.com',
  },
});
```

The loader snippet is pasted verbatim from the docs — only wrapped in a function so
the page can initialize on the first run using whatever is in the form fields.

## Notes / gotchas

- **Init happens once.** The API key, Rokt domain, dev-mode flag and the identity
  email are locked in when the SDK loads (those fields disable themselves after the
  first run). Reload the page to change them. The `identifier` and the email sent as
  a `selectPlacements` attribute can be changed between runs.
- **`isDevelopmentMode`** routes data to the Development environment (`env=1` on the
  `app.js` URL). Leave it on while testing `.stg.` identifiers; turn it off for
  production data.
- **`use("ThankYouPageJourney")`** is off by default — it's only needed for the Thank
  You Page Journey / shoppable-ads flow, not for a plain confirmation placement. The
  checkbox is there if the page under test needs it.
- **Embedded placements** render into `<div id="rokt-placeholder">`. If the placement
  is configured with a different placeholder id, rename that div to match. Overlays
  ignore it.
- **Domain allow-listing.** Whether `localhost` will serve placements is account/config
  dependent. If the call resolves but nothing renders, that's the first thing to check
  with the Rokt team — or host the page on an allow-listed staging domain.
- **First-party domain.** If the account uses a first-party subdomain (e.g.
  `rkt.example.com`), put it in the Rokt domain field instead of `apps.rokt-api.com`.
- **Ad blockers** will block `app.js`; the log will show the SDK never becoming ready.
