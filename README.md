# MarketPulse

A self-hosted market data platform built as a learning and portfolio project:
Python, React, distributed systems, and Kubernetes infrastructure on Raspberry Pi 5.
The platform will collect market data, store historical prices, display charts,
calculate analytics, and trigger alerts. Its scope is data analysis, without trade execution.

## Status

The repository currently contains the project structure and documentation.
Applications, container images, manifests, and CI/CD pipelines have not been
implemented yet. There is no command to start the entire platform at this stage.

## Stack and target environment

| Area | Technology / purpose |
| --- | --- |
| Frontend | Next.js, React, TypeScript — dashboard, charts, watchlists |
| API | Python, FastAPI — HTTP API for the frontend |
| Processing | Python — collector, analytics, alerts |
| Data | PostgreSQL — prices, watchlists, analytics results, alert rules |
| Events | NATS, with JetStream planned for durable processing |
| Runtime | Linux ARM64 containers, K3s on Raspberry Pi 5 with a 1 TB SSD |
| Deployment | Helm, Argo CD, GitOps |
| CI and images | GitHub Actions and GHCR planned |
| Observability | Prometheus, Grafana, Loki, OpenTelemetry |

The target host is `rpi5-01`, running Ubuntu Server ARM64. Cluster configuration
and resource settings will be verified during deployment. Development takes place
on a laptop.

## Monorepo structure

```text
marketpulse/
├── apps/
│   ├── frontend/        # Next.js
│   └── api/             # FastAPI
├── services/
│   ├── collector/       # Data collection and normalization
│   ├── analytics/       # Asynchronous calculations
│   └── alerts/          # Alert rules and events
├── infra/
│   ├── k8s/             # Cluster bootstrap and resources outside application charts
│   ├── helm/            # Charts and deployment configuration
│   └── argocd/          # GitOps application definitions
├── docs/
│   └── architecture.md
├── AGENTS.md
├── README.md
└── .gitignore
```

Empty directories are tracked through `.gitkeep` files; remove these when adding content.

## Roadmap

All stages below are planned.

1. **v1 — initial data flow:** AAPL/MSFT/NVDA → collector → PostgreSQL
   → FastAPI → Next.js chart. Start with one provider and one interval.
2. **v2 — watchlists and comparisons:** custom lists, instrument comparisons,
   charts normalized to 100, and 1D/1W/1M/YTD changes.
3. **v3 — asynchronous processing:** NATS, an analytics worker, returns,
   moving averages, volatility, correlations, and drawdown.
4. **v4 — price alerts:** threshold rules, trigger history, and deduplication.
5. **v5 — live updates:** WebSocket; update frequency depends on the data provider.
6. **v6 — observability:** Prometheus/Grafana, Loki, and OpenTelemetry.
7. **v7 — full GitOps and CI/CD:** GitHub Actions → GHCR → Helm → Argo CD.
8. **v8 — resilience:** tests of pod restarts, event reprocessing,
   and data restoration from backups.

Infrastructure is added gradually: host and SSD preparation, K3s, the first
deployment, PostgreSQL with persistent storage, ingress, and Helm. Basic logging,
health checks, and error handling are developed alongside the applications.

## First implementation stage

- Choose a data interval and verify the provider. Twelve Data is a candidate
  from the initial project discussions; its limits and terms still need verification.
- Define instrument and OHLCV candle models, along with PostgreSQL migrations.
- Implement idempotent imports and historical data access through FastAPI.
- Add a single Next.js chart, followed by containers and deployment on K3s.

A public demo must use data that can legally be shared; synthetic data is an
alternative. Secrets and local runtime data stay outside the repository.

Data flows and design decisions: [architecture](docs/architecture.md).
Contributor and agent guidance: [AGENTS.md](AGENTS.md).
