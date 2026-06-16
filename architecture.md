# Architecture: Observability, Logging & Monitoring

How the observability, logging & monitoring landscape is shaped, and how the Quest tests it.

## The landscape at a glance

| Tool | Tier | License | Focus | Notes |
|------|------|---------|-------|-------|
| LangSmith | A | Proprietary | Observability + Evaluation | 1B+ events/day; ~35% Fortune 500; LangChain native |
| Langfuse | A | MIT (core) | LLM Engineering Platform | 21K+ stars; self-hosted; acquired by ClickHouse Jan 2026 |
| Braintrust | A | Proprietary | Eval-first Observability | $124M raised; Notion/Stripe/Vercel; CI/CD quality gates |
| Arize Phoenix | A | Elastic-2.0 | AI Observability + Evaluation | 2.5M+ monthly downloads; RAG debugging; local-first OTel |
| Arize AX | A | Proprietary | Enterprise AI Monitoring | Production-scale telemetry; 50+ research-backed eval metrics |
| Helicone | A | Apache-2.0 (partial) | LLM Gateway + Observability | YC W23; proxy-based; one-line integration; 100+ models |
| Pydantic Logfire | A | Proprietary | AI-native Observability | 10M free spans/mo; $2/M after; OTel-native; SQL-queryable |
| Portkey | A | Partially open | AI Gateway + Observability | 1T tokens/day; 250+ models; 20-40ms latency; MCP Gateway |
| OpenLIT | A | Apache-2.0 | AI Engineering Platform | LLM + GPU + VectorDB observability; 60+ integrations; self-h |
| Datadog LLM Observability | A | Proprietary | APM Extension | Unified infra + LLM tracing; agentless; OTel GenAI support |
| Weights & Biases Weave | A | Proprietary | LLM Tracing + Eval | ML + LLM unified; $50/user/mo; auto-instruments major framew |
| TruLens | A | MIT | RAG Evaluation Framework | RAG Triad: groundedness, relevance, correctness; OTel traces |
| AgentOps | A | Partially open | Agent Observability | Agent session lifecycles; tool calls; replay; CrewAI/AutoGen |
| Galileo | A | Proprietary | AI Evaluation + Guardrails | $68M raised; Luna-2 evaluators; sub-200ms scoring; runtime g |
| DeepEval | A | Apache-2.0 | LLM Evaluation Framework | 50+ metrics; CI-native; pytest integration; RAG/agent/multi- |
| Ragas | A | Open source | RAG Evaluation | Faithfulness, context precision/recall; research-backed; pai |
| Promptfoo | A | MIT | Eval + Red Teaming | YAML-based assertions; 50+ vulnerability types; GitHub Actio |
| Fiddler | A | Proprietary | AI Governance + Monitoring | $100M raised; explainability, bias, compliance; ML+LLM unifi |
| Arthur AI | A | Proprietary | Model Performance + Governance | $60M raised; risk, bias, drift; compliance dashboards |
| LiteLLM | A | Open source | LLM Gateway + Proxy | 18K+ stars; 100+ LLM APIs in OpenAI format; YC W23 |

## How the Quest tests a tool

Same harness for all entries; the judge was frozen before any tool ran:

```
Adapter[frozen CategoryAdapter contract]
  ├── setup()    → install, configure
  ├── load()     → ingest workload
  ├── await_ready() → async barrier
  ├── query()    → run test, get response
  └── teardown() → cleanup, measure
       ↓
Telemetry: latency · tokens · $ · ops notes
       ↓
Grading: deterministic + LLM judge (frozen prompts)
       ↓
Raw results JSON (published)
```

The `await_ready()` barrier is where async designs get their cost measured instead of hidden.

## License

Content is licensed CC BY 4.0 — share and adapt with attribution to **ArdurAI / Observability, Logging & Monitoring Almanac**.
