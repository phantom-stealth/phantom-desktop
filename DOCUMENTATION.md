# Phantom Codebase Documentation

## Overview
Phantom is an enterprise-grade synthetic identity and social automation platform. It provides a stealthy anti-detect browser environment combined with massive AI capabilities to simulate users and drive social engagements on platforms like Tinder, Bumble, TikTok, and Twitter.

## Repository Structure (`src/`)

### `src/automation/`
The workhorse of the application. Contains platform-specific automation sequences.
- **`dating/`**: Tinder/Bumble swiping logic, AI bio generation, reply engines, and escalation state machines.
- **`tiktok/`**: Video upload routines, shadowhash video processing to avoid duplicate detection, and AI content studios.
- **`ama/`** (Ask Me Anything / Engagement): Engines tailored to Instagram, Threads, and Twitter to build passive audience engagement.
- **`reply-engine/`**: LLM-powered dynamic messaging system using contextual awareness and RAG.

### `src/core/`
The backbone infrastructure that enables automation to run safely and consistently.
- **`ai/`**: `ModelRouter.ts` manages dispatching text/image tasks. Load balances between local LLMs (Llama/Ollama) and cloud APIs (Opencode, Claude, OpenAI).
- **`fingerprint/`**: Obfuscates browser fingerprints (Canvas, WebGL, Audio, Battery) to avoid bot detection.
- **`persona/`**: Manages the generated synthetic identities (bio, stats, demographics, assigned proxies).
- **`security/`**: `ThreatDetectionEngine.js` for IP banning, DDOS mitigation, and WebRTC leak protection.

### `src/python-ai/`
A FastAPI standalone server providing native local AI inference for both text and images, mimicking cloud APIs without the privacy risk.
- **`main.py`**: Maps `/v1/images/generations` to local Diffusers/ComfyUI and `/v1/chat/completions` to Transformers/Ollama.

### `src/saas/`
Multi-tenant architecture modules.
- **Billing**: Stripe integrations for tenant subscription management.
- **White-Labeling**: Custom branding logic so agencies can resell Phantom access.
- **Usage Metering**: Tracks API costs and generation tokens per tenant to enforce quotas.

### `src/ui/` (Frontend)
Vite + React SPA (Single Page Application) providing the dashboard interface.
- **`pages/`**: 
  - `Personas/PersonaBuilderPage.tsx`: Highly interactive form for configuring synthetic identities.
  - `Settings/SettingsPage.tsx`: Configures AI keys, local API bindings, and webhooks.
  - `Security/EvasionHubPage.tsx`: The command center for antidetect fingerprint tweaking.
- **`styles/`**: Utilizes `design-system.css` featuring a strict 8pt grid, design tokens, and glassmorphic UI patterns.

## Feature Pipelines

1. **Synthetic Identity Pipeline**
   - User inputs traits into `PersonaBuilderPage` -> Backend invokes AI Generator -> `PersonaStore` saves profile -> Cloud Cookie Vault caches sessions.

2. **Dating Automation Pipeline**
   - Native AVD (Android Virtual Device) spins up -> Appium bridges the connection -> AI reads screen/DOM -> Swipe/Message decision executed via `dating-engine.ts`.

3. **Stealth Operation Pipeline**
   - Proxy Manager routes traffic -> Threat Engine strips headers -> WebRTC is proxied -> Fingerprint generator perturbs canvas readout -> Browser is launched via Puppeteer/Playwright.

## Setup & Deployment
Phantom leverages Docker Compose for isolated microservices:
- `phantom-server`: The main Node.js backend & UI host.
- `python-ai`: The local ML inference engine (GPU accelerated).
- `appium-server`: The mobile emulation layer.
- `prometheus / grafana`: Health and metrics monitoring.
