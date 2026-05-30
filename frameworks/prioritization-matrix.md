# Framework — Prioritisation matrix

> How the 10 findings were ranked, and the logic for the fix order. Method: rank by **impact on trust/revenue ÷ effort to fix** (and legal risk), then sequence so the things that lose an evaluator *today* — or can halt the business — are fixed before the slower strategic bets.

## Impact × effort table

| Finding | Impact | Effort | Priority | Rationale |
|---|---|---|---|---|
| **F8** Core call fails + raw error | High | Med | **P0** | The product visibly doesn't work on its main public surface. Nothing else matters until this is fixed. |
| **F1** No self-serve signup | High | Med | **P0** | Loses the entire stated (developer) ICP at the front door. |
| **F3** Hollow API docs | High | Med | **P0** | Even with a key, integration is trial-and-error; "docs are the product" for infra. |
| **F2** Orchestration vs "proprietary" | High | High | **P1** | Strategic re-messaging (low cost) + a longer build to make the moat real. |
| **F4** No India compliance (TCCCPR/DLT/DPDP) | High | High | **P1** | A non-negotiable legal/sales gate for India enterprise verticals; partly registration + process, partly product features. |
| **F6** Dead Chrome extension | Med | Low | **P1** | Cheap credibility + security/trust fix; a broken advertised capability. |
| **F7** Test-app on Play Store | Med | Low | **P1** | Cheap positioning fix; removes a confusing public artifact. |
| **F9** Missing integrations | Med | High | **P2** | High-value partnerships (scheduler, WhatsApp, payments) but larger build. |
| **F5** Incoherent GitHub org | Low | Low | **P2** | Quick hygiene; low individual impact. |
| **F10** Validate analytics claim | Med | — | **P2** | Research/validation step, not a code fix (see README §7, Day-1 validations). |

> Note: the audio **barge-in / full-duplex** question (previously tracked as a standalone finding) is a *code-derived hypothesis*, not a confirmed defect — so it now lives in the README's ["one thing I couldn't test"](../README.md#7--the-one-thing-i-couldnt-test--and-how-id-evaluate-it-on-day-1) Day-1 validation plan rather than as a numbered finding. F4 (India compliance) takes its place in the top 10 because it is evidenced and consequential.

## Fix order (sequencing logic)

```mermaid
flowchart LR
    P0["P0 — Stop the bleeding\nF8 broken call\nF1 no signup\nF3 hollow docs"] --> P1["P1 — Coherence, trust & compliance\nF2 re-message\nF4 compliance gate\nF6 dead extension\nF7 test-app"]
    P1 --> P2["P2 — Depth & roadmap\nF9 integrations\nF5 org hygiene\nF10 validate + Day-1 tests"]
```

**Why F8 outranks F2:** F2 is the most intellectually interesting finding, but a prospect never reaches a moat debate if "Start Call" crashes. Reliability earns the right to have the strategy conversation. **First impressions (P0) gate everything downstream**, and legal risk (F4) sits just behind them because it gates the entire India enterprise motion.
