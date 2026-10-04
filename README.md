<p align="center">
  <a href="https://github.com/getplumber/plumber"><img src="https://raw.githubusercontent.com/getplumber/plumber/HEAD/assets/plumber-banner.png" alt="Plumber"></a>
</p>

<h1 align="center">AI-written pipelines</h1>

<p align="center">
  <b>Five AI models. One prompt. One CI/CD pipeline each. Then Plumber reads what they wrote.</b>
</p>

<p align="center">
  <a href="https://github.com/getplumber/plumber"><img src="https://img.shields.io/badge/scanned%20with-Plumber-33DD4E" alt="Scanned with Plumber"></a>
  <a href="./LICENSE"><img src="https://img.shields.io/badge/license-MIT-blue" alt="MIT License"></a>
</p>

---

## The experiment

AI agents write CI/CD pipelines now, and they are good at it: the pipelines run, the checks pass, the deploy goes out. This repository asks a different question. Is a pipeline that works also a pipeline that is safe?

1. This repo holds a small, realistic project: an API, a web interface, a Docker image, two environments.
2. Each model gets [the same prompt](./PROMPT.md), once, in a fresh session. No hints, no follow-up, no retry.
3. Each answer is committed untouched as `.github/workflows/<model>.yml`.
4. [Plumber](https://github.com/getplumber/plumber) scans every workflow for risky CI/CD patterns.

## The contestants

| Model               | Vendor    | Workflow                                    | Findings    |
| ------------------- | --------- | ------------------------------------------- | ----------- |
| Claude Fable 5.1    | Anthropic | `.github/workflows/claude-fable-5-1.yml`    | not run yet |
| GPT-6 Astra         | OpenAI    | `.github/workflows/gpt-6-astra.yml`         | not run yet |
| Gemini 4 Argon      | Google    | `.github/workflows/gemini-4-argon.yml`      | not run yet |
| Grok 4.7            | xAI       | `.github/workflows/grok-4-7.yml`            | not run yet |
| DeepSeek V4.1 Flash | DeepSeek  | `.github/workflows/deepseek-v4-1-flash.yml` | not run yet |

## Check it yourself

```bash
brew tap getplumber/plumber
brew install plumber

plumber analyze github.com/getplumber-examples/ai-written-pipelines
```

Every finding names the file, the line, why it matters and how to fix it. Do not take our word for it, run the scan.

## The project: hello-pipeline

A hello world with enough moving parts to need a real pipeline.

```
apps/
  api/        Hono API on Node 22 (TypeScript, Vitest)
  web/        React + Vite interface (TypeScript, Vitest, Testing Library)
scripts/
  smoke.sh    smoke test for a deployed instance
Dockerfile    one image: the API serves the built interface
fly.toml      Fly.io config for staging and production
PROMPT.md     the prompt every model received
```

| Endpoint                  | What it does                            |
| ------------------------- | --------------------------------------- |
| `GET /healthz`            | Liveness check                          |
| `GET /api/version`        | The deployed version (commit SHA)       |
| `GET /api/hello?name=Ada` | `{"message": "Hello, Ada!"}`            |
| `GET /api/greetings`      | The 20 most recent greetings            |
| `POST /api/greetings`     | Stores a greeting for `{"name": "Ada"}` |

### Run it

```bash
npm ci
npm run dev          # API on :3000, interface on :5173
```

### What a pipeline has to run

```bash
npm run format:check
npm run lint
npm run typecheck
npm run test:coverage   # writes coverage/coverage-summary.json in each app
npm run build
docker build --build-arg APP_VERSION=$(git rev-parse --short HEAD) -t hello-pipeline .
./scripts/smoke.sh http://localhost:3000
```

## Rules of the experiment

- Same prompt for everyone, copied from [`PROMPT.md`](./PROMPT.md).
- One shot. The first answer is the answer.
- The workflow files are never edited by hand.
- The prompt does not mention security, in either direction.
- The workflows are exhibits. This repository has no deploy secrets and GitHub Actions is disabled on it, so nothing here actually ships.

## Why it matters

None of this is about a model being bad at its job. The pipelines do what was asked. The patterns Plumber reports (actions pinned to tags that can move, tokens with more permissions than the job needs, untrusted text pasted into a shell, scripts downloaded and run unchecked) are the same ones found in pipelines written by people. Agents just write them faster, and fewer people read the result.

Let the agents build. Check what they build.

## License

[MIT](./LICENSE)
