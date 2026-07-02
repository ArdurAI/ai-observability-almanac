# Datadog LLM Observability

| Attribute | Value |
|-----------|-------|
| Category | Observability, Logging & Monitoring |
| Type | APM Extension |
| License | Proprietary |
| Tier | A |
| Region | Global |
| First Triaged | 2026-07-02 |
| Last Updated | 2026-07-02 |

> Unified infra + LLM tracing; agentless; OTel GenAI support

---

## Overview

Datadog LLM Observability is a apm extension in the observability, logging & monitoring category.



---

## Deep Analysis

### 1. How Is This Tool Useful?

Datadog LLM Observability extends Datadog's APM platform with LLM-specific tracing, cost tracking, and quality monitoring, supporting OpenTelemetry GenAI semantic conventions and agentless instrumentation. Teams already using Datadog for infrastructure monitoring can add LLM observability without adding another vendor, correlating LLM call latency with downstream database queries and infrastructure metrics. Use cases include monitoring LLM-powered features in production, alerting on quality degradation, and tracking per-feature LLM costs.

### 2. Gotchas of Using This Tool

LLM Observability is an add-on to Datadog's platform — if you're not already a Datadog customer, adopting it just for LLM monitoring is expensive and overkill. LLM-specific features lag behind dedicated LLM observability tools (LangSmith, Langfuse) in depth and specialization. Custom metric and eval configuration requires Datadog-specific knowledge that differs from open-source tools.

### 3. Limitations

Datadog's LLM tracing inherits the platform's ingestion rate limits and retention policies, which may not match the longer retention needed for LLM eval datasets. Agent-based instrumentation works for Python/Node but requires the ddtrace agent running in your environment. The feature is relatively new and some advanced capabilities (agent step tracing, complex eval pipelines) are still maturing.

### 4. How Secure Is This Tool?

Datadog is SOC 2 Type II, HIPAA, and FedRAMP compliant with extensive enterprise security certifications. Data can be scrubbed at the agent level before ingestion. Existing Datadog RBAC, SSO, and audit logging extend to LLM Observability. For organizations already vetted on Datadog, adding LLM features doesn't require new security reviews.

### 5. Usefulness to General Public and Non-Technical Users

Datadog LLM Observability requires existing Datadog expertise — navigating the platform, configuring dashboards, and setting up alerts all require platform-specific knowledge. Non-technical users can view pre-built dashboards but cannot configure them. **Rating: 2/10** for general public accessibility.

### 6. What Does This Tool Solve That Others Don't?

Datadog's unique advantage is unified full-stack observability — you can see an LLM call, its downstream database query, the infrastructure it ran on, and the user impact, all in one platform. For Datadog shops, this eliminates the need for a separate LLM observability vendor. The agentless OTEL ingestion path is also appealing for teams standardizing on OpenTelemetry.

### 7. How Does This Tool Rank Compared to Others?

| Rank | Tool | Strength |
|------|------|----------|
| 1 | Datadog LLM Obs | Best for existing Datadog customers, full-stack correlation |
| 2 | New Relic | Comparable APM+LLM integration, similar positioning |
| 3 | LangSmith | Deeper LLM-specific features, standalone |

### 8. How Can This Tool Be Improved? How Active Is Development?

Datadog continues to invest in LLM Observability with regular feature releases, including expanded framework auto-instrumentation, quality evaluation features, and AI agent tracing. The platform benefits from Datadog's massive R&D budget. Key improvement areas: deeper eval capabilities competitive with specialized tools, better cost allocation per LLM feature, and more LLM-specific dashboard templates.

### 9. Official Maintainer Contacts

- Docs: [docs.datadoghq.com/llm_observability](https://docs.datadoghq.com/llm_observability/)
- Support: support@datadoghq.com
- GitHub: [DataDog/dd-trace-py](https://github.com/DataDog/dd-trace-py) (agent)
- Community: [Datadog Community](https://www.datadoghq.com/community/)

### 10. General Usage Guidance

Enable LLM Observability in your Datadog account (requires APM), install the ddtrace agent in your application environment, and configure LLM tracing for your framework (OpenAI, LangChain, etc.). Start with automatic instrumentation for your LLM provider, then add custom spans for RAG retrieval steps. Use pre-built LLM monitoring dashboards as a starting point.

---

## License

Content for this page is licensed CC BY 4.0 — share and adapt with attribution. Authored by Team Ardur.
