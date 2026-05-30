# Vocallabs.ai — Product Teardown

![Type](https://img.shields.io/badge/Type-Product_Teardown-6f42c1)
![Findings](https://img.shields.io/badge/Findings-10-blue)
![Pillars](https://img.shields.io/badge/Pillars_covered-5%2F5-success)
![Frameworks](https://img.shields.io/badge/Frameworks-SWOT_·_Porter's_·_Moat_Audit-orange)
![Lens](https://img.shields.io/badge/Includes-Founder's_Lens-e36209)

> A product teardown of **Vocallabs.ai** (AI voice agents) for the Product Intern assignment.
> Instead of grading the polish of the marketing site, this teardown does something most reviews don't: it **takes Vocallabs' own four claimed moats and stress-tests each one against evidence** — the live website, the Android app, the Chrome extension, the public API docs, and the company's own open-source code on GitHub — then reframes the findings through a **founder's lens** (what's deliberate startup debt vs what now costs real revenue, and what I'd do in week one).
>
> **The one-line takeaway:** Vocallabs presents as *"Voice AI infrastructure for developers"*, but underneath it is a **sales-gated orchestration layer running on Subspace's backend**, where the headline developer surfaces (self-serve signup, the app, the extension, the docs' response schemas) are either missing, broken, or unshipped — and the India-specific *compliance* and *closed-loop* work that would actually make it defensible isn't there yet. The good news: the real moat is hiding in plain sight, and most of this is fixable fast.

---

## Table of contents

1. [TL;DR — the 60-second read](#1--tldr--the-60-second-read)
2. [How I tested (and what I couldn't)](#2--how-i-tested-and-what-i-couldnt)
3. [Plain-English glossary](#3--plain-english-glossary)
4. [What Vocallabs actually is — the architecture](#4--what-vocallabs-actually-is--the-architecture)
5. [The Moat Audit — testing their 4 claimed moats](#5--the-moat-audit--testing-their-4-claimed-moats)
6. [The 10 findings](#6--the-10-findings)
7. [The one thing I couldn't test — and how I'd evaluate it on Day 1](#7--the-one-thing-i-couldnt-test--and-how-id-evaluate-it-on-day-1)
8. [Founder's Lens — deliberate debt, the real moat, and my first week](#8--founders-lens--deliberate-debt-the-real-moat-and-my-first-week)
9. [Prioritisation — what to fix first, and why](#9--prioritisation--what-to-fix-first-and-why)
10. [What I'd genuinely keep (strengths)](#10--what-id-genuinely-keep-strengths)
11. [Frameworks & deep dives](#11--frameworks--deep-dives)
12. [Repository map](#12--repository-map)

---

## 1 · TL;DR — the 60-second read

- **What it is:** Vocallabs sells AI voice agents that make and take business phone calls (sales, support, booking). The website pitches it as developer infrastructure that "scales from zero to millions of concurrent calls."
- **What I did:** Used every public surface like a real customer — and, because Vocallabs open-sources its SDK, n8n node and an app, I also **read their code and their full API docs**. That combination is what surfaced the non-obvious findings.
- **The sharpest finding (F2):** The product is, on the evidence, an **orchestration layer over third-party models** (Deepgram for speech-to-text, swappable LLMs, third-party TTS) running on **Subspace's backend** (`api1.superflow.run`, Hasura, a "green-balance" wallet). That directly tests their claim of *"proprietary conversation intelligence."*
- **The most damaging finding (F8):** On the public Android app, the core action — placing a call — **fails with a raw backend error leaked to the user**.
- **The one most applicants will miss (F4):** An India company auto-dialing business calls sits squarely under **TRAI's TCCCPR (incl. the Feb-2025 auto-dialler/robocall + DLT rules)** and the **DPDP Act 2023 / Rules 2025** — yet nothing in the product surfaces consent, DLT, or AI-disclosure handling. This is a real adoption blocker for their own target verticals.
- **Coverage:** 10 findings across all five product pillars (GTM & ICPs, Competitor Analysis, Features/Services, UX, Potential Collaborations) — not stacked in one area.
- **Balance:** This is not a hit piece. Vocallabs has a genuinely **broad API** and one **real India-first moat** (in-call Aadhaar/PAN identity verification) that I'd tell them to double down on. See [Strengths](#10--what-id-genuinely-keep-strengths).

---

## 2 · How I tested (and what I couldn't)

**Submitted by:** Rohan ([@rohanbalu05](https://github.com/rohanbalu05)) · Co-founder, **AIz Yantra**.

**Surfaces I exercised:**

| Surface | What I did |
|---|---|
| `vocallabs.ai` website | Walked the full funnel: Home → Solutions → Docs → "Get Started" → Contact. |
| Contact / demo flow | Confirmed the *only* way in is a human demo request. |
| Android app ("Vocallabs") | Installed, attempted a real call, captured the failure. |
| Chrome extension | Tried to install from the official Chrome Web Store. |
| API docs (`docs.vocallabs.ai`) | Read the full 65-page reference end-to-end. |
| Public GitHub org (`github.com/Vocallabsai`) | Read the SDK, the n8n node + its bundled API docs, the `vocalflow` app, and the extension repo. |

**What I could *not* access — and why that is itself a finding:** I could not test a live AI agent call. There is **no self-serve way to get API credentials** (they are provisioned only after a sales demo), and a demo could not be arranged inside the assignment window. So the deepest technical claims (audio quality, barge-in, "emotion/tone" analytics, the hybrid handoff) are assessed from **their own code and API**, and flagged as *evidence-based but not live-verified* where relevant. I treat that gap honestly in [section 7](#7--the-one-thing-i-couldnt-test--and-how-id-evaluate-it-on-day-1). The access wall itself is **Finding F1**.

> **A note on method:** Because the depth here comes partly from reading Vocallabs' public code, every technical claim is tied to a specific file, endpoint, or screenshot. Nothing in this teardown requires you to take my word for it.

---

## 3 · Plain-English glossary

This teardown is written so a junior engineer (or a non-engineer) can follow every point. Quick definitions for the terms used below:

- **Self-serve / PLG (product-led growth):** You can sign up and start using the product yourself, instantly, without talking to a salesperson. The opposite is "sales-gated."
- **ASR (speech-to-text):** Software that turns spoken audio into text. (Vocallabs appears to use **Deepgram** for this.)
- **TTS (text-to-speech):** Software that turns text into a spoken voice.
- **LLM:** The "brain" (e.g. GPT, Claude, Llama) that decides what the agent says next.
- **Orchestration layer:** A product that doesn't build its own ASR/LLM/TTS, but *wires together* other companies' models and adds workflow on top.
- **Half-duplex vs full-duplex audio:** Half-duplex = only one side can "talk" at a time (like a walkie-talkie). Full-duplex = both sides can talk and listen at once (like a real phone call). Full-duplex is what makes an agent feel human and lets you interrupt it ("barge-in").
- **Hasura:** An off-the-shelf tool that auto-generates a GraphQL API over a database. Seeing it means parts of the backend are built on Hasura.
- **SIP / DID:** Telephony plumbing. SIP is the protocol for internet phone calls; a DID is a phone number you can rent.
- **DLT (here):** *Distributed Ledger Technology* — India's TRAI mandates registering commercial senders/templates on a blockchain-based ledger before they can call/message.
- **TCCCPR / DND / DPDP:** India's rulebooks for commercial calls (TCCCPR), the Do-Not-Disturb registry (DND), and personal-data protection (DPDP Act 2023 + Rules 2025).
- **Sample rate (8 kHz / 16 kHz):** How much audio detail is captured. 8 kHz is old-school phone quality; 16 kHz+ sounds clearer.

---

## 4 · What Vocallabs actually is — the architecture

The marketing says "full-stack ownership." The evidence (their API docs + their open-source app) points to a different, more honest picture: **client apps → Subspace's backend → third-party AI models.**

```mermaid
flowchart TD
    subgraph CLIENT["1 — Client surfaces"]
        A1["Website + demo form"]
        A2["React Native SDK"]
        A3["n8n community node"]
        A4["Android app"]
        A5["Chrome extension — delisted"]
    end
    subgraph BACKEND["2 — api1.superflow.run/b2b — Subspace backend"]
        B1["Hasura GraphQL + Action webhooks"]
        B2["Wallet / green-balance billing"]
    end
    subgraph MODELS["3 — Third-party models, orchestrated"]
        C1["ASR — Deepgram"]
        C2["LLM — chosen via getAIModels"]
        C3["TTS — voice_id catalogue"]
    end
    A1 --> BACKEND
    A2 --> BACKEND
    A3 --> BACKEND
    A4 --> BACKEND
    BACKEND --> MODELS
```

**How I know each box (evidence):**

- **`api1.superflow.run`, Hasura, wallet:** Every endpoint in the official docs is served from `api1.superflow.run/b2b/...` — *not* `vocallabs.ai`. The docs include `getGreenBalance` and `whatsubTransactionHistory` (a wallet ledger). "WhatSub" was Subspace's original product name; "Superflow" is Subspace's backend. The Android app's crash exposes a **Hasura GraphQL** error. A `402 — insufficient balance` error on billable endpoints confirms the **pay-per-call wallet**.
- **Deepgram + swappable LLM + TTS catalogue:** The API exposes `getAIModels` (you pick the LLM), `voice_id` and `getVoicesByLanguageComment` (a TTS voice catalogue), and per-agent `analytics_prompt` (analytics is *a prompt over a transcript*). Vocallabs' own open-source `vocalflow` app documents its stack as **Deepgram (ASR) + Groq/OpenRouter (LLM)** — strong evidence for the same orchestration approach in the core product.

This architecture isn't bad — orchestration is a legitimate, fast way to build. The problem is only that it **contradicts the "proprietary / full-stack" story** the company tells. See the Moat Audit next.

---

## 5 · The Moat Audit — testing their 4 claimed moats

The assignment lists Vocallabs' four claimed moats. Here is each one, tested against evidence. This is the spine of the whole teardown.

| # | Claimed moat (their words) | Verdict | Why | Finding |
|---|---|---|---|---|
| 1 | **Full-stack ownership** + **data flywheel: proprietary conversation intelligence** | ⚠️ **Overstated** | The stack orchestrates Deepgram + swappable LLMs; "analytics beyond transcription" is an LLM prompt over a transcript. Real data is captured, but the *intelligence* is third-party. | [F2](#f2--proprietary-conversation-intelligence-is-mostly-orchestration-over-third-party-models-) |
| 2 | **India-first: tuned for local languages, accents & workflows** | 🟡 **Mixed** | One part is a **genuine moat** — in-call **Aadhaar/PAN KYC** competitors don't have. The "tuned languages" part looks like prompt-based transliteration on third-party ASR, not proprietary India-tuned models. | [F2](#f2--proprietary-conversation-intelligence-is-mostly-orchestration-over-third-party-models-) / [Strengths](#10--what-id-genuinely-keep-strengths) |
| 3 | **Founder conviction (post-Subspace)** | ✅ **Confirmed — but it is a dependency too** | The product literally runs on Subspace's `superflow.run` + wallet rails. That is real leverage (speed, lower cost) **and** a structural coupling/single-point-of-failure risk. | [F2](#f2--proprietary-conversation-intelligence-is-mostly-orchestration-over-third-party-models-) |
| 4 | **Data flywheel at scale** | 🟡 **Partial** | They do capture every conversation (a real asset), but nothing observed turns that into a defensible, compounding model advantage yet. | [F2](#f2--proprietary-conversation-intelligence-is-mostly-orchestration-over-third-party-models-) / [F10](#f10--validate-or-puncture-the-emotiontone-analytics-claim-on-a-live-call-reserved) |

**Bottom line:** three of the four moats are weaker than advertised, and the *strongest real moat they have (India KYC) isn't even in their moat list.* That's a strategic mispositioning, not just a feature gap. The founder-facing version of this argument is in [section 8](#8--founders-lens--deliberate-debt-the-real-moat-and-my-first-week).

---

## 6 · The 10 findings

Each finding follows the assignment's structure — **(a) Observed → (b) Problem → (c) Ship instead** — plus a severity and effort rating so the team can prioritise.

**Severity:** 🔴 Critical · 🟠 High · 🟡 Medium · ⚪ Low  **Effort:** how much work the fix is (Low / Med / High).

| # | Finding | Pillar | Severity | Effort |
|---|---|---|---|---|
| F1 | No self-serve signup, yet positioned "for developers" | GTM & ICPs · UX | 🟠 High | Med |
| F2 | "Proprietary intelligence" is mostly third-party orchestration | Competitor Analysis | 🟠 High | High |
| F3 | Polished API docs, but hollow (no response schemas, bugs) | Features · Dev UX | 🟠 High | Med |
| F4 | Outbound AI calling with no visible India compliance (TCCCPR/DLT/DPDP) | GTM · Risk · Features | 🟠 High | High |
| F5 | The public GitHub org sends an incoherent brand signal | GTM · Brand | ⚪ Low | Low |
| F6 | Chrome extension is delisted & only on ad-laden mirror sites | Features · UX · Trust | 🟡 Medium | Low |
| F7 | A developer test-app is shipped to the public Play Store | GTM · UX | 🟡 Medium | Low |
| F8 | Core call fails; raw backend error shown to the user | Features · Reliability | 🔴 Critical | Med |
| F9 | "Booking" agents have no scheduler; WhatsApp is notify-only | Potential Collaborations | 🟡 Medium | High |
| F10 | "Emotion/tone" analytics needs live validation | Competitor · Features | 🟡 Medium | — |

---

### F1 · No self-serve signup, yet positioned "for developers"
**Pillar:** GTM & ICPs · UX  **Severity:** 🟠 High  **Effort:** Medium

**(a) Observed.** The homepage positions Vocallabs as *"Voice AI infrastructure for developers… that scales from zero to millions of concurrent calls,"* and the top CTA reads **"Get Started ⚡ 2 mins."** But every CTA — "Get Started," "Talk to an Expert," "Schedule Your Free Demo" — leads to the same **sales/demo form**. There is no signup, no free tier, no instant API key anywhere on the site, in the docs, or in the repos. Meanwhile the docs and GitHub clearly show a developer console exists (it issues `clientId`/`clientSecret`).

<p>
  <img src="evidence/screenshots/F1-homepage-developer-infra.png" width="48%" alt="Vocallabs homepage: 'Voice AI infrastructure for developers... scales from zero to millions of concurrent calls'">
  <img src="evidence/screenshots/F1-contact-demo-only.png" width="48%" alt="Vocallabs contact page: the only entry point is 'Schedule Your Free Demo'">
</p>

> 🔍 **What to look at:** the homepage promises developer self-serve and "2 mins"; the only actual door (right) is a sales demo form.

**(b) Problem.** Developers — the *stated* ideal customer — self-serve. They evaluate by getting a key and making a test call in minutes. A sales wall makes the high-intent developer bounce immediately, and the "2 mins / from zero" copy sets a promise the funnel then breaks (worst of both worlds: wrong promise *and* wrong motion). Direct competitors (Vapi, Retell, Bland, Synthflow) all hand you a free API key in ~60 seconds, so the comparison shopper is gone before a salesperson ever calls back. **In plain terms:** the website invites developers in, then locks the only door.

**(c) Ship instead.** Pick one identity and align the funnel to it:
- *If the real ICP is developers:* expose the existing console behind "Get Started" with a **free sandbox tier + instant API key** (even rate-limited). This is the single highest-leverage change for the dev ICP.
- *If the real motion is enterprise sales:* drop "for developers / 2 mins / from zero," and reposition honestly as **managed enterprise voice**, so the message matches the funnel.

---

### F2 · "Proprietary conversation intelligence" is mostly orchestration over third-party models ⭐
**Pillar:** Competitor Analysis / Moats  **Severity:** 🟠 High (strategic)  **Effort:** High

> **Read this as a gift, not a gotcha.** The point isn't "your tech is weak" — it's that you're *marketing the wrong moat*. Your real, defensible moat (India telephony + KYC + payments) is sitting unused while you claim one that diligence can puncture. Fixing the *story* is nearly free; the section below shows how.

**(a) Observed.** Three independent pieces of evidence converge:
1. **It runs on Subspace's backend.** The official docs serve every endpoint from `api1.superflow.run/b2b/...`, and include `getGreenBalance` + `whatsubTransactionHistory` — Subspace's ("WhatSub") wallet. A `402 — insufficient balance` error gates billable calls.
2. **It orchestrates swappable models.** The API exposes `getAIModels` (choose the LLM), a `voice_id` TTS catalogue, and per-agent `analytics_prompt` (so "analytics" = an LLM prompt run over the transcript).
3. **Their own app reveals the stack.** Vocallabs' open-source `vocalflow` app documents **Deepgram (ASR) + Groq/OpenRouter (LLM)**.

<p>
  <img src="evidence/screenshots/F2-docs-superflow-base-url.jpg" width="70%" alt="Vocallabs API docs showing the base URL api1.superflow.run/b2b and the getGreenBalance wallet endpoint">
</p>

> 🔍 **What to look at:** the endpoint is `api1.superflow.run/b2b/createAuthToken` — Subspace's backend — and just below it, `getGreenBalance` (a wallet). This is not a `vocallabs.ai` API; it's a voice product layered on Subspace's rails.

**(b) Problem.** The brand sells *"full-stack ownership"* and *"proprietary conversation intelligence,"* and the site claims *"emotion, intent & tone beyond transcription."* The evidence says this is **prompts over Deepgram transcripts on third-party LLMs** — a capability a competent team can rebuild in weeks. The moat narrative is therefore fragile to anyone (a technical buyer, an investor's diligence) who looks one layer down. The Subspace coupling is simultaneously a strength (speed, shared cost) and a **single-point-of-failure / strategic-dependency risk**. There's also a quiet **margin** question: a pure orchestration layer pays Deepgram + LLM + telephony per call, and that input cost is commoditising — so "proprietary" also has to mean *defensible economics*, not just messaging.

**(c) Ship instead.** Two honest paths:
- *Make the moat real and visible:* ship one thing competitors can't trivially copy — e.g. **India-accent-tuned ASR**, **acoustic emotion detection from the waveform** (not the transcript), and publish a benchmark.
- *Reposition around the true moat:* orchestration *speed* + **India telephony + Aadhaar/PAN KYC + (proposed) payments + compliance** + the Subspace data/billing rails. Stop claiming proprietary intelligence that can't be demonstrated.

> **Honesty note:** I could not run a live call, so the model-stack claim is a strong, evidence-backed inference (from `getAIModels` + the `vocalflow` stack), not a lab-proven fact. F10 is the live test that would convert it to proof.

---

### F3 · Polished API docs, but hollow underneath
**Pillar:** Features / Developer UX  **Severity:** 🟠 High  **Effort:** Medium

**(a) Observed.** The docs *portal* looks great — clean tables, multi-language code samples, a "Try it Out" runner. But the substance is thin and buggy:
- **Response bodies are undocumented.** Every endpoint lists its response as the same generic `data: object · "endpoint-specific payload."` You can authenticate but never learn what fields actually come back.
- **Errors are copy-pasted boilerplate** (the same 401/400/402/404 block on every endpoint).
- **Method bugs:** "Add multiple contacts to group" is labelled **POST** but its code sample is `curl -X GET`, with a description about creating a *single* contact.
- **Insomnia artifacts leak through:** examples still carry `User-Agent: insomnia/12.0.0` — i.e. the docs are exported from the Insomnia REST client, not authored.

<p>
  <img src="evidence/screenshots/F3-docs-402-wallet-generic-response.jpg" width="32%" alt="Docs: generic 'endpoint-specific payload' response and a 402 insufficient balance error">
  <img src="evidence/screenshots/F3-docs-get-post-mismatch.jpg" width="32%" alt="Docs: endpoint labelled POST but the curl example uses GET">
  <img src="evidence/screenshots/F3-docs-insomnia-useragent.jpg" width="32%" alt="Docs: Bridge Call example still carrying User-Agent insomnia/12.0.0">
</p>

> 🔍 **What to look at:** (left) the response is a generic `object`; (middle) a POST endpoint with a GET example; (right) `User-Agent: insomnia/12.0.0` left in the public docs.

**(b) Problem.** For a company whose pitch is *"infrastructure for developers,"* **the docs *are* the product.** A developer here can log in but can't predict a single response shape, can't trust the HTTP verbs, and copies broken examples. Polished-but-hollow is arguably worse than sparse-but-honest, because it *looks* finished, so the developer wastes time discovering it isn't. This compounds F1: even if you got a key, integration is trial-and-error.

**(c) Ship instead.** Generate the docs from a real **OpenAPI spec** with typed response schemas per endpoint, real per-endpoint errors, human-written one-line descriptions, correct verbs, and strip the Insomnia `User-Agent`. Add a **"Get your API keys" quickstart** at the top (which also fixes F1).

---

### F4 · Outbound AI calling with no visible India compliance (TCCCPR / DLT / DPDP)
**Pillar:** GTM · Regulatory Risk · Features  **Severity:** 🟠 High  **Effort:** High

> **Why this is the finding most applicants won't have:** it requires stepping out of the app and asking *"is this even legal to operate at scale in India?"* For a company whose target verticals are telecom, finance, and real estate, the answer is the difference between a demo and a deployable product.

**(a) Observed.** Vocallabs auto-dials business calls (sales, support, booking) to Indian phone numbers — which makes it, in regulatory terms, a sender of **commercial communication via auto-diallers/robocalls**. Two live Indian regimes apply directly:
- **TRAI's TCCCPR 2018**, tightened by the **Second Amendment of 12 Feb 2025**: commercial senders/templates must be registered on **DLT**; **auto-diallers and robocalls must be notified to the origin access provider in advance**; promotional vs transactional calls must use the correct caller-ID series (**140** promotional, **1600** transactional/service); **explicit consent** for a commercial transaction is valid only ~7 days, inferred consent only for the contractual relationship; using ordinary **10-digit numbers for telemarketing is treated as spam**, with operators able to disconnect/blacklist offending senders for up to 1–2 years. TRAI has also signalled it is **actively considering AI-specific telemarketing rules** (AI disclosure + consent verification). [(TRAI Second Amendment, 2025)](https://www.trai.gov.in/sites/default/files/2025-02/Regulation_12022025.pdf)
- **The DPDP Act 2023 + DPDP Rules 2025** (notified 13 Nov 2025; consent/notice/retention obligations enforceable by 13 May 2027): a consent-based regime for personal data, with notice, data-principal rights, retention limits and breach reporting. Voice agents **record calls** and process personal data (and, via the KYC feature, **Aadhaar/PAN**) — so Vocallabs and its customers are data fiduciaries with consent and call-recording-consent obligations. [(DPDP Rules 2025, PIB)](https://static.pib.gov.in/WriteReadData/specificdocs/documents/2025/nov/doc20251117695301.pdf)

In the entire public API and docs I found **no visible support** for any of this — no consent capture, no DLT-aware number provisioning, no 140/1600 routing, no DND scrubbing before dial-out, and no AI-disclosure opener.

**(b) Problem.** This is not paperwork — it's an **adoption blocker for Vocallabs' own target verticals**. A bank, telco, or real-estate enterprise will not deploy an outbound voice agent that can get its numbers blacklisted (taking down its OTPs and support traffic with them) or that can't evidence consent under DPDP. And because TRAI is moving toward AI-calling rules specifically, an AI-voice company that *isn't* ahead of this is exposed precisely where it should be strongest. **In plain terms:** the product can place calls, but not provably *lawful, consented* calls at scale — which is the only kind an Indian enterprise can buy.

**(c) Ship instead.** Make **compliance a first-class product feature**, not the customer's problem: (1) DLT-aware number provisioning with correct **140/1600** routing; (2) **DND + consent scrubbing** before any dial-out; (3) mandatory **consent capture + call-recording consent**, logged immutably; (4) a built-in **AI-disclosure** line at call open; (5) DPDP **retention/erasure** controls and a compliance dashboard. This turns a liability into a *wedge*: “compliant-by-default India voice” pairs perfectly with the KYC moat (see F2 / Strengths) and is something the US incumbents can’t easily match.

> **Not legal advice.** I am not a lawyer; specifics above are current as of early 2026 and TRAI/DPDP rules are actively evolving — the exposure is durable, but exact obligations should be confirmed with counsel. Sources linked are the TRAI gazette notification and the Government of India PIB release.

---

### F5 · The public GitHub org sends an incoherent brand signal
**Pillar:** GTM / Brand  **Severity:** ⚪ Low  **Effort:** Low

**(a) Observed.** The most polished, most SEO-stuffed repo in `github.com/Vocallabsai` is **`vocalflow` — a free macOS *dictation* app** (a Wispr Flow clone), unrelated to the voice-agent business. Its README is keyword soup ("FREE FREE FREE", competitor names). Meanwhile the core SDK has no self-serve path, build binaries (`.dmg`/`.pkg`) are committed straight into git, and a stray `oplo_square.png` hints at a rebrand leftover.

**(b) Problem.** A developer who lands on the org to evaluate "Vocallabs the voice infra company" gets a confusing signal about what the company even *is*, and sees the most love went to a non-core side project. It quietly undercuts the "serious infrastructure" positioning. *(Caveat: a side project as an SEO/recruiting magnet can be deliberate — see the [Founder's Lens](#8--founders-lens--deliberate-debt-the-real-moat-and-my-first-week) on debt-vs-blind-spot.)*

**(c) Ship instead.** Move `vocalflow` to its own org (or clearly label it a Deepgram-powered demo of their audio chops), **pin the core repos** with one-line positioning, and move binaries to GitHub Releases instead of committing them.

---

### F6 · The Chrome extension is delisted and only reachable via ad-laden mirror sites
**Pillar:** Features · UX · Trust  **Severity:** 🟡 Medium  **Effort:** Low

**(a) Observed.** Vocallabs advertises a "Chrome extension for workflow automation." Its official Chrome Web Store listing now returns **"This item is not available"** (delisted or pulled). The only places it still surfaces are third-party download sites like **Softonic**, wrapped in Avast/Opera/CCleaner ads, described there as a tool to *"obtain direct contact numbers for business professionals."*

<p>
  <img src="evidence/screenshots/F6-chrome-store-unavailable.png" width="48%" alt="Chrome Web Store: 'This item is not available' for the Vocallabs extension">
  <img src="evidence/screenshots/F6-softonic-ads.png" width="48%" alt="Softonic mirror page for the Vocallabs Chrome extension, surrounded by ads">
</p>

> 🔍 **What to look at:** (left) the official store listing is dead; (right) the only live copy is an ad-wrapped mirror — an uncontrolled, repackaged `.crx`.

**(b) Problem.** Three compounding issues: (1) a **publicly advertised capability is dead** through legitimate channels — a broken promise that signals neglect; (2) anyone hunting for it lands on **mirror sites serving an uncontrolled extension file**, a real security and brand risk (a trojaned clone could ship under "Vocallabs"); (3) the described purpose — scraping personal mobile numbers for outreach — is **off-strategy** for a voice-agent platform and privacy-sensitive under DPDP. *(Privacy point flagged as a risk, not legal advice.)*

**(c) Ship instead.** Decide its fate explicitly. If deprecated → remove all references from the site/brief and redirect them. If strategic → **relist on the Chrome Web Store** (own the distribution channel) and **re-scope it on-strategy** (e.g. "launch/monitor an AI voice agent from your CRM tab") rather than contact scraping.

> ⚠️ Do **not** install the mirror-site build — third-party extension repackages are a known malware vector and it is not Vocallabs' controlled artifact.

---

### F7 · A bare-bones developer test-app is shipped to the public Play Store as a "product"
**Pillar:** GTM · UX · Positioning  **Severity:** 🟡 Medium  **Effort:** Low

**(a) Observed.** The public "Vocallabs" Android app contains exactly: a phone-number field, an 8/16 kHz toggle, a "Start Call" button, a mute control, and a Disconnected/Inactive status row — with a vague Play Store description. This is an **internal SDK test client**, not a consumer product.

<p>
  <img src="evidence/screenshots/F7-mobile-call-setup.jpeg" width="38%" alt="Vocallabs Android app: a minimal 'Call Setup' screen with a phone field and 8/16 kHz toggle">
</p>

> 🔍 **What to look at:** no agent selection, no onboarding, no explanation — just a raw call tester, published as if it were the product. (Note: it *does* offer a 16 kHz option, so audio isn't locked to 8 kHz.)

**(b) Problem.** It's the same incoherence as F5/F6: a public surface that contradicts the company story and confuses positioning (is Vocallabs a consumer calling app, a dev SDK, or B2B infra?). A prospect or investor's analyst who downloads it gets something that looks purposeless — a poor, trust-eroding first impression.

**(c) Ship instead.** Unpublish it, or relabel it explicitly (e.g. "Vocallabs SDK Demo") and gate it behind the docs. A *public* app should demonstrate the real product — pick an agent, hear a real AI call — not expose a raw call tester.

---

### F8 · The core action (place a call) fails, leaking a raw backend error to the user
**Pillar:** Features · Reliability · UX (+ security)  **Severity:** 🔴 Critical  **Effort:** Medium

**(a) Observed.** Tapping **"Start Call"** in the public Android app returns a modal reading: *"Call Failed — parsing Hasura.GraphQL.Execute.Action.Types.ActionWebhookErrorResponse failed, key 'message' not found."* The one thing the app exists to do does not work, and the failure dumps an internal stack error to the end user.

<p>
  <img src="evidence/screenshots/F8-mobile-call-failed.jpeg" width="38%" alt="Vocallabs app modal: 'Call Failed' showing a raw Hasura GraphQL Action webhook error">
</p>

> 🔍 **What to look at:** a normal user is shown a **Hasura GraphQL** parse error. Users should never see this.

**(b) Problem.** Read the error closely — it's more diagnostic than it first appears. Three layers, top to bottom:
1. **Reliability** — the headline capability fails on the primary public surface, awkward for a company claiming to "scale to millions of concurrent calls."
2. **A specific internal contract bug** — `ActionWebhookErrorResponse … key 'message' not found` means a **Hasura Action webhook** (their own backend logic) returned an error object that didn't match the shape Hasura expects. In other words, Vocallabs' own client ↔ their own API integration is broken on the happy path — a sign the end-to-end flow isn't covered by tests.
3. **Information disclosure** — leaking `Hasura.GraphQL.Execute.Action.Types…` tells an attacker the backend is Hasura + Action webhooks (useful for crafting attacks) and confirms, for this teardown, the **Subspace/Hasura backend behind F2**. A user should see a friendly message; an attacker shouldn't see the stack.

**(c) Ship instead.** Wrap all backend calls in a user-facing error layer ("We couldn't connect your call — please try again"), log the raw error **server-side only**, add an end-to-end test on the call-initiation path, and fix the failing Action webhook's error contract (return the `message` key Hasura expects).

---

### F9 · "Booking" agents have no scheduler; WhatsApp is notify-only; KYC has no payment rail
**Pillar:** Potential Collaborations  **Severity:** 🟡 Medium  **Effort:** High

**(a) Observed.** Across the entire API (agents, actions, library, campaigns) there is **no calendar/scheduling integration** (no Google Calendar / Cal.com / Calendly) — despite the brand selling "booking" agents. WhatsApp appears only as `updateWhatsappNotification` — a one-way **notification toggle**, not a conversational channel. They ship in-call **Aadhaar/PAN KYC** (`getIdentityUrl`) but **no payment rail** to pair with it.

**(b) Problem.** The three highest-value Indian voice workflows — *book an appointment*, *confirm/collect over WhatsApp*, *take a payment after verifying identity* — each dead-end at an integration that doesn't exist. Agents can talk, but can't **close the loop**. These are exactly the partnerships that turn a demo into ROI.

**(c) Ship instead.** Three concrete partnerships: (1) **Cal.com / Google Calendar** as a first-class agent Action, so "booking" actually books; (2) **WhatsApp Business API** as a conversational channel (handoff + confirmations), not just notifications; (3) **Razorpay / UPI** in-call payment links, leveraging the existing KYC step. The Subspace (fintech) parentage makes the payments integration especially natural.

---

### F10 · Validate (or puncture) the "emotion/tone" analytics claim on a live call *(reserved)*
**Pillar:** Competitor · Features  **Severity:** 🟡 Medium  **Effort:** —

**(a) Observed (from code).** Analytics is configured via a per-agent `analytics_prompt` and prompt-based `updatePostCallData` (you write a prompt to extract each field) on a Deepgram + LLM stack. That points to **"emotion, intent & tone beyond transcription"** being an **LLM prompt over a transcript**, not acoustic analysis of the audio.

**(b) Problem.** If true, the claim oversells what is essentially transcript summarisation — the same fragility as F2.

**(c) Ship instead / next step.** This needs **a single live call to confirm** — see [section 7](#7--the-one-thing-i-couldnt-test--and-how-id-evaluate-it-on-day-1) for exactly how I'd test it. If confirmed, it strengthens F2; if calls stay broken, **F8 stands on its own** and the analytics claim is cited as evidenced-but-unverified.

---

## 7 · The one thing I couldn't test — and how I'd evaluate it on Day 1

In the interest of intellectual honesty: **I never got to assess an actual agent conversation** — the thing Vocallabs actually sells — because there's no self-serve access (F1) and the app call failed (F8). A teardown that pretended otherwise would be dishonest. So here is exactly what I'd test on day one with a provisioned account, and what evidence I'd look for. *(This doubles as my answer to "how would you operate this from the inside?")*

| Unverified claim | How I'd test it on a live call | What would confirm / refute it |
|---|---|---|
| **Conversation quality** ("human-like, unscripted") | Run 10 calls across sales/support/booking scripts; measure response latency, naturalness, and recovery from off-script input. | Latency under ~1s and graceful handling of interruptions = real; long pauses / rigid flows = IVR-like. |
| **Barge-in / full-duplex** | Deliberately talk over the agent mid-sentence. *(The SDK exposes `halfDuplexRms/Peak` + an 8 kHz default — which **suggests** a half-duplex pipeline that would struggle here, but this is a hypothesis from code, not a proven defect.)* | Agent stops and listens instantly = full-duplex; agent talks over you / ignores you = half-duplex limitation confirmed. |
| **"Emotion/tone beyond transcription" (F10)** | Say the same words in a happy vs angry tone; compare the analytics output. | Different emotion read from *tone alone* = acoustic analysis (real moat); identical read = it's just an LLM reading the transcript. |
| **Hybrid AI ↔ live-agent handoff** (a headline feature in their brief) | Trigger an escalation and watch the transfer to a human. | A clean warm-transfer with context = real; **I found no clear API evidence for this capability**, so it may be unbuilt — worth verifying first. |
| **Reliability at volume** | Fire concurrent calls and watch failure rates (given F8 failed at N=1). | Stable concurrency = the "millions of calls" claim holds; failures = it doesn't yet. |

The honest headline: **the most important evaluation — of the core product — is the one I was structurally prevented from doing.** That itself is a finding (it's F1's consequence), and I'd make running these five tests my literal first task on the job.

---

## 8 · Founder's Lens — deliberate debt, the real moat, and my first week

A teardown is easy; *owning* the fixes is the job. Three reframes that matter if I were building this from the inside.

### 8.1 · Which of these are deliberate startup debt vs a blind spot?

A fast-moving, early-stage India team **knows** its docs are auto-generated and its app is a test harness — calling those "discoveries" would be naive. The owner's question isn't "is there debt?" (there always is) but **"which debt now costs real revenue and must graduate to *fix-this-quarter*?"**

| Rough edge | Likely *deliberate* debt? | Does it now cost real money? | Verdict |
|---|---|---|---|
| Auto-generated docs (F3) | Yes — shipped fast | **Yes**, if the ICP is developers — blocks integration | **Graduate to fix** |
| Test-harness app on store (F7) | Maybe — internal tool leaked out | Low, but a trust ding | Cheap unpublish/relabel |
| `vocalflow` side project (F5) | Yes — plausible SEO/recruiting play | Low | Leave, or relabel as a demo |
| No self-serve (F1) | Maybe — sales-led *by design* | **Yes**, if the ICP is developers | Decide the ICP first, then fix |
| Broken app call (F8) | **No** — a bug, not a choice | **Yes** — visible breakage on the core action | **Fix now** |
| Orchestration vs "proprietary" (F2) | Yes — fastest path to market | **Risk** to fundraising + positioning | Re-message now, build the real moat over time |
| No India compliance (F4) | **No** — this is *risk*, not debt | **Yes** — legal + enterprise-sales blocker | Address before scaling outbound |

The pattern: **F8 and F4 aren't "debt" at all — one is a bug, one is risk — so they jump the queue. F1/F2/F3 are debt that has started charging interest.** The rest is cheap hygiene.

### 8.2 · The real moat (the gift in F2)

Vocallabs is marketing the moat it *wishes* it had ("proprietary intelligence") and under-marketing the one it *actually* has. Its defensible wedge is the **India operations layer for voice**: **telephony + Aadhaar/PAN KYC + (proposed) UPI payments + compliance + workflow.** No global incumbent (Vapi/Retell/Bland) has in-call KYC; no pure model-player (e.g. Sarvam) has the telephony+payments+workflow bundle. The move isn't to out-feature Vapi — it's to own *"verify → talk → collect → confirm,"* compliant-by-default, for regulated Indian verticals. That single repositioning makes F2 and F4 strengths instead of weaknesses.

### 8.3 · If I owned this on Day 1 — my first sprint

Concrete, ordered, and tied to the findings:

1. **Instrument + fix the front door.** Add funnel analytics, then fix the **broken call (F8)** and ship a gated **self-serve sandbox + instant key spike (F1)** — stop losing high-intent developers today.
2. **Run the 5 core-product tests** from [section 7](#7--the-one-thing-i-couldnt-test--and-how-id-evaluate-it-on-day-1) so we *know* where conversation quality, barge-in, and analytics actually stand (not where marketing says).
3. **Draft the "compliant-by-default" spec (F4)** — DLT/140-1600 routing, consent + recording consent, DPDP retention — because it's both a legal gate and the wedge.
4. **Rewrite the homepage moat message (F2)** to lead with India ops + KYC, and **back-fill response schemas in the docs (F3)**.
5. **Cheap hygiene:** unpublish/relabel the app (F7), relist or kill the extension (F6), tidy the GitHub org (F5).

---

## 9 · Prioritisation — what to fix first, and why

Ordered by **impact on trust/revenue ÷ effort**. The logic: fix the things that make a serious evaluator walk away *today* (and the risks that can halt the business) before the slower strategic bets.

| Priority | Findings | Why this bucket first |
|---|---|---|
| **P0 — Stop the bleeding** | **F8** (broken call), **F1** (no door), **F3** (unusable docs) | These three kill the evaluation *in the first session*. A failed call, a sales wall, and undocumented responses each independently lose a high-intent developer. All are concrete and shippable. |
| **P1 — Coherence, trust & compliance** | **F2** (re-message the moat), **F4** (compliance gate), **F6** (dead extension), **F7** (test-app on store) | The strategic re-message (F2) is near-free; the compliance work (F4) is a non-negotiable gate for India enterprise sales; F6/F7 are cheap credibility wins. Together they answer "what *is* this company, and can I legally deploy it?" |
| **P2 — Depth & roadmap** | **F9** (integrations), **F5** (org hygiene), **F10** (validate analytics) + the [Day-1 validations](#7--the-one-thing-i-couldnt-test--and-how-id-evaluate-it-on-day-1) | Higher-effort or research-dependent. These build the *durable* product and moat once the basics hold. |

**Why F8 is #1 not F2:** F2 is the most *intellectually* interesting finding, but a prospect never reaches a moat debate if the app crashes on "Start Call." Reliability buys you the right to have the strategy conversation. **First impressions (P0) gate everything downstream.**

---

## 10 · What I'd genuinely keep (strengths)

A fair teardown names what's working — and Vocallabs has real assets:

- **A broad, mature B2B API.** Agents, campaigns, contact groups, a DID number marketplace, conferencing (bridge two numbers), and **custom Actions** = mid-call function-calling via `external_curl` with explicit success/failure/**interruption** responses. This is a genuinely capable platform surface.
- **RAG built in.** `insertAgentDocx` with `webcrawler_depth` lets an agent learn from crawled sites/documents.
- **Operational depth.** Keyword/pronunciation replacement, per-agent success metrics, and post-call structured-data extraction.
- **The real moat (and they're underselling it):** **in-call Aadhaar/PAN identity verification** (`getIdentityUrl`). US-centric competitors (Vapi, Retell, Bland) have nothing like it. *This* — India telephony + KYC + (proposed) payments + compliance — is the moat I'd put on the homepage, instead of "proprietary intelligence."

### Quick competitor positioning (as of early 2026, structural — not a price sheet)

| Capability | Vocallabs | Vapi | Retell | Bland | Synthflow | Sarvam (IN) |
|---|---|---|---|---|---|---|
| Self-serve signup + instant API key | ❌ | ✅ | ✅ | ✅ | ✅ | ✅ |
| Developer-first docs | 🟡 (hollow) | ✅ | ✅ | ✅ | ✅ | ✅ |
| Transparent public pricing | ❌ | ✅ | ✅ | ✅ | ✅ | ✅ |
| India languages / accents focus | ✅ (claimed) | 🟡 | 🟡 | 🟡 | 🟡 | ✅✅ |
| In-call Aadhaar/PAN KYC | ✅ | ❌ | ❌ | ❌ | ❌ | ❌ |
| Owns core models vs orchestrates | Orchestrates | Orchestrates | Orchestrates | Mixed | Orchestrates | ✅ Owns (India LLM/ASR) |

> Read this as *positioning*, not a benchmark: Vocallabs' way to win is **not** "be Vapi for India on features" — it's to own **India telephony + KYC + payments + compliance + workflow**, where it already has a head start nobody else does.

---

## 11 · Frameworks & deep dives

The supporting analysis lives in this repo:

- [`frameworks/moat-audit.md`](frameworks/moat-audit.md) — the four-moats stress test, expanded.
- [`frameworks/swot.md`](frameworks/swot.md) — SWOT.
- [`frameworks/porters-five-forces.md`](frameworks/porters-five-forces.md) — Porter's Five Forces.
- [`frameworks/prioritization-matrix.md`](frameworks/prioritization-matrix.md) — impact × effort detail.
- [`research/01-architecture-and-stack.md`](research/01-architecture-and-stack.md) — how the Subspace/Hasura/Deepgram stack was reverse-engineered.
- [`research/02-api-surface-analysis.md`](research/02-api-surface-analysis.md) — the full annotated API surface.
- [`research/03-github-org-teardown.md`](research/03-github-org-teardown.md) — repo-by-repo findings.
- [`research/04-competitor-landscape.md`](research/04-competitor-landscape.md) — competitor positioning notes.
- [`evidence/`](evidence/) — every screenshot, mapped to its finding.

---

## 12 · Repository map

```
.
├── README.md                         ← this teardown (start here)
├── evidence/
│   ├── README.md                     ← index: which screenshot proves which finding
│   └── screenshots/                  ← F1…F8 evidence images
├── research/
│   ├── 01-architecture-and-stack.md
│   ├── 02-api-surface-analysis.md
│   ├── 03-github-org-teardown.md
│   └── 04-competitor-landscape.md
└── frameworks/
    ├── moat-audit.md
    ├── swot.md
    ├── porters-five-forces.md
    └── prioritization-matrix.md
```

---

<sub>Teardown by Rohan ([@rohanbalu05](https://github.com/rohanbalu05)) · Co-founder, AIz Yantra. All findings are based on publicly available surfaces (website, app, Chrome Web Store, public API docs, and Vocallabs' own open-source repositories) exercised on 30 May 2026. Technical claims that could not be verified on a live call are explicitly flagged. Regulatory references (TCCCPR, DPDP) are current as of early 2026 and are not legal advice.</sub>
