---
tags: [claude-code, engineering, testing, evals, mlflow]
type: engineering
related:
  - "[[Loop-Engineering]]"
  - "[[Observability]]"
  - "[[Prompt-Engineering]]"
  - "[[SDLC-with-AI]]"
  - "[[Anatomy-of-a-Skill]]"
created: 2026-08-02
status: stable
---

# Harness Engineering

> **Source**: notes from *AI Builders Meetup Guadalajara #1*, hosted by Wizeline (Guadalajara, 2026-08-02).

## What Is a Test Harness for AI Agents?

Infrastructure that lets you run, evaluate, and compare an agent's behavior systematically. Key difference from traditional unit tests: a harness evaluates **probabilistic quality**, not binary pass/fail determinism — the same prompt can produce slightly different outputs across runs.

It's critical because, without a harness, you can't know whether a prompt, skill, or model change made the system better or worse — you only have anecdotal impression ("it feels better"), which doesn't scale or hold up to a stakeholder.

## Components of a Complete Harness

1. **Test dataset (fixtures)**: representative inputs of real usage — happy paths, edge cases, adversarial cases
2. **Expected outputs / golden set**: what the agent should produce for each dataset input
3. **Evaluators (judges)**: a deterministic function or LLM-as-judge that scores the output (0-1 scale or categorical rubric)
4. **Metrics**: accuracy, average latency, cost per call, hallucination rate, tool call accuracy
5. **Experiment logging**: which prompt was used, which model, which parameters — to compare runs against each other
6. **Regression suite**: minimal set of cases that must pass before each deploy, to catch regressions

## Types of Evals

### Unit Eval
An isolated prompt + input → expected output. Tests one component in isolation (e.g., "given this ticket, does the agent classify it correctly?").

### Integration Eval
A complete tool flow — e.g., the agent calls 3 tools in sequence and produces a result; both the final result and the call sequence get evaluated.

### End-to-End Eval
The complete flow from real user input to final output, across every system component (see [[Loop-Engineering]] and [[Graph-Engineering]] for the architectures this evaluates).

### Adversarial Eval
Inputs deliberately designed to break the agent — prompt injection, ambiguous inputs, extreme edge cases (see [[Security]] for the related threat model).

## MLflow for Eval Tracking

```python
import mlflow

def evaluate_prompt_version(prompt_template: str, dataset: list[dict], version: str):
    with mlflow.start_run(run_name=f"prompt-eval-{version}"):
        mlflow.log_param("prompt_version", version)
        mlflow.log_param("model", "claude-sonnet-5")

        scores = []
        for example in dataset:
            output = run_agent(prompt_template, example["input"])
            score = judge_output(output, example["expected"])
            scores.append(score)

        mlflow.log_metric("mean_accuracy", sum(scores) / len(scores))
        mlflow.log_metric("min_accuracy", min(scores))
        mlflow.log_artifact("dataset.json")

        return sum(scores) / len(scores)

# Compare two prompt versions
score_v1 = evaluate_prompt_version(PROMPT_V1, golden_dataset, version="v1")
score_v2 = evaluate_prompt_version(PROMPT_V2, golden_dataset, version="v2")

print(f"v1: {score_v1:.2%} | v2: {score_v2:.2%}")
# Results land in the MLflow tracking server for visual comparison
# between runs, including parameters, metrics, and artifacts
```

With this: one experiment gets registered per prompt version, evaluation metrics get logged, and the MLflow UI lets you visually compare both runs (parameters, metrics, artifacts) side by side.

## Golden Datasets: Construction and Maintenance

- **How to collect the first examples**: real production logs (anonymized if they contain sensitive data) and already-documented use cases from specs
- **How many examples you need to start**: 10-20 well-chosen examples are enough to start iterating — don't wait until you have hundreds before starting to measure
- **How to update it**: every time the system evolves (new feature, new edge case discovered in production), add the case to the dataset
- **Automation**: instrument production to automatically detect cases where the agent had low confidence or the user corrected its output — those are natural candidates for new golden set cases

## CI/CD Integration

- Run evals on every PR that modifies a prompt, a skill, or the project's CLAUDE.md
- Define **thresholds**: what minimum metrics must be met to allow the merge (e.g., accuracy ≥ 90% on the golden set, no regressions on the security subset)
- Example GitHub Action:

```yaml
name: Agent Evals
on:
  pull_request:
    paths:
      - "prompts/**"
      - "skills/**"
      - "CLAUDE.md"

jobs:
  eval:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Run eval suite
        run: uv run python evals/run_suite.py --dataset golden_set.json
      - name: Check thresholds
        run: |
          uv run python evals/check_thresholds.py \
            --min-accuracy 0.90 \
            --results results.json
```

## Anti-patterns

- Relying on subjective impression ("it seems to work better") instead of running the harness against the golden set
- An outdated golden set that doesn't reflect the real cases the system faces today
- Evals that only measure the happy path, without coverage of edge cases or adversarial cases
- Not versioning the dataset or the results — without this, you can't objectively compare "before" vs "after" a change

## Related Notes

- [[Loop-Engineering]] — what gets evaluated: the quality of the agent's decisions within the loop
- [[Observability]] — the harness produces eval metrics; observability monitors them in production over time
- [[Prompt-Engineering]] — the harness is the systematic way to iterate prompts (instead of ad hoc trial and error)
- [[SDLC-with-AI]] — where the harness fits within the full development lifecycle
- [[Anatomy-of-a-Skill]] — skills also benefit from systematic evals, not just standalone prompts
