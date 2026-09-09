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
| `attributes.survey` | `"true"` (toggleable to `"false"`, or omitted) |
| `isDevelopmentMode` | `true` |

## Run it — hosted

The repo is public and this branch is the default branch, so GitHub Pages can serve it
as a real URL. In **Settings → Pages**, set Source to *Deploy from a branch*, pick this
branch and the `/ (root)` folder, and save. The page then lives at:

```
https://rokt-jordan-gan.github.io/claude-code-repo/
```

Query-string overrides work the same way there:

```
https://rokt-jordan-gan.github.io/claude-code-repo/?survey=false&autorun=1
```

Note this puts the JS Web API key on a public URL. That key is public by nature — it ships
in client-side JS on any site using the SDK — but it is a real key, so rotate it or make
the repo private if that matters for this account.

## Run it — locally

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

Supported params: `apiKey`, `roktDomain`, `identifier`, `email`, `survey`, `dev`, `tyj`,
`autorun`. `survey` accepts `true` / `false` / `omit`.

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
    survey: 'true', // radio: "true" | "false" | not sent at all
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
- **`survey` is sent as a string**, `"true"` or `"false"` — Rokt attribute values are
  strings, not JSON booleans. The third radio option leaves the attribute out entirely,
  which is a different case from sending `"false"`.
- **`isDevelopmentMode`** routes data to the Development environment (`env=1` on the
  `app.js` URL). Leave it on while testing `.stg.` identifiers; turn it off for
  production data.
- **`use("ThankYouPageJourney")`** is off by default — it's only needed for the Thank
  You Page Journey / shoppable-ads flow, not for a plain confirmation placement. The
  checkbox is there if the page under test needs it.
- **Embedded placements** render into `<div id="rokt-confirmation-placeholder">`, matching
  the placeholder configured on this page. Overlay placements render over the page and
  ignore that container; this page is expected to serve both.
- **Domain.** Intended to be run from `localhost`. If the call resolves but nothing
  renders, check with the Rokt team whether the account restricts placements to
  allow-listed domains.
- **First-party domain.** Not in use here — the Rokt domain stays `apps.rokt-api.com`.
  If a first-party subdomain is added later, put it in the Rokt domain field instead.
- **Ad blockers** will block `app.js`; the log will show the SDK never becoming ready.
