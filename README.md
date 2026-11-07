# Sentinel AI Threat Detections

KQL detections for attacks on an AI chatbot hosted in Azure, written for
Microsoft Sentinel. Each rule includes a threat mapping, false positives,
tuning ideas, and response steps.

> **Note:** All data in this project is simulated. The `AIUsage` table was
> created for practice and is not a real Azure log table. Users, callers, and
> IPs are fictional (IPs use documentation ranges). The rules have not been
> tested in a production environment, and field names should be checked against
> real Azure OpenAI logs before use.

## Scenario

An internal chatbot runs on Azure. An attacker sends repeated prompt injection
attempts, then uses a service key to generate unusually high token usage.

## Detections

| # | Rule | What it detects |
|---|------|-----------------|
| 1 | Prompt injection burst | Several injection attempts from one IP in a short time |
| 2 | Token usage spike | A caller using far more tokens than its own baseline |

More detections are planned.

## Repository layout

    test-data/     Simulated AIUsage data
    queries/       Runnable KQL for each rule
    detections/    Documentation for each rule
    screenshots/   Query results

## Skills shown

KQL, Microsoft Sentinel, detection engineering, alert tuning, threat mapping
(MITRE ATLAS, OWASP LLM Top 10), detection documentation
