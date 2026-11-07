# Rule 4: Correlated AI Attack

## What it detects
One caller and IP address showing all three warning signs together: a burst of
prompt injection attempts, a token usage spike, and a request from a country the
caller has never used. Each signal alone is common and noisy. All three from the
same source is a high-confidence sign of a stolen credential being abused.


## How it works
1. Signal 1 (Rule 1): 3 or more prompt injection events from one caller and IP
   in a 10-minute bucket.
2. Signal 2 (Rule 2): average tokens per request at least 5 times the caller's
   own baseline.
3. Signal 3 (Rule 3): requests from a country that is not in the caller's
   history.
4. Alert only when the same caller and IP matches all three signals.

The window, threshold, multiplier and detection start time are variables at the
top of the query, so they are easy to change when tuning.

## Why correlate
Single rules produce many alerts, and analysts learn to ignore them. Requiring
several independent signals from the same source cuts noise and raises
confidence, so the alert gets high severity.

## Threat mapping
- MITRE ATT&CK: T1078.004, Valid Accounts: Cloud Accounts
- MITRE ATLAS and OWASP LLM Top 10: the same entries as Rules 1 to 3

(Verify all IDs on attack.mitre.org, atlas.mitre.org and owasp.org before
publishing.)

## Simulated attack
The service key "key-svc" normally calls from India. From 203.0.113.50 in the
Netherlands it sends 6 prompt injection requests, then three very large
requests.

## Expected result
One row for key-svc from 203.0.113.50, country NL, with 6 injection attempts,
a baseline of about 910 tokens, a current average of about 6,369 tokens, and 9
new-country requests. Severity is High.

## Negative test
The caller user3 makes a 9,000-token request at 14:40 from India. That fires the
token spike signal on its own (Rule 2 would alert), but it has no injection burst
and no new country, so this combined rule stays quiet.

## False positives
- A penetration test or red-team exercise run through a legitimate key from a
  new location
- A service moved to a new region that also has a launch-day usage spike and a
  tester sending injection prompts

These are rare because all three signals must match.

## Tuning ideas
- Require only two of the three signals for medium severity, and all three for
  high severity
- Add a time limit so the signals must happen within a few hours of each other
- Exclude approved test IP addresses

## Limitations
- The signals are matched on caller and IP only. The rule does not check the
  order of events or require them to happen close together in time.
- If one signal is missed, such as an attack the content filter does not flag,
  the combined rule will not fire. Rules 1 to 3 still run on their own.
- The token signal uses an average, which can hide a spike when normal requests
  are mixed into the window. Using the maximum or the total would be more robust.
- All data is simulated, and the baseline and history are tiny.

## Response steps
1. Treat as a likely stolen credential and prioritise the incident.
2. Restrict or rotate the key immediately.
3. Review the prompts sent and the responses returned, and check for data that
   should not have been exposed.
4. Block the source IP and look for other keys or callers used from it.
5. Write an incident report with the timeline.
