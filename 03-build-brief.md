# 03 Build brief: Tap method extensions

**For:** external developer. **Assumes:** existing Tap connection (auth, card purchase) is live; you're extending it. Diff against the current connector before starting and tell me what's already there.

## Scope

Add: Mada (purchase, authorize, recurring), KNET (purchase), Benefit (purchase), STC Pay (purchase), card authorize / capture / refund / MIT.
Not in scope: KNET authorize, Apple Pay, BNPL, marketplace splits. Void: see section 4, blocked.

## 1. Requests

All: `POST https://api.tap.company/v2/...`, `Authorization: Bearer <sk>`. Send `reference.idempotent` = our operation ID on every charge, authorize and refund (24 h replay window). Send `post.url` (our webhook) and `redirect.url` on every charge and authorize, including Mada.

| Operation | Endpoint | Key fields |
|---|---|---|
| Purchase, card | `/charges` | `source.id`: `tok_...`, `threeDSecure: true`, `customer_initiated: true`, `save_card` per intent |
| Purchase, Mada | `/charges` | `currency: SAR`, `source.id: src_sa.mada` (or card token) |
| Purchase, KNET | `/charges` | `currency: KWD`, `source.id: src_kw.knet`. Pass `customer.id` only (not full customer) so KFAST shows |
| Purchase, Benefit | `/charges` | `currency: BHD`, `source.id: src_bh.benefit`. SDK tokens expire after 5 min |
| Purchase, STC Pay | `/charges` then `PUT /charges/{id}` | Create with `source.id: src_sa.stcpay` + `source.phone`. Then PUT `gateway_response.name: STC_PAY`, `gateway_response.response.reference.otp`. OTP window 3 min |
| Authorize, card / Mada | `/authorize` | `auto: {type: VOID, time: 168}` default (see 4). Mada: `authorize_debit: true` |
| Capture | `/charges` | `source.id: auth_...`, same currency, amount <= outstanding. Don't send `threeDSecure: false` or `customer_initiated: false` |
| Refund | `/refunds` | `charge_id`, `amount` (decimal), `currency`, `reason` (`duplicate`, `fraudulent`, `requested_by_customer`), `post.url` |
| Save card | any CIT with `save_card: true` | Store `card.id`, `customer.id`, `payment_agreement.id` from the response |
| MIT / renewal | Create a Token (Saved Card) then `/charges` | Token from `card_id` + `customer_id`. Charge with `customer_initiated: false`, `threeDSecure: false`, `payment_agreement.id`. Missing agreement returns 1216 |

## 2. Flows

- **Redirect methods** (3DS cards, KNET, Benefit, Mada): status `INITIATED` + `transaction.url`. Return it to our redirect handler. Final status comes from the GET in section 3, never from the redirect query string alone.
- **Direct** (non-enrolled card, MIT, capture): final status in the response.
- **STC Pay**: no URL. Our checkout must render an OTP input and call back. Expose this as a separate step in the connector, not a redirect.
- **Async**: `IN_PROGRESS` is pending, not failed. Keep the operation open.

## 3. Webhooks and status

1. Verify `hashstring` header: HMAC-SHA256, key = secret key, message =
   `x_id{id}x_amount{amount}x_currency{currency}x_gateway_reference{reference.gateway}x_payment_reference{reference.payment}x_status{status}x_created{transaction.created}`
   - amount formatted to currency decimals (KWD/BHD/OMR/JOD 3 dp, others 2 dp) **after** parsing
   - empty string if `reference.gateway` is missing
   - same pattern for refunds (`created` from the refund object)
2. Reject on mismatch (constant-time compare). Return 200 fast on match.
3. Treat the webhook as a trigger: GET the object and update from that. Retry policy isn't documented and there's no event ID, so dedupe on `(object id, status)`.
4. If no webhook within transaction expiry + 5 min, poll GET.

## 4. Void: blocked

No on-demand void endpoint is documented. Don't build against undocumented endpoints from community SDKs. Implement void as **"not supported"** for now and leave the operation behind a flag. I'll update this once Tap answers (see 06, Q1).

## 5. Errors

Map per [05-error-code-mapping.md](05-error-code-mapping.md). Use `response.code` for transaction outcomes, `errors[].code` for 4xx. Keep the raw Tap code and message in `provider_error_code` / `provider_error_message`. Unknown codes go to 7710, not 7000, so we can find them.

## 6. Done means

The QA checklist in [04-engineering-story.md](04-engineering-story.md) passes in sandbox, with request/response logs attached per case. Send me logs, not screenshots.

## Test data

Cards, KNET, Benefit, STC Pay numbers: https://developers.tap.company/reference/testing-cards. Decline simulation via expiry date (05/22 declined, 04/27 expired, 08/28 timeout). KNET: choose "KNET1" in the bank dropdown.
