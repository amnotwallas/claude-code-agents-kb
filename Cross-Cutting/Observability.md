---
tags: [claude-code, observability, monitoring, logging, mlflow, tracing]
type: cross-cutting
related:
  - "[[Loop-Engineering]]"
  - "[[Graph-Engineering]]"
  - "[[Harness-Engineering]]"
  - "[[SDLC-with-AI]]"
  - "[[Context-Engineering]]"
  - "[[Six-Pillars-of-Agents]]"
created: 2026-08-02
status: stable
---

# Observability

> **Source**: notes from *AI Builders Meetup Guadalajara #1*, hosted by Wizeline (Guadalajara, 2026-08-02).

Covers half of Pillar 6 (observe) of the [[Six-Pillars-of-Agents|6 pillars of agents]] — the other half, governance, lives in [[Security]].

## Why AI Observability Is Different

Traditional systems are deterministic: the same input always produces the same output — an error reproduces and gets debugged with a breakpoint.

LLM-based systems are **probabilistic**: the same input can produce slightly different outputs across runs. You need to observe **behavior distributions**, not just individual cases. You can't "debug" an LLM with a traditional breakpoint — you need full logs of all the context that produced a decision, to be able to reconstruct why the model responded that way.

## The 3 Pillars Adapted to AI

### Logs

What to log: full prompt, full completion, tool calls with their arguments and responses, timestamps for each step.

**Structured logging**: JSON with standardized fields.

```json
{
  "session_id": "sess_8f3a2b",
  "timestamp": "2026-08-02T14:32:10Z",
  "model": "claude-sonnet-5",
  "tokens_in": 1450,
  "tokens_out": 320,
  "latency_ms": 2100,
  "tool_calls": [
    {"name": "read_file", "args": {"path": "app/main.py"}, "duration_ms": 15}
  ],
  "error": null
}
```

### Metrics

- **Latency**: p50, p95, p99 per call type — the average hides outliers that affect the real experience
- **Tokens**: average and distribution per call, cost trend over time
- **Error rate**: classified by type (`timeout`, `max_tokens`, `safety_filter`, `tool_error`) — each type requires a different response
- **Eval scores**: accuracy, hallucination rate, task completion rate — these come from the harness (see [[Harness-Engineering]]) but should also be monitored in production, not just in CI
- **Cost per task**: the most important metric for business optimization — without this, you can't justify decisions about which model or architecture to use

### Traces

- Distributed traces for [[Loop-Engineering|loops]] and [[Graph-Engineering|graphs]] in multi-agent systems — each loop step or graph node is a span
- OpenTelemetry for instrumenting Python agents in a standard, stack-compatible way
- Connect agent traces with backend system traces (e.g., the agent call that triggers a slow DB query should show up in the same trace)

## Tooling: Recommended Observability Stack

- **MLflow**: experiment tracking, eval logging, model registry (see the code example in [[Harness-Engineering]])
- **LangFuse / LangSmith**: LLM-specific observability — traces and evals with a UI specialized for inspecting full conversations
- **OpenTelemetry**: standard instrumentation for integrating agent telemetry with the rest of the existing stack
- **Grafana + Prometheus**: operational dashboards for cost, latency, errors — for the ops team that already lives in that stack

Example of connecting all of them in a FastAPI architecture:

```python
from opentelemetry import trace
from opentelemetry.exporter.otlp.proto.http.trace_exporter import OTLPSpanExporter
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.trace.export import BatchSpanProcessor

provider = TracerProvider()
provider.add_span_processor(BatchSpanProcessor(OTLPSpanExporter()))
trace.set_tracer_provider(provider)
tracer = trace.get_tracer("agent-service")

async def run_agent_step(session_id: str, user_input: str):
    with tracer.start_as_current_span("agent.step") as span:
        span.set_attribute("session_id", session_id)
        span.set_attribute("input.length", len(user_input))

        with tracer.start_as_current_span("agent.tool_call") as tool_span:
            result = await call_tool("read_file", {"path": "app/main.py"})
            tool_span.set_attribute("tool.name", "read_file")
            tool_span.set_attribute("tool.duration_ms", result.duration_ms)

        return result
```

## Cost Monitoring: The Forgotten Pillar

- Set up budget alerts **before** deploying to production, not after the first surprise bill
- Key metrics: cost per user, per session, per task type — lets you identify which flows are disproportionately expensive
- Cost-reduction strategies: caching repeated responses, prompt compression (see [[Context-Engineering]]), model routing (using a smaller model for simple tasks)
- Example alert: "if daily cost exceeds $X, notify the team and throttle non-critical features"

## Alerts and Anomaly Detection

- **Degraded behavior**: metrics that shift persistently (not just a single spike) usually indicate a real problem, not noise
- **Hallucination spikes**: detect them with automated evals running continuously over a sample of real traffic, not just in CI
- **Infinite loops**: detect sessions with an abnormally high number of tool calls relative to baseline — a typical sign of a stuck agent (see [[Loop-Engineering]] for the flow control that should prevent this in the first place)
- **Latency spikes**: detect and alert before the user reports slowness, not after

## Complete Example

Observability setup for a FastAPI agent with MLflow + OpenTelemetry + Grafana:

```python
import mlflow
import time
from fastapi import FastAPI

app = FastAPI()
mlflow.set_tracking_uri("http://mlflow-server:5000")

@app.post("/agent/run")
async def run_agent(payload: dict):
    start = time.monotonic()
    with mlflow.start_run(run_name="production-agent-call"):
        try:
            result = await execute_agent_task(payload["input"])
            latency_ms = (time.monotonic() - start) * 1000

            mlflow.log_metric("latency_ms", latency_ms)
            mlflow.log_metric("tokens_used", result.tokens_used)
            mlflow.log_metric("cost_usd", result.cost_usd)

            # These metrics also get exposed to Prometheus via an
            # exporter to feed Grafana dashboards
            record_prometheus_metrics(latency_ms, result.tokens_used, result.cost_usd)

            return {"output": result.output}
        except Exception as e:
            mlflow.log_param("error_type", type(e).__name__)
            raise
```

## Anti-patterns

- Logging only the final output without the full context that produced it — impossible to debug afterward
- Measuring only average latency, hiding outliers affecting a meaningful subset of users
- Not monitoring cost until it's already spiked — setting up budget alerts is cheap compared to a surprise bill
- Confusing "no explicit errors" with "the system works well" — an agent can complete the loop with no error and still produce an incorrect or hallucinated result

## Related Notes

- [[Loop-Engineering]] — instrumenting each loop step to detect anomalous behavior
- [[Graph-Engineering]] — distributed tracing needed for multi-agent systems
- [[Harness-Engineering]] — offline eval metrics should keep being monitored in production
- [[SDLC-with-AI]] — observability feeds the Monitoring & Maintenance phase of the lifecycle
- [[Context-Engineering]] — token consumption metrics as part of context observability
- [[Six-Pillars-of-Agents]] — this note covers "observe," one of the three axes of Pillar 6
