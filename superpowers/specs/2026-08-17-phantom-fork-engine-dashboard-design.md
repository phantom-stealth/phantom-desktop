# Phantom — Undetectable Fork Engine + Competitive Dashboard (Design)

- **Date:** 2026-08-17
- **Status:** Draft for review
- **Delivery:** One coherent milestone (Approach 1) — both tracks land together.

## Objective

Make Phantom competitive with Gologin/Multilogin class tools on two fronts:

1. **Track A — Browser undetectability via a Chromium fork.** Add an engine
   abstraction that can drive `adryfish/fingerprint-chromium` (a BSD-3 licensed,
   ungoogled Chromium fork with deterministic `--fingerprint` seed flags and
   prebuilt Windows x64 binaries) so sessions are indistinguishable from a real
   user browser at the *browser* layer, not just the JS layer. The engine is
   auto-detected now, and a `phantom install-fork` command auto-downloads it later.
2. **Track B — Dashboard GUI overhaul.** Replace the inline single-page dashboard
   (`DASHBOARD_HTML` in `src/server.ts`) with a static-asset application that
   reaches Gologin/Multilogin feature parity: table+card views, configurable
   columns, search/filter/sort, bulk actions, running view, profile detail drawer
   with fingerprint + coherence preview, tags/status/notes/folders with
   persistence, keyboard shortcuts, and a brand-new visual identity.

## Context & Current State

Key facts verified in code (all paths under `C:\Users\Robocop\Desktop\Projects\Phantom`):

- `src/config.ts` — `Config` has `chromePath?`, `headless`, `launchTimeoutMs`,
  `navTimeoutMs`, `retries`, `defaultRegion`, `verifyTargets`, `serveHost`,
  `servePort`, `keepAlive`. `loadConfig(env)` reads `PHANTOM_*` env vars.
- `src/core/profile.ts` — `Profile` = `id, name, status("created"|"active"|"retired"),
  fingerprint, proxy?, region, egressGeo?, createdAt, lastLaunchedAt?`. Helpers
  `createProfile`, `markActive`, `markRetired`.
- `src/core/store.ts` — `ProfileStore` persists one JSON file per profile
  (`{id}.json`), `get()` runs `parseProfile` (strict required-field check), `list()`
  sorts by createdAt/name, `remove()` deletes the file.
- `src/core/validate.ts` — `isProfile` + `parseProfile`; rejects files missing
  required fields. Optional additive fields pass through untouched.
- `src/core/fingerprint/types.ts` — `Fingerprint` = `profileId, os, userAgent,
  platform, languages, hardwareConcurrency, deviceMemory, maxTouchPoints, screen,
  windowChrome, gpu, timezone, fonts, noiseSeed, createdAt`. `OsFamily` =
  windows|macos|linux|android.
- `src/verify/harness.ts` — `checkCoherence(profile, reports)` currently checks
  **4 segments**: `tz`, `offset`, `lang`, `region` (the "TZ/OFF/LNG/REG" bits).
  `SignalReport` captures `userAgent`, `navigatorPlatform`, timezone, screen, gpu,
  languages, canvas/audio fingerprints, `webdriverDetected`.
- `src/browser/launcher.ts` — `resolveExecutable()` (lines 10–41) walks a
  `CANDIDATE_EXECUTABLES` list, `LaunchOptions` (43–51), `launch()` (line 204),
  `puppeteer.launch` (line 238) with `args` at line 243, and an init script that
  overrides `navigator`/`screen`/`window` properties.
- `src/ops.ts` — `Ops.verify()`/`create()`/`spawnSession()`/`launch()` all pass
  `executable: this.cfg.chromePath`. Keep-alive relaunch logic in
  `scheduleRelaunch()` (backoff 1s→2s→4s→max 30s).
- `src/server.ts` — API handlers, `ops.launch(profile)` at line 259, `DASHBOARD_HTML`
  template literal starting ~line 297 (~300 lines: dark theme, card grid, 5 stats,
  create form). `/api/overview` already exists.
- `src/index.ts` — CLI dispatch for `create|verify|list|serve`; `usage()` text;
  `serve()` receives explicit opts (not the whole config).
- `tests/*.test.ts` — 5 files, 51 tests. Current gates must stay green:
  `npm run build`, `npx vitest run`, `npm audit` (0).
- Only profile on disk: `test-profile-1`
  (`c635391a-03a8-4f18-8ac9-a67be8b6421f`), intentionally incoherent → verify/launch
  returns **409** via the coherence gate (`Ops.assertCoherent`).
- Coherence gate semantics: 409 only when a *prior* check concluded incoherent;
  first-ever verify is allowed.
- Dev server: PID 38916 on `127.0.0.1:4173` (`node dist/index.js serve`).
- Egress (ipinfo) currently Lagos, NG · Africa/Lagos; Nigeria is unmapped in
  `src/core/geo.ts`, so `deriveFromGeo` falls back to `defaultRegion` (`us-east`).
  All checks must remain egress-agnostic.
- The assistant cannot view images; GUI verification is headless via
  puppeteer-core (scripts under `C:\Users\Robocop\AppData\Local\Temp\opencode\`).

## Decisions (locked via clarifying Q&A)

| Question | Decision |
| --- | --- |
| Fork delivery | **Both**: auto-detect now + `phantom install-fork` auto-download later. |
| Dashboard architecture | **Static asset folder** (`src/dashboard/`, vanilla, no framework). |
| GUI feature scope | **UI + tags/folders persistence** (full option). |
| Fork flag depth | **Full coherent flag driving** (`--fingerprint=<seed>`, platform/brand/version, lang/timezone); JS/CDP overrides remain as a safety layer. |
| Visual direction | **New identity** (fresh accent, typography, art direction). |
| Delivery approach | **Approach 1** — one coherent milestone, both tracks together. |

Fork choice: **`adryfish/fingerprint-chromium`** (2.9k★, BSD-3, ungoogled Chromium
+ fingerprint patches, prebuilt Windows/Linux/macOS, puppeteer-core compatible via
`executablePath`). Flags (verify exact spellings at implementation time):
`--fingerprint=<seed>`, `--fingerprint-platform`, `--fingerprint-platform-version`,
`--fingerprint-brand`, `--fingerprint-brand-version`, `--disable-spoofing`.
Also considered and rejected: CloakBrowser (29k★, commercial, license key),
clearcote-browser (68★), clark-browser (119★, no Windows builds), KC Browser.

---

## Track A — Fork Engine

### A1. Engine abstraction (`src/browser/engine.ts`, new)

```ts
export type EngineKind = "stock" | "fingerprint-chromium";

export interface ResolvedEngine {
  kind: EngineKind;
  executable: string;
  supportedFlags: Set<string>; // probed from `--help`
}

export async function resolveEngine(opts: {
  explicit?: string;      // PHANTOM_CHROME_PATH (or config.chromePath)
  mode?: EngineMode;      // config.engine
  forkDir?: string;       // cached install dir
  probeMs?: number;       // default 5000
}): Promise<ResolvedEngine>
```

- `EngineMode = "auto" | "stock" | "fingerprint-chromium"` (new).
- **Resolution order (auto):**
  1. If `explicit` (PHANTOM_CHROME_PATH) is set, it always wins and is treated as
     `stock` (user opted in to a specific binary; it can never be the cache install,
     which lives at `~/.cache/phantom/`).
  2. Else if a verified fork install exists in the cache dir → `fingerprint-chromium`.
  3. Else → `stock` via the existing `resolveExecutable()` candidate walk.
  To force the fork explicitly (bypassing auto), set `PHANTOM_ENGINE=fingerprint-chromium`
  — it then requires a verified install and fails open to stock only with a warning.
- **Mode `stock`:** always use `explicit` or the candidate walk; never the fork.
- **Mode `fingerprint-chromium`:** require the fork install; if absent, log a clear
  warning and **fail-open** to stock (never crash launches).
- **Probe** (network-free): run `<exec> --version` and `<exec> --help` with a short
  timeout (default 5s). Parse the flag names from `--help` into `supportedFlags`.
  Probe failures are logged and treated as "no fork flags supported" (fail-open).
  Probe results are cached per executable in-process.
- New config: `PHANTOM_ENGINE` env (`auto` default), `Config.engine` field.
- `launch()` (launcher.ts) keeps its signature; engine resolution happens in
  `Ops`/`create`/`server` and is passed in as `engine: ResolvedEngine`, or resolved
  inside `launch()` from config when not supplied. Prefer: resolve in `Ops` once and
  pass down, so `server.ts` and `index.ts` share the same path.

### A2. Fork argument builder (`buildForkArgs`, new, in engine.ts or launcher.ts)

```ts
export function buildForkArgs(
  profile: Profile,
  engine: ResolvedEngine,
): string[]
```

- **Deterministic seed (critical):** `--fingerprint=ph-<sha256(profile.id).slice(0,16)>`.
  This is **not** `profile.fingerprint.noiseSeed` — the seed must be stable across
  every relaunch so the fork reproduces the identical fingerprint each time.
- **Platform/brand/version:** derived from `profile.fingerprint.os` +
  `profile.fingerprint.userAgent` (parse OS + Chrome major version from the UA;
  brand-version mirrors it). Map `OsFamily` → fork platform value
  (windows/mac/linux/android).
- **Language:** `--lang=<profile.fingerprint.languages[0]>` (e.g. `en-US`).
- **Timezone:** pass `profile.fingerprint.timezone.id` using the fork's timezone
  flag (exact name to verify — see Open Questions; expected `--timezone=<IANA>`).
  If unsupported, omit and rely on the existing JS override (fail-open).
- **Fail-open rule:** every candidate flag is only emitted if it is in
  `engine.supportedFlags`; unsupported flags are skipped with a one-line log, never
  an error.
- **Stock engine:** `buildForkArgs` returns `[]` (stock gets no fork flags; existing
  JS/CDP overrides still apply).
- Existing init-script overrides in launcher.ts stay untouched as a **safety layer**
  so stock engines remain as stealthy as today.

### A3. Coherence: new `engine` segment (5th)

- `checkCoherence(profile, reports)` currently returns bits `tz/offset/lang/region`
  (4). Add an `engine` bit, making segments **TZ/OFF/LNG/REG/ENG** (5).
- `Profile` gains optional additive field `engine?: EngineKind`, recorded at
  creation time by `create()` / `buildProfile`.
- The `engine` segment is coherent when the engine used for the verified session
  (`resolveEngine` result) equals `profile.engine`. If a profile was built expecting
  `fingerprint-chromium` but launched with `stock` (or vice versa) → incoherent.
  This catches "fingerprint promises fork-level stealth but a stock binary was used".
- Update everything that assumes 4 segments: README, dashboard segment display,
  `phantom-dash-check.mjs`, and any unit tests asserting the bit count.
- `CoherenceRule`/segment listing in `src/core/fingerprint/types.ts` is extended to
  document the 5th segment.

### A4. `phantom install-fork` CLI command

- New command kind `install-fork` in `src/index.ts`; add to `usage()`.
- Behavior:
  1. Resolve cache dir `~/.cache/phantom/fingerprint-chromium/` (Windows:
     `join(os.homedir(), ".cache", "phantom", "fingerprint-chromium")`).
  2. Download the pinned Windows x64 release zip from the fingerprint-chromium
     GitHub release (URL + exact version pinned at implementation time — see Open
     Questions), streaming to a temp file.
  3. Verify the file against a **pinned SHA-256**; mismatch → abort with clear error.
  4. Extract to the cache dir; verify a `chrome.exe` exists; write a small
     `installed.json` (version, checksum, installedAt) for `resolveEngine` to trust.
  5. If not on win32 → clear error: no prebuilt offered for this platform; instruct
     manual `PHANTOM_CHROME_PATH`.
- Idempotent: re-running refreshes/verifies the pinned install.
- `resolveEngine` treats an install as valid only when `installed.json` checksum
  matches and the binary exists.

---

## Track B — Dashboard GUI

### B1. Static asset dashboard (`src/dashboard/`, new)

- New folder `src/dashboard/` with `index.html`, `styles.css`, `app.js` — vanilla
  HTML/CSS/JS, no build step, no framework.
- `DASHBOARD_HTML` literal in `src/server.ts` is **deleted**; `GET /` serves
  `index.html`; static files served with correct content types; CSP nonce applies
  to inline `<script>`/`<style>`.
- Server resolves the dashboard dir from `import.meta.url` (`new URL("../dashboard/",
  import.meta.url)`) so it works from `src/` (dev) and `dist/` (prod).
- **Build step change:** the current `build` is just `tsc`, which will not copy the
  static assets. Add a tiny copy step so `dist/dashboard/` exists for prod:
  `"build": "tsc && node scripts/copy-dashboard.mjs"` (copy `src/dashboard/*` →
  `dist/dashboard/*` with a portable Node fs script). The dev path (`tsx src/index.ts`)
  needs no copy.

### B2. API + persistence extensions

**Profile metadata (optional additive fields — no migration, old files stay valid):**
- `Profile` gains `tags?: string[]`, `notes?: string`, `engine?: EngineKind`.
- Store **normalize-on-read**: `ProfileStore.get()`/`list()` fill defaults
  (`tags: []`, `notes: "", engine: "stock"`) on profiles that lack them, without
  rewriting files. `isProfile`/`parseProfile` are untouched (fields optional).

**`PATCH /api/profiles/{id}`** — body `{ tags?, status?, notes? }`:
- Validate: `status` ∈ {created, active, retired}; `tags` array of ≤ 20 unique,
  trimmed strings, each ≤ 40 chars; `notes` string ≤ 2000 chars. 400 on violation.
- Persists via `store.save`, returns the updated profile.

**Folder store (`src/core/folderStore.ts`, new):**
- Index file `data/folders.json`:
  ```json
  { "version": 1,
    "folders": [{ "id", "name", "color", "createdAt" }],
    "memberships": { "<folderId>": ["<profileId>", ...] } }
  ```
  Multi-membership (a profile can belong to several folders).
- Atomic writes (write temp file + rename) and an in-process serialization queue to
  avoid read-modify-write races.
- Endpoints:
  - `GET /api/folders` → folders with member counts.
  - `POST /api/folders` `{ name, color? }` → create (400 if name empty/dup).
  - `PATCH /api/folders/{id}` `{ name?, color? }` → rename/recolor.
  - `DELETE /api/folders/{id}` → delete folder + its memberships.
  - `POST /api/profiles/{id}/folders` `{ folderId }` → add membership (idempotent).
  - `DELETE /api/profiles/{id}/folders/{folderId}` → remove membership.
  - 404s for unknown folder/profile ids.

**`/api/overview` extension:** return folders (with counts) plus per-profile
metadata the UI needs in one call: `tags, status, notes, engine, folderIds,
running, lastCoherence` (coherence summary), `lastLaunchedAt`.

### B3. Frontend (new identity)

**Visual direction:** brand-new accent palette, typography ramp, and art direction.
Light/dark toggle (defaults to `prefers-color-scheme`, persisted to localStorage).
WCAG AA contrast; `prefers-reduced-motion` respected.

**Layout:**
- **Top bar:** brand, global search, view toggle (table/card), running-count,
  theme toggle, Create button.
- **Left sidebar:** folder list (name + count, colored dot, editable/removable),
  filter chips: All · Running · Incoherent · Banned · per-tag.
- **Content area:** table view and card view.
- **Bottom status bar:** server stats (profiles, running, folders, last sync).

**Table view:** configurable columns (show/hide, reorder via drag, resize),
sortable, persisted to localStorage. **Card view:** profile cards with avatar/initials,
name, tag chips, status badge, region, last-launched, inline quick actions.

**Bulk selection toolbar:** select-all/select-range; bulk launch, stop, add-to-folder,
assign tags, delete (with confirm), move to folder.

**Profile detail drawer** (right-side, focus-trapped, Esc closes):
- Fingerprint preview: OS, UA, platform, languages, screen, GPU, timezone, fonts count.
- 5-segment coherence breakdown (TZ/OFF/LNG/REG/ENG) with per-segment pass/fail.
- Live egress info, proxy config summary.
- Notes editor, tags editor, status control, folder membership.
- Actions: Verify / Launch / Stop (respecting the 409 gate with an inline error).

**Running view:** list of currently-running profiles with egress + session age.

**Right-click context menu** on profiles (same actions as bulk toolbar).

**Keyboard shortcuts:** `c` create, `/` focus search, `t` theme, `d` open drawer,
`g` grid/cards, `Esc` close drawer/menu. Documented in the UI (help hint).

**Accessibility:** keyboard-navigable, aria-labels, focus trap in drawer/menu,
sufficient contrast, visible focus states.

---

## Verification Plan

- **Unit tests (vitest):**
  - Store normalize-on-read (missing tags/notes/engine get defaults; files untouched).
  - FolderStore CRUD + multi-membership + idempotent add + atomic write.
  - `PATCH /api/profiles` validation (bad status, tag overflow/dup, oversized notes).
  - `resolveEngine` probe with a mocked child-process (flag parsing, fail-open on
    probe error, mode precedence, explicit-path wins).
  - `buildForkArgs` determinism: same profile → identical args across calls; seed
    derives from profile.id; unsupported flags skipped.
  - `checkCoherence` 5th `engine` segment (mismatch → incoherent; match → ok).
  - Update any existing tests asserting 4 segments.
- **Headless GUI check** — extend `phantom-dash-check.mjs` (temp dir) to cover:
  drawer open/close, table↔card switch, theme toggle persistence, bulk-select,
  tags/status/notes/folders via API, sidebar filter chips, keyboard shortcut.
- **Gates:** `npm run build`, `npx vitest run` (51 existing + new all green),
  `npm audit` (0). Server rebuilt + restarted after implementation.

## Risks & Mitigations

- **Fork flag names/values** — must be confirmed against the fork's actual
  `--help`/source at implementation time. Mitigation: fail-open probing; exact
  spellings tracked in Open Questions.
- **Seed stability** — using `sha256(profile.id)` prefix (not `noiseSeed`) keeps the
  fingerprint byte-identical across relaunches. A unit test asserts determinism.
- **Install integrity** — pinned version + pinned SHA-256; `resolveEngine` refuses a
  corrupt/partial install and fails open to stock.
- **Auto-detect stays network-free** — probe and resolution never hit the network.
- **Static serving + CSP nonce** — must work identically from `src/` and compiled
  `dist/`; covered by headless check.
- **Concurrent writes to folders.json** — serialization queue + atomic rename.
- **Regression surface** — 51 existing tests plus gates; keep-alive/409 behavior must
  not change (covered by `phantom-mgmt-check.mjs`).

## Out of Scope

- Fork installers for macOS/Linux (Windows x64 only for now).
- Direct integration with Gologin/Multilogin services.
- Android fingerprint behavior in the fork (fork is a desktop browser; Android
  profiles keep JS-override coverage).
- Multi-user auth / team management / remote sync.

## Open Questions (resolve at implementation time)

1. Exact fingerprint-chromium flag spelling for timezone and language
   (candidate: `--timezone=<IANA>`; confirm via `--help`). If absent, rely on JS
   override.
2. Required value formats for `--fingerprint-platform-version` / `--fingerprint-brand`
   / `--fingerprint-brand-version` (e.g. `"126.0.0.0"`, `"Chrome"`).
3. Pin exact release tag + SHA-256 for the Windows x64 zip in `install-fork`.
4. Whether `--fingerprint` accepts an arbitrary string seed or requires a specific
   format; if numeric-only, hash to a numeric form instead.