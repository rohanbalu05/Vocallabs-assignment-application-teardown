# Framework — Porter's Five Forces

> Applied to Vocallabs.ai in the India voice-AI-agent market. Each force notes the pressure level and the strategic implication.

## 1. Competitive rivalry — **HIGH**
Global developer-first platforms (Vapi, Retell, Bland, Synthflow) plus India-native players (Sarvam and a fast-growing local wave). Most compete on self-serve, pricing transparency, and docs — areas where Vocallabs currently trails (**F1, F3**).
*Implication:* differentiate on India workflow/KYC, not on being a better generic agent builder.

## 2. Threat of new entrants — **HIGH**
Because the core is **orchestration over third-party models** (**F2**), the technical barrier to entry is low — what Vocallabs built, a competent team can rebuild. The defensibility has to come from distribution and regulated-workflow depth, not the model.
*Implication:* build moats at the **integration + compliance** layer (KYC, payments, telephony), which are harder to copy than prompts.

## 3. Supplier power — **MEDIUM–HIGH**
Key inputs are external: **Deepgram** (ASR), **LLM providers** (via OpenRouter/Groq), **telephony/DID** suppliers, and — critically — the **Subspace backend** the whole product runs on. Each can squeeze margin or create dependency risk.
*Implication:* multi-vendor the model layer and progressively de-risk the Subspace single point of failure.

## 4. Buyer power — **MEDIUM–HIGH**
For SMBs, switching costs are low and alternatives are one signup away — so a broken first impression (**F8**) or a sales wall (**F1**) loses them instantly. Enterprises hold leverage by demanding reliability, compliance, and SLAs.
*Implication:* reliability + transparent self-serve directly reduce buyer churn at the top of the funnel.

## 5. Threat of substitutes — **MEDIUM**
Substitutes include in-house builds on OpenAI Realtime / open models, generic no-code agent tools, and traditional human BPO/call centres. The KYC + payments + India-telephony bundle is the hardest of these to substitute.
*Implication:* the more Vocallabs owns the *closed-loop India workflow*, the less substitutable it becomes.

## Net read
Forces are **structurally tough** for a generic orchestration play. Vocallabs' viable strategy is to retreat *up* the value chain into **regulated, India-localised, closed-loop workflows** (verify → talk → pay → confirm) where rivalry, entrants, and substitutes are all weaker.
