# 06 Open questions for Tap

To send to the Tap account manager in week 1, together with the enablement request (02-B). Ordered by what blocks most.

**Blocking**

1. **Void.** Is there an endpoint to void an authorization on demand? Docs only show `auto.type: VOID` at authorize time. If not, can `auto.time` be updated after creation (Update an Authorize)?
2. **KNET authorize.** The KNET page says authorization is supported. Request format, capture, void and hold duration?
3. **Multiple partial captures.** Can one `auth_` be captured more than once? Error 1150 mentions "outstanding" amount.

**Needed before QA sign-off**

4. **Refunds by method.** Which of Mada, KNET, Benefit, STC Pay support refunds, partial refunds and multiple refunds per charge? Refund window per method?
5. **Webhook delivery.** Retry schedule, max attempts, timeout, and is there a stable event ID? Can webhook URL be set at account level instead of per request?
6. **Response code 303 "Deferred".** When is it returned and what's the final state?
7. **Mada redirect.** Required or not? (Payment Methods page vs Mada page disagree.)
8. **MIT 3DS flag.** For merchant-initiated charges, `threeDSecure: false` or `true`? (MIT guide text vs its sample.)
9. **Error codes.** Full list including codes not on the published page (e.g. 1216).

**Nice to have**

10. Rate limits per endpoint.
11. Name of the query param on `redirect.url`.
12. STC Pay: card/account saving, refunds.
13. KNET: any merchant-initiated option beyond KFAST (customer present)?
14. A test trigger for async `IN_PROGRESS` and for refund failure codes in sandbox.
