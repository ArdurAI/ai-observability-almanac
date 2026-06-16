# Benchmark Methodology

How the AI Infrastructure Almanac tests tools across all eight categories. The same philosophy everywhere: **frozen methodology, independent verification, ops-first evaluation**.

## Core principle

A tool must clear two bars to justify its existence:
1. **Beat the naive baseline** on accuracy / quality / performance
2. **Beat the full-capability baseline** on cost / ops burden / complexity

If it can't do both, it has no reason to exist as infrastructure.

## The seven scored dimensions

Every tool is scored across seven dimensions, each 0-100:

| Dimension | What it measures | How it's tested |
|-----------|-----------------|-----------------|
| **Accuracy / Quality** | Does it produce correct, useful, safe outputs? | Standard benchmarks + custom PlatformOps suites |
| **Latency** | Time to first result, throughput, tail latency | Instrumented measurements; p50, p95, p99 |
| **Token Economics** | Cost per unit of work, pricing predictability | Standardized workloads; $/1K requests, $/1M tokens |
| **Scale Behavior** | What happens at 10x, 100x load? | Load tests; saturation curves; degradation points |
| **Ops Burden** | Time to first result, dependency conflicts, upgrade pain | Measured setup time; smoke-gate sweep; dependency matrix |
| **Developer Experience** | Documentation quality, error messages, debugging, community | Structured rubric; community health metrics |
| **Data Sovereignty** | Self-hosting viability, audit trails, compliance alignment | Feature matrix; EU AI Act / GDPR / SOC 2 mapping |

## The three benchmark types

### Type 1: Standard benchmarks (comparability)

Run every tool through published benchmark suites to verify vendor claims.

| Category | Standard Benchmark | What it tests |
|----------|-------------------|-------------|
| Code Editors | SWE-bench, Exercism, Terminal-Bench | Code generation correctness |
| Agent Frameworks | GAIA, WebArena, SWE-bench | Agent task completion |
| Observability | RAGAS, DeepEval, custom trace fidelity | Evaluation accuracy, trace completeness |
| Vector Databases | ANN-Benchmarks, BEIR, MS MARCO | Retrieval accuracy, latency |
| Model Serving | LLMPerf, AnyScale serving benchmark | Throughput, TTFT, TPOT |
| Security & Guardrails | OWASP LLM Top 10, Gandalf, custom injection | Defense effectiveness, false positive rate |
| LLMOps Platforms | Custom workflow reliability, prompt regression | Workflow execution, prompt versioning |
| Context & Protocols | MCP compliance test, A2A interop | Protocol conformance, security posture |

Every ranking ships a **published vs. reproduced** table.

### Type 2: PlatformOps custom benchmarks (ops reality)

Custom benchmarks that test how a platform engineer actually lives with the tool:

- **Setup experience**: Time from `git clone` to first working result. Dependency count. Conflict resolution time.
- **Smoke gate**: Identical 3-turn scenario across all tools. Store → search → export → wipe. Every tool must pass without bugs.
- **Stress suite**: Contradiction storms, near-duplicate floods, concurrent writers, kill-the-backing-store chaos, cost-runaway measurement.
- **Upgrade path**: Can you upgrade from version N to N+1 without rewriting everything?
- **Debugging experience**: When it fails, can you find out why in <5 minutes?

### Type 3: Cross-category integration tests

Test how tools from different categories work together:
- Agent framework + vector DB + observability: Does the full stack trace through?
- Code editor + agent framework: Can the editor's agent harness call the framework?
- Security layer + model serving: Do guardrails add <50ms latency?
- Protocol layer + all categories: MCP server compliance, A2A interoperability.

## The harness architecture

```
┌─────────────────────────────────────────┐
│  CategoryAdapter (frozen contract)        │
│  ├── setup()   → install, configure        │
│  ├── load()    → ingest workload           │
│  ├── await_ready() → async barrier         │
│  ├── query()   → run test, get response   │
│  └── teardown() → cleanup, measure        │
└─────────────────────────────────────────┘
              ↓
┌─────────────────────────────────────────┐
│  Telemetry Collector                      │
│  ├── latency (p50/p95/p99)               │
│  ├── token count & cost                    │
│  ├── memory & CPU usage                   │
│  ├── error rate & failure mode taxonomy   │
│  └── ops notes (setup time, deps, bugs)   │
└─────────────────────────────────────────┘
              ↓
┌─────────────────────────────────────────┐
│  Grading Pipeline                         │
│  ├── Deterministic grader (exact match)   │
│  ├── LLM judge (frozen prompts, SHA-256)   │
│  ├── Second pass (confidence < 0.7)        │
│  └── Failure mode taxonomy               │
└─────────────────────────────────────────┘
              ↓
┌─────────────────────────────────────────┐
│  Results Publisher                        │
│  ├── Raw JSON (per question, per run)     │
│  ├── Summary tables (per tool)            │
│  ├── Cross-verification analysis          │
│  └── Insight extraction                   │
└─────────────────────────────────────────┘
```

## The canary

The first run of every batch is the **no-tool baseline** through the identical pipeline. If the benchmark leaked answers anywhere, it would score above zero. The canary must score exactly zero on answerable categories and exactly the abstention rate on adversarial categories.

## Judge model freeze

Before any tool runs:
- Judge model is pinned (e.g., `claude-opus-4-8`)
- Judge prompts are SHA-256-frozen
- Control variables are fixed (same tool-internal LLM everywhere configurable)
- Random seeds are fixed
- Sampling policy is documented

## Update cadence

- **Monthly**: Roster metadata refresh (GitHub stars, last push, new releases)
- **Quarterly**: Benchmark results release per category
- **Annually**: Methodology review and revision (with backward-compatible raw data)

## Disclosure

ArdurAI contributes to some tools on the roster. Mitigation: identical harness for every tool, methodology frozen and published before results, raw data always published.

## License

Methodology and harness code are licensed MIT. Benchmark data is licensed CC BY 4.0.
