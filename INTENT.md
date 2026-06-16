# Project Intent & Philosophy

## Why this almanac exists

The AI observability landscape is fractured. Every week, a new tracing tool claims to "solve LLM monitoring," a new evaluation framework promises perfect RAG metrics, and a new cost-tracking platform announces the next revolution in token economics. But **nobody independently verifies these claims**. The benchmarks are self-reported, the comparisons are marketing, and the "best tool" lists are affiliate SEO.

This almanac is the **public record of independent verification** for observability, evaluation, and monitoring tools. It exists because platform engineers need a single source of truth that answers:

- Does this tracing tool actually capture all spans in a production pipeline?
- What's the real overhead of running an evaluation framework on every request?
- How does a drift detector behave when the baseline shifts gradually vs. abruptly?
- What's the total cost per eval run at 100K requests per day?
- Can I trust the vendor's RAGAS score when they designed the benchmark?

## Core principles

### 1. Frozen methodology before results

The harness, judge model, prompts, scoring rubric, and evaluation datasets are **fixed and published before any tool is tested**. This prevents "cherry-picking" the methodology that favors a particular vendor. If a tool doesn't fit the harness, we adapt the adapter — not the rules.

### 2. Ops-first evaluation

Most benchmarks measure accuracy or recall. We measure **what a platform engineer actually lives with**:
- Time from `pip install` to first trace captured
- Dependency conflicts when installing Langfuse alongside Braintrust
- Time to debug why a span is missing from the trace tree
- Upgrade pain when the tracing SDK version N → N+1 breaks async instrumentation
- Cost predictability of running RAGAS on every production query

### 3. Raw data is always published

Every benchmark run produces a JSON file with every trace, every metric, every token count, every latency measurement, and every evaluation score. These raw files are published alongside the summary. If you disagree with a ranking, you can re-analyze the data yourself.

### 4. No tool is above criticism

Every tool on the roster has been through a smoke gate. Every tool has bugs — missing spans, incorrect metric calculations, or broken async support. We document them honestly. A vendor relationship or sponsorship does not influence rankings. The only way a tool improves its score is by actually improving.

### 5. Living document, not a static snapshot

The almanac is updated monthly. Tools enter and exit the roster. Scores change as tools improve or degrade. The "founding edition" is a snapshot; the current edition is the truth.

## Design philosophy

### The two-bar test

Every tool must clear two bars to justify its existence:
1. **Beat the naive baseline** on accuracy/quality/performance (e.g., does the tracing tool capture more than a simple `print()` wrapper? Does the eval framework score more reliably than human spot-checking?)
2. **Beat the full-capability baseline** on cost/ops burden/complexity (e.g., is a full RAGAS suite cheaper than a team of manual evaluators? Is the drift detector less noisy than a simple threshold alert?)

If a tool can't do both, it has no reason to exist as infrastructure. A tool that is 5% more accurate on RAGAS but adds 50ms of latency to every request is not worth adopting.

### The seven dimensions

We score every tool on seven dimensions because no single number captures "good observability infrastructure":

| Dimension | Why it matters for observability |
|-----------|----------------------------------|
| **Accuracy** | Do evaluation metrics (RAGAS, DeepEval, custom) match human judgment? Does drift detection catch real regressions without false positives? |
| **Latency** | What's the tracing overhead per request? Does the eval framework add unacceptable latency to the inference path? |
| **Token economics** | What's the cost per eval run? Does the tool make expensive LLM calls for scoring? Can it run on a local judge model? |
| **Scale behavior** | What happens when you trace 10K concurrent requests? Does the eval framework degrade at 100K rows? |
| **Ops burden** | How much of your life does the tool consume? Setup complexity, dashboard maintenance, alert fatigue? |
| **Developer experience** | Is the trace UI pleasant? Are error messages actionable? Can you export data easily? |
| **Data sovereignty** | Can you run the trace collector on-premise? Can you audit eval scores? Does the tool phone home? |

### The adapter pattern

Every tool is tested through a **CategoryAdapter** — a frozen interface that the tool must satisfy. The adapter handles setup, ingestion, querying, and teardown. For observability tools, this means:
- **Tracing adapters**: instrument a model, collect spans, query the trace tree
- **Evaluation adapters**: load a dataset, run metrics (RAGAS, DeepEval, custom), retrieve scores
- **Drift detection adapters**: establish baseline, inject drift, query alerts, measure detection latency
- Tools are tested identically; the adapter is the only thing that changes per tool
- New tools can be added without changing the harness
- The adapter is published and open for review

### The canary

Every benchmark batch starts with a **no-tool baseline** (the "canary"). If the benchmark leaked answers anywhere, the canary would score above zero. The canary must score exactly zero — this is a hard invariant. If it doesn't, the entire batch is invalid.

## Who this is for

- **Platform engineers** evaluating which observability tool to adopt for tracing, evaluation, or monitoring
- **CTOs/CIOs** making build-vs-buy decisions on LLM observability with actual data
- **ML engineers** choosing between RAGAS, DeepEval, and custom evaluation frameworks
- **Open-source maintainers** who want independent benchmarking of their tracing or eval project
- **Researchers** studying the AI observability and evaluation landscape
- **Vendors** who want to improve their tools based on real evidence

## What this is NOT

- Not a marketing site for any vendor (LangSmith, Langfuse, Arize, etc.)
- Not a "best of" list based on GitHub stars or funding rounds
- Not a tutorial on how to use any observability tool
- Not a replacement for your own due diligence on trace retention policies or SOC 2 compliance
- Not a static document that never changes

## The "Quest"

The "Observability Engineer's Quest for the Best" is the ongoing effort to test, measure, and rank every tool on the roster. It's not a one-time effort. It's a continuous process of:
1. **Discovery** — finding new tools via research, community, and submissions
2. **Triage** — deciding if a tool is serious enough to enter the roster
3. **Smoke gate** — running every tool through an identical 3-turn scenario: instrument model → collect trace → query metrics
4. **Benchmark** — running standard eval suites (RAGAS, DeepEval) + custom stress tests + drift detection scenarios
5. **Publication** — publishing raw data + summary + per-tool deep-dives
6. **Iteration** — re-testing as tools update, as methodology improves, as new benchmarks emerge

## How to challenge a result

If you believe a ranking or score is wrong:
1. Check the **raw results JSON** — the trace data, metric scores, and evaluation outputs are public
2. Check the **adapter implementation** — the adapter code that instruments the tool is public
3. Check the **judge prompts** — the prompts used for LLM-as-judge scoring are frozen and public
4. File an issue with a specific claim and evidence
5. We'll re-run the test or update the methodology if warranted

## Governance

- **ArdurAI** maintains the almanac and runs the Quest
- **Methodology changes** require a public RFC and at least one edition cycle of notice
- **Tool additions/removals** are decided by the triage criteria (stars, last push, community activity, seriousness)
- **Benchmark results** are machine-generated; summaries are human-reviewed for fairness
- **Conflicts of interest** are disclosed (e.g., ArdurAI contributes to some tools on the roster); mitigation is identical harness for all

## License

Content: CC BY 4.0  
Harness code: MIT  
Raw data: CC BY 4.0
