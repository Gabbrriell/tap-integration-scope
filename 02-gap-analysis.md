# 02 Gap analysis

What the merchant asked for vs what Tap documents. Written so CS can take section A straight back to the merchant.

## A. Merchant requirements Tap can't meet (as documented)

| Ask | Problem | Proposed answer to merchant |
|---|---|---|
| Auth now / capture on shipment, **all methods** | Benefit and STC Pay don't support authorize. KNET authorize is claimed but undocumented. | Auth/capture on cards and Mada. Benefit and STC Pay are purchase-only (charge at checkout). KNET pending Tap confirmation. |
| Monthly renewals on saved methods | Benefit saved cards need a PIN on every payment, so no MIT. STC Pay has no recurring. KNET MIT not documented (KFAST is customer-present). | Renewals on cards and Mada only. Benefit / STC Pay / KNET customers get a pay-by-link or in-app prompt each cycle. |
| Cancel an order before shipping (void) | No on-demand void endpoint in the docs. Only a scheduled auto-void set at authorize time (min 1 h). | Pending Tap. Fallback: capture-then-refund, which costs the merchant fees and shows on the customer's statement. Merchant should know this before go-live. |
| Partial refunds on every method | Supported at API level, but Tap has a refund code for "partial refund not supported" and doesn't say which methods it applies to. | Confirm per method with Tap before promising. |

## B. Things Tap must switch on for our account

1. **Benefit** acceptance (contact account manager) [BEN]
2. **STC Pay** acceptance (contact account manager) [STC]
3. **KFAST** saved cards on KNET [KFAST]
4. **Non-3DS / merchant-initiated** charges in live, needed for any MIT renewal [MIT]
5. **Save card** feature (error 1108 "Save card features are not enabled" shows it's account-gated) [CODES]

Ask for all five in one request; each is a separate dependency on the go-live date.

## C. What sandbox won't let us test

- **KFAST** is not available in sandbox at all [KFAST]. First real test is in live.
- **Authorize in test mode** doesn't place a real hold [AUTH], so auto-void / auto-capture timing can only be checked as status changes, not on a real card.
- **Webhook retries:** no documented way to force a retry or a delayed delivery.
- **Async payments** (`IN_PROGRESS`): no test card listed that triggers it.
- **Refund failure codes** (insufficient balance, partial not supported): no test data listed.

Plan: a small live-money test pass (minimum amount 0.100) per method after go-live switch, refunded same day.

## D. Contradictions and errors in Tap's docs

| # | Where | Contradiction | Treat as |
|---|---|---|---|
| 1 | Mada | Payment Methods overview and Create Charge say Mada uses redirect and needs `redirect.url`. Mada page says no further redirection. | Always send `redirect.url`. Confirm with Tap. |
| 2 | MIT | MIT guide says set `threeDSecure: false`; its own sample request sends `true`. Authorize and Capture guide says never send `false` on capture. | Send `false` for MIT charges, omit or `true` on captures. Confirm. |
| 3 | Refunds | `amount` typed as integer in the OpenAPI spec, but description says up to 3 decimal places. | Send decimals. Test 0.250 KWD partial refund in sandbox. |
| 4 | KNET | KNET page says the response "will redirect the customer to the Benefit Payment Page". Copy-paste error. | Ignore. |
| 5 | MIT error | Missing agreement returns code 1216, which isn't in the published error code list. | Add 1216 to the mapping; ask Tap for the full list. |

## E. Risks to the 6-week date

- Benefit / STC Pay / KFAST / non-3DS enablement depends on Tap, not us. Request in week 1.
- Void behaviour decides whether the merchant's cancel flow works. Blocker until answered.
- KNET authorize may turn out to be unusable through the API, which changes the merchant's Kuwait flow.

Sources: see [01-capability-matrix.md](01-capability-matrix.md#sources).
