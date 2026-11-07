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
high token usage. The rules detect each stage on its own, and a fourth rule
combines them into one high-confidence alert.

## Detections

| # | Rule | What it detects | Query | Documentation |
|---|------|-----------------|-------|---------------|
| 1 | Prompt injection burst | Several injection attempts from one IP in a short time | [query](queries/01-prompt-injection-burst.kql) | [doc](detections/01-prompt-injection-burst.md) |
| 2 | Token usage spike | A caller using far more tokens than its own baseline | [query](queries/02-token-usage-spike.kql) | [doc](detections/02-token-usage-spike.md) |
| 3 | New country for caller | A caller seen from a country it has not used before | [query](queries/03-new-country-for-caller.kql) | [doc](detections/03-new-country-for-caller.md) |
| 4 | Correlated AI attack | All three signals from the same caller and IP | [query](queries/04-correlated-ai-attack.kql) | [doc](detections/04-correlated-ai-attack.md) |

## Repository layout

    queries/       Runnable KQL for each rule (simulated data included)
    detections/    Documentation for each rule
    screenshots/   Query results (to be added)

## How to test

1. Open a free KQL environment, such as Azure Data Explorer
   (dataexplorer.azure.com) or a Log Analytics workspace.
2. Copy one file from `queries/` and paste it into the query editor.
3. Run it. Each file already includes its own simulated data, so nothing else
   needs to be set up.

## Moving to real data

- Replace the simulated `AIUsage` table with the real Azure OpenAI diagnostic
  table in your workspace, and map the field names.
- Replace the fixed `detectStart` time with a relative one such as `ago(1h)`.
- Use a much longer baseline and history than the simulated data has.
- Validate thresholds against your own traffic before enabling any rule.

## Skills shown

KQL, Microsoft Sentinel, detection engineering, alert correlation, alert tuning,
threat mapping (MITRE ATT&CK, MITRE ATLAS, OWASP LLM Top 10), detection
documentation
