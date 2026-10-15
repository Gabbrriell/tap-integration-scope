# Tap Payments: method-level integration scope

A worked example of scoping a payment provider integration from public API documentation, the way an integrations lead would hand it to engineering and an external developer.

**What this is:** a portfolio exercise. I picked a realistic intake (a GCC subscription merchant asking for local methods through Tap), read Tap's public API docs, and produced the full scoping package: capability matrix, gap analysis, build brief, engineering story with QA acceptance criteria, and an error-code mapping onto MoneyHash's public unified status codes.

**What this isn't:** affiliated with Tap or MoneyHash, or based on any non-public information. Everything is sourced from public docs read in October 2026. Where the docs don't answer something, it's marked as an open question instead of guessed.

## The intake

> *From CS:* Merchant (subscriptions + one-off orders, sells in Kuwait, Saudi Arabia, Bahrain) wants to route through Tap: KNET, Mada (incl. recurring), Benefit, STC Pay, plus Visa/MC with auth now / capture on shipment, partial refunds, and saved cards for monthly renewals. Go-live target: 6 weeks.

**Classification (step 1 of intake):** MoneyHash announced a Tap integration in Jan 2026, so this is **not a new provider**. It's a set of **new methods and capability extensions on an existing provider**. The first internal action is checking which Tap methods and operations are already live before anything is briefed. From outside I can't see that, so the scope below covers everything the merchant asked for and flags what to diff against the existing connection.

## Files

| File | For | What's in it |
|---|---|---|
| [01-capability-matrix.md](01-capability-matrix.md) | Integrations / PM | Method x operation matrix: integration type, currency, operations, tokenization, recurring, 3DS, webhooks. Each cell sourced or marked unknown. |
| [02-gap-analysis.md](02-gap-analysis.md) | CS, merchant, eng lead | What the merchant asked for that Tap can't do, what Tap must enable on our account, what sandbox can't test, and doc contradictions. |
| [03-build-brief.md](03-build-brief.md) | External developer | Self-contained brief: endpoints, flows, fields, webhook verification, idempotency. |
| [04-engineering-story.md](04-engineering-story.md) | Squad backlog | Story with QA checklist as acceptance criteria. |
| [05-error-code-mapping.md](05-error-code-mapping.md) | Engineering | Tap charge / authorize / refund / API error codes mapped to MoneyHash unified codes, with judgement calls flagged. |
| [06-open-questions.md](06-open-questions.md) | Tap account manager | The questions to send Tap before build starts. |

## Headline findings

1. **Three of the merchant's asks can't be met as written.** STC Pay has no authorize and no recurring. Benefit has no authorize, and saved Benefit cards still need a PIN per payment, so no merchant-initiated renewals. KNET MIT/recurring isn't documented.
2. **Manual void isn't documented.** Tap's documented void is a scheduled auto-void (1 to 168 hours) set at authorize time. Our void operation needs either an undocumented endpoint confirmed by Tap, or a defined fallback.
3. **Webhooks are a trigger, not a source of truth.** Retry policy isn't documented and there's no event ID, so every webhook should be verified (HMAC) then followed by a GET on the object.
4. **Five things need Tap to switch on for our account:** Benefit, STC Pay, KFAST, save card, and non-3DS / MIT in live.
5. **Docs contradict themselves in three places** (Mada redirect, MIT 3DS flag, refund amount type), plus an undocumented error code. Listed in the gap analysis with sources.

## Sources

Tap: developers.tap.company (Charges, Authorize, Refunds, Webhook, Payment Methods, KNET, KFAST, Mada, Mada Recurring, Benefit, STC Pay, Merchant Initiated Transaction, Idempotency, Authorize and Capture, Charges Response Codes, Test Cards).
MoneyHash: docs.moneyhash.io (Status Codes, Webhook Types, Capture Operation), moneyhash.io/blog (Tap partnership announcement).
Full URLs are listed at the bottom of each file.
