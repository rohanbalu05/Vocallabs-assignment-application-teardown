# Deep dive 03 — GitHub org teardown (`github.com/Vocallabsai`)

> Supports **F2, F3, F4, F5, F6**. Vocallabs open-sources several repos; reading them is where most non-obvious findings came from. Four public repos:

## `vocal-sdk` — React Native client SDK

- **What it is:** a React Native-only SDK for real-time calls over WebSocket (`v1.1.8`).
- **Audio:** defaults to **8 kHz** (`sampleRate: 8000`); audio config exposes **half-duplex** params (`halfDuplexRms`, `halfDuplexPeak`), noise gate, ducking, DC blocker. → **F4** (half-duplex weakens barge-in).
- **Transport:** connects to `wss://rupture2.vocallabs.ai/ws?callId=...` — an internal-sounding host leaking into public docs.
- **Maturity tells:** the "test suite" is a single `node test.js`; the README shows how to *connect* a call but not how to *get* a `callId` or credentials (you can't actually place a call from the SDK docs alone). → reinforces **F1/F3**.
- **Reach:** React Native only — no web/JS, Python, or server SDK, despite a "for developers" pitch. Narrow for the stated ICP.

## `n8n-nodes-vocallabs` — n8n automation node

- Ships a **21 KB `apidocs.md`** that dumps the entire B2B API — the single most revealing file in the org (see deep dive 02).
- The docs are clearly **exported from Insomnia** (User-Agent artifacts, `?0=` params) rather than authored. → **F3**.

## `vocalflow` — free macOS *dictation* app (the off-brand one)

- **What it is:** a free, open-source macOS menu-bar **dictation** tool (a Wispr Flow clone) — **unrelated to the voice-agent business**. → **F5**.
- **Reveals the stack:** documents **Deepgram (ASR)** + **Groq/OpenRouter (LLM)**. → key corroboration for **F2**.
- **Brand hygiene:** README is SEO keyword soup ("FREE FREE FREE", competitor names). Build binaries (`.dmg`, `.pkg`, `.app.zip`) are committed straight into git. A stray `oplo_square.png` suggests a rebrand leftover. → **F5**.
- **Note:** it's the most *polished* repo in the org — effort went to a non-core side project while the core SDK lacks self-serve.

## `chrome-extension-for-tab-recording`

- The repo behind the advertised Chrome extension. The published Web Store listing is now **"item not available"**; the extension only survives on ad-laden mirror sites. → **F6**.

## Open gap (transparency)

I attempted to pull commit history to gauge how actively each repo is maintained, but the call timed out (an MCP/network issue on my side), so I'm not asserting a maintenance cadence. The docs were last updated **27 May 2026**, so the project is clearly live. This is a minor, honestly-flagged gap.
