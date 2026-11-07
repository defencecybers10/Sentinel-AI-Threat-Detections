# Rule 6: New Caller

## What it detects
A caller that has no earlier history and then sends several requests. Rules 2
and 3 compare a caller to its own past behaviour, so they skip callers that have
no past. An attacker using a freshly created key, or a key that was never used
before, would slip past both. This rule closes that gap.

## How it works
1. Build a list of all callers seen before the detection window.
2. Look at activity inside the detection window.
3. Remove every caller that is in the known list (an anti-join).
4. Alert when a remaining caller has 3 or more requests.

The detection start time and the minimum request count are variables at the top
of the query, so they are easy to change when tuning. In a real workspace,
replace the fixed start time with a relative one such as ago(1h).

## Threat mapping
- MITRE ATT&CK: T1078.004, Valid Accounts: Cloud Accounts, or T1098, Account
  Manipulation, if the attacker created the new key themselves
- OWASP Top 10 for LLM Applications: Unbounded Consumption, if the new caller
  is also generating heavy usage

(Verify these IDs on attack.mitre.org and owasp.org before publishing. OWASP
names and numbers change between versions.)

## Simulated attack
A caller named "key-new" has never appeared before. From 15:30 it sends 5
requests of 4,200 tokens each from 203.0.113.88 in the Netherlands.

## Expected result
One row for key-new with 5 requests, 21,000 total tokens, IP 203.0.113.88 and
country NL.

## Negative tests
- user4 is also a new caller but sends only 1 request, below the minimum of 3,
  so it does not alert.
- user1 is a known caller, so the anti-join removes it.

## False positives
- A new employee or a newly deployed application starting to use the chatbot
- A key rotated as part of normal security practice, which appears as a new
  caller name
- A new project or team onboarding

## Tuning ideas
- Raise the minimum request count if onboarding causes noise
- Compare against an approved list of newly created keys, for example from a
  change ticket or deployment record
- Combine with a new country (Rule 3) or a token spike (Rule 2) before raising
  the severity

## Limitations
- The rule only knows about callers it has seen in the data. A short history
  makes ordinary callers look new, so the history window should be long in a
  real workspace.
- A new caller that sends only 1 or 2 requests will not alert.
- If the caller name is reused or spoofed, the rule cannot tell.
- All data is simulated and very small.

## Response steps
1. Find out who created the key or account, and whether it was approved.
2. Review what the requests asked for and the volume of tokens used.
3. Check the source IP and country against what is expected.
4. Disable the key if the owner cannot be confirmed.
5. Look for related activity from the same IP, such as prompt injection (Rule 1)
   or throttled requests (Rule 5).
