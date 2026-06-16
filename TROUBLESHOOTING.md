# Troubleshooting & Debugging

How to understand the observability almanac codebase, debug issues, and resolve common problems when working with the almanac.

## Table of Contents

1. [Understanding the Codebase](#understanding-the-codebase)
2. [Common Issues](#common-issues)
3. [Debugging the Data Pipeline](#debugging-the-data-pipeline)
4. [Debugging Benchmark Runs](#debugging-benchmark-runs)
5. [FAQ](#faq)
6. [Getting Help](#getting-help)

---

## Understanding the Codebase

### High-level flow

```
Research Agents → Research Output (Markdown) →
  Python Script → roster.json (Structured Data) →
    Manual Review → Edition Markdown →
      Git Commit → GitHub Publication
```

### Key files and their roles

| File | Role | When to read it |
|------|------|-----------------|
| `README.md` | Project overview, quick reference | First thing you read |
| `INTENT.md` | Philosophy, why we do things this way | When you disagree with a decision |
| `IMPLEMENTATION.md` | How things are built, how to add tools | When you want to contribute |
| `TESTING.md` | Benchmark methodology, scoring, harness details | When you want to reproduce or challenge a result |
| `TROUBLESHOOTING.md` | This file | When something is broken |
| `architecture.md` | Stack architecture diagram | When you want to understand the big picture |
| `editions/YYYY-MM.md` | Monthly snapshot of the observability landscape | When you want historical data |
| `data/roster.json` | Machine-readable catalog of 97 observability tools | When you want to query or analyze the data |
| `methodology/benchmark-harness.md` | Harness specification for observability tools | When you want to build an adapter or run benchmarks |
| `benchmarks/` | Benchmark results (JSON + markdown) | When you want raw trace data, eval scores, or drift detection results |

### The data model

The almanac is fundamentally a **directed graph** of data:

```
Research findings → Tool metadata → Roster JSON → Edition Markdown → README
                                      ↓
                               Benchmark results → Per-tool pages
```

- **Research findings** are the raw output of the research swarm. They're saved in `research/` (not in the public repo).
- **Tool metadata** is extracted from research and stored in `data/roster.json`.
- **Roster JSON** is the single source of truth. Everything else derives from it.
- **Edition markdown** is human-written based on the roster and research.
- **README** is auto-generated from the roster and the latest edition.

### Understanding `data/roster.json`

This is the most important file in the repo. It is the single source of truth.

**Structure**:
```json
{
  "meta": { ... },
  "categories": {
    "observability": {
      "name": "Observability, Logging & Monitoring",
      "description": "...",
      "estimated_total": 150,
      "tools": [
        { "name": "...", "type": "Tracing|Evaluation|Drift Detection|Cost Monitoring|APM", "license": "...", "tier": "A|B|C", "notes": "..." }
      ]
    }
  }
}
```

**How to query it**:
```bash
# Find all Tier A tracing tools
jq '.categories.observability.tools[] | select(.tier == "A" and .type == "Tracing") | .name' data/roster.json

# Count tools by tier
jq '.categories.observability.tools | group_by(.tier) | map({tier: .[0].tier, count: length})' data/roster.json

# Find all MIT-licensed evaluation frameworks
jq '.categories.observability.tools[] | select(.license == "MIT" and .type == "Evaluation") | .name' data/roster.json

# Find all on-premise-capable tools (search notes)
jq '.categories.observability.tools[] | select(.notes | contains("on-premise")) | .name' data/roster.json
```

### The edition markdown

Editions are **human-written** summaries, not machine-generated. They are based on the roster but include analysis, interpretation, and narrative that a machine can't produce.

**How editions are structured**:
1. Front matter: date, research method, context
2. Landscape at a glance: summary table by subcategory (Tracing, Evaluation, Drift Detection, Cost Monitoring)
3. Per-subcategory sections: findings, roster, analysis
4. Quest diary: what was tested this month (which eval suites, which trace overhead tests, which drift detection runs)
5. Cross-category findings: patterns that span subcategories (e.g., "all tracing tools added async support this month")

### The benchmark harness (separate repo)

The actual benchmark code lives in a separate repository. The almanac repo contains:
- The methodology specification
- The results (JSON + markdown)
- The adapter interface definitions

The harness repo contains:
- The runner code
- The adapter implementations (Langfuse, LangSmith, Braintrust, RAGAS, DeepEval, etc.)
- The judge model integration
- The telemetry collector
- The trace completeness analyzer
- The eval correlation engine
- The drift detection validator

**Why separate?** Because the harness is code that runs, and the almanac is data that is published. They have different lifecycles and different audiences.

## Common Issues

### Issue: `roster.json` is invalid JSON

**Symptoms**:
- `jq` fails to parse it
- GitHub Actions fails on JSON validation
- Python `json.load()` raises `JSONDecodeError`

**Diagnosis**:
```bash
python3 -c "import json; json.load(open('data/roster.json'))"
```

**Resolution**:
1. Find the line with the error: `python3 -m json.tool data/roster.json`
2. Common causes: trailing commas, unescaped quotes, incorrect nesting
3. Fix the JSON and re-validate
4. Consider using a JSON linter in your editor

### Issue: Edition markdown has broken links

**Symptoms**:
- Links to tools return 404
- Links to benchmarks don't exist yet
- Relative links work locally but break on GitHub

**Diagnosis**:
```bash
# Check all links in the repo
find . -name "*.md" -exec grep -oP '\[.*?\]\(.*?\)' {} + | grep -v "http" | grep -v "mailto"
```

**Resolution**:
1. For internal links, use relative paths: `../data/roster.json`
2. For external links, verify the URL is correct
3. For tools without a per-tool page yet, link to their homepage or GitHub repo
4. Run a link checker as part of CI

### Issue: Tier assignment is wrong

**Symptoms**:
- A tool is Tier A but has no production usage (e.g., a new tracing tool with 50 stars)
- A tool is Tier C but is widely adopted (e.g., a well-known APM with thousands of users)
- A tool's tier changed without explanation

**Diagnosis**:
1. Check the tier assignment rules in `IMPLEMENTATION.md`
2. Verify the tool's adoption, activity, and community health
3. Check the edition notes for the rationale

**Resolution**:
1. File an issue with evidence (GitHub stars, last push, production references, eval framework citations)
2. The tier will be reviewed in the next edition cycle
3. Tiers are not changed mid-edition; they are updated at edition boundaries

### Issue: Benchmark results can't be reproduced

**Symptoms**:
- Running the harness produces different RAGAS scores
- The adapter fails with a different tool version (e.g., Langfuse SDK v2.50 vs. v2.45)
- The judge model is unavailable or has changed behavior
- Trace overhead measurements differ significantly

**Diagnosis**:
1. Check the `results.json` metadata for the exact commit, seed, hardware, and tool version
2. Check if the tool version has changed since the published run (trace SDKs update frequently)
3. Verify the judge model is accessible and hasn't been deprecated
4. Check if the async environment differs (e.g., event loop implementation)

**Resolution**:
1. Use the exact commit and dependencies from the results metadata
2. If the tool version changed, the results are for a different version — this is expected; check the edition notes for version pinning
3. If the judge model changed, that's a methodology issue — file an issue
4. If trace overhead differs, check if the test model or framework version changed (different frameworks produce different call patterns)

### Issue: Research agent missed a tool

**Symptoms**:
- A well-known observability tool is not in the roster (e.g., a popular new tracing platform)
- A tool from a specific region is missing
- A newly launched evaluation framework is not in the latest edition

**Diagnosis**:
1. Check if the tool meets triage criteria in `IMPLEMENTATION.md`
2. Check if it was added in a previous edition and later removed
3. Check if it falls outside the search scope (e.g., a generic logging tool without LLM-specific features)

**Resolution**:
1. File an issue with the tool name, URL, and evidence of adoption/activity
2. The tool will be triaged for the next edition
3. If it meets criteria, it will be added

### Issue: Monthly update cron failed

**Symptoms**:
- No new edition was published on the 15th
- The cron job is missing from the scheduler
- The research agent timed out

**Diagnosis**:
```bash
# Check cron status
cron status

# Check the cron job list
# (use the Kimi Work cron interface)
```

**Resolution**:
1. Check if the cron job is still registered
2. Check if the research agent timed out (increase timeout if needed — observability research often surfaces 100+ tools)
3. Manually trigger the update if the cron missed a cycle
4. Check the workspace path in the cron job configuration

### Issue: GitHub push fails

**Symptoms**:
- `git push` returns 403 or 401
- The remote is not configured
- The branch is behind origin

**Diagnosis**:
```bash
git remote -v
git status
git log --oneline -5
```

**Resolution**:
1. Verify the remote URL is correct: `git remote set-url origin https://github.com/ArdurAI/...`
2. Verify GitHub CLI auth: `gh auth status`
3. If behind origin, pull first: `git pull origin main`
4. If there are conflicts, resolve them manually

## Debugging the Data Pipeline

### Research output → roster.json

**Problem**: Research agents produce markdown, but the roster JSON is incomplete or wrong.

**Debug steps**:
1. Read the research output files in `research/` (local workspace, not in the repo)
2. Check if the Python extraction script correctly parsed the tool tables (especially multi-type tools like "Tracing + Evaluation")
3. Check if tools were dropped during triage (check the triage log)
4. Verify the JSON schema is correct

**Common bugs**:
- Tool names with special characters break JSON parsing → Escape them properly
- Tools with no tier get dropped → Default to Tier C if unsure
- Tools with no notes get empty strings → Add a minimal note (e.g., "LLM tracing and evaluation platform")
- Multi-type tools get assigned only one type → Use the primary type or split into separate entries if significantly different products

### roster.json → edition markdown

**Problem**: The edition doesn't reflect the roster.

**Debug steps**:
1. Compare the tool counts in the roster vs. the edition by subcategory
2. Check if the edition was written before the roster was updated
3. Check if tools were manually edited in the edition but not in the roster

**Resolution**:
1. The edition should be derived from the roster, not the other way around
2. If the edition has manual additions, ensure they are also in the roster
3. The edition is a human-readable summary; the roster is the source of truth

### Edition markdown → README

**Problem**: The README roster-at-a-glance doesn't match the latest edition.

**Debug steps**:
1. Check which edition is referenced in the README
2. Check if the README was updated after the edition was published

**Resolution**:
1. The README should always reference the latest edition
2. Update the README when a new edition is published
3. Consider automating README updates from the roster JSON

## Debugging Benchmark Runs

### The adapter fails

**Symptoms**:
- `setup()` crashes (e.g., can't connect to trace collector)
- `instrument()` throws an exception (e.g., callback signature mismatch)
- `query()` returns nothing or garbage (e.g., trace not found, eval score is NaN)

**Debug steps**:
1. Run the adapter in isolation (without the harness)
2. Check the tool's documentation for setup requirements (e.g., "create a project in the dashboard first")
3. Check if environment variables are set (API keys, collector endpoints)
4. Check if the tool version matches what the adapter expects (SDKs change callback signatures frequently)
5. Check if the async instrumentation is compatible with the test framework's event loop

**Common fixes**:
- Missing Docker daemon for self-hosted collector → Start Docker
- Missing API key → Set the environment variable (e.g., `LANGFUSE_PUBLIC_KEY`)
- Wrong SDK version → Update the adapter or pin the version in `requirements.txt`
- Dependency conflict → Use a virtual environment or container (e.g., LangSmith and Langfuse SDKs may conflict)
- Async instrumentation not firing → Check if the model uses `asyncio` and the SDK supports it

### The canary fails

**Symptoms**:
- The no-tool baseline scores above zero on eval metrics
- The abstention score is not 1.000 for drift detection
- The canary trace has spans (it should be empty)

**Debug steps**:
1. Check if the benchmark workload has leaked answers (e.g., ground truth embedded in the prompt)
2. Check if the grading pipeline has a bug (e.g., the judge model is not using the frozen prompt)
3. Check if the random seed was set
4. Check if the canary adapter is truly a no-op (no hidden imports or environment side effects)

**Resolution**:
1. If the workload leaked answers, redesign the workload
2. If the grader has a bug, fix the grader and rerun all tests
3. This is a critical failure — the entire batch is invalid

### Results are inconsistent

**Symptoms**:
- Same tool, same test, different RAGAS scores across runs
- Trace overhead varies by more than the confidence interval
- Drift detection AUC changes between runs

**Debug steps**:
1. Check if the tool has non-deterministic behavior (e.g., eval framework uses temperature > 0, async timing varies, sampling rate not fixed)
2. Check if the hardware was different between runs (CPU contention affects trace overhead)
3. Check if the tool version changed between runs (common with rapidly evolving tracing SDKs)
4. Check if the judge model changed (model updates can alter eval scores)
5. Check if trace sampling is enabled (some tools sample traces by default, causing inconsistent completeness scores)

**Resolution**:
1. Set temperature to 0 for all LLM calls in the adapter and eval framework
2. Record hardware specs and disable CPU throttling in the results metadata
3. Pin tool versions and record them (e.g., `langfuse==2.45.0`)
4. Disable trace sampling during benchmarks to ensure deterministic completeness
5. Use the same judge model version and record it

## FAQ

### Q: Why is tool X not in the roster?

A: Either it doesn't meet triage criteria, it hasn't been discovered yet, or it was removed for inactivity. File an issue with evidence and we'll triage it. Note: generic logging or monitoring tools without LLM-specific features (tracing, RAG eval, token cost tracking) are out of scope.

### Q: Why did tool X's score change?

A: Either the tool was updated (e.g., Langfuse v2.45 → v2.50), the methodology was refined (e.g., new RAGAS metric added), or we found a bug in our previous test. All three are valid reasons. Check the edition notes for the rationale.

### Q: Can I run the benchmarks myself?

A: Yes. The harness is published separately. Clone it, install dependencies, and run the adapter for the tool you want to test. See `TESTING.md` for reproducibility instructions. You'll need API keys for cloud-hosted tools and Docker for self-hosted collectors.

### Q: How do I challenge a ranking?

A: File an issue with specific evidence. Check the raw results JSON (trace data, eval scores, drift detection results), the adapter code, and the judge prompts. If you find a real problem, we'll re-run or update the methodology.

### Q: Can I add a tool to the roster?

A: Yes. See `CONTRIBUTING.md` for instructions. The tool must meet triage criteria and pass the smoke gate (instrument → collect → query). If it's an eval framework, it must run on our standard RAGAS or DeepEval dataset.

### Q: Why are there separate category almanacs?

A: Each category is deep enough to warrant its own dedicated repo with per-tool pages, category-specific benchmarks, and focused community. The parent almanac is the master catalog. Observability alone has 97+ tools — it needs its own space.

### Q: How often are benchmarks re-run?

A: Standard: every quarter for each subcategory (Tracing, Evaluation, Drift Detection, Cost Monitoring). Stress: annually. Integration: quarterly. If a tool releases a major version (e.g., Langfuse v3.0, RAGAS v1.0), we may re-run early.

### Q: What's the difference between Tier A, B, and C?

A: Tier A = market leader or strongest technical merit in observability. Tier B = solid option, specific use cases (e.g., on-premise only, specific cloud provider). Tier C = niche, early-stage, or specialized. See `IMPLEMENTATION.md` for full rules.

### Q: Can vendors sponsor the almanac?

A: No. The almanac is independently funded. Sponsorship would compromise the core mission. Vendors can improve their scores by actually improving their tools (lower trace overhead, better eval accuracy, faster setup).

### Q: Why does trace overhead vary so much between tools?

A: Trace overhead depends on the SDK's implementation (synchronous vs. asynchronous buffering), the network latency to the collector, and whether the tool blocks the request path. We measure p50/p95/p99 across 1,000 requests to capture this variance. Some tools buffer traces locally and flush asynchronously (low overhead), while others block on every span (high overhead).

### Q: Why are eval scores sometimes inconsistent with the vendor's published numbers?

A: Vendors often report scores on their own curated datasets, with tuned prompts, or on specific model versions. We run on a frozen dataset with frozen judge prompts and a fixed model. If the scores differ, we report both and explain why. This is the whole point of independent verification.

### Q: How do you handle tools that require cloud dashboards?

A: We test cloud-hosted tools with their free tier or evaluation license. We document any limitations (e.g., trace retention limits, eval run quotas). If a tool has no free tier and no evaluation path, it is excluded from the roster.

## Getting Help

### File an issue

GitHub issues are the primary support channel. Use the appropriate template:

- **Tool request**: "Add [Tool Name] to Observability"
- **Data correction**: "[Tool Name] metadata is wrong: [what's wrong]"
- **Benchmark challenge**: "Challenge [Tool Name] ranking on [Dimension]: [evidence]"
- **Bug report**: "[Bug description] in [file/process]"
- **Feature request**: "[Feature description] for [use case]"

### Discussion

GitHub Discussions are for:
- General questions about the almanac
- Sharing experiences with tools on the roster (e.g., "We run Langfuse at 50K RPM and here's what we learned")
- Proposing methodology changes (e.g., "Should we add a new drift detection benchmark?")
- Community announcements

### Email

For private or sensitive inquiries: Use the contact info in the ArdurAI org profile.

## License

Content: CC BY 4.0
