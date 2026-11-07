# Rule 5: Rate Limit Abuse

## What it detects
Many throttled requests from one IP address in a short time. When a client
sends requests faster than the service allows, the service rejects them with
HTTP status 429 (too many requests). A burst of 429 responses from one source
suggests an automated attack, such as a script flooding the chatbot, trying
many stolen keys, or trying to exhaust the service.

## How it works
1. Keep only events where the status code is 429.
2. Group them by source IP into 5-minute time buckets.
3. Alert when one IP has 5 or more throttled events in a bucket.

The window (5 minutes) and threshold (5) are variables at the top of the query,
so they are easy to change when tuning.

## Threat mapping
- OWASP Top 10 for LLM Applications: Unbounded Consumption (resource abuse)
- MITRE ATT&CK: look up the technique that best matches a request flood
  against an application (for example under Impact, Endpoint Denial of
  Service) and record the ID here.

(Verify the category names and IDs on owasp.org and attack.mitre.org before
publishing. OWASP names and numbers change between versions.)

## Simulated attack
IP 203.0.113.77, calling as "bot-svc", sends 8 requests between 15:00 and 15:03
and every one is throttled with status 429.

## Expected result
One row for 203.0.113.77 with 8 throttled requests, callers ["bot-svc"]. The
two throttled requests from user2 at 15:10 are below the threshold of 5, so
they do not alert. That is the negative test.

## False positives
- A legitimate application with a bug that retries too aggressively
- A batch job that is not allowed enough capacity
- Many users behind one shared office or VPN IP address
- A real traffic surge that exceeds the configured limits

## Tuning ideas
- Raise the threshold or lengthen the window if alerts are too noisy
- Count distinct callers per IP to separate one attacker from many users
- Exclude approved batch jobs or known internal IP ranges
- Combine with other signals, such as prompt injection (Rule 1) or a new
  country (Rule 3), before raising the severity

## Limitations
- The rule only sees requests that reached the service and were throttled.
  An attacker staying just under the rate limit will not be caught.
- Fixed time buckets can split a burst across a bucket boundary. A sliding
  window would fix this.
- Rate limits and status codes depend on how the service is configured. The
  field names and status codes must be checked in real logs.
- All data is simulated and very small.

## Response steps
1. Identify the source IP and which caller or key it used.
2. Block or rate-limit the IP at the gateway.
3. Check whether any requests from the same source succeeded, and what they
   asked for.
4. Check for earlier activity from the same IP, such as prompt injection (Rule 1)
   or a new-country alert (Rule 3).
5. Rotate the key if a stolen credential is suspected.
