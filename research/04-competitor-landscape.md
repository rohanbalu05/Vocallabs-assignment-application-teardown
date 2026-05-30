# Deep dive 04 — Competitor landscape

> Supports the [Strengths/competitor section](../README.md#8--what-id-genuinely-keep-strengths). **Scope note:** this is a *positioning-level* comparison as of **early 2026**, built from durable structural facts (motion, focus) — not a live price sheet. Treat specifics as directional.

## The map

- **Global developer-first platforms — Vapi, Retell AI, Bland AI, Synthflow:** self-serve signup, instant API keys, transparent usage-based pricing, strong docs. Primarily US/global, English-first. All are **orchestration layers** too (they wire together ASR/LLM/TTS), so Vocallabs is *architecturally similar* — it just hides it while they embrace it.
- **India-native — Sarvam AI (and the broader Indian voice-AI wave):** differentiates by **owning India-language models** (ASR/LLM tuned for Indian languages/accents). This is the most direct threat to Vocallabs' "India-first" claim, because Sarvam's India advantage is at the *model* layer, while Vocallabs' (on the evidence) is prompt-based.

## Where Vocallabs actually wins

The honest competitive truth: Vocallabs should **not** try to out-feature Vapi for India. Its defensible wedge is the **India operations stack**:

- **In-call Aadhaar/PAN KYC** (`getIdentityUrl`) — no global competitor has this.
- **India telephony / DID marketplace** already built in.
- **Proposed:** UPI/Razorpay in-call payments (natural, given the Subspace fintech parentage) + WhatsApp as a conversational channel.

That bundle — *verify identity → talk → collect payment → confirm on WhatsApp*, all India-localised — is something neither the US incumbents nor a pure-model player like Sarvam offers end-to-end today.

## Positioning recommendation

| Today (fragile) | Recommended (defensible) |
|---|---|
| "Proprietary conversation intelligence" | "The India operations layer for voice: telephony + KYC + payments + workflow" |
| Competes on model quality (a losing race vs owners) | Competes on **regulated India workflows** (a race it leads) |
| Sales-gated, opaque pricing | Self-serve sandbox + transparent pricing to win developers (F1) |
