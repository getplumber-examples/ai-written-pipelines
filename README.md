# hello-pipeline

A small greetings service: a Hono API on Node 22 and a React interface, shipped together as one Docker image and deployed to Fly.io.

[![MIT License](https://img.shields.io/badge/license-MIT-blue)](./LICENSE)

## Layout

```
apps/
  api/        Hono API on Node 22 (TypeScript, Vitest)
  web/        React + Vite interface (TypeScript, Vitest, Testing Library)
scripts/
  smoke.sh    smoke test for a deployed instance
Dockerfile    one image: the API serves the built interface
fly.toml      Fly.io config for staging and production
```

## API

| Endpoint                  | What it does                            |
| ------------------------- | --------------------------------------- |
| `GET /healthz`            | Liveness check                          |
| `GET /api/version`        | The deployed version (commit SHA)       |
| `GET /api/hello?name=Ada` | `{"message": "Hello, Ada!"}`            |
| `GET /api/greetings`      | The 20 most recent greetings            |
| `POST /api/greetings`     | Stores a greeting for `{"name": "Ada"}` |

## Development

Requires Node 22 (see `.nvmrc`).

```bash
npm ci
npm run dev          # API on :3000, interface on :5173
```

Configuration is read from the environment; `.env.example` lists the variables.

## Checks

```bash
npm run format:check
npm run lint
npm run typecheck
npm run test            # or npm run test:coverage
npm run build
```

## Container

```bash
docker build --build-arg APP_VERSION=$(git rev-parse --short HEAD) -t hello-pipeline .
docker run --rm -p 3000:3000 hello-pipeline
./scripts/smoke.sh http://localhost:3000
```

`APP_VERSION` is what `GET /api/version` reports.

## Deployment

`fly.toml` is shared by both environments; the app name is chosen at deploy time:

```bash
flyctl deploy --app hello-pipeline-staging --image <image>   # staging
flyctl deploy --app hello-pipeline --image <image>           # production
```

After a deploy, `./scripts/smoke.sh <base-url>` checks that the instance answers.

## License

[MIT](./LICENSE)
