# Testing & Benchmarking

How the observability almanac tests tools, what the harness does, how scoring works, and how to reproduce results.

## Table of Contents

1. [The Three Benchmark Types](#the-three-benchmark-types)
2. [The Seven Dimensions](#the-seven-dimensions)
3. [The Harness Architecture](#the-harness-architecture)
4. [The Canary](#the-canary)
5. [Standard Benchmarks](#standard-benchmarks)
6. [ObservabilityOps Custom Benchmarks](#observabilityops-custom-benchmarks)
7. [Stress Suite](#stress-suite)
8. [Cross-Category Integration Tests](#cross-category-integration-tests)
9. [Scoring](#scoring)
10. [Reproducibility](#reproducibility)
11. [Failure Mode Taxonomy](#failure-mode-taxonomy)

---

## The Three Benchmark Types

Every observability tool is tested across three types of benchmarks:

| Type | Purpose | Frequency |
|------|---------|-----------|
| **Standard benchmarks** | Verify vendor claims with published test suites (RAGAS, DeepEval) | Every benchmark run |
| **ObservabilityOps custom benchmarks** | Test ops reality: trace overhead, setup, eval cost, drift detection latency | Every benchmark run |
| **Cross-category integration tests** | Test how observability tools work with vector DBs, agent frameworks, and model serving in a full RAG stack | Quarterly |

## The Seven Dimensions

Every tool is scored 0-100 on each dimension. The final score is a weighted average, but the per-dimension scores are always published.

| Dimension | Weight | What it measures (observability-specific) | How it's tested |
|-----------|--------|-------------------------------------------|-----------------|
| **Accuracy / Quality** | 25% | Do eval metrics (RAGAS, DeepEval) correlate with human judgment? Does drift detection catch real regressions? | Standard benchmarks (RAGAS, DeepEval) + human-labeled comparison set + drift injection tests |
| **Latency** | 15% | Tracing overhead per request; eval framework latency; time to first alert in drift detection | Instrumented measurements; p50, p95, p99 of trace overhead; eval run time per 1K samples |
| **Token Economics** | 15% | Cost per eval run; cost per trace stored; pricing predictability at scale | Standardized workloads; $/1K eval runs, $/1M traces, $/1M tokens scored |
| **Scale Behavior** | 15% | What happens at 10K concurrent traces? Does the eval framework degrade at 100K rows? Can the drift detector handle streaming data? | Load tests; trace volume floods; eval batch size curves; streaming drift detection |
| **Ops Burden** | 15% | Time to first trace; dependency conflicts between tracing SDKs; upgrade pain when the eval framework changes metric APIs | Measured setup time; smoke-gate sweep; dependency matrix across Langfuse, LangSmith, Braintrust, etc. |
| **Developer Experience** | 10% | Trace UI quality; metric interpretability; exportability; debugging missing spans; alert configurability | Structured rubric; community health metrics; dashboard walkthroughs |
| **Data Sovereignty** | 5% | Self-hosting viability (trace collector on-premise); audit trails for eval scores; SOC 2 / GDPR alignment; does the tool phone home? | Feature matrix; EU AI Act / GDPR / SOC 2 mapping; network traffic inspection during benchmark runs |

### Why these weights?

The weights reflect what an observability engineer actually cares about. Accuracy is the most important — a drift detector that fires false positives is worse than no detector at all. But ops burden is nearly as important because an eval framework that consumes your team's life with broken SDK upgrades and conflicting dependencies is not worth the accuracy gain.

Weights are reviewed annually. Changes require an RFC and a public comment period.

## The Harness Architecture

```
┌─────────────────────────────────────────┐
│  CategoryAdapter (frozen contract)        │
│  ├── setup()   → install, configure      │
│  ├── instrument() → wrap model/agent     │
│  ├── load()    → ingest workload         │
│  ├── await_ready() → async barrier       │
│  ├── query()   → run test, get response  │
│  └── teardown() → cleanup, measure       │
└─────────────────────────────────────────┘
              ↓
┌─────────────────────────────────────────┐
│  Telemetry Collector                    │
│  ├── latency (p50/p95/p99)             │
│  ├── token count & cost                  │
│  ├── memory & CPU usage                 │
│  ├── trace completeness (span coverage) │
│  ├── error rate & failure mode taxonomy │
│  └── ops notes (setup time, deps, bugs)  │
└─────────────────────────────────────────┘
              ↓
┌─────────────────────────────────────────┐
│  Grading Pipeline                         │
│  ├── Deterministic grader (exact match)  │
│  ├── LLM judge (frozen prompts, SHA-256) │
│  ├── Human correlation (eval vs. human)  │
│  ├── Second pass (confidence < 0.7)      │
│  └── Failure mode taxonomy               │
└─────────────────────────────────────────┘
              ↓
┌─────────────────────────────────────────┐
│  Results Publisher                        │
│  ├── Raw JSON (per trace, per eval, per run) │
│  ├── Summary tables (per tool)          │
│  ├── Cross-verification analysis          │
│  └── Insight extraction                 │
└─────────────────────────────────────────┘
```

### The `await_ready()` barrier

This is where async-ingestion designs get their cost measured instead of hidden. Many observability tools (trace collectors, eval dashboards, drift indexers) claim "fast" write paths because the actual work happens in the background. The `await_ready()` barrier forces the tool to finish all background work (trace flush, index commit, metric computation) before the query is run, so the true latency and completeness are measured.

### The Telemetry Collector

Every adapter call is instrumented:
- **Latency**: `time.monotonic()` around every call; p50, p95, p99 computed across all runs
- **Trace overhead**: Latency difference between instrumented and non-instrumented model calls
- **Tokens**: If the eval tool uses LLM APIs for scoring, token counts are captured from API responses
- **Cost**: Token counts × provider pricing; or measured cloud spend for self-hosted tools; per-trace storage cost
- **Memory**: `psutil` or container metrics for memory usage during the run (trace buffer bloat, eval framework memory leaks)
- **CPU**: CPU utilization during the run (drift detection algorithms, metric computation)
- **Trace completeness**: Percentage of expected spans actually captured vs. dropped
- **Errors**: Every exception, timeout, missing span, or metric mismatch is logged with full traceback
- **Ops notes**: Human observations about setup friction, dependency conflicts, documentation quality, dashboard usability

## The Canary

The first run of every batch is the **no-tool baseline** (the "canary"). If the benchmark leaked answers anywhere, the canary would score above zero on answerable categories.

**Canary rules**:
- The canary must score exactly **0.000** on all answerable categories (e.g., the canary eval framework with no logic should score 0 on RAGAS metrics)
- The canary must score exactly **1.000** on abstention categories (e.g., the canary drift detector should abstain on everything — no false positives)
- If the canary fails, the entire batch is invalid and must be rerun
- The canary run is published alongside the real results
- The canary adapter is a no-op: setup does nothing, instrument does nothing, load does nothing, query returns empty/zeros, teardown does nothing

## Standard Benchmarks

### By subcategory

| Subcategory | Benchmark | What it tests | Source |
|-------------|-----------|-------------|--------|
| Evaluation | RAGAS | RAG evaluation accuracy (faithfulness, answer relevance, context recall) | [RAGAS](https://github.com/explodinggradients/ragas) |
| Evaluation | DeepEval | LLM evaluation metrics (hallucination, bias, toxicity, answer relevance) | [DeepEval](https://github.com/confident-ai/deepeval) |
| Evaluation | Custom RAG Suite | Domain-specific RAG evaluation on technical documentation | Custom harness |
| Tracing | Trace Completeness | Span coverage, parent-child correctness, attribute capture | Custom harness |
| Tracing | OpenTelemetry Compliance | OTLP conformance, span attribute standards | [OTLP Spec](https://opentelemetry.io/docs/specs/otlp/) |
| Drift Detection | Synthetic Drift Injection | Detection latency, false positive rate, AUC-ROC on known drift patterns | Custom harness |
| Drift Detection | Concept Drift Benchmark | Gradual vs. abrupt drift detection accuracy | Custom harness |
| Cost Monitoring | Token Cost Accuracy | Predicted cost vs. actual billed cost | Custom harness |
| Cost Monitoring | Pricing Model Transparency | Hidden fees, egress charges, seat minimums | Custom harness |

### Published vs. reproduced

Every standard benchmark ranking ships a table:

| Tool | Published Claim | Our Result | Delta | Verdict |
|------|----------------|------------|-------|---------|
| Tool A | "95% on RAGAS" | 92% on RAGAS | -3% | ✅ Close |
| Tool B | "Zero tracing overhead" | 8ms p99 overhead | +8ms | ⚠️ Misleading |
| Tool C | No claim | 87% on DeepEval | N/A | — |

## ObservabilityOps Custom Benchmarks

### Setup experience

**Measured**:
- Time from `pip install` to first captured trace or first eval run
- Number of dependency conflicts when installing alongside other roster tools (e.g., Langfuse and LangSmith SDKs in the same environment)
- Time to resolve dependency conflicts
- Number of undocumented steps required (e.g., "create a project in the dashboard first" not mentioned in quickstart)
- Time to find the answer in the docs when stuck (e.g., "how do I trace a streaming response?")

**Scored on**:
- Sub-5 minutes: 90-100
- 5-30 minutes: 70-89
- 30-60 minutes: 50-69
- 60+ minutes or unresolved: 0-49

### Smoke gate

Every tool must pass an identical 3-turn scenario before entering the roster:

```
Turn 1: Instrument a model call and generate a trace
Turn 2: Collect the trace and verify span completeness
Turn 3: Query a metric (eval score, trace latency, or drift alert) and verify the result
```

**Pass criteria**:
- No crashes, no silent failures, no trace loss
- Results must be deterministic (same input → same trace structure / same eval score)
- Tool must handle the basic case without workarounds (e.g., no manual span ID injection)

**What the smoke gate surfaced** (observability-specific examples):
- Trace loss: Async model calls not captured because the SDK only hooks synchronous methods
- Metric inflation: Eval framework double-counting tokens because it re-runs the scorer internally
- Cross-project contamination: Tracing tool writing to a global shared project because the default config is not isolated
- Cloud tethers: "Self-hosted" trace collector still calling cloud APIs for metadata enrichment
- Drift false positives: Baseline window too small, causing random variance to trigger alerts
- Eval latency spread: Sub-100ms to ~35 seconds for 3-turn eval depending on judge model choice

### Stress suite

| Test | What it does | What it reveals |
|------|-------------|---------------|
| **Contradiction storms** | Rapidly store contradictory facts and evaluate them | How the eval framework handles conflicting ground truth; metric stability |
| **Near-duplicate floods** | Store thousands of similar traces | Deduplication quality, trace index bloat, search latency degradation |
| **Temporal paradoxes** | Store traces that span long durations with nested async calls | Temporal trace correctness, parent-child span integrity across async boundaries |
| **Concurrent writers** | Multiple threads/agents writing traces simultaneously | Race conditions in trace buffers, span ID collisions, collector locking |
| **Kill-the-backing-store** | Crash the trace database or eval cache mid-operation | Recovery, data integrity, trace loss on restart, eval re-computation |
| **Cost-runaway** | Run the eval framework at maximum scale for 1 hour | Cost predictability, billing accuracy, token budget exhaustion |
| **Drift avalanche** | Inject gradual drift over 10K samples | Detection latency, false positive rate under sustained change, alert fatigue |
| **Trace volume flood** | Send 100K traces in 60 seconds | Collector throughput, backpressure handling, drop rate under saturation |

### Upgrade path

**Tested**:
- Can you upgrade the tracing SDK from version N to N+1 without rewriting all instrumentation?
- Are there breaking changes in the eval metric APIs?
- Is there a migration guide for dashboard queries or saved eval configurations?
- Does the tool maintain backward compatibility for trace export formats?

### Debugging experience

**Tested**:
- When a trace is missing, can you find out why in <5 minutes?
- When an eval score is unexpected, can you inspect the intermediate steps?
- Are error messages clear and actionable? (e.g., "Span dropped: max depth exceeded" vs. "Error: 400")
- Is there a debug mode or verbose logging for trace export?
- Are there known issues documented for async instrumentation, streaming, or custom callbacks?
- Can you trace the execution path of a single request through all spans?

## Cross-Category Integration Tests

These tests run quarterly and check how observability tools work with tools from other categories in a realistic stack:

| Integration | What it tests | Tools involved |
|-------------|-------------|----------------|
| **Agent + Vector DB + Observability** | Full RAG stack with tracing: does the trace capture retriever, reranker, and generator spans? | Agent framework, vector DB, observability tool |
| **Model Serving + Observability** | Does the tracing SDK add <10ms overhead to inference? Does it capture streaming tokens? | Model serving platform, observability tool |
| **Security + Observability** | Do guardrail events appear as spans in the trace? Can eval metrics score filtered outputs? | Security tool, observability tool |
| **Protocol + Observability** | MCP server traces: are tool calls within an agent trace visible and queryable? | Protocol implementation, observability tool |

## Scoring

### Per-dimension scoring

Each dimension is scored 0-100 using a rubric. The rubric is published before any scoring happens.

**Example: Accuracy rubric (for eval tools)**

| Score | Criteria |
|-------|----------|
| 90-100 | ≥95% correlation with human judgment on RAGAS/DeepEval; no critical failures in drift detection; <1% false positive rate |
| 80-89 | 85-95% correlation; minor failures in drift detection; 1-5% false positive rate |
| 70-79 | 75-85% correlation; some stress suite failures; 5-10% false positive rate |
| 60-69 | 65-75% correlation; frequent stress suite failures; 10-20% false positive rate |
| 50-59 | 55-65% correlation; significant reliability issues; >20% false positive rate |
| 0-49 | <55% correlation or fundamentally unreliable; drift detector is unusable |

**Example: Latency rubric (for tracing tools)**

| Score | Criteria |
|-------|----------|
| 90-100 | <1ms p99 overhead per request; eval runs complete in <10s per 1K samples |
| 80-89 | 1-5ms p99 overhead; eval runs 10-30s per 1K samples |
| 70-79 | 5-15ms p99 overhead; eval runs 30-60s per 1K samples |
| 60-69 | 15-50ms p99 overhead; eval runs 60-120s per 1K samples |
| 50-59 | 50-100ms p99 overhead; eval runs >120s per 1K samples |
| 0-49 | >100ms p99 overhead or fundamentally blocking |

### Composite score

The composite score is a weighted average of the seven dimensions:

```
Composite = (Accuracy × 0.25) + (Latency × 0.15) + (TokenEconomics × 0.15) +
            (ScaleBehavior × 0.15) + (OpsBurden × 0.15) + (DevEx × 0.10) +
            (DataSovereignty × 0.05)
```

The composite is used for ranking, but the per-dimension scores are always published. A tool with a high composite but low ops burden score is a warning sign — it might be accurate but painful to maintain.

### Confidence intervals

Every score is reported with a confidence interval computed from the standard error across runs. If the intervals overlap between two tools, the difference is not statistically significant.

## Reproducibility

### How to reproduce a benchmark

1. Clone the benchmark harness repo (published separately)
2. Check out the exact commit used for the run (recorded in the results JSON)
3. Install the exact dependencies (lockfile is published)
4. Run the harness with the same adapter and same seed
5. Compare your results to the published results

### What is frozen

| Element | How it's frozen | Where to find it |
|---------|---------------|------------------|
| Judge model | Pinned model name and version | `results.json` metadata |
| Judge prompts | SHA-256 hash | `methodology/benchmark-harness.md` |
| Evaluation datasets | Published JSON files | `benchmarks/` directory |
| Control variables | Documented values | `results.json` metadata |
| Random seeds | Published integer | `results.json` metadata |
| Adapter code | Published in harness repo | Separate repo |
| Test workloads | Published JSON files (traces, eval datasets, drift baselines) | `benchmarks/` directory |

### What is NOT frozen (and why)

| Element | Why it changes | How we handle it |
|---------|---------------|------------------|
| Tool versions | Tools update SDKs and collectors | We re-run benchmarks for new versions; old results are archived |
| Provider pricing | Cloud pricing for trace storage, eval compute changes | Cost is computed at runtime using current pricing; historical results are annotated |
| Judge model pricing | Token costs for eval scoring change | Cost per eval run is computed at runtime; historical results are annotated |
| Hardware | We may upgrade machines | Hardware spec is recorded in `results.json`; results are hardware-specific |

## Failure Mode Taxonomy

Every failure is classified into a taxonomy. This helps identify patterns across tools and subcategories.

| Category | Failure Modes |
|----------|--------------|
| **Setup** | `install_failed`, `dependency_conflict`, `config_error`, `missing_env_var`, `docs_incomplete`, `api_key_required_no_trial` |
| **Instrumentation** | `instrument_failed`, `async_trace_loss`, `streaming_not_captured`, `span_id_collision`, `context_propagation_broken`, `callback_not_fired` |
| **Ingestion** | `write_timeout`, `write_crash`, `trace_loss`, `span_dropped`, `metric_corruption`, `async_lag`, `backpressure_drop` |
| **Query** | `query_timeout`, `query_crash`, `wrong_metric`, `missing_span`, `incomplete_trace`, `false_positive_drift`, `false_negative_drift`, `irrelevant_alert` |
| **Scale** | `throughput_degradation`, `memory_leak`, `cpu_spike`, `connection_pool_exhaustion`, `rate_limit_hit`, `trace_buffer_overflow`, `eval_queue_backlog` |
| **Ops** | `upgrade_breaking`, `undocumented_behavior`, `debug_opacity`, `community_unresponsive`, `dashboard_slow`, `alert_fatigue` |
| **Security** | `data_leakage`, `pii_exposure`, `trace_exfiltration`, `eval_data_phones_home`, `unencrypted_storage` |
| **Integration** | `mcp_noncompliant`, `a2a_noncompliant`, `auth_failure`, `protocol_mismatch`, `otel_incompatible`, `export_format_unsupported` |

## License

Content: CC BY 4.0  
Code: MIT
