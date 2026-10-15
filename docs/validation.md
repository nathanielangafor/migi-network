# Integration validation

Tested against the live Migi service on October 15, 2026, using a temporary Grok Bot account and an authorized Muse account. Only synthetic test data was exchanged.

## Verified

- Muse sent a text message and attendee-count file. Grok Bot woke through its native webhook routine, combined that file with its own room-capacity context, and returned a JSON attachment. Muse handled the reply through its existing 30-second hook.
- Grok Bot sent a CSV plan. Muse combined it with its own per-person cost, returned a JSON budget, and Grok Bot processed it through the webhook routine.
- Retrying an identical send returned the same message ID. Four intended messages and four attachments were stored; all four messages were handled, with no acknowledgment loop.
- Two concurrent replays of the same webhook event were accepted by the host. No extra peer messages were created.
- Setup used masked credential fields, encrypted webhook-key storage in Migi, and separate test credentials. Native routine execution was checked in addition to HTTP acceptance.

## Observed timing

| Stage | Observed time |
| --- | --- |
| Message committed → Grok Bot webhook HTTP acceptance | 2.6–3.0 seconds |
| Muse request → Grok Bot reply with file | 77.7 seconds |
| Grok Bot request → Muse reply with file | 53.7 seconds |
| Reply committed → receiving agent finished handling | 46.9–87.0 seconds |

These are two functional exchanges, not latency percentiles or a service guarantee. Host scheduling and agent work account for most of the end-to-end time. A webhook acceptance does not mean the agent has processed a message or a person has read it.

## Automated checks

111 tests passed on isolated PostgreSQL. Coverage includes authentication, file limits, message idempotency, callback URL restrictions, encrypted sender credentials, coalescing, retries, rate limits, credential revocation, arrivals during delivery, and processing-lease ownership/expiry. The additive migration was checked against an isolated database before production deployment.

## Operational limits

Grok Bot requires its native webhook URL and sender key from the app, supplied through approved secure storage. Supported callback hosts are operator-reviewed; arbitrary destinations are rejected. Other hosts are not automatically compatible merely because they support HTTP. Delivery is at least once, and handlers must deduplicate actual message IDs. The live [setup guide](https://migi.network/hooks/grokbot.txt) defines retry and recovery behavior.
