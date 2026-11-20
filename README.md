# Practice

Small self-contained web apps, served from one static host. Each app is a folder; drop
one in and it is served.

## The apps

| App                          | Address |
| ---------------------------- | ------- |
| [Clockwork](apps/clockwork/) | `/calc` |

This table does not say what any of them do. What an app is lives in its own README, one
level down.

## Running them

```bash
npm install
npm start
```

Every app is served under its own slug: <http://localhost:8080/calc/>. There is no build
step — the apps are plain files and the host is plain Node, which is also why a deploy is
a checkout and a process start.

```bash
npm test             # the host, then every app's own checks
npm run test:browser # drives real Chromium at phone size
```

`npm run check` is the one that matters. If it ever reports less than 100% exact,
something that must be true is not.

## Adding an app

```
apps/<name>/
  app.json        {"name": "…", "slug": "…"}   ← the whole registration
  public/         everything the world can see
  check.mjs       proves whatever must be exact
  smoke.mjs       drives a browser
  README.md       how it works. Never served.
```

Drop the folder in and the host finds it. No shared code changes, so a new app cannot
break one that is already in use.

### Addresses

Every app is served at `/<slug>`, and the slug defaults to the folder name — `apps/heron/`
is at `/heron` with nothing configured. Set `slug` in `app.json` only when the folder name
should not be the address.

Lower-case letters, digits and hyphens only. Two apps claiming one address, an app sitting
on a route the host answers itself, or an address that would need URL-escaping all stop
the host from starting, naming the folder and the fix. The alternative is an app that is
silently unreachable.

**Only `public/` is ever served.** Everything beside it — the README, the checks, the
tooling — is unreachable over HTTP. That boundary is a directory rather than a list of
exclusions precisely so it cannot rot as files are added.

Nothing advertises what is hosted: no index page, no directory listing, no app names in
`/healthz`, and a flat 404 at the bare domain.

## Deploying

`render.yaml` is a Render blueprint: `npm ci --omit=dev` and `npm start`, no build. The
host has no runtime dependencies at all.

Each app's address can be changed on the host without a commit — `SLUG_CLOCKWORK`, named
after the folder. Setting one empty takes that app off the air.

A value set in the Render dashboard overrides the blueprint and is not updated by
re-applying it.
