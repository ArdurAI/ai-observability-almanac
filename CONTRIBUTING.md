# Contributing to the Observability Almanac

How to add observability tools, fix data, challenge rankings, and improve the methodology.

## Table of Contents

1. [Ways to Contribute](#ways-to-contribute)
2. [Adding a New Tool](#adding-a-new-tool)
3. [Fixing Data](#fixing-data)
4. [Challenging a Ranking](#challenging-a-ranking)
5. [Improving the Methodology](#improving-the-methodology)
6. [Code of Conduct](#code-of-conduct)
7. [License](#license)

---

## Ways to Contribute

You can contribute to the observability almanac in several ways:

| Contribution Type | What you do | Impact |
|-------------------|-------------|--------|
| **Add a tool** | File an issue with a new observability, evaluation, or monitoring tool | Expands the roster |
| **Fix data** | Correct incorrect metadata (license, type, tier, pricing notes) | Improves accuracy |
| **Challenge a ranking** | Provide evidence that a trace overhead, eval accuracy, or drift detection score is wrong | Drives quality |
| **Share experience** | Write about using an observability tool in production at scale | Adds real-world context |
| **Improve methodology** | Propose a better benchmark, scoring rubric, or eval dataset | Improves fairness |
| **Build an adapter** | Implement the adapter for a new tracing, eval, or drift detection tool | Enables testing |
| **Review an edition** | Proofread, fact-check, suggest improvements | Improves quality |
| **Spread the word** | Share the almanac with your MLOps or platform engineering community | Grows the ecosystem |

## Adding a New Tool

### Before you submit

Check if the tool meets the triage criteria:

1. **Seriousness**: Is it a real tool with real users, not a toy or demo? A "trace pretty-printer" with 10 stars does not qualify.
2. **Activity**: Has it had a push or release in the last 6 months? Observability tools evolve fast — stale projects are excluded.
3. **Documentation**: Does it have a README, docs, or landing page? Must document how to instrument a model, collect traces, run evaluations, or detect drift.
4. **Accessibility**: Is it testable (open source, free tier, or evaluation license)? Proprietary tools with no trial path are excluded.
5. **Scope**: Does it fit the observability category? A generic logging framework needs LLM-specific features (tracing, eval, token tracking) to qualify.

### How to submit

**Option 1: GitHub Issue (preferred)**

File an issue with this template:

```markdown
## Tool Request: [Tool Name]

### Category
Observability — Subcategory: [Tracing / Evaluation / Drift Detection / Cost Monitoring / APM]

### Tool URL
[GitHub repo or homepage URL]

### License
[e.g., MIT, Apache-2.0, Proprietary]

### Description
[What does it do? One paragraph. Be specific: does it trace LLM calls? Run RAG evaluations? Detect model drift?]

### Why it should be on the roster
[Evidence of adoption, production usage, or technical merit. E.g., "Used by 500+ companies," "Integrated with LangChain," "Published at NeurIPS."]

### Evidence
- GitHub stars: [N]
- Last release: [date]
- Notable users: [companies, if known]
- Funding: [amount, if known]
- Eval frameworks supported: [RAGAS, DeepEval, custom, etc.]
- Deployment model: [Cloud-only, Self-hosted, Hybrid]

### Tier suggestion
[A, B, or C — and why]
```

**Option 2: Pull Request**

If you want to add the tool directly:

1. Fork the repo
2. Edit `data/roster.json` to add the tool to the `observability` category
3. Update `README.md` if the tool is Tier A
4. Submit a PR with the same template as above

### What happens after submission

1. **Triage**: We check if the tool meets criteria (within 7 days)
2. **Smoke gate**: We run the tool through the 3-turn scenario: instrument → collect → query (within 14 days)
3. **Decision**: Accepted, rejected, or deferred with a note
4. **Publication**: If accepted, it appears in the next edition

## Fixing Data

### If you find incorrect metadata

File an issue with:

```markdown
## Data Correction: [Tool Name]

### Current (incorrect) data
[What does the roster say?]

### Correct data
[What should it say?]

### Evidence
[Link to the source that proves the correct data. E.g., LICENSE file, release notes, pricing page.]
```

### Common corrections

| Field | Common errors | How to verify |
|-------|--------------|---------------|
| License | Wrong SPDX identifier | Check the repo's LICENSE file |
| Stars | Out of date | Check the GitHub API |
| Last push | Wrong date | Check the GitHub repo |
| Tier | Wrong tier | Check the tier rules in IMPLEMENTATION.md |
| Notes | Outdated description | Check the tool's homepage/docs |
| Type | Wrong subcategory | Check if the tool is primarily tracing, evaluation, drift detection, or cost monitoring |
| Pricing notes | Outdated | Check the vendor's pricing page |

### What happens after submission

Data corrections are reviewed and applied in the next edition cycle. We don't edit editions retroactively; we correct the data and note it in the next edition.

## Challenging a Ranking

### If you believe a score is wrong

File an issue with:

```markdown
## Challenge: [Tool Name] on [Dimension]

### Current score
[What does the almanac say? E.g., "Latency score: 45 (p99 overhead: 120ms)"]

### Your evidence
[What data do you have? E.g., "We measured 8ms p99 overhead in production at 10K RPM."]

### What you did to verify
[Steps you took to reproduce or verify. E.g., "Ran the adapter with langfuse==2.45.0 on an m5.xlarge."]

### Suggested resolution
[What should change? Re-run? Different score? Methodology update?]
```

### What evidence is valid

| Evidence Type | Strength | Example |
|---------------|----------|---------|
| Raw results JSON analysis | Strong | "I re-analyzed the trace JSON and found 30% of spans were dropped due to async callbacks" |
| Independent reproduction | Strong | "I ran the RAGAS harness on the same dataset and got faithfulness=0.82, not 0.91" |
| Documentation of a bug | Medium | "The tool has a known bug where streaming responses are not traced" |
| Production metrics | Medium | "We run this tool at 50K RPM and the p99 overhead is 5ms, not 50ms" |
| Vendor claim | Weak | "The vendor says Z" — but we already test vendor claims |
| Anecdote | Weak | "It worked for me" — not reproducible |

### What happens after submission

1. **Review**: We review the evidence (within 7 days)
2. **Reproduction**: If the claim is reproducible, we re-run the test
3. **Update**: If the re-run confirms the challenge, we update the score
4. **Publication**: The update appears in the next edition

## Improving the Methodology

### If you want to propose a methodology change

File an issue with:

```markdown
## Methodology Proposal: [Title]

### Current state
[What does the methodology say now?]

### Proposed change
[What should it say?]

### Rationale
[Why is this better? What problem does it solve?]

### Impact
[Which tools/subcategories would be affected?]

### Backward compatibility
[Can old results be re-scored with the new method?]
```

### Methodology change process

1. **RFC**: The proposal is posted as an RFC for public comment (30 days)
2. **Discussion**: Community feedback is collected
3. **Decision**: ArdurAI makes the final decision based on feedback
4. **Announcement**: If accepted, a public announcement is made with a transition plan
5. **Implementation**: The change is implemented in the next edition cycle
6. **Re-run**: Affected benchmarks are re-run with the new methodology

### What kinds of changes are accepted

| Change Type | Likelihood | Example |
|-------------|------------|---------|
| Bug fix in harness | High | "The trace adapter incorrectly handles async generators" |
| New benchmark | Medium | "Add a new streaming trace benchmark for token-level spans" |
| New eval dataset | Medium | "Add a multilingual RAG evaluation dataset" |
| Weight adjustment | Medium | "Increase ops burden weight from 15% to 20% for tracing tools" |
| New dimension | Low | "Add a 'compliance' dimension for SOC 2 / GDPR alignment" |
| Remove dimension | Very low | "Remove latency as a dimension" |

### What kinds of changes are rejected

- Changes that favor a specific vendor (e.g., "RAGAS should weight faithfulness higher because Tool X scores well on it")
- Changes that reduce reproducibility (e.g., "Use the vendor's curated dataset instead of the frozen one")
- Changes that increase complexity without clear benefit (e.g., "Add a 50-page manual scoring rubric")
- Changes that are not backward-compatible without a migration plan

## Code of Conduct

### Be respectful

This is a collaborative project. Treat others with respect, even when you disagree about whether LangSmith or Langfuse is better.

### Be evidence-based

Claims should be backed by evidence. "I think X is better" is not enough. "I measured X and found Y" is. "Our production traces show Z" is even better.

### Be constructive

Criticism is welcome if it's constructive. "This is wrong" is not helpful. "This is wrong because the adapter misses async spans, and here's a fix" is.

### Be patient

The almanac is maintained by a small team. Responses may take time. Repeated pings are not helpful.

### No spam

Don't submit the same tool multiple times. Don't submit tools that clearly don't meet criteria (e.g., a generic log viewer with no LLM features). Don't use the almanac for marketing.

## License

By contributing to the almanac, you agree that your contributions are licensed under CC BY 4.0 for content and MIT for code.

## Attribution

Contributors are recognized in the edition notes. If you make a significant contribution (e.g., adding 5+ tools, fixing major data issues, improving methodology, building an adapter for a widely-used tool), you will be listed as a contributor in the next edition.

## License

Content: CC BY 4.0  
Code: MIT
