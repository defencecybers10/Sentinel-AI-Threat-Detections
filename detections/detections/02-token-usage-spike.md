# Rule 2: Token Usage Spike

## What it detects
A caller using far more tokens per request than its own normal level. Stolen
or abused API keys are often used to run large volumes of requests, which
shows up as a sudden jump in token usage.

## How it works
1. Build a baseline: the average tokens per request for each caller, using
   activity before the detection window.
2. Look at activity inside the detection window and calculate each caller's
   current average.
3. Alert when the current average is 5 or more times the caller's own baseline.

The multiplier (5) and the detection start time are variables at the top of the
query, so they are easy to change when tuning. In a real workspace, replace the
fixed start time with a relative one such as ago(1h).

## Why compare to a baseline
A fixed token limit would flag busy but legitimate users and miss quiet
accounts that suddenly spike. Comparing each caller to its own history avoids
both problems.

## Threat mapping
- OWASP Top 10 for LLM Applications: LLM10, Unbounded Consumption
- MITRE ATLAS: look up the technique that best matches abuse of a stolen
  credential or model access, and record the ID here.

(Verify these on atlas.mitre.org and owasp.org before publishing. The OWASP
category names and numbers can change between versions.)

## Simulated attack
The service key "key-svc" normally averages about 910 tokens per request from
198.51.100.20. From 14:02 it is used from 203.0.113.50: six small prompt
injection requests, then three very large requests (18,000 to 19,000 tokens)
at 14:20 to 14:22.

## Expected result
One row for key-svc from 203.0.113.50 with 9 requests, a current average of
about 6,369 tokens against a baseline of about 910. The average is pulled down
by the six small injection requests, so it clears the 5x threshold (4,550) with
less margin than the three large requests alone would suggest. Regular users do
not appear because they have no activity in the detection window.

## False positives
- Batch jobs, such as summarising a large set of documents
- A product launch or other legitimate busy period
- A caller whose baseline is based on very few requests

## Tuning ideas
- Raise or lower the multiplier depending on how noisy the alerts are
- Require a minimum number of requests in the window
- Exclude known batch jobs by caller name

## Limitations
- Baseline poisoning: if the baseline period already contains the attack, the
  baseline is inflated and the rule can miss it. Exclude the most recent hours
  from the baseline, or use a longer history.
- A brand-new caller has no baseline, so it will not match the join. A separate
  rule for new callers is needed.
- The simulated baseline has only three events. A real baseline needs much more
  history.

## Response steps
1. Check the source IP and country against the caller's usual location.
2. Review what the requests were asking for and what was returned.
3. Restrict or rotate the key.
4. Check for earlier activity from the same IP, such as prompt injection
   attempts (see Rule 1).
