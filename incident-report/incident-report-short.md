# Incident Report (Short): AI Service Key Abuse

> Simulated incident. All users, keys, and IPs are fictional. Based on the
> simulated `AIUsage` data in this repository.

| Field | Detail |
|-------|--------|
| Incident | Suspected stolen service key used against an internal AI chatbot |
| Date (simulated) | 2026-11-10 |
| Severity | High |
| Detected by | Rule 4: Correlated AI Attack |
| Affected asset | Service key `key-svc` |
| Source IP | 203.0.113.50 (country: NL) |
| Status | Contained (recommended actions, simulated) |

## Summary
The service key `key-svc` normally calls the chatbot from India (IN) at about
910 tokens per request. At 14:02 it was used from 203.0.113.50 in the
Netherlands. Six prompt injection attempts were sent in six minutes, followed by
three requests of 18,000 to 19,000 tokens. The combined detection fired because
the same caller and IP showed a prompt injection burst, a token spike, and a new
country.

## Key evidence
- 6 flagged prompt injection events between 14:02 and 14:08
- Average tokens per request of about 6,369 against a baseline of about 910
- First use of the key from a country other than IN

## Impact
About 57,320 tokens consumed by the suspicious source. Whether any sensitive
data was returned is unknown and must be checked in the response logs.

## Actions recommended
1. Rotate the key immediately.
2. Block 203.0.113.50.
3. Review the prompts and responses for exposed data.
4. Check whether the key was stored or shared insecurely.

## Lesson
Single signals are noisy. Correlating three signals from the same source gave a
high-confidence alert.
