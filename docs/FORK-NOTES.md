# Fork notes (mr-socialmedia/quran.com-frontend-next)

Audit date: 2026-11-26. Base: upstream quran.com-frontend-next at `aff1a035` ("Ramadan release 4 (#3251)", 2026-03-03).

## What differs from upstream

Nothing in application code. Every commit in history is an upstream commit; this fork has only
this notes file on top. No deploy config, secrets or app changes were added.

## Required environment

Copy `.env.example` to `.env.local` and fill in values (never commit real values). Groups:
- Sentry: `NEXT_PUBLIC_SENTRY_DSN`, `NEXT_PUBLIC_SENTRY_URL`, `NEXT_PUBLIC_SENTRY_PROJECT`,
  `NEXT_PUBLIC_SERVER_SENTRY_ENABLED`, `NEXT_PUBLIC_CLIENT_SENTRY_ENABLED`, `SENTRY_AUTH_TOKEN` (upload only).
- Auth/proxy: `SIGNATURE_TOKEN`, `PROXY_SIGNATURE_TOKEN`, `INTERNAL_CLIENT_ID`, `ALLOWED_ORIGINS`,
  `NEXT_PUBLIC_SSO_ENABLED`, `NEXT_PUBLIC_AUTH_PROFILE_TIMEOUT_MS`.
- Content/services: `NEXT_PUBLIC_SANITY_PROJECT_ID`, `NEXT_PUBLIC_TARTEEL_VS_API_KEY`, Novu
  (`NEXT_PUBLIC_NOVU_*`, `NEXT_PUBLIC_ENABLE_NOTIFICATIONS`), `NEXT_PUBLIC_QURAN_REFLECT_URL`,
  `NEXT_PUBLIC_EMBED_URL`, `LOKALISE_API_KEY`/`LOKALISE_PROJECT_ID` (translation sync only).
- Misc: `NEXT_PUBLIC_VERCEL_ENV`, `NEXT_PUBLIC_VERCEL_URL`, `NEXT_PUBLIC_ENABLE_FS_LOGGING`.

`yarn build` succeeded with the unmodified `.env.example` values (dummy), so no variable is
build-blocking; runtime features depending on them will not work against real services.

## Commands

Package engines pin Node 18.x; Node 22 works but yarn refuses to run scripts unless
`--ignore-engines` is used (or call tools directly with `npx`). Prefer Node 18 for parity.

- Install: `yarn install --frozen-lockfile`
- Lint: `npx eslint . --ext=tsx,ts,js,jsx`
- Type-check: `npx tsc --noEmit`
- Unit tests: `npx vitest run` (101 files, 934 tests)
- Build: `IS_BUILD_TIME=true npx next build` (about 3 min)
- AI standards links: `node scripts/ai/check-links.mjs` (or `yarn ai:check`)

## Known issues (baseline 2026-11-26, all inherited from upstream)

- Install: `yarn.lock` pins `@fastify/otel` (via `@sentry/node`) to a `codeload.github.com` tarball.
  Networks that block codeload (e.g. restricted proxies) fail with 403. Workaround: clone
  `getsentry/fastify-otel` at `ae3088d6`, tar it, and point a temporary copy of the lockfile at it.
  Do not commit that change.
- Lint: 2 errors, 9 warnings. Errors are `import/no-unresolved` for `msw` in
  `tests/mocks/msw/*.js` (`msw` is not in package.json; the mocks are unused).
- Type-check: 9 errors, all in test files (`styles.test.ts`, `utils.test.ts` x2,
  `audioPlayerMachine.reset.test.ts`, Playwright specs). No errors in application sources.
- `yarn audit`: 974 advisories (112 low, 368 moderate, 431 high, 63 critical), mostly transitive
  through Next 14, Storybook and old tooling. Not remediated: a fix needs major upgrades, which
  is outside minimal-change scope.
- No committed secrets found (pattern scan; `.env.example` holds placeholders only).
