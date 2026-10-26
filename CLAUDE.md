# Day.js

2 KB immutable date library with a Moment.js-compatible API. Core is tiny; everything else (plugins, locales) is opt-in and built as separate bundles.

## Architecture
- `src/index.js`: the whole core (`Dayjs` class, `dayjs()` factory, `extend`/`locale`). `src/utils.js`, `src/constant.js`: shared helpers.
- `src/plugin/<name>/index.js`: one directory per plugin, default-exports `(option, Dayjs, dayjs) => {...}` and patches `Dayjs.prototype`.
- `src/locale/<code>.js`: one file per locale, starts with a `// Name [code]` comment (the build parses it).
- `types/`: hand-written `.d.ts`; `types/plugin/<name>.d.ts` is **required** for every plugin (`test/plugin/declarations.test.js` fails without it).
- `test/`: core tests at top level, `test/plugin/<name>.test.js`, `test/locale/<code>.test.js`, `test/issues/issue<N>.test.js` for regressions.
- `docs/`: only stubs and translated READMEs; real docs live on day.js.org (separate repo), so don't add API docs here.

## Commands (Node + npm; run `npm ci` first)
- One test file: `npx jest test/plugin/isToday.test.js --coverage=false`
- One test by name: `npx jest test/parse.test.js -t "some name" --coverage=false`
- Timezone-sensitive test: `TZ=America/New_York npx jest test/plugin/timezone.test.js --coverage=false`
- Lint (this is also the pre-commit hook): `npm run lint`
- Full suite: `npm test`. It re-runs the timezone tests under 4 `TZ` values and then enforces **100% line coverage**; takes minutes, so use it once at the end.
- Build bundles: `npm run build` (rollup, then size check). Only needed when touching `build/` or checking the 2.99 KB size limit.

## Conventions
- ES6 modules, **no semicolons**, no trailing commas, 2-space indent (airbnb-base + overrides in `.eslintrc.json`). A line starting with `[` or `(` hits `semi-style`; restructure instead of prefixing `;`.
- Imports in tests are `import dayjs from '../src'` / `'../../src/plugin/x'`, never `'dayjs'` (except inside `src/locale/*`, which imports `'dayjs'` on purpose).
- Test idiom: top-level `it(...)` (no wrapping `describe` in most files), `MockDate.set(new Date())` in `beforeEach` and `MockDate.reset()` in `afterEach`, and `moment` as the oracle: `expect(dayjs(x).foo()).toBe(moment(x).foo())`.
- Plugins are enabled in-test with `dayjs.extend(plugin)` at module top.
- Regression tests go in `test/issues/issue<N>.test.js` with a `// issue N` comment.
- Commits and PR titles use semantic prefixes (`fix:`, `feat:`, `chore:`).

## Guardrails
- Open PRs against **`dev`**, not `master` (master is the release branch; semantic-release publishes from it).
- Do not hand-edit generated/ignored output: `/dayjs.min.js`, `/esm`, `/plugin`, `/locale`, `/index.d.ts`, `locale.json`, `coverage/`. Source of truth is `src/` + `types/`.
- Do not edit `CHANGELOG.md` or `version` in `package.json` (`0.0.0-development` is intentional; semantic-release owns both) or `patches/`.
- Do not run `npm run prettier` or format `src/`; prettier is configured only for `docs/**/*.md`.
- Core size is budgeted (`size-limit`, 2.99 KB). Avoid adding code to `src/index.js` when a plugin would do.
- Many guards look redundant (`|| 0`, `+ 0`, `isNaN` checks) but exist for old bugs; `git log -S '<snippet>'` before removing one.
- New plugin checklist: `src/plugin/x/index.js` + `types/plugin/x.d.ts` + `test/plugin/x.test.js`. Plugin docs live on day.js.org, not in `docs/en/Plugin.md`. New locale: `src/locale/x.js` + `test/locale/x.test.js`.
