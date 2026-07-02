# New Relic

| Attribute | Value |
|-----------|-------|
| Category | Observability, Logging & Monitoring |
| Type | APM + LLM Observability |
| License | Proprietary |
| Tier | B |
| Region | Global |
| First Triaged | 2026-07-02 |
| Last Updated | 2026-07-02 |

> OpenLIT integration; pre-built dashboards; OTLP endpoint

---

## Overview

New Relic is a apm + llm observability in the observability, logging & monitoring category.



---

## Deep Analysis

### 1. How Is This Tool Useful?

New Relic is an established APM platform that has added LLM observability through OpenLIT integration, pre-built GenAI dashboards, and OTLP endpoint support for AI workloads. Teams already using New Relic for application monitoring can add LLM traces and metrics via OpenTelemetry without a separate vendor. Use cases include correlating LLM call latency with application performance, tracking LLM costs per service, and monitoring AI-powered feature quality.

### 2. Gotchas of Using This Tool

New Relic's LLM features are bolted onto an APM platform rather than built AI-first — the LLM-specific experience is less deep than dedicated tools like LangSmith or Langfuse. OpenLIT integration means an extra component to manage. Pricing for high-volume LLM trace data can be significant on top of existing APM costs.

### 3. Limitations

New Relic handles enterprise-scale observability but LLM trace volume (with rich token-level data) can strain ingestion and storage. The OTEL pipeline for LLM traces inherits New Relic's general data retention policies (typically 8 days on free tier, 400 days on Pro). LLM-specific evaluation features are limited compared to specialized eval platforms.

### 4. How Secure Is This Tool?

New Relic is SOC 2 Type II, HIPAA, and FedRAMP compliant with enterprise SSO, RBAC, and data residency. LLM trace data inherits the same security posture as all New Relic data. PII redaction is configurable at ingestion.

### 5. Usefulness to General Public and Non-Technical Users

New Relic requires platform expertise — configuring dashboards, alerts, and NRQL queries needs New Relic-specific knowledge. Non-technical users can view dashboards but cannot configure them. **Rating: 2/10** for general public accessibility.

### 6. What Does This Tool Solve That Others Don't?

New Relic's unique value for AI observability is unified APM + LLM monitoring for existing customers — correlating LLM calls with database queries, cache hits, and infrastructure metrics in one platform. The NRQL query language is powerful for custom analytics on trace data. For New Relic shops, it eliminates a separate LLM observability vendor.

### 7. How Does This Tool Rank Compared to Others?

| Rank | Tool | Strength |
|------|------|----------|
| 1 | New Relic | Best for existing NR customers, unified APM + LLM |
| 2 | Datadog LLM Obs | Comparable APM + LLM integration, larger market share |
| 3 | Honeycomb | Better high-cardinality tracing, OTel-native |

### 8. How Can This Tool Be Improved? How Active Is Development?

New Relic continues to invest in AI observability features with regular releases. The team is expanding LLM-specific dashboards, evaluation features, and framework integrations. Roadmap includes deeper agent tracing, cost analytics, and improved OpenLIT integration.

### 9. Official Maintainer Contacts

- Website: [newrelic.com](https://newrelic.com)
- Docs: [docs.newrelic.com](https://docs.newrelic.com)
- Support: support.newrelic.com
- GitHub: [newrelic](https://github.com/newrelic)
- Community: [discuss.newrelic.com](https://discuss.newrelic.com)

### 10. General Usage Guidance

Enable AI monitoring in your New Relic account, install OpenLIT or configure OTEL export to New Relic's OTLP endpoint, and use the pre-built GenAI dashboards. Start with LLM call tracing via OpenLIT auto-instrumentation, then customize dashboards with NRQL for your specific metrics.

---

## License

Content for this page is licensed CC BY 4.0 — share and adapt with attribution. Authored by Team Ardur.
