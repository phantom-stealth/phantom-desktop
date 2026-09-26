# 🕵️⚡ Phantom Anti-Detect Browser & Autonomous Growth Operating System
## Complete System Architecture, Specification, API Reference & AI Gap Analysis Blueprint

> **Version:** 2.5.0 Enterprise  
> **Target Audience:** AI Models (GPT-4o, Claude 3.5 Sonnet, Gemini 1.5 Pro, DeepSeek R1/V3), System Architects, Security Researchers, and Farm Engineers.  
> **Document Purpose:** Complete, unambiguous, and exhaustive specification of the entire Phantom codebase for automated analysis, feature synthesis, gap identification, and headless agent orchestration.

---

# Table of Contents
1. [Executive Summary & System Paradigm](#1-executive-summary--system-paradigm)
2. [Core Architecture & Component Topology](#2-core-architecture--component-topology)
3. [Anti-Detect & Stealth Fingerprint Engine](#3-anti-detect--stealth-fingerprint-engine)
   - 3.1 Data Contracts & Type Schemas (`src/core/fingerprint/types.ts`)
   - 3.2 Fingerprint Generation & Templates (`src/core/fingerprint/generator.ts`)
   - 3.3 Main-World JavaScript Injections & Overrides (`src/browser/launcher.ts`)
   - 3.4 Multi-Engine Abstraction (`src/browser/engine.ts`)
   - 3.5 Stealth Execution Flags & Windows Interactive GUI Launcher
4. [Egress-Driven Coherence & Network Layer](#4-egress-driven-coherence--network-layer)
   - 4.1 Universal Proxy Parser & CDP Auth (`src/core/proxy.ts`)
   - 4.2 Egress Geo Resolution & The "Permanent Region Fix" (`src/core/geo.ts`)
   - 4.3 The 409 Coherence Gate (`src/ops.ts`)
5. [Social Automation Suite & Multi-Platform Engines](#5-social-automation-suite--multi-platform-engines)
   - 5.1 Task Queue & Concurrency Controller (`src/automation/task-queue.ts`)
   - 5.2 Human Biometrics & Physics Engine (`src/automation/human-timing.ts`)
   - 5.3 TikTok Automation Studio (`src/automation/tiktok/`)
   - 5.4 Instagram Automation Suite (`src/automation/instagram/`)
   - 5.5 Facebook Automation Suite (`src/automation/facebook/`)
   - 5.6 Cross-Platform Syndication Bridge (`src/automation/cross-platform-bridge.ts`)
   - 5.7 ShadowHash Anti-Deduplication Engine (`src/automation/tiktok/shadowhash.ts`)
6. [Media Vault & Storage Subsystem](#6-media-vault--storage-subsystem)
   - 6.1 Folder Hierarchy & Account Isolation (`src/core/vault/`)
   - 6.2 MIME Detection & Intelligent Routing
   - 6.3 Sidecar Captions, Hashtag Sets & Brand Kits
   - 6.4 Disk Scanner & Asset Resolver
7. [Browser Extension: QuickFill Pro](#7-browser-extension-quickfill-pro)
   - 7.1 Deep DOM & Shadow DOM Traversal Algorithm
   - 7.2 React / SPA Prototype Descriptor Hook
   - 7.3 Multi-Locale Persona Generation (`Faker`)
   - 7.4 Mail.tm Hydra Client & Regex OTP Extractor
8. [Complete REST API Specification](#8-complete-rest-api-specification)
   - 8.1 Profile Operations (18 Endpoints)
   - 8.2 Batch Operations (4 Endpoints)
   - 8.3 Folders & Proxy Diagnostics (7 Endpoints)
   - 8.4 Automation Queue & SSE Telemetry (9 Endpoints)
   - 8.5 TikTok Automation Endpoints (8 Endpoints)
   - 8.6 Instagram Automation Endpoints (9 Endpoints)
   - 8.7 Facebook Automation Endpoints (7 Endpoints)
   - 8.8 Media Vault Endpoints (14 Endpoints)
   - 8.9 Accounts Vault Endpoints (2 Endpoints)
9. [Dashboard UI & State Management](#9-dashboard-ui--state-management)
   - 9.1 Navigation Rail & Collapsible Layout
   - 9.2 Contextual Topbar, Breadcrumbs & Global Shortcuts
   - 9.3 Dynamic View Switching & Popstate History
10. [Verification Harness & Survival Telemetry](#10-verification-harness--survival-telemetry)
    - 10.1 CreepJS & Pixelscan In-Page Probe Scrapers
    - 10.2 Longitudinal Survival Telemetry
11. [Comprehensive Gap Analysis & Missing Features](#11-comprehensive-gap-analysis--missing-features)
    - 11.1 Anti-Detect & Kernel Spoofing Gaps
    - 11.2 Verification & Identity Infrastructure Gaps
    - 11.3 Social Growth & Automation Gaps
    - 11.4 Enterprise UI/UX & Cloud Sync Gaps
12. [Architectural Roadmap to Market Leadership](#12-architectural-roadmap-to-market-leadership)

---

# 1. Executive Summary & System Paradigm

Phantom is a unified **Autonomous Anti-Detect Browser & Multi-Account Social Growth Operating System**. It bridges two historically disconnected software categories:
1. **Commercial Anti-Detect Isolators** (e.g., Multilogin, AdsPower, GoLogin, Dolphin{anty}, Octo Browser), which isolate hardware parameters and network proxies but lack native AI content engines, automated warming, and organic social growth pipelines.
2. **Social Automation Bots** (e.g., TokPilot, GramAddict, AutoTok, InstaPy, Jarvee), which automate clicks and posts but run on detectable vanilla browsers or private mobile APIs that result in rapid account bans.

Phantom eliminates this divide by providing:
- **Egress-Driven Coherence:** The browser fingerprint (OS, Screen, WebGL, Timezone, UTC Offset, Languages, Regional Audio/Font templates) is strictly derived from and locked to the proxy egress exit node.
- **Multi-Engine Execution:** Native support for C++ anti-detect kernels (`fingerprint-chromium`, `camoufox`) alongside standard stock Chromium with dynamic CDP injections.
- **Deterministic Cryptographic Video Uniquification:** The `ShadowHash` engine applies profile-seeded micro-transformations (crop, brightness/contrast jitter, speed factor, audio pitch shift, and EXIF strip) to defeat platform perceptual deduplication (pHash/MD5).
- **Multi-Platform Growth Automations:** Pre-built, multi-day warming protocols, AI scriptwriting, comment engagement, and 1-click cross-platform syndication across **TikTok, Instagram, and Facebook**.
- **Isolated Media Vaults:** Deterministic, account-specific folder trees with sidecar caption and hashtag management.

---

# 2. Core Architecture & Component Topology

```
┌────────────────────────────────────────────────────────────────────────────────────────┐
│                               PHANTOM APPLICATION TOPOLOGY                             │
└────────────────────────────────────────────────────────────────────────────────────────┘
                                           │
         ┌─────────────────────────────────┼─────────────────────────────────┐
         ▼                                 ▼                                 ▼
┌──────────────────┐             ┌──────────────────┐             ┌──────────────────┐
│ Presentation &   │             │ Core Engine &    │             │ Social Hub &     │
│ Dashboard UI     │             │ Stealth Layer    │             │ Automation Mesh  │
│ - index.html     │             │ - Fingerprint Gen│             │ - TaskQueue      │
│ - styles.css     │             │ - Proxy Parser   │             │ - HumanTiming    │
│ - app.js         │             │ - Geo Egress     │             │ - TikTokManager  │
│ - QuickFill Pro  │             │ - Multi-Engine   │             │ - IGManager      │
│ - REST & SSE API │             │ - Windows WSH    │             │ - FBManager      │
└────────┬─────────┘             └────────┬─────────┘             └────────┬─────────┘
         │                                │                                │
         └────────────────────────────────┼────────────────────────────────┘
                                          ▼
┌────────────────────────────────────────────────────────────────────────────────────────┐
│                               PERSISTENCE & DATA LAYER                                 │
│ - Profile JSON Store (`data/profiles/`)         - Survival JSONL (`data/survival/`)     │
│ - Media Vault (`phantom-data/vault/`)           - Task Rolling Queue Buffer (500 logs)  │
│ - Accounts DB (`data/accounts.json`)            - Brand Kits & Hashtag Sets Index       │
└────────────────────────────────────────────────────────────────────────────────────────┘
```

---

# 3. Anti-Detect & Stealth Fingerprint Engine

### 3.1 Data Contracts & Type Schemas (`src/core/fingerprint/types.ts`)
The identity data contract defines the complete browser fingerprint:

```typescript
export type OsFamily = "windows" | "macos" | "linux" | "android";
export type WebRtcMode = "altered" | "disabled" | "real";

export interface ScreenSpec {
  width: number;
  height: number;
  availWidth: number;
  availHeight: number;
  colorDepth: 24 | 30 | 48;
  pixelDepth: number;
  devicePixelRatio: number;
}

export interface WindowChromeSpec {
  innerWidth: number;
  innerHeight: number;
  outerWidth: number;
  outerHeight: number;
}

export interface GpuSpec {
  vendor: string;     // e.g. "Google Inc. (NVIDIA)"
  renderer: string;   // e.g. "ANGLE (NVIDIA, NVIDIA GeForce RTX 3080 Direct3D11 vs_5_0 ps_5_0, D3D11)"
  webglVersion: 1 | 2;
}

export interface TimezoneSpec {
  id: string;          // IANA ID, e.g. "America/New_York", "Europe/Paris"
  offsetMinutes: number; // Signed minutes, e.g. -300 for UTC-5
}

export interface NoiseSettings {
  canvas: boolean;
  webgl: boolean;
  audio: boolean;
  clientRects: boolean;
}

export interface Fingerprint {
  os: OsFamily;
  platform: string;
  userAgent: string;
  languages: string[];
  screen: ScreenSpec;
  windowChrome: WindowChromeSpec;
  gpu: GpuSpec;
  timezone: TimezoneSpec;
  webrtc: WebRtcMode;
  hardwareConcurrency: number;
  deviceMemory: number;
  maxTouchPoints: number;
  fonts: string[];
  noiseSeed: number; // 32-bit uint
  noise: NoiseSettings;
  mobile?: MobileDeviceProfile;
}
```

### 3.2 Fingerprint Generation & Templates (`src/core/fingerprint/generator.ts`)
- **OS Templates (`OS_TEMPLATES`):**
  - `windows`: Realistic 1080p/1440p/864p resolutions with taskbar bounds deduction (40px bottom offset), 8–16 cores, 8–32 GB RAM, ANGLE Direct3D11 NVIDIA/Intel strings, 25 standard Windows fonts.
  - `macos`: Retina resolutions with DPR 2.0 (1440x900, 1728x1117, 1512x982), menu bar bounds deduction (25px top offset), 8–12 cores, 8–64 GB RAM, ANGLE Metal Apple M1/M2/M3 / Intel UHD 630 renderers, macOS typography (SF Pro Display, Monaco, Menlo, Helvetica Neue).
  - `linux`: FreeDesktop font catalog, Mesa/NVIDIA OpenGL 4.5 renderers.
  - `android`: 5 touch points, DPR 2.625–2.75, Qualcomm Adreno / ARM Mali GPUs, Roboto fonts.
- **Deterministic Linear Congruential Generator (LCG):**
  $$\text{state}_{n+1} = (\text{state}_n \times 1664525 + 1013904223) \pmod{2^{32}}$$
  Generates reproducible pseudo-random floats in $[0, 1)$ per profile ID.

### 3.3 Main-World JavaScript Injections (`src/browser/launcher.ts`)
Injected via `page.evaluateOnNewDocument()` prior to any page execution:
1. **`navigator` Overrides:** Deletes `navigator.webdriver` from `Navigator.prototype`, overrides `platform`, `languages`, `language`, `hardwareConcurrency`, `deviceMemory`, `maxTouchPoints`.
2. **`screen` & `window` Overrides:** Patches `screen.width`, `height`, `availWidth`, `availHeight`, `colorDepth`, `pixelDepth`, and `window.devicePixelRatio`.
3. **`Intl` Fallback:** Patches `Intl.DateTimeFormat.prototype.resolvedOptions` to return `fp.timezone.id`.
4. **WebGL Vendor/Renderer Spoofing:** Hooks `WebGLRenderingContext.prototype.getParameter` and `WebGL2RenderingContext.prototype.getParameter` for parameters `37445` (`UNMASKED_VENDOR_WEBGL`) and `37446` (`UNMASKED_RENDERER_WEBGL`).
5. **2D Canvas Noise:** Hooks `CanvasRenderingContext2D.prototype.getImageData`, applying $\pm 1$ deterministic byte noise every 400 bytes using `fp.noiseSeed`.
6. **Font Enumeration Masking:** Overrides `document.fonts.check()` and `document.fonts.size` to match profile font whitelist.

### 3.4 Multi-Engine Abstraction (`src/browser/engine.ts`)
Supports 4 engine execution modes:
- `fingerprint-chromium`: Pinned Ungoogled-Chromium C++ binary with native `--fingerprint=ph-<sha256(id)[0..16]>` seed flags.
- `camoufox`: Pinned C++ Firefox fork with native C++ fingerprint cloaking.
- `stock`: Local Chrome, Microsoft Edge, Brave, or Puppeteer Chromium with CDP overrides.
- `auto`: Fallback order `fingerprint-chromium` $\to$ `camoufox` $\to$ `stock`.

### 3.5 Windows Interactive GUI Spawning
To ensure top-level Win32 desktop window station attachment:
```powershell
$wsh = New-Object -ComObject WScript.Shell
$wsh.Run('chrome.exe ...', 1, $false)
```
Followed by dynamic `DevToolsActivePort` polling, `puppeteer.connect()`, and `Browser.setWindowBounds` (`windowState: "normal"`).

---

# 4. Egress-Driven Coherence & Network Layer

### 4.1 Universal Proxy Parser & CDP Auth (`src/core/proxy.ts`)
- Parses URL schemes (`http://`, `https://`, `socks4://`, `socks5://`, `socks5h://`), `host:port:user:pass`, `user:pass@host:port`, and IPv6 brackets.
- Strips credentials from `--proxy-server` and intercepts CDP `Fetch.authRequired` events at runtime via `Fetch.enable` and `Fetch.continueWithAuth` (`ProvideCredentials`).

### 4.2 Egress Geo Resolution & The "Permanent Region Fix" (`src/core/geo.ts`)
- Executes pre-flight probe through the proxy session using `https://ipinfo.io/json`.
- Dynamically resolves:
  - **Timezone:** Real IANA timezone ID from egress exit.
  - **UTC Offset:** Calculated via `Intl.DateTimeFormat("en-US", { timeZone, timeZoneName: "longOffset" })`.
  - **Languages:** Country-to-locale dictionary mapping 25+ countries to primary and fallback languages (e.g. `DE` $\to$ `["de-DE", "de"]`, `JP` $\to$ `["ja-JP", "ja"]`).
  - **Region:** Mapped to geographical zones (`us-east`, `us-west`, `eu-west`, `jp-tok`, `au-east`).

### 4.3 The 409 Coherence Gate (`src/ops.ts`)
`buildProfile()` and `launchProfile()` validate that profile region, timezone, and language match the proxy's real egress IP. If an impossible combination is detected (e.g. proxy in Paris, FR but profile configured as `us-east` with `America/New_York`), execution is rejected with a **409 Conflict** error.

---

# 5. Social Automation Suite & Multi-Platform Engines

### 5.1 Task Queue & Concurrency Controller (`src/automation/task-queue.ts`)
- **State Lifecycle:** `pending` $\to$ `running` $\to$ `done` / `failed` / `cancelled`.
- **Concurrency Rules:**
  - Global limit: 6 concurrent tasks.
  - Type limits: Account create (2), Video upload (3), Comments (4), Warmup (4), Scrub (2), Full pipeline (2).
  - **Strict Profile Lock:** `runningProfiles: Set<string>`. A profile can only run one task at a time.
- **Rolling Logs:** 500-entry in-memory FIFO log buffer per task.

### 5.2 Human Biometrics & Physics Engine (`src/automation/human-timing.ts`)
- **Gaussian Delay (Box-Muller Transform):**
  $$\text{delay} = \max(20, \lfloor z_0 \cdot \sigma + \mu \rfloor), \quad z_0 = \sqrt{-2\ln u_1}\cos(2\pi u_2)$$
- **Keystroke Cadence (`typeSlowly`):** Target 65 WPM ($\mathcal{N}(\text{baseDelay}, 0.35\cdot\text{baseDelay})$), cognitive pauses after punctuation, and a 2% simulated typo/backspace correction rate.
- **Cubic Bézier Mouse Curves (`humanMouseMove`):**
  $$\mathbf{B}(t) = (1-t)^3\mathbf{P}_0 + 3(1-t)^2 t\mathbf{P}_1 + 3(1-t)t^2\mathbf{P}_2 + t^3\mathbf{P}_3$$
  Interpolated across 15–25 steps with micro-delays (4–9ms).
- **Bounding Box Targeting (`humanClick`):** Clicks restricted to the inner 20%–80% area of target elements with pre-hover dwell, mouse-down hold, and post-click settle periods.

### 5.3 TikTok Automation Studio (`src/automation/tiktok/`)
- **`TikTokAccountFactory`:** Mail.tm automated signup with 6-digit regex OTP polling.
- **`AutoTokScheduler`:** Auto-scrubs video with `ShadowHash` and uploads via Creator Center.
- **`TokPilotCommentCampaign`:** Keyword-triggered AI comment replies and targeted likes.
- **`TaktikBot`:** Organic For You feed browsing with stochastic likes ($p=0.25$) and follow actions.
- **`ReachOptimizer`:** 3-day progressive warming protocol (180s $\to$ 240s $\to$ 300s).
- **`AIContentStudio`:** GPT-4o viral scriptwriting (Hooks, Scenes, CTA, Audio style) and 30-day calendar generation.

### 5.4 Instagram Automation Suite (`src/automation/instagram/`)
- **`IGAccountFactory`:** Automated account registration with birthday pickers and OTP confirmation.
- **`IGPostEngine` & `IGReelsEngine`:** Feed photo, carousel, and 9:16 vertical Reel publishing.
- **`IGEngagementBot`:** Safe daily limits (80 likes, 40 follows, 30 unfollows, 15 comments, 60 story views) with 5-day progressive warm-up.
- **`IGHashtagIntel`:** Multi-tier tag grouping and volume/shadowban scrapers.
- **`IGDMAutomator`:** GPT-4o cold outreach message generation.

### 5.5 Facebook Automation Suite (`src/automation/facebook/`)
- **`FBAccountFactory`:** Automated profile creation at `facebook.com/r.php`.
- **`FBProfileBot`:** Timeline status and media posting.
- **`FBPageManager`:** Automated Page creation and announcement publishing.
- **`FBGroupManager`:** Group posting and member profile scraping.
- **`FBEngagementBot`:** Newsfeed browsing, likes, comments, and friend requests.

### 5.6 Cross-Platform Syndication Bridge (`src/automation/cross-platform-bridge.ts`)
Syndicates a single video and AI script across platforms with staggered timing:
- **TikTok:** $t_0$ (High energy hook + inline hashtags).
- **Instagram:** $t_0$ (Aesthetic body + hashtags in first comment).
- **Facebook Profile:** $t_0 + 5\text{ min}$ (Long-form narrative).
- **Facebook Page:** $t_0 + 10\text{ min}$ (Broadcast post + discussion prompt).

### 5.7 ShadowHash Anti-Deduplication Engine (`src/automation/tiktok/shadowhash.ts`)
Deterministic video mutation seeded by `SHA256(profileId)`:
- Speed factor: $1.0100\times - 1.0400\times$.
- Pixel crop: 2–6px on X and Y axes.
- Brightness/Contrast: $\pm 3\%$ micro-adjustments.
- Audio pitch/tempo: Time-stretched via `atempo` filter.
- Metadata: Stripped via `-map_metadata -1`.

---

# 6. Media Vault & Storage Subsystem

### 6.1 Folder Hierarchy & Account Isolation (`src/core/vault/`)
```
phantom-data/vault/
├── vault-db.json
├── _global/ (fonts/, watermarks/, music/, brand-kits/)
├── instagram/{profileId.slice(0,8)}__{accountRef}/
│   ├── posts/ (images/, carousels/, videos/)
│   ├── reels/ (raw/, processed/, published/)
│   ├── stories/ (images/, videos/, templates/)
│   ├── generated/ (captions/, hashtags/, scripts/, thumbnails/)
│   └── archive/ (published/, rejected/)
└── facebook/
    ├── profiles/{profileId.slice(0,8)}__{accountRef}/ (posts/, stories/, generated/, archive/)
    └── pages/{profileId.slice(0,8)}__{accountRef}/ (posts/, generated/, archive/)
```

### 6.2 MIME Detection & Intelligent Routing
Automatically routes uploaded assets: `.jpg`/`.png` $\to$ `posts/images`, `.mp4`/`.mov` $\to$ `reels/raw`, `.txt` $\to$ `generated/captions`. Dropping files into `reels/raw` automatically triggers the `reel:needs-processing` anti-shadowban scrub event.

### 6.3 Sidecar Captions, Hashtag Sets & Brand Kits
- Writes `${baseName}.txt` sidecar files in `generated/captions/`.
- Persists `${set.name}.json` in `generated/hashtags/`.
- Stores brand color palettes and typography in `_global/brand-kits/[brand]/colors.json`.

---

# 7. Browser Extension: QuickFill Pro

Located in `extensions/quickfill-pro/` (Manifest V3):

### 7.1 Deep DOM & Shadow DOM Traversal Algorithm
Recursively traverses closed/nested shadow roots to locate form inputs:
```javascript
function findInputs(root = document) {
  let inputs = Array.from(root.querySelectorAll("input, select, textarea"));
  for (const host of root.querySelectorAll("*")) {
    if (host.shadowRoot) inputs = inputs.concat(findInputs(host.shadowRoot));
  }
  return inputs;
}
```

### 7.2 React / SPA Prototype Descriptor Hook
Bypasses React/Vue synthetic event state tracking by invoking native property setters:
```javascript
const setter = Object.getOwnPropertyDescriptor(window.HTMLInputElement.prototype, "value").set;
setter ? setter.call(input, val) : (input.value = val);
input.dispatchEvent(new Event("input", { bubbles: true }));
input.dispatchEvent(new Event("change", { bubbles: true }));
```

### 7.3 Multi-Locale Persona Generation (`Faker`)
Generates realistic names, usernames, passwords, and phone numbers for 17 countries (`US`, `UK`, `DE`, `FR`, `ES`, `IT`, `NL`, `BR`, `JP`, `KR`, `CN`, `AU`, `CA`, `IN`, `RU`, `PL`, `SE`), synchronized with proxy egress.

### 7.4 Mail.tm Hydra Client & Regex OTP Extractor
Creates disposable mailboxes via Mail.tm API (with 1secmail fallback) and extracts verification codes using regex patterns (`/\b\d{4,8}\b/g`, `/code\s+is\s+([a-zA-Z0-9]+)/i`).

---

# 8. Complete REST API Specification

### 8.1 Profile Operations
- `GET /api/overview` — Full dashboard summaries and profile lists.
- `POST /api/profiles` — Creates a new anti-detect profile.
- `PATCH /api/profiles/:id` — Updates profile settings.
- `DELETE /api/profiles/:id` — Deletes profile and storage.
- `POST /api/profiles/:id/launch` — Launches browser profile.
- `POST /api/profiles/:id/stop` — Stops running browser.
- `POST /api/profiles/:id/verify` — Runs CreepJS/Pixelscan verification.
- `POST /api/profiles/:id/rename` — Renames profile.
- `POST /api/profiles/:id/recalibrate` — Re-derives fingerprint from proxy egress.
- `POST /api/profiles/:id/regenerate-fingerprint` — Re-rolls canvas/WebGL noise.
- `POST /api/profiles/:id/clone` — Clones profile with unique ID.
- `GET /api/profiles/:id/cookies` — Exports Netscape/JSON cookies.
- `POST /api/profiles/:id/cookies` — Imports Netscape/JSON cookies.
- `POST /api/profiles/:id/clear-data` — Clears cache and cookies.
- `GET /api/profiles/:id/accounts` — Lists accounts bound to profile.
- `POST /api/profiles/:id/accounts` — Binds account credentials.
- `POST /api/profiles/:id/delete-account` — Deletes bound account.

### 8.2 Batch Profile Operations
- `POST /api/profiles/batch-launch` — `{ ids: string[] }`
- `POST /api/profiles/batch-stop` — `{ ids: string[] }`
- `POST /api/profiles/batch-proxy` — `{ ids: string[], proxy?: string }`
- `POST /api/profiles/batch-delete` — `{ ids: string[] }`

### 8.3 Folders & Proxy Diagnostics
- `GET /api/folders` — Lists folders.
- `POST /api/folders` — Creates folder.
- `PATCH /api/folders/:id` — Renames folder.
- `DELETE /api/folders/:id` — Deletes folder.
- `POST /api/profiles/:id/folders` — Adds profile to folder.
- `DELETE /api/profiles/:id/folders/:folderId` — Removes profile from folder.
- `POST /api/proxy/check` — Checks proxy latency, IP, and location.

### 8.4 Automation Queue & Telemetry
- `GET /api/automation/tasks` — Lists active and completed tasks.
- `POST /api/automation/task` — Enqueues automation task.
- `GET /api/automation/tasks/:id` — Returns single task status and logs.
- `POST /api/automation/tasks/:id/cancel` — Cancels task.
- `GET /api/automation/events` / `GET /api/automation/stream` — SSE telemetry stream.
- `POST /api/automation/studio/generate-script` — AI script generation.
- `POST /api/automation/studio/calendar` — 30-day content calendar generation.
- `POST /api/automation/shadowhash/scrub` — Anti-shadowban video processing.
- `POST /api/automation/cross-platform/syndicate` — Cross-platform video syndication.

### 8.5 Social Platform Endpoints
- **Instagram:** `/api/automation/instagram/account-factory`, `warm-up`, `post`, `reel`, `story`, `engage`, `dm-campaign`, `pipeline`, `hashtags/strategy`.
- **Facebook:** `/api/automation/facebook/account-factory`, `warm-up`, `post`, `engage`, `messenger`, `page/create`, `page/post`.

### 8.6 Media Vault Endpoints
- `POST /api/vault/create`
- `GET /api/vault/list`
- `GET /api/vault/:id/stats`
- `POST /api/vault/:id/scan`
- `GET /api/vault/:id/files`
- `POST /api/vault/:id/files/add`
- `POST /api/vault/:id/files/move`
- `POST /api/vault/:id/files/archive`
- `POST /api/vault/:id/import`
- `GET /api/vault/brand-kits` & `POST /api/vault/brand-kits`
- `GET /api/vault/:id/hashtag-sets` & `POST /api/vault/:id/hashtag-sets`

---

# 9. Dashboard UI & State Management

- **Navigation Rail:** Persistent left sidebar with collapse toggle (`«` / `»`), real-time profile/vault badge counters, and theme toggle.
- **Contextual Topbar:** Dynamic breadcrumbs, global search (`Ctrl+K`), and dynamic context action button (`#btnCreate`).
- **Keyboard Shortcuts:** `1`–`7` (Switch tabs), `Ctrl+K` (Search), `c` (Create profile), `t` (Toggle view), `Esc` (Close modals).
- **History Synchronization:** Clean deep linking across `/`, `/vault`, `/automation`, `/automation/social`, `/accounts`, `/proxies`, `/settings`.

---

# 10. Verification Harness & Survival Telemetry

### 10.1 Probe Scrapers (`src/verify/harness.ts`)
Evaluates live detection against `creepjs`, `pixelscan`, and `live` targets. Evaluates `navigator.webdriver`, WebGL unmasked parameters, 2D canvas data URLs, Web Audio oscillator signatures, and font availability widths.

### 10.2 Survival Telemetry (`src/telemetry/survival.ts`)
Maintains an append-only JSONL log (`events.jsonl`) and atomic per-profile survival records (`data/survival/<profileId>.json`), tracking 30-day lifecycle states: `launched`, `checkpoint_ok`, and `banned`.

---

# 11. Comprehensive Gap Analysis & Missing Features

| Vertical | Feature | Current Phantom State | Industry Standard (AdsPower / Multilogin / Dolphin) | Gap Severity | Recommended Solution |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Anti-Detect** | **TLS / JA3 / JA4 Fingerprint** | Stock Chromium TLS handshake | Custom NetworkService modifying TLS ClientHello & cipher suite order | **CRITICAL** | Integrate CycleTLS or C++ network hooks into `fingerprint-chromium`. |
| **Anti-Detect** | **Client Hints (`Sec-CH-UA`)** | Overrides `navigator.userAgent` only | Native synchronization of `navigator.userAgentData` and HTTP headers | **HIGH** | Patch `NavigatorUAData` and `Sec-CH-UA-*` headers on stock engine. |
| **Anti-Detect** | **Canvas `toDataURL` Bypass** | Patches `getImageData` only | Native GPU shader emulation and `toDataURL`/`toBlob` hooks | **HIGH** | Intercept `HTMLCanvasElement.prototype.toDataURL` and `toBlob`. |
| **Anti-Detect** | **Audio & WebGPU Noise** | None in JS injection | Deterministic noise on `OfflineAudioContext` and WebGPU adapter | **MEDIUM** | Hook `AudioBuffer.prototype.getChannelData` and `GPUAdapter`. |
| **Verification** | **SMS / PVA Number Provider** | None (Email only) | API integrations with SMS-Activate, 5SIM, DaisySMS | **CRITICAL** | Build `sms-manager.ts` with auto-order, OTP polling, and refund on timeout. |
| **Verification** | **CAPTCHA Solver Bridge** | None (Manual) | CapSolver / 2Captcha integration for Turnstile and slider puzzles | **HIGH** | Implement automated CAPTCHA solver module for TikTok/IG puzzles. |
| **Verification** | **Custom Catch-All SMTP** | mail.tm only | Private domain catch-all via Cloudflare Workers / IMAP | **HIGH** | Build custom domain email receiver to bypass disposable email bans. |
| **Automation** | **Cookie Robot Warmup** | None | Automated crawler visiting top 50 regional sites before social login | **HIGH** | Implement `cookie-robot.ts` to accumulate browsing cookies and history. |
| **Automation** | **Interactive Story Actions** | Story posting only | Auto-answering story polls, sliders, quizzes | **MEDIUM** | Add story interaction handler in `IGEngagementBot`. |
| **Automation** | **TTS Voiceover Engine** | Text script only | Edge-TTS / ElevenLabs audio generation and automatic FFmpeg merge | **MEDIUM** | Integrate Edge-TTS for automatic voiceover synthesis in `AIContentStudio`. |
| **UI & Cloud** | **Inline Cell Spreadsheet Edit** | Drawer/Modal editing | Double-click inline cell editing (Dolphin{anty} style) | **MEDIUM** | Implement contenteditable inline table editing in `app.js`. |
| **UI & Cloud** | **Visual RPA Workflow Studio** | Predefined Node.js runners | Drag-and-drop node graph builder (AdsPower RPA style) | **LOW** | Build visual node editor canvas for custom macro chains. |

---

# 12. Architectural Roadmap to Market Leadership

```
┌────────────────────────────────────────────────────────────────────────────────────────┐
│                                 PHANTOM ROADMAP MATRIX                                 │
└────────────────────────────────────────────────────────────────────────────────────────┘

 [ PHASE 1: HARDENED STEALTH CORE ]
 ├── TLS/JA3/JA4 Handshake Normalization
 ├── Full User-Agent Client Hints (`navigator.userAgentData`)
 ├── Canvas `toDataURL` / `toBlob` & Web Audio API Noise
 └── Automated Cookie Robot Crawler (`cookie-robot.ts`)

 [ PHASE 2: IDENTITY & VERIFICATION MESH ]
 ├── SMS Gateway Integration (SMS-Activate / 5SIM / SMSPool)
 ├── Custom Catch-All IMAP & Cloudflare Email Gateway
 ├── Automated CAPTCHA Solver Bridge (CapSolver / 2Captcha)
 └── RFC 6238 TOTP 2FA Token Generator in QuickFill Pro

 [ PHASE 3: PRODUCTION VIDEO & VOICE STUDIO ]
 ├── Edge-TTS / ElevenLabs Voiceover Synthesis
 ├── Automated Subtitle Generation & FFmpeg Video Assembly
 ├── Persistent SQLite Action & Interaction Memory
 └── Omnichannel Expansion: X/Twitter & YouTube Shorts

 [ PHASE 4: ENTERPRISE UI & RPA MARKETPLACE ]
 ├── Inline Spreadsheet Table Editing (Dolphin-speed)
 ├── Visual Drag-and-Drop RPA Workflow Canvas
 └── One-Click Profile Extension Center
```
