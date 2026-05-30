# Deep dive 02 — API surface analysis

> Supports **F3** (docs quality) and the [Strengths](../README.md#8--what-id-genuinely-keep-strengths) section. Source: the official docs at `docs.vocallabs.ai` (read end-to-end) and the API reference bundled in the `n8n-nodes-vocallabs` repo (`apidocs.md`).

## What the API can do (this part is genuinely strong)

The B2B API is broad and mature. Grouped by the docs' own navigation:

| Group | Notable capabilities |
|---|---|
| **Auth** | `createAuthToken` via `clientId`+`clientSecret` → Bearer token. |
| **Wallet** | `getGreenBalance`, `whatsubTransactionHistory` (a credit ledger). |
| **SIP / Call** | `createSIPCall`, `initiateVocallabsCall`, `createDirectCall`, `hangupCall`, call timeline, daily-call stats. |
| **Agents** | Create/update agents, `agent_prompt`, `welcome_message`, `analytics_prompt`, `voice_id`, templates, FAQs, **keyword/pronunciation replacement**, **success metrics**, WhatsApp/email notification toggles, agent documents. |
| **Analytics** | Call conversation, summary, status, and **prompt-based post-call data extraction** (`updatePostCallData`). |
| **Contacts & groups** | Full CRUD, bulk add, metadata, v1 **and** v2 variants. |
| **Library / Actions** | **Custom Actions = mid-call function-calling** via `external_curl` with explicit `success_response` / `failure_response` / `interruption_response`. Action templates, parameters, fields. |
| **Identity** | `getIdentityUrl` with `verification_type: aadhaar or pan` — **in-call KYC**. |
| **Campaigns** | Create/manage campaigns, queueing, add contact groups to a campaign. |
| **Marketplace** | `fetchAvailableNumbers`, `getNumbers`, `fetchCountries` — rent DIDs. |
| **Conference** | `Bridge Call` (`conference-call`) — connect two phone numbers. |
| **RAG** | `insertAgentDocx` with `webcrawler_depth` — agents learn from crawled sites/docs. |

**Takeaway:** as a *capability surface*, this competes with the serious players. The problem is not breadth — it's packaging (F1), documentation depth (F3), reliability (F8), and positioning (F2).

## Where the docs fall down (F3 evidence)

1. **Response bodies are undocumented.** Every endpoint shows the identical generic response: `status: string`, `data: object — "endpoint-specific payload."` A developer never learns the real return shape.
2. **Errors are boilerplate.** The same 401/400/402/404 block is pasted onto every endpoint, rather than endpoint-specific failures.
3. **Generic auto-filled descriptions.** e.g. "Agent prompt History — call this endpoint to interact with agent"; "Update fields on an existing resource."
4. **Verb bugs.** "Add multiple contacts to group" is labelled **POST** but its sample is `curl -X GET`, and its description talks about creating a *single* contact.
5. **Insomnia export artifacts.** Examples carry `User-Agent: insomnia/10.3.0 … 12.0.0`, and stray query params like `?0=` / `?1=`. The bundled `apidocs.md` even leaves a **ChatGPT share link** as a placeholder `file_url`.

## Billing model (inferred)

A `402 — insufficient balance — "wallet cannot cover the request (if billable)"` on billable endpoints indicates **pay-per-call metering against a prepaid wallet** (the Subspace "green" balance). There is no public pricing page, so the unit economics are opaque to a prospective customer (ties to F1).
