# Rule 3: New Country for a Caller


## What it detects
A caller making requests from a country it has never used before. If a service
key or user account suddenly appears from a new country, the credential may have
been stolen and is being used by someone else.

## How it works
1. Build a history: the set of countries each caller has used, from activity
   before the detection window.
2. Look at activity inside the detection window.
3. Alert when a caller is seen from a country that is not in its own history.

The detection start time is a variable at the top of the query. In a real
workspace, replace the fixed start time with a relative one such as ago(1h).

## Threat mapping
- MITRE ATT&CK: T1078.004, Valid Accounts: Cloud Accounts (use of a stolen
  credential)
- MITRE ATLAS: look up the technique that best matches use of a valid account
  or credential to reach a model, and record the ID here.

(Verify these IDs on attack.mitre.org and atlas.mitre.org before publishing.)

## Simulated attack
The service key "key-svc" normally calls from 198.51.100.20 in India (IN). From
14:02 it is used from 203.0.113.50 in the Netherlands (NL), a country it has
never used.

## Expected result
One row for key-svc from NL, with 9 requests, first seen at 14:02, and known
countries ["IN"]. The normal request from user1 at 14:30, from a country it has
used before, does not alert. That is the negative test.

## False positives
- Employees travelling or working remotely
- VPN or proxy use that exits in a different country
- Mobile carriers and cloud-hosted services whose IP addresses geolocate
  unexpectedly
- A service legitimately moved to a new region

## Tuning ideas
- Only alert on service keys, which should rarely change country, and treat
  human users with a higher threshold
- Combine with other signals, such as a token spike (Rule 2) or prompt
  injection (Rule 1), before raising the severity
- Keep a list of approved countries or known VPN ranges

## Limitations
- IP geolocation is not exact, so a country can be wrong
- A caller with no history will not match the join, so a separate rule for
  brand-new callers is needed
- The simulated history is tiny. A real history needs weeks of activity, and
  a short history will cause false alerts
- An attacker using an IP in the caller's usual country will not be caught

## Response steps
1. Confirm with the key owner whether the new location is expected.
2. Review what the requests asked for and the volume of tokens used.
3. Restrict or rotate the key if the activity is not expected.
4. Look for other activity from the same IP, such as prompt injection attempts
   (Rule 1) or token spikes (Rule 2).
