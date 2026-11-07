# Sentinel AI Threat Detections

KQL detections for attacks on an AI chatbot hosted in Azure, written for
Microsoft Sentinel. Each rule includes a threat mapping, false positives,
tuning ideas, limitations, and response steps.

> **Note:** All data in this project is simulated. The `AIUsage` table was
> created for practice and is not a real Azure log table. Users, callers, and
> IPs are fictional (IPs use documentation ranges). The rules have not been
> deployed or tested in a production environment, and table and field names
> must be checked against real Azure OpenAI logs before use.

## Scenario

An internal chatbot runs on Azure. An attacker sends repeated prompt injection
attempts, then uses a stolen service key from a new country to generate unusually
high token usage. Separate rules cover request flooding and brand-new callers.
A correlated rule combines three signals into one high-confidence alert.

## Detections

| # | Rule | What it detects | Query | Documentation |
|---|------|-----------------|-------|---------------|
| 1 | Prompt injection burst | Several injection attempts from one IP in a short time | [query](queries/01-prompt-injection-burst.kql) | [doc](detections/01-prompt-injection-burst.md) |
| 2 | Token usage spike | A caller using far more tokens than its own baseline | [query](queries/02-token-usage-spike.kql) | [doc](detections/02-token-usage-spike.md) |
| 3 | New country for caller | A caller seen from a country it has not used before | [query](queries/03-new-country-for-caller.kql) | [doc](detections/03-new-country-for-caller.md) |
| 4 | Correlated AI attack | All three signals above from the same caller and IP | [query](queries/04-correlated-ai-attack.kql) | [doc](detections/04-correlated-ai-attack.md) |
| 5 | Rate limit abuse | Many throttled (HTTP 429) requests from one IP | [query](queries/05-rate-limit-abuse.kql) | [doc](detections/05-rate-limit-abuse.md) |
| 6 | New caller | A caller with no history that sends several requests | [query](queries/06-new-caller.kql) | [doc](detections/06-new-caller.md) |

## Incident report

The simulated attack is written up as a short one-page report and a full report
with timeline, impact, indicators, response steps, and lessons learned:

- [Short report](incident-report/incident-report-short.md)
- [Full report](incident-report/incident-report-full.md)

## Repository layout

    queries/           Runnable KQL for each rule (simulated data included)
    detections/        Documentation for each rule
    incident-report/   Short and full reports for the simulated attack
    screenshots/       Query results (to be added)

## How to test

1. Open a free KQL environment, such as Azure Data Explorer
   (dataexplorer.azure.com) or a Log Analytics workspace.
2. Copy one file from `queries/` and paste it into the query editor.
3. Select all and run it. Each file already includes its own simulated data,
   so nothing else needs to be set up.

## Moving to real data

- Replace the simulated `AIUsage` table with the real Azure OpenAI diagnostic
  table in your workspace, and map the field names.
- Replace the fixed `detectStart` time with a relative one such as `ago(1h)`.
- Use a much longer baseline and history than the simulated data has.
- Validate thresholds against your own traffic before enabling any rule.

## Known limitations

- Simulated data is small, so baselines and histories are far thinner than in
  a real environment.
- Rules use fixed time buckets, which can split a burst across a boundary.
- The token rule uses an average, which can hide a spike when normal requests
  are mixed into the window.
- Rule 4 matches on caller and IP only and does not check the order or timing
  of events.

## Skills shown

KQL, Microsoft Sentinel, detection engineering, alert correlation, alert tuning,
threat mapping (MITRE ATT&CK, MITRE ATLAS, OWASP LLM Top 10), detection
documentation, incident reporting
