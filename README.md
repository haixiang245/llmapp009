# llmapp09 — DOAIS Day 3: LLMSecOps CI/CD

NUS-ISS *Deploying and Operating AI Solutions* Day 3 workshop, based on
[darryl1975/llmapp09](https://github.com/darryl1975/llmapp09).

## Components
- `llm-multiroute/` — FastAPI backend: multi-model routing, guardrails, metrics, Langfuse tracing
- `llm-frontend-python/` — Flask web UI calling the backend
- `promptfoo-tests/`, `deepeval-tests/` — LLM evaluation suites against the live backend
- `.github/workflows/` — 4 GitHub Actions pipelines

## CI/CD pipelines
| Workflow | Stages |
|---|---|
| LLM Multiroute CI | Ruff → pytest → Docker build → Trivy scan → push to Docker Hub |
| LLM Frontend Python CI | Ruff → Docker build → Trivy scan → push to Docker Hub |
| PromptFoo Tests | start backend (docker compose) → smoke test → 4 PromptFoo evals |
| DeepEval Tests | start backend (docker compose) → smoke test → 4 DeepEval suites |

## Repository configuration
Settings → Secrets and variables → Actions

- **Variable** `DOCKERHUB_USERNAME` — Docker Hub account images are pushed to
- **Secrets** `DOCKERHUB_TOKEN`, `OLLAMA_API_KEY`, `OLLAMA_BASE_URL`, `OPENAI_API_KEY`

## Changes from the original
1. Docker Hub account read from the `DOCKERHUB_USERNAME` repo variable instead of hardcoded
2. Trivy `ignore-unfixed: true` so OS CVEs with no available fix don't block the build
3. PromptFoo/DeepEval jobs write the backend `.env` from secrets, rebuild with `--build`, run a classify smoke test and dump backend logs on failure
4. DeepEval suites use `--reruns 2` (pytest-rerunfailures) for non-deterministic LLM-judge results
5. Removed committed `venv/`, `__pycache__/` and DeepEval run cache; added `.gitignore` and `.dockerignore`

## Run locally
```bash
cp llm-multiroute/.env.example .env   # fill in OLLAMA_API_KEY etc.
docker compose up -d --build
# backend  http://localhost:8080   frontend http://localhost:5000
```
