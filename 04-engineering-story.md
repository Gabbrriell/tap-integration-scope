# 04 Engineering story

**Title:** Tap: add Mada, KNET, Benefit, STC Pay + card authorize / capture / refund / MIT

**As a** merchant routing to Tap in KW / KSA / BH, **I want** the local methods and card lifecycle operations available through MoneyHash, **so that** I can offer local checkout and recurring billing without a second integration.

**Depends on:** Tap enablement of Benefit, STC Pay, KFAST, non-3DS/MIT, save card (see 02-B). Void blocked on 06-Q1.
**Out of scope:** KNET authorize, void, Apple Pay, BNPL.
**Brief:** [03-build-brief.md](03-build-brief.md). **Mapping:** [05-error-code-mapping.md](05-error-code-mapping.md).

## Acceptance criteria (QA checklist)

Every case: sandbox, logs attached, MoneyHash operation status and code match the expected column.

### Purchase

| # | Case | Test data | Expect |
|---|---|---|---|
| P1 | Card, 3DS enrolled, success | Visa 4508750015741019, 01/39 | redirect, then 6000 |
| P2 | Card, not enrolled | MC 5111111111111118 | no URL, 6000 in response |
| P3 | Card, declined | expiry 05/22 | 7001 |
| P4 | Card, expired | expiry 04/27 | 7387 |
| P5 | Card, timeout | expiry 08/28 | 7152 |
| P6 | Mada, 3DS | 4464040000000007 | 6000, `card.scheme: MADA` |
| P7 | KNET success | 8888880000000001, 09/30, PIN 1234 | 6000 |
| P8 | KNET not captured | 8888880000000001, 05/21 | 7xxx per mapping, not 8000 |
| P9 | Benefit success / not captured / declined | 4600..., 7777..., 1111... (12/27, 1234) | 6000 / 7xxx / 7001 |
| P10 | STC Pay OTP correct | 966 548220713, OTP 123456 | 6000 after PUT |
| P11 | STC Pay OTP not submitted within 3 min | same | 7488 (expired) |
| P12 | Wrong currency per method (KNET in SAR) | | 7503, request never reaches Tap if we validate first |
| P13 | Customer abandons hosted page | close tab | stays 8000 until expiry, then 7017 (Tap 301) or 7488 (Tap 304). Document which one Tap sends |

### Authorize / capture

| # | Case | Expect |
|---|---|---|
| A1 | Card authorize | `auth_` id, 6000 on authorize op, `auto.status: SCHEDULED` |
| A2 | Full capture | capture op 6000 |
| A3 | Partial capture | 6000, `available_to_capture` reduced |
| A4 | Second partial capture | Result documented either way (feeds 06-Q2) |
| A5 | Capture over amount | Tap 1150, MoneyHash 7013, no money moved |
| A6 | Mada authorize with `authorize_debit` | Result documented |
| A7 | Benefit / STC Pay authorize request | Rejected by us before Tap call, method not eligible |
| A8 | Void requested | Returns not supported, no Tap call |

### Refund

| # | Case | Expect |
|---|---|---|
| R1 | Full refund, card | 6000 (or 8000 then 6000) |
| R2 | Partial refund, 3 dp currency (0.250 KWD on KNET) | Correct amount, hash verifies |
| R3 | Refund over remaining | 1161, 7013 |
| R4 | Two partial refunds same charge | Result documented (06-Q4) |
| R5 | Refund on Benefit / STC Pay | Result documented (06-Q3) |

### Tokenization / MIT

| # | Case | Expect |
|---|---|---|
| T1 | CIT with save_card | `card_`, `cus_`, `payment_agreement.id` stored |
| T2 | MIT with agreement | 6000, no redirect |
| T3 | MIT without agreement | 1216 mapped (7003), no retry |
| T4 | Benefit saved card reuse | Redirect for PIN, never offered for MIT |
| T5 | Mada recurring | 6000 with `customer_initiated: false` |

### Webhooks / resilience

| # | Case | Expect |
|---|---|---|
| W1 | Valid webhook | hash verifies, GET issued, status updated once |
| W2 | Tampered body (amount changed) | rejected, no status change, alert logged |
| W3 | Same webhook delivered twice | one status transition |
| W4 | Webhook for KWD/BHD amount | hash uses 3 dp, verifies |
| W5 | Webhook missing `reference.gateway` | empty string in hash, verifies |
| W6 | No webhook received | poll after expiry + 5 min resolves status |
| W7 | Duplicate charge request (same idempotent key) | same Tap charge id returned, one charge |
| W8 | 429 from Tap | backoff, retry with same idempotent key |
| W9 | Unknown Tap code | 7710, raw code preserved |
