# Implementation Plan — Phantom Fork Engine + Dashboard

Date: 2026-08-17
Feature: Fork-engine launch support (fingerprint-chromium) + dashboard overhaul
Source spec: `docs/superpowers/specs/2026-08-17-phantom-fork-engine-dashboard-design.md` (user-approved)

## Scope note

The spec covers two subsystems (Track A: fork engine, Track B: dashboard). Per user decision (Approach 1) these ship as **one milestone** in one plan. The phases below are ordered so Track A and Track B land incrementally and gates stay green throughout. If scope needs trimming later, phases A1–A7 and B1–B5 are individually shippable.

## Environment facts (verify before starting)

- Not a git repo yet → Task 0.2 initializes it so commit points work. If the user prefers no git, skip commit points marked "(commit)".
- `npm run build` is `tsc` only (no asset copy) — Task B1 changes it.
- `parseProfile`/`isProfile` (`src/core/validate.ts`) is a loose shape-check; it ignores extra keys, so new optional profile fields (`tags`, `notes`, `engine`) persist/round-trip without changes.
- `checkCoherence` (`src/verify/harness.ts:239`) is the 4-bit verify-time authority (tz/offset/lang/region). `computeCoherence` (`src/server.ts:76`) is a display-only replica that early-returns `checked:false` when no live geo. **Both** must learn the 5th `engine` segment. The server replica's gate must skip `null` segments (a non-running profile has no known resolved engine — it must not flip to incoherent).
- `launch()` (`src/browser/launcher.ts`) already receives `opts.executable`; engine integration goes inside `launch()` via a new `opts.engine`, so CLI verify, ops.launch, and the egress probe all flow through one place. Egress probe (`src/create.ts` `resolveEgress`) passes no `engine` → stays stock (backward compatible, no fork probing on create-time egress).
- Node v24 (global `fetch` available), Windows host (win32).
- Testing style: `vitest` + `node:fs/promises` temp dirs, `describe/it/expect` (see `tests/store.test.ts`).
- Manual E2E is headless puppeteer-core from `C:\Users\Robocop\AppData\Local\Temp\opencode\phantom-dash-check.mjs`. Keep `phantom-mgmt-check.mjs` (409 gate regression) green.

## File structure map

### New files
- `src/browser/engine.ts` — `EngineKind`, `EngineMode`, `ResolvedEngine`, `FINGERPRINT_CHROMIUM` install/cache constants, `resolveEngine()`, `buildForkArgs()`, `computeLaunchArgs()`.
- `src/core/profileMeta.ts` — `normalizeProfile()`, `validateProfilePatch()` (pure, unit-testable without HTTP).
- `src/core/folderStore.ts` — `FolderStore` (atomic writes + serialization queue), `Folder`, folder list + memberships shape.
- `src/cli/installFork.ts` — `installFork()` (download → verify SHA-256 → extract → `installed.json`), `isForkInstalled()`.
- `src/dashboard/index.html`, `src/dashboard/styles.css`, `src/dashboard/app.js` — new GUI (vanilla, static).
- `scripts/copy-dashboard.mjs` — copies `src/dashboard/*` → `dist/dashboard/`.
- `tests/engine.test.ts`, `tests/profileMeta.test.ts`, `tests/folderStore.test.ts`, `tests/harness.test.ts`.

### Changed files
- `src/config.ts` — `PHANTOM_ENGINE` env → `Config.engine: EngineMode` (default `"auto"`).
- `src/core/profile.ts` — `Profile` gains `tags?`, `notes?`, `engine?`; `createProfile` accepts them; type-only import of `EngineKind` (type-only cycle with `engine.ts` is safe).
- `src/core/store.ts` — `get()`/`list()` run `normalizeProfile()` after `parseProfile()` (defaults `tags:[]`, `notes:""`, `engine:"stock"`; files untouched).
- `src/browser/launcher.ts` — `LaunchOptions.engine?: EngineMode`; `launch()` resolves engine and appends `buildForkArgs`/proxy/window args via `computeLaunchArgs()`.
- `src/verify/harness.ts` — `checkCoherence(profile, reports, resolvedEngine?)` gains 5th segment.
- `src/create.ts` — `BuildProfileOptions.engine?: EngineKind`; `buildProfile` records it on the created profile.
- `src/ops.ts` — passes `cfg.engine` into `launch()`; records resolved engine at create; exposes `engineFor(id)`; stores resolved engine per running profile; `VerifyOutcome` gains `engine`.
- `src/index.ts` — new `Cmd.kind = "install-fork"`; passes `engine: config.engine` into `launch()`/`serve()`; `usage()` updated.
- `src/server.ts` — delete `DASHBOARD_HTML`; static file serving with CSP nonce; `PATCH /api/profiles/{id}`; folder endpoints; `/api/overview` gains `folders` + per-profile `tags/status/notes/engine/folderIds`; `computeCoherence` gains engine segment + skip-null gate.
- `package.json` — `"build": "tsc && node scripts/copy-dashboard.mjs"`.
- `README.md` — engine mode, `install-fork`, dashboard features, 5-segment coherence, env vars.
- `tests/store.test.ts` — normalize-on-read assertions.

## Tasks

### Phase 0 — Baseline and scaffolding

#### Task 0.1: Confirm baseline gates are green
- [ ] Run `npm run build`, `npx vitest run`, `npm audit`. Record counts (expected: 51 tests passing, audit 0). Any failure blocks start.
- Test: n/a (baseline).
- How to test: run the three commands above.

#### Task 0.2: Initialize git + baseline commit
- [ ] If `.git` absent: `git init`, add `.gitignore` (`node_modules/`, `dist/`, `data/`, `*.log`), initial commit "chore: baseline".
- Test: `git status` clean.
- How to test: `git log --oneline -1` shows the baseline commit.
- Commit: baseline commit.

### Phase A — Fork engine (Track A)

#### Task A1: Engine types + config
- [ ] In `src/browser/engine.ts`: `export type EngineKind = "stock" | "fingerprint-chromium";` `export type EngineMode = "auto" | "stock" | "fingerprint-chromium";` plus constants: cache dir `~/.cache/phantom/fingerprint-chromium`, `installed.json` name.
- [ ] In `src/config.ts`: `Config.engine: EngineMode` from `env.PHANTOM_ENGINE` (default `"auto"`); reject invalid values.
- Test (first): `tests/engine.test.ts` — `loadConfig({ PHANTOM_ENGINE: "fingerprint-chromium" }).engine === "fingerprint-chromium"`; invalid value throws.
- How to test: `npx vitest run tests/engine.test.ts`.

#### Task A2: `resolveEngine` with injectable probe
- [ ] `resolveEngine(opts: { explicit?: string; mode?: EngineMode; probe?: Probe }): Promise<ResolvedEngine>` where `Probe = (exec: string) => Promise<string[]>` (returns supported flags; defaults to spawning `<exec> --version` then `<exec> --help`, 5s timeout each).
- [ ] Precedence (auto): explicit `PHANTOM_CHROME_PATH` → `{ kind:"stock", executable }`; else fork install present (verified via `isForkInstalled()`) → probe fork → `{ kind:"fingerprint-chromium", executable: forkExe, supportedFlags }`; else walk stock candidates → `{ kind:"stock" }`.
- [ ] Mode `"stock"` forces stock. Mode `"fingerprint-chromium"` requires a verified install; probe failure → **fail-open** to stock with a warning (log param). Cache probe result per executable in a `Map`.
- Test (first): precedence table — explicit path wins as stock even when fork installed; mode stock never probes; mode fork + install present probes and returns fork; install absent → stock; probe throws → stock (fail-open); probe result cached (probe called once for same exec).
- How to test: unit tests with injected fake `probe`.

#### Task A3: `buildForkArgs` (deterministic) + `computeLaunchArgs`
- [ ] `buildForkArgs(profile, resolved): string[]` — stock → `[]`. Fork: `--fingerprint=ph-<sha256(profile.id).slice(0,16)>` (deterministic per profile id — **not** `noiseSeed`); platform/brand/version candidates derived from `profile.fingerprint.os` + `userAgent`; `--lang=<languages[0]>`; timezone candidate. Only emit flags present in `resolved.supportedFlags`; log+skip unknown flags (fail-open).
- [ ] **Open question to resolve at impl time**: exact fork flag spellings and value formats (`--timezone=<IANA>`? `--fingerprint-platform-version`/`--fingerprint-brand`/`--fingerprint-brand-version` value shapes; whether `--fingerprint` accepts an arbitrary string seed). Pin the final table in a constants object in `engine.ts` and verify against the fork's actual `--help` before finalizing.
- [ ] `computeLaunchArgs(profile, resolved, headless): string[]` — merges existing `--no-sandbox`, `--disable-blink-features=AutomationControlled`, `--window-size`, proxy args, plus `buildForkArgs`. Pure — no browser needed.
- Test (first): same id → identical `--fingerprint` seed across calls; different ids → different seeds; seed matches `^ph-[0-9a-f]{16}$`; stock → no fork flags; unsupported flag filtered out; `--lang` from `fingerprint.languages[0]`.
- How to test: `npx vitest run tests/engine.test.ts`.

#### Task A4: Profile metadata fields + normalize-on-read
- [ ] `src/core/profile.ts`: `Profile` gains `tags?: string[]`, `notes?: string`, `engine?: EngineKind`; `createProfile` opts gain `tags/notes/engine`.
- [ ] `src/core/profileMeta.ts`: `normalizeProfile(p)` returns `{ ...p, tags: p.tags ?? [], notes: p.notes ?? "", engine: p.engine ?? "stock" }`.
- [ ] `src/core/store.ts`: apply `normalizeProfile` in `get()` and `list()` after `parseProfile`.
- Test (first): a profile JSON written without tags/notes/engine loads with defaults; fields are **not** written back to the file (file bytes unchanged); stored profile round-trips.
- How to test: `npx vitest run tests/profileMeta.test.ts tests/store.test.ts`.

#### Task A5: Wire engine into `launch()`
- [ ] `LaunchOptions.engine?: EngineMode`; in `launch()` resolve once (`resolveEngine({ explicit: opts.executable, mode: opts.engine })`) and pass `ResolvedEngine` through to `launchOnce`; replace inline args array with `computeLaunchArgs(...)`.
- [ ] `launch()` returns (or exposes on session) `resolvedEngine` so callers can record it.
- Test: no new browser tests — pure `computeLaunchArgs` already covered (A3). Assert `launch` with `engine:"stock"` calls `computeLaunchArgs` with stock (use dependency-injected args builder if needed to avoid a real browser; otherwise rely on A3 units + E2E).
- How to test: `npx vitest run` + later E2E via `phantom-mgmt-check.mjs`.

#### Task A6: Engine coherence segment + record at create
- [ ] `src/verify/harness.ts`: `checkCoherence(profile, reports, resolvedEngine?)` — when `resolvedEngine` provided and `resolvedEngine !== profile.engine` → incoherent with reason `engine: resolved <X> vs profile <Y>`. Absent `resolvedEngine`/absent profile engine → no constraint (coherent).
- [ ] `src/create.ts`: `BuildProfileOptions.engine?: EngineKind`; set on created profile.
- [ ] `src/ops.ts`: on create, `const resolved = await resolveEngine({ explicit: cfg.chromePath, mode: cfg.engine });` record `profile.engine = resolved.kind` before save; pass `engine: cfg.engine` into all `launch()` calls; store `this.engineByProfile.set(id, resolved.kind)` on launch, delete on stop; expose `engineFor(id)`.
- [ ] `src/server.ts` `computeCoherence`: add `engine` segment = `resolvedEngine === (profile.engine ?? "stock")` when a resolved engine is known, else `null`; **gate skips null segments** (`filter(v => v !== null).every(v => v === true)`). Pass `ops.engineFor(id)` for running profiles via `loadViews`.
- [ ] `src/index.ts`: verify flow passes `resolvedEngine` into `checkCoherence`; create passes `engine` into `buildProfile`.
- Test (first): `tests/harness.test.ts` — checkCoherence incoherent on engine mismatch; coherent when matched; no constraint when resolvedEngine undefined; server-style gate ignores null engine segment (replicate the segment reducer in the test).
- How to test: `npx vitest run tests/harness.test.ts`.

#### Task A7: `phantom install-fork` CLI
- [x] `src/cli/installFork.ts`: `isForkInstalled(): Promise<boolean>` (reads `fork-install.json` (engine.ts's `FORK_INSTALL_FILE`) + verifies exe exists); `installFork({ force })` — on non-win32 throw a clear error pointing to `PHANTOM_CHROME_PATH`; download pinned Windows-x64 release zip (pinned tag + SHA-256 in constants) via `fetch`; verify checksum (node `crypto`); extract to cache dir via `tar -xf` (bundled on Windows 10+) with `child_process.execFile`, fallback to `Expand-Archive` via PowerShell if `tar` missing; write `fork-install.json` (`{ version, checksum, executable, installedAt }`); idempotent (skip when already installed unless `--force`).
- [x] `src/index.ts`: add `Cmd.kind = "install-fork"` to union + allowed set; call `installFork`; update `usage()`.
- [x] Test (first): unit tests with a **fixture zip** (tiny zip bytes on disk) + injected downloader AND extractor: writes `fork-install.json`, checksum mismatch throws (extract never called), `isForkInstalled` false when exe missing. Network download itself is manual/integration (do NOT hit GitHub in unit tests).
- [x] How to test: `npx vitest run` + `npm start -- install-fork` against a mocked download URL env if provided; otherwise `isForkInstalled` path only. (Verified E2E: fresh install, idempotent skip, `--force` reinstall against a local mock server; real `tar` extraction confirmed.)
- Commit: Phase A complete (engine + coherence + install-fork) — run full gates first.

### Phase B — Dashboard (Track B)

#### Task B1: Static dashboard serving (drop `DASHBOARD_HTML`)
- [x] `scripts/copy-dashboard.mjs`: recursive copy `src/dashboard/` → `dist/dashboard/`.
- [x] `package.json`: `"build": "tsc && node scripts/copy-dashboard.mjs"`.
- [x] Create minimal `src/dashboard/` placeholder files (index.html with `__NONCE__` placeholder + script tag, styles.css, app.js) so serving works before the full UI lands.
- [x] `src/server.ts`: remove `DASHBOARD_HTML`; serve `GET /` (read `dist|src/dashboard/index.html` via `new URL("./dashboard/", import.meta.url)` — `./` resolves to `dist/dashboard/` when built and `src/dashboard/` under tsx, replaces `__NONCE__` with a fresh `crypto.randomBytes` nonce, sets CSP `default-src 'self'; script-src 'self' 'nonce-<n>'; style-src 'self' 'nonce-<n>'; img-src 'self' data:; connect-src 'self'`), `GET /styles.css`, `GET /app.js` (static, no-store), keep favicon 204.
- Test: none unit (server integration) — verify via E2E.
- How to test: `npm run build; npm start -- serve` then headless check: `GET /` returns 200 + nonce present + CSP header; assets 200.

#### Task B2: Profile PATCH (tags/status/notes) + overview extension
- [x] `src/core/profileMeta.ts`: `validateProfilePatch(body)` → normalized `{ tags?, status?, notes? }` or error. Rules: tags unique array ≤20, each trim-nonempty ≤40 chars (400 on violation); notes string ≤2000; status ∈ created/active/retired.
- [x] `src/server.ts`: `PATCH /api/profiles/{id}` (in the `profileId && !action` branch) — load, validate, apply, `store.save` (normalize fields), return updated view. `rename` stays POST for compat.
- [x] `ProfileView` + `loadViews`: add `tags`, `notes`, `status`, `engine`, `folderIds`; `/api/overview` keeps existing summaries.
- [x] Test (first): `tests/profileMeta.test.ts` — valid patch normalized; 21 tags rejected; >40-char tag rejected; duplicate tags rejected; status enum enforced; notes >2000 rejected.
- How to test: `npx vitest run tests/profileMeta.test.ts`; manual `curl -X PATCH http://127.0.0.1:4173/api/profiles/<id> -d '{"tags":["a","b"]}'`.

#### Task B3: FolderStore + folder endpoints
- [x] `src/core/folderStore.ts`: `data/folders.json` `{ version: 1, folders: [{ id, name, color?, createdAt }], memberships: { [folderId]: profileId[] } }`. API: `list()`, `createFolder({name,color})`, `renameFolder`, `removeFolder` (drops memberships), `addToFolder(folderId, profileId)` (multi-membership, idempotent), `removeFromFolder`, `membershipFor(profileId)`. Atomic writes: write `folders.json.tmp` then `rename`; serialize writes through a promise chain (`this.q = this.q.then(work)`). 404 for unknown folder/profile.
- [x] `src/server.ts`: `GET/POST /api/folders`, `PATCH/DELETE /api/folders/{id}`, `POST /api/profiles/{id}/folders` `{ folderId }`, `DELETE /api/profiles/{id}/folders/{folderId}`. `/api/overview` returns `folders`.
- [x] Test (first): `tests/folderStore.test.ts` — CRUD; multi-membership idempotent add; remove folder clears memberships; concurrent `addToFolder`+`removeFolder` leaves valid JSON (serialization queue, no partial file); unknown folder → 404 sentinel.
- How to test: `npx vitest run tests/folderStore.test.ts`.

#### Task B4: New dashboard UI
- [x] Build `src/dashboard/index.html`, `styles.css`, `app.js` (vanilla, no framework) per spec B3: new visual identity (light/dark via `prefers-color-scheme` + localStorage override), top bar (brand, search, view toggle table/cards, running count, theme, create), left sidebar (folders + filter chips All · Running · Incoherent · Banned · per-tag), main content, status bar; table view (configurable/reorderable/resizable/sortable columns persisted in localStorage) + card view; bulk-select toolbar (launch, stop, add-to-folder, set tags, delete with confirm, move); detail drawer (fingerprint preview, 5-segment coherence with engine, live egress, proxy, notes/tags/status, Verify/Launch/Stop respecting 409); running view; right-click context menu; keyboard shortcuts `c` create, `t` table, `d` drawer, `g` search focus, `Esc` close; WCAG AA (focus states, aria labels, contrast), `prefers-reduced-motion`.
- [x] Poll `/api/overview` (5s), reuse existing `api()` fetch pattern. Handle 409 from verify/launch with clear incoherence message.
- Test: none unit (static UI) — covered by E2E.
- How to test: extend `phantom-dash-check.mjs`: drawer opens and shows 5 segments; table↔card toggle persists across reload (localStorage); theme toggle; bulk-select two profiles then stop; PATCH tags renders as chip + filters by tag; folder create + assign + filter; shortcuts work; running view shows running profile. Then run `phantom-mgmt-check.mjs` (409 gate) + full gates.

#### Task B5: Docs + gates
- [x] `README.md`: engine env (`PHANTOM_ENGINE`, `PHANTOM_CHROME_PATH` precedence), `phantom install-fork`, dashboard features, 5-segment coherence, folder file location.
- Test: n/a.
- How to test: final gate run: `npm run build`, `npx vitest run` (51 existing + all new), `npm audit`, `phantom-mgmt-check.mjs`, `phantom-dash-check.mjs`; manual serve + keep-alive smoke.
- Commit: final.

## Gates (run after each phase and at the end)
1. `npm run build` — clean.
2. `npx vitest run` — 51 existing + new all pass.
3. `npm audit` — 0 vulnerabilities.
4. `phantom-mgmt-check.mjs` — green, including 409 gate regression (test-profile-1 must stay blocked).
5. `phantom-dash-check.mjs` — green after B4.

## Risks / notes
- Fork flag names/values are the main uncertainty — resolve against real `--help` before finalizing A3 constants; fail-open keeps launch safe if a flag is unsupported.
- `--fingerprint` seed must be stable per profile id across relaunches (determinism is a hard requirement for the engine coherence segment).
- Server `computeCoherence` gate change (skip null segments) is behavior-safe but must not regress the 409 gate — verify test-profile-1 still returns 409 after B2.
- `install-fork` download is network + pinned-artifact dependent; unit tests must never hit the network.
- Duplicated coherence logic (`harness.ts` vs `server.ts`) is a known drift risk; both are updated in Task A6.