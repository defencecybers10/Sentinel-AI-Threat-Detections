# Rule 1: Prompt Injection Burst

## What it detects
Several prompt injection attempts from the same IP address within a short time.
A single flagged prompt can be an accident or a curious user. Repeated attempts
in a few minutes suggest someone is deliberately trying to override the
chatbot's instructions.

## How it works
1. Keep only events where the content filter flagged a prompt injection.
2. Group them by source IP into 10-minute time buckets.
3. Alert when one IP has 3 or more flagged events in a bucket.

The window (10 minutes) and threshold (3) are variables at the top of the query,
so they are easy to change when tuning.

## Threat mapping
- MITRE ATLAS: LLM Prompt Injection (AML.T0051)
- OWASP Top 10 for LLM Applications: LLM01, Prompt Injection

(Verify these IDs on atlas.mitre.org and owasp.org before publishing.)

## Simulated attack
IP 203.0.113.50 sends 6 flagged prompts between 14:02 and 14:08.

## Expected result
One row for 203.0.113.50 with 6 attempts. The single flagged prompt from
user2 at 11:15 does not alert, because it is below the threshold.

## False positives
- Security testers or red-team exercises
- Curious employees testing the chatbot
- Shared office or VPN IP addresses where several users each trigger one flag

## Tuning ideas
- Raise the threshold, or lengthen the window, if alerts are too noisy
- Exclude known tester IP addresses
- Count distinct users per IP to separate one attacker from many casual users

## Limitations
- Time buckets are fixed, so an attack that straddles a bucket boundary can be
  split and missed. A sliding window would fix this.
- The rule depends on the content filter flagging the prompt. Attacks the
  filter misses will not be counted.

## Response steps
1. Review the flagged prompts and the caller identity.
2. Block or rate-limit the source IP.
3. If a service key is involved, check where else it was used and rotate it.
4. Check for follow-up activity, such as unusual token usage (see Rule 2).
