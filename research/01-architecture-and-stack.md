# Deep dive 01 — Architecture & stack (how it was reverse-engineered)

> Supports **F2**. This note shows the receipts behind the claim that Vocallabs is an **orchestration layer over third-party models, running on Subspace's backend** — and exactly how each layer was identified from public sources.

## The one-paragraph version

Vocallabs' marketing says "full-stack ownership" and "proprietary conversation intelligence." But its own public API docs serve every endpoint from **`api1.superflow.run`** (Subspace's backend, not `vocallabs.ai`), bill via a **wallet/credit system** (`getGreenBalance`, `402 insufficient balance`), expose **swappable LLMs** (`getAIModels`) and a **TTS voice catalogue** (`voice_id`), and run **"analytics" as an LLM prompt over the transcript** (`analytics_prompt`). Vocallabs' own open-source dictation app independently documents a **Deepgram + Groq/OpenRouter** stack. Together this is the signature of an orchestration product, not an in-house model house.

## The evidence chain

| Layer | What I observed | Where |
|---|---|---|
| **Backend = Subspace** | Endpoints are `https://api1.superflow.run/b2b/...`. "Superflow" is Subspace's backend; the docs even include `whatsubTransactionHistory` — "WhatSub" was Subspace's original product name. | Official API docs (every endpoint) |
| **Wallet billing** | `getGreenBalance` returns a "green" balance (Subspace's credits); billable endpoints return `402 — insufficient balance — wallet cannot cover the request`. | API docs, wallet + agent endpoints |
| **Backend tech = Hasura** | The Android app crash leaks `Hasura.GraphQL.Execute.Action.Types.ActionWebhookErrorResponse`. Hasura = an off-the-shelf GraphQL-over-Postgres engine; "Action webhooks" are how it calls custom business logic. | Android app failure modal (F8) |
| **Hosting / assets** | The docs site loads assets from `cdn.subspace.money`. | `docs.vocallabs.ai` |
| **Swappable LLMs** | `getAIModels` lets a caller choose the model — you don't expose a model picker if the intelligence is your own single proprietary model. | API docs (Agents) |
| **Third-party TTS** | `voice_id` + `getVoicesByLanguageComment` = a catalogue of selectable voices. | API docs (Agents) |
| **"Analytics" = prompts** | `analytics_prompt` per agent and `updatePostCallData` (you supply a prompt to extract each field) — i.e. analysis is an LLM reading the transcript, not acoustic processing. | API docs (Analytics) |
| **Confirmed stack via their own app** | `vocalflow` (their open-source macOS dictation app) documents **Deepgram (ASR)** + **Groq/OpenRouter (LLM)**. Strong evidence the same building blocks power the core product. | `github.com/Vocallabsai/vocalflow` |
| **Realtime transport** | The SDK connects to `wss://rupture2.vocallabs.ai/ws?callId=...&sampleRate=8000` — an internal-sounding host, 8 kHz default. | `vocal-sdk` README |

## Why this matters strategically

- **The moat narrative is fragile.** Anything built as "prompts over Deepgram" can be rebuilt by a competent team quickly. A technical buyer or investor who looks one layer down sees this immediately.
- **The Subspace coupling cuts both ways.** Upside: speed, shared infra, lower cost, an existing customer base to cross-sell. Risk: a structural single point of failure and shared-billing complexity — Vocallabs' fate is tied to Subspace's backend.
- **There is a real moat, just not the advertised one.** See `../frameworks/moat-audit.md`: India telephony + Aadhaar/PAN KYC + (proposed) payments is defensible; "proprietary intelligence" is not.

## Honesty / limitations

This is a **strong, evidence-backed inference**, not a lab-verified fact — I could not run a live call (no self-serve credentials; the app call failed). `getAIModels` + the `vocalflow` stack make orchestration the most likely architecture by far, and **F10** is the single live test that would convert this from inference to proof.
