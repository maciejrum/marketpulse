# MarketPulse — project context and working guidelines

## Purpose and current state

We are building a self-hosted market data platform to learn infrastructure and
create a portfolio project. Target environment: Raspberry Pi 5, a 1 TB SSD,
Linux ARM64, and K3s. Prioritize understandable data flows and incremental learning.
Trade execution is outside the scope of this project.

The repository currently contains only a scaffold and documentation. Do not assume
working services, tests, a cluster, or a pipeline exist. Read README.md and
docs/architecture.md before making changes; update the status as implementation progresses.

## Agreed stack and boundaries

- `apps/frontend`: Next.js, React, TypeScript; communicates through the HTTP API.
- `apps/api`: Python/FastAPI; market, watchlists, and alert rules start as modules
  within one API. Do not split them into additional microservices prematurely.
- `services/collector`: Python; provider integration, normalization, and price storage.
- `services/analytics`: Python worker; market data calculations.
- `services/alerts`: Python worker; rule evaluation and alert event storage.
- PostgreSQL for durable storage, NATS for event messaging.
- K3s, Helm, and Argo CD; GitHub Actions and GHCR are planned.
- Prometheus/Grafana for metrics, Loki for logs, OpenTelemetry for instrumentation.

## Implementation approach

1. Start with v1: AAPL/MSFT/NVDA → collector → PostgreSQL → API → chart.
2. Follow the README roadmap incrementally; do not install the entire stack up front.
3. Treat component directories as responsibility boundaries and future container images.
4. Choose solutions that fit on a single Pi. Do not assume high availability,
   multiple nodes, or a specific amount of RAM without checking the environment.
5. Document new decisions and distinguish proposals from implemented functionality.
6. Keep all repository content in English: documentation, code identifiers, comments,
   docstrings, API contracts, user-facing text, configuration comments, and commit messages.
   Do not introduce Polish text into repository files.

## Data and reliability

- Separate provider integration from the domain model; API limits must be configurable.
- Use timeouts, bounded retries with backoff, and rate-limit handling.
- Use UTC timestamps, explicit currencies, and precise numeric types for prices.
- Imports and event consumers must tolerate duplicates. Do not promise exactly-once
  processing; design for deduplication and safe retries.
- Manage database schema changes through versioned migrations.
- Use fixtures instead of external API dependencies in unit tests.
- Verify data usage rights before including data in a public repository or demo.

## Deployments and secrets

- Support `linux/arm64` images; add other architectures as needed.
- Choose dependency versions during implementation, commit lockfiles, and pin images.
- Keep application manifests in Helm. Use `infra/k8s` for bootstrap and resources
  outside charts; do not create two configuration sources for the same resource.
- Add probes appropriate to each process, plus resource requests and limits at deployment.
- PostgreSQL and durable messaging require PVCs, retention policies, and backup/restore plans.
- Never commit `.env` files, tokens, kubeconfigs, keys, or plaintext Kubernetes Secrets.
  `.env.example` files must contain placeholders only. Choose and document SOPS or
  Sealed Secrets before introducing secrets into GitOps.
- The frontend must not connect directly to the database or broker. PostgreSQL and
  NATS remain internal; exposing the application is a separate stage.

## Verification

Run linting, type checks, and tests relevant to the changed component once its
tooling exists. For Helm changes, lint and render charts; for container changes,
verify ARM64 support. For documentation changes, check links, naming consistency,
and `git diff --check`. Do not create superficial tests for an empty scaffold.
Report the checks performed and their limitations; do not claim deployments or
tests that were not actually run.
