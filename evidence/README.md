# Evidence index

Every screenshot used in the [teardown](../README.md), mapped to the finding it supports and what to look for. Image files live in [`screenshots/`](screenshots/).

| File | Finding | What it proves |
|---|---|---|
| `F1-homepage-developer-infra.png` | **F1** | Homepage positions Vocallabs as "Voice AI infrastructure for developers… scales from zero to millions" with a "Get Started ⚡ 2 mins" CTA. |
| `F1-contact-demo-only.png` | **F1** | The only entry point is a sales form ("Schedule Your Free Demo") — no self-serve signup. |
| `F2-docs-superflow-base-url.jpg` | **F2** | API docs serve endpoints from `api1.superflow.run/b2b/...` (Subspace backend) and expose `getGreenBalance` (a wallet). |
| `F3-docs-402-wallet-generic-response.jpg` | **F3** | Generic `data: object — endpoint-specific payload` response + boilerplate `402 insufficient balance` error. |
| `F3-docs-get-post-mismatch.jpg` | **F3** | Endpoint labelled POST whose code sample uses `GET` — a docs correctness bug. |
| `F3-docs-insomnia-useragent.jpg` | **F3** | "Bridge Call" example still carries `User-Agent: insomnia/12.0.0` — docs exported from Insomnia, not authored. |
| `F6-chrome-store-unavailable.png` | **F6** | Official Chrome Web Store listing returns "This item is not available." |
| `F6-softonic-ads.png` | **F6** | The extension survives only on an ad-laden third-party mirror (Softonic). |
| `F7-mobile-call-setup.jpeg` | **F7** | The public Android app is a bare-bones call tester (phone field + 8/16 kHz toggle), not a product. |
| `F8-mobile-call-failed.jpeg` | **F8** | "Start Call" fails with a raw `Hasura.GraphQL … ActionWebhookErrorResponse` error shown to the user. |
