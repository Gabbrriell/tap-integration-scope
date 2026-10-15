# 01 Capability matrix: Tap, requested methods

Legend: **Yes** = documented. **No** = documented as unsupported. **?** = docs don't say (see open questions). **Enable** = Tap must switch it on for our account.
Source codes in brackets refer to the list at the bottom.

## Method x capability

| | Visa / MC / Amex | Mada | KNET | Benefit | STC Pay |
|---|---|---|---|---|---|
| **Country / currency** | Per merchant account ? | KSA / SAR only [MADA] | KW / KWD only [KNET] | BH / BHD only [BEN] | KSA / SAR only [STC] |
| **Source ID** | `tok_` (Card SDK), `src_card`, `src_all` [CHG] | `src_sa.mada` or card token [MADA] | `src_kw.knet` [KNET] | `src_bh.benefit` or Card SDK token [BEN] | `src_sa.stcpay` + `source.phone` [STC] |
| **Integration type** | Hosted page (redirect), or tokenized via Card SDK with 3DS redirect; direct (no URL) if card not 3DS-enrolled [PM] | Hosted page. Redirect required per [PM][CHG], not required per [MADA] (conflict) | Redirect to KNET page, mandatory [KNET] | Redirect to Benefit page for PIN, both flows [BEN] | Server to server: create charge, customer gets OTP, merchant PUTs OTP to the charge. No redirect URL [STC] |
| **Purchase** | Yes | Yes | Yes | Yes | Yes |
| **Authorize** | Yes [AUTH] | Yes, with `authorize_debit: true` [AUTH] | "Supports authorization now", no flow detail [KNET] ? | **No** [BEN] | **No** [STC] |
| **Capture** | Yes: `POST /charges` with `authorize_id` as source. Amount <= authorized [A&C] | ? (assumed same as cards) | ? | n/a | n/a |
| **Multiple partial captures** | ? (error 1150 says "outstanding authorized amount", hints yes) [RC] | ? | ? | n/a | n/a |
| **Void (on demand)** | **? No endpoint documented.** Only scheduled `auto.type: VOID`, 1-168 h (Visa/MC tokens up to 720 h) [AUTH] | ? | ? | n/a | n/a |
| **Auto-capture after N hours** | Yes, `auto.type: CAPTURE` [AUTH] | ? | ? | n/a | n/a |
| **Refund full / partial** | Yes, `POST /v2/refunds`, full or partial [REF] | Yes at API level; per-method support ? | ? | ? | ? |
| **Multiple refunds per charge** | ? | ? | ? | ? | ? |
| **Refund window** | ? | ? | ? | ? | ? |
| **Tokenization (save card)** | Yes, `save_card: true` returns `card_` + `cus_` [CHG] | Yes [MADA-R] | KFAST: saved on KNET page, **Enable**, not in sandbox [KFAST] | Yes, but **PIN required on every use** [BEN] | ? (not documented) |
| **CIT with saved card** | Yes, token from saved card [TOK] | Yes | Yes via KFAST list on KNET page | Yes, PIN each time | ? |
| **MIT / recurring** | Yes: `customer_initiated: false` + `payment_agreement.id`. Non-3DS in live: **Enable** [MIT] | Yes [MADA-R] | ? not documented | **No** (PIN per payment) [BEN] | **No** [STC] |
| **3DS** | `threeDSecure` flag; test cards for enrolled / not enrolled [TEST] | Test cards for both [TEST] | n/a (PIN on KNET page) | n/a (PIN) | n/a (OTP) |
| **Async final status** | Possible: `IN_PROGRESS` for async methods [PM] | | | | |
| **Webhook** | Per-request `post.url` on charge, authorize, refund [WH] | same | same | same | same |
| **Account enablement** | Non-3DS / MIT live [MIT] | | KFAST [KFAST] | **Enable** [BEN] | **Enable** [STC] |
| **Sandbox testable** | Yes, incl. decline simulation by expiry date [TEST] | Yes | Yes (select "KNET1" bank). KFAST no | Yes (capture / not captured / declined cards) | Yes (4 test numbers, OTP 123456) |

## Cross-cutting

| Area | Behaviour | Source |
|---|---|---|
| Auth | `Authorization: Bearer sk_test_...` / `sk_live_...`. Mode decided by key | [PM][CODES] |
| Base URL | `https://api.tap.company/v2/` | [REF][STC] |
| Amounts | Decimal in major units; 3 dp for KWD/BHD/OMR/JOD, 2 dp otherwise. Min 0.100 for any currency | [CHG-C][WH] |
| Idempotency | `reference.idempotent`, 24 h window, charges / authorize / refunds. Replay returns original response or same 3DS page | [IDEM] |
| Hosted page / txn expiry | `transaction.expiry` 30 min typical; STC Pay 3 min | [KNET][STC] |
| Webhook auth | HMAC-SHA256 over a fixed field string, keyed with secret key, in `hashstring` header | [WH] |
| Webhook retries | ? not documented. Payload has `post.attempt` | [WH] |
| Rate limits | 429 returned, exponential backoff recommended. Limits not published | [CODES] |
| Redirect back | `redirect.url`; Tap appends the object ID as a query param. Param name not stated in Tap docs (community SDKs read `tap_id`) ? | [AUTH] |
| Arabic UI | `lang_code: ar` header on hosted pages | [AUTH] |

## Out of scope for this intake

Apple Pay, Google Pay, Samsung Pay, Tabby, Tamara, Deema, Fawry, OmanNet, NAPS/QPay, marketplace splits, payouts. All exist in Tap's docs; the merchant didn't ask for them.

## Sources

- [PM] Overview of Payment Methods: https://developers.tap.company/docs/payment-methods
- [CHG] Charges: https://developers.tap.company/docs/charges-1
- [CHG-C] Create a Charge: https://developers.tap.company/reference-link/create-a-charge
- [AUTH] Create an Authorize: https://developers.tap.company/reference/create-an-authorize
- [A&C] Authorize and Capture: https://developers.tap.company/docs/authorize-and-capture
- [REF] Create a Refund: https://developers.tap.company/reference/create-a-refund
- [TOK] Create a Token (Saved Card): https://developers.tap.company/reference/create-a-token-from-saved-card
- [MIT] Merchant Initiated Transaction: https://developers.tap.company/docs/merchant-initiated-transaction
- [MADA] Mada: https://developers.tap.company/docs/mada
- [MADA-R] Mada Recurring: https://developers.tap.company/docs/mada-recurring
- [KNET] KNET: https://developers.tap.company/docs/knet
- [KFAST] KFAST: https://developers.tap.company/docs/kfast
- [BEN] Benefit: https://developers.tap.company/docs/benefit
- [STC] STC Pay: https://developers.tap.company/docs/stcpay
- [WH] Webhook: https://developers.tap.company/docs/webhook
- [IDEM] Idempotency: https://developers.tap.company/docs/idempotency
- [RC] / [CODES] Charges Response Codes: https://developers.tap.company/reference/charge-response-codes
- [TEST] Test Cards: https://developers.tap.company/reference/testing-cards
