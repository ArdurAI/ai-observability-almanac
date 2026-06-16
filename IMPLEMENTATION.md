# Implementation Guide

How the observability almanac is built, how to add a tool, how to update an edition, and how the data pipeline works.

## Table of Contents

1. [Repository Structure](#repository-structure)
2. [The Data Pipeline](#the-data-pipeline)
3. [Adding a New Tool](#adding-a-new-tool)
4. [Updating an Edition](#updating-an-edition)
5. [The Roster JSON Schema](#the-roster-json-schema)
6. [Directory Conventions](#directory-conventions)
7. [Building the Adapter](#building-the-adapter)
8. [Automation](#automation)

---

## Repository Structure

```
ai-observability-almanac/
├── README.md                          # Project overview + roster at a glance
├── INTENT.md                          # Philosophy, design principles, governance
├── IMPLEMENTATION.md                  # This file
├── TESTING.md                         # Benchmark methodology, harness details
├── TROUBLESHOOTING.md                 # Common issues, debugging, FAQ
├── CONTRIBUTING.md                    # How to contribute
├── architecture.md                    # Stack architecture + test philosophy
├── SETUP.md                           # How to push to GitHub
├── .gitignore
│
├── editions/                          # Monthly editions
│   └── 2026-06.md                   # Founding edition
│
├── benchmarks/                        # Benchmark results (rolling)
│   └── (populated as results land)
│
├── methodology/
│   └── benchmark-harness.md         # Detailed harness spec for observability tools
│
├── data/
│   └── roster.json                  # Machine-readable catalog (97 tools)
│
├── tools/                             # Per-tool deep-dive pages (placeholder)
│   └── (populated as deep-dives are written)
│
└── assets/                            # Charts, diagrams, screenshots
    └── (populated by editions)
```

## The Data Pipeline

The almanac data flows through four stages:

```
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│  Discovery      │────▶│  Triage         │────▶│  Research       │────▶│  Publication    │
│  (find tools)   │     │  (decide entry) │     │  (deep dive)    │     │  (write edition) │
└─────────────────┘     └─────────────────┘     └─────────────────┘     └─────────────────┘
```

### Stage 1: Discovery

Tools are discovered through:
- **Monthly research swarm**: 8-10 parallel agents search for new observability, evaluation, and monitoring tools
- **Community submissions**: Issues, PRs, email, social media
- **Vendor announcements**: Funding rounds, product launches, major releases (e.g., new LangSmith features, Arize Phoenix updates)
- **GitHub trending**: New repos with significant star growth in observability/evaluation
- **Conference talks**: Papers, demos, blog posts from major events (NeurIPS, ICML, MLOps World)

### Stage 2: Triage

A tool enters the roster if it meets ALL of these criteria:
1. **Seriousness**: Not a toy/demo. Must have a real use case, real users, or real funding. A "trace pretty-printer" with 10 stars does not qualify.
2. **Activity**: Last push or release within 6 months. Exceptions for "stable/mature" tools (e.g., established APM platforms with steady release cadence).
3. **Documentation**: Must have a README, docs, or at least a landing page explaining what it does. Must document how to instrument a model, collect traces, or run evaluations.
4. **Accessibility**: Must be accessible to test (open source, free tier, or evaluation license available). Proprietary tools with no trial path are excluded.
5. **Scope**: Must fit the observability category definition. A general-purpose logging framework doesn't enter unless it has LLM-specific observability features. A generic dashboard tool needs AI-native tracing or evaluation capabilities to qualify.

A tool is **excluded** if:
- It's a fork with no meaningful divergence from the parent
- It's a thin wrapper around another tool with no added value (e.g., a simple OpenTelemetry exporter repackaged)
- It has no users, no community, and no evidence of real-world use
- It requires an enterprise-only license with no evaluation path

### Stage 3: Research

For each new tool, we collect:
- Name, type (Tracing, Evaluation, Drift Detection, Cost Monitoring, APM), license, language, GitHub URL, stars
- Last push date, release cadence
- Key features and differentiators (e.g., async trace support, on-premise deployment, RAGAS integration)
- Known bugs and sharp edges (from smoke gate: missing spans, incorrect metric aggregation, broken async hooks)
- Community health (issues, PRs, maintainer responsiveness)
- Pricing model (per-seat, per-trace, per-eval, open-source)

This data is stored in `data/roster.json` and summarized in the edition.

### Stage 4: Publication

The edition is a markdown file that includes:
- Landscape at a glance table
- Per-subcategory findings and trends (Tracing, Evaluation, Drift Detection, Cost Monitoring)
- New tools added and tools removed
- Notable releases and acquisitions
- Quest diary (what was tested this month: which tools were benchmarked, which adapters were built)

## Adding a New Tool

### Step 1: Verify the tool meets triage criteria

Check: seriousness, activity, documentation, accessibility, scope.

### Step 2: Add to the roster JSON

Edit `data/roster.json` and add the tool to the observability category:

```json
{
  "name": "ToolName",
  "type": "Tracing|Evaluation|Drift Detection|Cost Monitoring|APM",
  "license": "License",
  "region": "Region",
  "tier": "A|B|C",
  "notes": "One-line description and key differentiators (e.g., async trace support, RAGAS native, on-premise collector)"
}
```

**Tier assignment rules**:
- **Tier A**: Market leader, widest adoption, or strongest technical merit in observability. Must be actively maintained and have real production usage tracing LLM applications or running evaluations at scale. Examples: LangSmith, Langfuse, Braintrust, Arize Phoenix.
- **Tier B**: Solid option, actively maintained, but not the market leader. Good for specific use cases (e.g., on-premise-only, specific cloud provider, niche evaluation framework). Examples: Lunary, Comet Opik, New Relic LLM Observability.
- **Tier C**: Niche, early-stage, or specialized. Worth knowing about but not a default choice. Examples: new open-source tracing projects, academic drift detection prototypes, region-specific monitoring tools.

### Step 3: Update the edition

Add the tool to the appropriate subcategory section in `editions/YYYY-MM.md`. If the tool is Tier A, add it to the roster-at-a-glance table in the README.

### Step 4: Update the category README

If the tool is Tier A, update the tier list in the README.

### Step 5: Run the smoke gate

Before the tool is officially "in," it must pass the smoke gate (see TESTING.md). If it fails, document the failure in the edition and assign it to Tier C with a note about the blocker.

## Updating an Edition

### Monthly update checklist

```
□ Check for new tools (discovery phase)
□ Triage new tools (add to roster or reject)
□ Update metadata for existing tools (stars, last push, releases, new pricing tiers)
□ Flag tools for removal (dead/abandoned, no releases in 12+ months)
□ Run smoke gate for new tools
□ Run benchmark updates for re-tested tools (RAGAS suite, DeepEval suite, trace overhead test)
□ Draft the edition markdown
□ Update README roster-at-a-glance
□ Commit and push
```

### Edition markdown template

```markdown
# Edition YYYY-MM — [Title]

*Research conducted YYYY-MM-DD. [Context about this month].*

## The landscape at a glance

| Subcategory | Tool Count | New This Month | Notable Changes |
|-------------|-----------|----------------|-----------------|
| Tracing     | N         | N              | [notes]         |
| Evaluation  | N         | N              | [notes]         |
| Drift Detection | N     | N              | [notes]         |
| Cost Monitoring | N     | N              | [notes]         |

## Tracing — [Theme]

### Tier A roster
[table]

### Findings
[bullets: new features, notable bugs, performance changes]

## Evaluation — [Theme]

### Tier A roster
[table]

### Findings
[bullets: new eval metrics, benchmark results, accuracy shifts]

## Quest diary — [Month] [Year]

- [what was tested: e.g., "Ran RAGAS suite on 12 tracing tools; Langfuse v2.45 added 12% latency overhead"]

## Coming next month

[what's planned: e.g., "DeepEval 0.50 benchmark run; drift detection stress suite"]

## License
Content is licensed CC BY 4.0.
```

## The Roster JSON Schema

```json
{
  "meta": {
    "name": "AI Observability Almanac Roster",
    "version": "YYYY-MM",
    "generated_at": "ISO-8601 timestamp",
    "total_tools": 97,
    "categories": 1,
    "research_method": "description"
  },
  "categories": {
    "observability": {
      "name": "Observability, Logging & Monitoring",
      "description": "Tools for tracing, evaluating, monitoring, and debugging AI/LLM systems including RAG pipelines, agent workflows, and model inference",
      "estimated_total": 150,
      "tools": [
        {
          "name": "Tool Name",
          "type": "Tracing|Evaluation|Drift Detection|Cost Monitoring|APM",
          "license": "License",
          "region": "Region",
          "tier": "A|B|C",
          "notes": "Description"
        }
      ]
    }
  }
}
```

**Field definitions**:
- `name`: The tool's common name. Use the name the tool calls itself (e.g., "Langfuse", not "Langfuse.io").
- `type`: What kind of observability tool is it? (e.g., "Tracing", "Evaluation Framework", "Drift Detection", "Cost Monitoring", "APM")
- `license`: The primary license. Use SPDX identifiers where possible.
- `region`: Where the tool is primarily developed (US, EU, China, Global, etc.)
- `tier`: A, B, or C (see tier rules above)
- `notes`: One-line description with key differentiators. Keep under 100 chars. Mention eval frameworks supported (RAGAS, DeepEval) if applicable.

## Directory Conventions

### `editions/`
- One file per month: `YYYY-MM.md`
- Never delete old editions. The history is part of the record.
- New editions are appended; old editions are never rewritten.

### `data/`
- `roster.json` is the single source of truth for the tool catalog.
- It is machine-generated from the research process.
- It should be valid JSON at all times.

### `benchmarks/`
- One file per benchmark run: `observability-<suite>-<date>.md`
  - Examples: `observability-ragas-2026-06.md`, `observability-trace-overhead-2026-06.md`, `observability-drift-stress-2026-06.md`
- Raw JSON files alongside the markdown: `observability-<suite>-<date>.json`
- Raw data is never deleted. It is the audit trail.

### `tools/`
- One file per tool: `<name>.md`
- Contains deep-dive analysis: setup experience, benchmark results, bug notes, comparison with peers
- Populated as deep-dives are written (not all tools have a page immediately)

### `assets/`
- Images, charts, diagrams referenced by editions and benchmarks
- Named descriptively: `trace-latency-2026-06.png`, `eval-accuracy-2026-06.png`, `drift-detection-roc-2026-06.png`

### `methodology/`
- The benchmark harness specification for observability tools
- Frozen before any results are generated
- Changes require an RFC and a public announcement

## Building the Adapter

When a new tool is added to the roster and is ready for benchmarking, an adapter must be built. The adapter is the bridge between the tool's API and the harness's fixed interface. For observability tools, the adapter type depends on what the tool does.

### The CategoryAdapter contract

```python
class CategoryAdapter:
    def setup(self) -> None:
        """Install, configure, and start the tool (collector, dashboard, eval runner)."""
        pass
    
    def instrument(self, model_or_pipeline) -> None:
        """Instrument the target model, agent, or RAG pipeline with the tool."""
        pass
    
    def load(self, workload) -> None:
        """Ingest the workload (traces, evaluation dataset, baseline metrics)."""
        pass
    
    def await_ready(self) -> None:
        """Wait for async ingestion to complete. Measure lag (trace flush, index ready)."""
        pass
    
    def query(self, query) -> Response:
        """Run the query (retrieve trace, run eval, check drift alert) and return the response."""
        pass
    
    def teardown(self) -> None:
        """Clean up, measure resource usage."""
        pass
```

### Adapter rules

1. The adapter must be **pure** — it should not modify the tool's behavior, only interface with it. If the tool has a custom `trace_callback`, the adapter uses it; it does not rewrite the tool's internals.
2. The adapter must be **documented** — every step should be explainable in plain English. "We call `langfuse.trace()` because that's the documented API for capturing a span."
3. The adapter must be **reproducible** — running it twice on the same machine should produce the same trace structure and metric scores.
4. The adapter must be **isolated** — it should not depend on other tools' adapters. The Langfuse adapter should not import the Braintrust adapter.
5. The adapter code is **published** in the benchmark harness repo (separate from the almanac repo).

### Example adapters (pseudocode)

**Tracing adapter**:
```python
class TracingAdapter(CategoryAdapter):
    def __init__(self, tool_name, config):
        self.tool = tool_name
        self.config = config
    
    def setup(self):
        if self.tool == "langfuse":
            self.client = Langfuse(public_key=..., secret_key=...)
        elif self.tool == "arize":
            self.client = ArizeClient(api_key=...)
    
    def instrument(self, model):
        # Wrap the model call so every invocation creates a trace
        self.original_call = model.call
        model.call = lambda *args, **kwargs: self._trace_call(*args, **kwargs)
    
    def _trace_call(self, *args, **kwargs):
        span = self.client.trace(name="model_call", input=args)
        result = self.original_call(*args, **kwargs)
        span.update(output=result)
        return result
    
    def query(self, trace_id):
        return self.client.get_trace(trace_id)
    
    def teardown(self):
        self.client.flush()
```

**Evaluation adapter**:
```python
class EvaluationAdapter(CategoryAdapter):
    def __init__(self, tool_name, config):
        self.tool = tool_name
        self.config = config
    
    def setup(self):
        if self.tool == "ragas":
            from ragas import evaluate
            self.evaluate = evaluate
        elif self.tool == "deepeval":
            from deepeval import evaluate
            self.evaluate = evaluate
    
    def load(self, dataset):
        self.dataset = dataset  # List of (question, context, answer, ground_truth)
    
    def query(self, metrics):
        # Run the evaluation and return scores per metric
        return self.evaluate(self.dataset, metrics=metrics)
    
    def teardown(self):
        pass
```

**Drift detection adapter**:
```python
class DriftAdapter(CategoryAdapter):
    def __init__(self, tool_name, config):
        self.tool = tool_name
        self.config = config
    
    def setup(self):
        self.detector = DriftDetectorFactory.create(self.tool, self.config)
    
    def load(self, baseline_data):
        self.detector.fit(baseline_data)
    
    def query(self, new_data):
        return self.detector.detect(new_data)  # Returns drift score, alert flag, confidence
    
    def teardown(self):
        pass
```

## Automation

### Monthly update cron

The monthly update is run by a scheduled job:
- **Trigger**: `cron` expression `0 7 15 * *` (monthly, 15th at 7:00 AM)
- **Action**: Runs a research agent to discover new observability tools, update metadata, and draft the next edition
- **Output**: Commits to the repo with the updated roster and new edition

### GitHub Actions (optional)

For automatic metadata refresh (GitHub stars, last push dates, latest releases), a GitHub Actions workflow can be configured:

```yaml
name: Monthly Metadata Refresh
on:
  schedule:
    - cron: '0 7 1 * *'
  workflow_dispatch:
jobs:
  refresh:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Refresh metadata
        run: python scripts/refresh_metadata.py
      - name: Commit and push
        run: |
          git config user.name "github-actions[bot]"
          git config user.email "github-actions[bot]@users.noreply.github.com"
          git add -A
          git commit -m "Monthly metadata refresh: $(date +%Y-%m)" || echo "No changes"
          git push
```

## License

Content: CC BY 4.0  
Code: MIT
