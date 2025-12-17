# Arize AI Sample Notebooks

A collection of Jupyter notebooks demonstrating Arize AI platform capabilities for LLM observability, evaluation, and experimentation.

## Notebooks

| Notebook | Description |
|----------|-------------|
| `01_ingest.ipynb` | OpenTelemetry tracing setup for LangChain agents with a weather tool example |
| `02_dataset.ipynb` | Create and read evaluation datasets via Arize REST API |
| `03_code_eval.ipynb` | Run experiments with custom code-based evaluators |
| `04_llm_eval.ipynb` | LLM-as-a-judge evaluation for hallucination detection |
| `05_eval_mmlu.ipynb` | Evaluate models on MMLU (multiple choice questions) |
| `06_eval_livecodebench.ipynb` | Evaluate code generation using Phoenix evaluators |

## Installation

```bash
pip install arize arize-otel arize-phoenix[evals]
pip install openinference-instrumentation-langchain
pip install langchain langchain-openai openai
pip install datasets  # for LiveCodeBench
```

## Quick Start

### 1. Tracing LangChain Agents

```python
from arize.otel import register, Transport
from openinference.instrumentation.langchain import LangChainInstrumentor

tracer_provider = register(
    endpoint="https://<HOST>/v1/traces",
    transport=Transport.HTTP,
    space_id="<SPACE_ID>",
    api_key="<API_KEY>"
)

LangChainInstrumentor().instrument(tracer_provider=tracer_provider)
```

### 2. Running Experiments with Code Evaluators

```python
from arize.experimental.datasets import ArizeDatasetsClient
from arize.experimental.datasets.experiments.evaluators.base import EvaluationResult, CodeEvaluator

client = ArizeDatasetsClient(api_key="<API_KEY>")

def task(dataset_row) -> str:
    # Your LLM call here
    return response

class MyEvaluator(CodeEvaluator):
    def evaluate(self, *, output, dataset_row, **_):
        score = 1.0 if output == dataset_row["expected"] else 0.0
        return EvaluationResult(score=score, label="pass" if score else "fail")

client.run_experiment(
    space_id="<SPACE_ID>",
    dataset_id="<DATASET_ID>",
    task=task,
    evaluators=[MyEvaluator()],
    experiment_name="my-experiment"
)
```

### 3. Running Experiments with LLM Evaluators

```python
from arize.experimental.datasets.experiments.evaluators.base import EvaluationResult, LLMEvaluator

class HallucinationEvaluator(LLMEvaluator):
    template = "Classify if this response is factual or hallucinated: {response}"

    def evaluate(self, *, output, dataset_row, **_):
        result = self.llm.invoke(self.template.format(response=output))
        label = "factual" if "factual" in result.lower() else "hallucinated"
        return EvaluationResult(score=1.0 if label == "factual" else 0.0, label=label)
```

## Configuration

Set these environment variables or pass directly to the client:

- `ARIZE_API_KEY` - Your Arize API key
- `ARIZE_SPACE_ID` - Your Arize space ID
- `OPENAI_API_KEY` - OpenAI API key (for LLM-based evaluators)

## Resources

- [Arize Documentation](https://docs.arize.com)
- [Arize Phoenix](https://github.com/Arize-ai/phoenix)
- [OpenInference](https://github.com/Arize-ai/openinference)
