# Repository guide

Small self-contained web apps, one folder each, served from one static host. They run on
a phone, in front of people, often with no signal.

## What these are, and what follows from it

These are props, not web apps that happen to be used in front of an audience. The
difference drives most decisions here:

- **It has to work with no signal.** Rooms have no bars. Every app installs a service
  worker so it runs from the phone, and is tested that way.
- **It has to survive being picked up.** Clockwork is a working calculator first. If an
  ordinary `12 + 3 × 4` behaves oddly, it is blown before it is used.
- **Nothing may say what it is.** The URL is dull on purpose, the health check does not
  name the apps, and only `public/` is served — a README explaining how something works
  must never be reachable over HTTP.
- **Failures must be loud in testing and impossible in use.** A wrong number that looks
  right is the worst outcome, because the first person to notice is in the audience.

## Commands

```bash
npm install
npm start              # serves every app on :8080 under its own slug
npm test               # host tests, then each app's own checks
npm run check          # the arithmetic alone
npm run test:browser   # drives real Chromium against an app, phone-sized
npm run format:check
```

## Layout

| Path                   | What                                                         |
| ---------------------- | ------------------------------------------------------------ |
| `apps/<name>/public`   | **Everything the world can see.** Served at that app's slug. |
| `apps/<name>`          | Its README, checks, tooling. Never served.                   |
| `apps/<name>/app.json` | Name and slug. The whole registration.                       |
| `host/`                | Static host: discovers apps, serves `public/`, nothing else. |

## Branches

`main` is the trunk and is what deploys. Work on one app happens on a branch named after
it — `clockwork-landscape`, `newthing-scaffold` — and merges into `main`. Never push to
`main` without being asked.

## Invariants — do not break these

1. **Only `public/` is published.** The boundary is a directory, not a list of
   exclusions, so it cannot rot as files are added. `host/apps.mjs` owns it and a test
   pins it.
2. **Nothing advertises what is hosted.** No index page, no directory listing, no app
   names in `/healthz`. The bare domain is a flat 404.
3. **What must be exact must be proved.** Each app ships a `check.mjs` that verifies its
   claim across the whole input space, from a terminal, with no browser and no phone. If
   it reports less than 100%, the app is dead. It is the one test that cannot fail.
4. **Never fake what can be checked.** Clockwork's arithmetic is genuine — the audience
   can follow it on their own calculators and will. The correction hides in what the
   performer controls, never in a number that has been fudged.
5. **Test in a real browser, on a phone viewport.** Whole-app breakage — a dead keypad,
   an overlay that never lifts, digits clipped off the side — is invisible to headless
   checks. Assert at the _end_ of a routine, not just after load.
6. **Every app works offline, and updates anyway.** Cache-first for assets; the page
   itself is fetched fresh when the network answers quickly. A cache that cannot be got
   out of the way means a fixed bug stays on a phone for days.
7. **Mounts are relative, slugs are not hard-coded.** Every URL inside an app is `./`, so
   it works at any slug. The bare mount redirects to its trailing slash — without that,
   relative URLs resolve against the domain root and the app silently loses its service
   worker and manifest.
8. **A stray touch must never change a committed number.** The audience cannot see the
   screen. Anything they can do while holding the phone has to be harmless.

## Adding an app

Make `apps/<name>/` with an `app.json` and a `public/`. That is the registration — the
host finds it, and no shared code changes. Give it a `check.mjs` proving whatever has to
be exact and a `smoke.mjs` driving a browser. Branch first.

**Folder names are codenames, not descriptions.** Ask for one rather than inventing a
descriptive name; if you must propose one, propose something that says nothing. The
folder name becomes the address unless `app.json` says otherwise, and the address is the
one part of a prop an audience can see — it is on screen the whole time it is being
installed, and again every time the phone is handed over.

Addresses must be lower-case letters, digits and hyphens. Duplicates, reserved routes and
anything needing URL-escaping refuse to start, because an app that is silently
unreachable would be found out at the worst moment.

## Where this came from

Clockwork and the host were written in `jasonmwong603/KeyDateTechnologies` and copied
here. That repository still has them, along with a second app and a shelved game, and is
not maintained from this one — nothing here should push to it or assume anything about
its state.
