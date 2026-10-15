# 05 Error-code mapping: Tap to MoneyHash unified codes

Target: MoneyHash public status codes (6000 success, 8000 pending, 7xxx declined; soft vs hard per MoneyHash docs).
**This is a proposal from public docs.** The live Tap connector already has a mapping; diff against it and keep whichever is more specific.
⚠ = judgement call, confirm with Tap or with how other PSP connectors map the same meaning.

## Charge / authorize `response.code`

| Tap | Tap message | MH | MH name | Type | Note |
|---|---|---|---|---|---|
| 000 | Captured | 6000 | Success | | |
| 001 | Authorized | 6000 | Success (authorize op) | | |
| 100 | Initiated | 8000 | Pending | | |
| 301 | Abandoned | 7017 | CUSTOMER_CANCELLATION | Hard | ⚠ abandon vs explicit cancel |
| 302 | Canceled | 7017 | CUSTOMER_CANCELLATION | Hard | |
| 303 | Deferred | 8000 | Pending | | ⚠ meaning not documented (06-Q6) |
| 304 | Expired | 7488 | TRANSACTION_EXPIRED | Hard | |
| 401 | Failed | 7000 | FAILED | Soft | |
| 402 | Failed, Invalid Parameter | 7003 | INVALID_REQUEST | Soft | |
| 403 | Failed, Duplicate | 7094 | DUPLICATION | Soft | |
| 404 | Failed, Locked | 7057 | TRANSACTION_NOT_PERMITTED | Hard | ⚠ |
| 405 | Failed, Invalid Card No | 7055 | INSTRUMENT_CREDENTIALS | Hard | |
| 406 | Failed, Invalid Expiry | 7387 | EXPIRY_DATE | Hard | |
| 407 | Failed, Expired Card | 7387 | EXPIRY_DATE | Hard | |
| 408 | Failed, Unspecified | 7000 | FAILED | Soft | |
| 501 | Declined | 7001 | BANK_DECLINE | Soft | |
| 502 | Declined, Incorrect CVV | 7055 | INSTRUMENT_CREDENTIALS | Hard | |
| 503 | Declined, 3DS Incorrect | 7151 | AUTHENTICATION_FAILED | Soft | |
| 504 | Declined, 3DS Card not Enrolled | 7154 | AUTHENTICATION_REQUIRED | Soft | ⚠ |
| 505 | Declined, Insufficient Funds | 7051 | INSUFFICIENT_FUNDS | Soft | |
| 506 | Declined, Txn Type Not Supported | 7057 | TRANSACTION_NOT_PERMITTED | Hard | |
| 507 | Declined, Card Issuer | 7001 | BANK_DECLINE | Soft | |
| 508 | Declined, Issuer No Reply | 7152 | TRANSACTION_TIMEOUT | Soft | |
| 509 | Declined, Issuer Do not Contact | 7057 | TRANSACTION_NOT_PERMITTED | Hard | ⚠ |
| 510 | Declined, Issuer Referral | 7001 | BANK_DECLINE | Soft | |
| 511 | Declined, Card Issuer Error | 7015 | ISSUER_ERROR | Hard | |
| 512 | Declined, Not Authenticated | 7154 | AUTHENTICATION_REQUIRED | Soft | |
| 513 | Declined, Acquirer Error | 7035 | ACQUIRER_ERROR | Soft | |
| 514 | Declined, Issuer Risk Check | 7104 | RISK_DECLINE | Hard | |
| 515 | Declined, Tap | 7104 | RISK_DECLINE | Hard | ⚠ Tap-side reason unknown |
| 516 | Declined, Authentication Failed | 7151 | AUTHENTICATION_FAILED | Soft | |
| 601 | Void | 6000 | Success (void op) | | |
| 701 | Restricted | 7057 | TRANSACTION_NOT_PERMITTED | Hard | |
| 702 | Restricted, Retry Limit | 7105 | LIMIT_EXCEEDED | Soft | |
| 703 | Restricted, Bank | 7057 | TRANSACTION_NOT_PERMITTED | Hard | |
| 704 | Restricted, Tap | 7104 | RISK_DECLINE | Hard | ⚠ |
| 801 | Timed Out | 7152 | TRANSACTION_TIMEOUT | Soft | |
| 901 | Unknown | 7000 | FAILED | Soft | Keep raw code |

## Refund `response.code`

| Tap | Tap message | MH | Note |
|---|---|---|---|
| 000 | Refunded | 6000 | |
| 100 | Pending | 8000 | |
| 200 | In Progress | 8000 | |
| 301 | Canceled | 7000 | ⚠ who cancels a refund? |
| 401 / 801 | Failed / Unknown | 7000 | |
| 402 | Invalid Parameter | 7003 | |
| 403 | Duplicate | 7094 | |
| 501 | Declined | 7001 | |
| 502 | Insufficient Balance | 7511 | ACCOUNTS_ERROR. Merchant balance, not cardholder. Don't use 7051 ⚠ |
| 503 | Refund not Supported | 7057 | |
| 504 | Partial Refund not Supported | 7057 | Surface to merchant: full refund only for this method |
| 601 / 602 | Restricted / Restricted, Bank | 7057 | |
| 701 | Timed Out | 7152 | Check status before retrying, refund may have gone through |

## API errors (`errors[].code`, HTTP 4xx/5xx)

These are request problems, mostly ours or config. Alert on them; they shouldn't reach the customer as declines.

| Tap | Meaning | MH | Action |
|---|---|---|---|
| HTTP 401, 1101, 2104, 2105, 2106 | Bad / wrong-env key | 7709 PSP_CONFIGURATION | Page on-call |
| 1108 | Save card not enabled | 7709 | Tap enablement |
| 1109 | Non-3DS not allowed | 7709 | Tap enablement (blocks MIT) |
| 1216 | Agreement id missing | 7003 | Bug on our side |
| 1110, 1111 | Redirect URL missing / invalid | 7706 INVALID_REQUEST | Bug |
| 1119 | Currency not supported | 7503 CURRENCY | Pre-validate per method |
| 1117, 9998 | Invalid amount | 7013 INVALID_AMOUNT | |
| 1150 | Capture exceeds outstanding | 7013 | |
| 1161 | Refund exceeds | 7013 | |
| 1113, 1114, 1115 | Authorize invalid / bad status / not found | 7003 | |
| 1126 | Source already used | 7003 | Create new token, don't retry same |
| 1151 | Gateway timed out | 7152 | Check status first |
| 1163 | Card not supported | 7078 PAYMENT_INSTRUMENT | |
| HTTP 429 | Rate limited | 7100 SYSTEM_BUSY | Backoff, same idempotent key |
| HTTP 5xx, 2101, 9999 | Tap down / internal | 7705 MH_PSP_UNAVAILABLE | Eligible for failover routing |
| anything else | | 7710 MAPPING_NOT_SUPPORTED | Log, add to this table |

## Sources

- Tap Charges Response Codes: https://developers.tap.company/reference/charge-response-codes
- Tap MIT (code 1216): https://developers.tap.company/docs/merchant-initiated-transaction
- MoneyHash Status Codes: https://docs.moneyhash.io/docs/status-codes
