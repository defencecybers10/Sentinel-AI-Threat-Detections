# Incident Report (Full): AI Service Key Abuse

> **Simulated incident.** All users, keys, and IPs are fictional (IPs use
> documentation ranges). The report is based on the simulated `AIUsage` data in
> this repository. Containment and recovery steps are recommended actions, not
> actions that were performed.

## 1. Executive summary
A service key used by an internal AI chatbot was used from a new country, sent
repeated prompt injection attempts, and then generated very large token
requests. The pattern is consistent with a stolen credential being abused. The
correlated detection raised a high-severity alert. The key should be rotated and
the responses reviewed for exposed data.

## 2. Incident details
| Field | Detail |
|-------|--------|
| Incident ID | SIM-2026-001 |
| Date (simulated) | 2026-11-10 |
| Severity | High |
| Detected by | Rule 4: Correlated AI Attack |
| Affected asset | Service key `key-svc` |
| Attacker source | 203.0.113.50 (country: NL) |
| Normal source | 198.51.100.20 (country: IN) |

## 3. Timeline (simulated)
| Time | Event |
|------|-------|
| 09:00 to 09:40 | Normal use: `key-svc` from 198.51.100.20 (IN), about 880 to 950 tokens per request |
| 14:02 to 14:08 | Six prompt injection events from 203.0.113.50 (NL), about 300 tokens each |
| 14:20 to 14:22 | Three large requests of 18,000, 18,500 and 19,000 tokens |
| After 14:22 | Rules 1, 2 and 3 each match, and Rule 4 combines them into one alert |

## 4. Detection
| Rule | Result |
|------|--------|
| Rule 1: Prompt injection burst | Fired: 6 events in one bucket from 203.0.113.50 |
| Rule 2: Token usage spike | Fired: average about 6,369 against a baseline of about 910 |
| Rule 3: New country | Fired: `key-svc` seen from NL, history only IN |
| Rule 4: Correlated attack | Fired: all three signals from the same caller and IP |

Benign events did not alert on the combined rule. `user2` had one flagged prompt
(below the threshold), and `user3` had a large request from India with no other
signals.

## 5. Analysis
- The key's history was one country and one IP, so a new country is unusual.
- Prompt injection from a service key is unexpected, because a service key
  should not be typed into by a person.
- The token jump is about 20 times the baseline for the three large requests.
- The sequence is probing first, then heavy use, which is consistent with an
  attacker testing a stolen key before exploiting it.
- **Likely cause (assumption):** the key was exposed in a place such as source
  code, a shared document or a log. This cannot be confirmed from the data.

## 6. Impact
- About 57,320 tokens consumed by the suspicious source (1,820 from the
  injection requests and 55,500 from the three large requests).
- Cost: unknown, depends on the model and pricing.
- Data exposure: **unknown.** The simulated logs do not include prompt or
  response content, so this needs a review of the responses.

## 7. Containment, eradication and recovery (recommended)
**Containment**
1. Disable or rotate `key-svc`.
2. Block 203.0.113.50 at the network or gateway.
3. Apply rate limits to the chatbot.

**Eradication**
4. Find where the key was exposed and remove it.
5. Search for other keys used from 203.0.113.50.

**Recovery**
6. Issue a new key through a secrets store, not in code.
7. Restore normal access and watch usage closely for several days.

## 8. Indicators
| Type | Value | Note |
|------|-------|------|
| IP address | 203.0.113.50 | Source of the abuse (documentation range, fictional) |
| Caller | key-svc | Abused key |
| Country | NL | New for this caller |
| Behaviour | 3 or more injection flags in 10 minutes | Rule 1 |
| Behaviour | Tokens 5 times above baseline | Rule 2 |

## 9. Threat mapping
- MITRE ATT&CK: T1078.004, Valid Accounts: Cloud Accounts
- MITRE ATLAS and OWASP LLM Top 10: see the rule documentation

(Verify IDs on attack.mitre.org, atlas.mitre.org and owasp.org.)

## 10. Lessons learned and recommendations
1. Correlating independent signals gave a high-confidence alert that single
   rules would have buried in noise.
2. Store service keys in a secrets manager and rotate them on a schedule.
3. Restrict service keys to known IP ranges where possible.
4. Log prompt and response metadata so data exposure can be assessed.
5. Improve the token rule by using maximum or total tokens, since averages can
   hide spikes.
6. Add a rule for brand-new callers that have no history.

## 11. Limitations
- All data is simulated, and the baseline and history are tiny.
- The simulated logs contain no prompt or response content.
- Rules match on caller and IP only and do not check the order of events or how
  close together they happened.
