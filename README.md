# Rokt `selectPlacements` test page

A single-file dummy site for testing a Rokt Web SDK page setup. It loads the Rokt
launcher, creates a launcher instance, and fires one `selectPlacements` call with
`email` as the only attribute.

Pre-filled values:

| Field | Value |
| --- | --- |
| `accountId` | `us2-5d8607c00a70614ea6d1e01686ee528d` |
| `identifier` | `testsurvey.stg.rokt.conf` |
| `attributes.email` | `jordan.rokt5@gmail.com` |

## Run it

Serve it over HTTP — don't open the file with `file://`, the SDK needs a real origin:

```bash
python3 -m http.server 8000
# or: npx serve .
```

Then open <http://localhost:8000/>, press **Run selectPlacements**, and watch the log
panel (everything is also mirrored to the browser console).

Any field can be overridden by query string:

```
http://localhost:8000/?email=someone@example.com&identifier=testsurvey.stg.rokt.conf&sandbox=1&autorun=1
```

## What the page does

```js
const launcher = await window.Rokt.createLauncher({
  accountId: 'us2-5d8607c00a70614ea6d1e01686ee528d',
});

await launcher.selectPlacements({
  identifier: 'testsurvey.stg.rokt.conf',
  attributes: { email: 'jordan.rokt5@gmail.com' },
});
```

The launcher script is loaded from `https://apps.rokt.com/wsdk/integrations/launcher.js`
using the documented dynamic-injection snippet (`id="rokt-launcher"`, `async`,
`crossOrigin="anonymous"`, `fetchPriority="high"`).

## Notes / gotchas

- **`sandbox` checkbox.** `sandbox: true` sends SDK traffic to `apps-demo.rokt.com`
  instead of production `apps.rokt.com`. This is *not* the same thing as the `.stg.`
  in the page identifier — that's just the naming convention for a staging page inside
  a normal account. Leave sandbox off unless Rokt has told you to use the demo env.
- **Embedded placements** render into `<div id="rokt-placeholder">`. If the placement
  on this page is configured with a different placeholder id, rename that div to match.
  Overlay placements ignore it.
- **Domain allow-listing.** Rokt accounts generally only serve placements on registered
  domains. If the call resolves but nothing renders, `localhost` may need to be
  allow-listed against the account.
- **Ad blockers** will block `launcher.js`. The log panel says so explicitly if the
  script fails to load.
