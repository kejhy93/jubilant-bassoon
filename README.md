# jubilant-bassoon — Twitch Chat → Prometheus Metrics

A Java service that watches a Twitch channel's chat and events (subs, bits, raids) and exposes the activity as Prometheus metrics, for a Grafana dashboard.

> **Status: planning.** No application code exists yet — this repo currently holds the project plan as GitHub issues. See [Roadmap](#roadmap) below.

## What it does

- Connects to Twitch chat over IRC (WebSocket, anonymous read-only login — no OAuth needed).
- Connects to Twitch EventSub (WebSocket) for subscriptions, bits, and raids.
- Tracks chat activity, unique chatters, message length, emote/keyword usage, and event counts — without ever emitting a per-user Prometheus label (unbounded cardinality is treated as a hard constraint, not an optimization).
- Exposes everything on a `/metrics` HTTP endpoint for Prometheus to scrape.
- Ships as a Docker image for use alongside an existing Prometheus + Grafana setup.

## Stack

| Concern | Choice |
|---|---|
| Language | Java 25 (structured concurrency, virtual threads, sealed types + pattern matching, `System.Logger` — no external logging dependency) |
| Build | Maven |
| Metrics | `prometheus-metrics-core` + `prometheus-metrics-exporter-httpserver` |
| JSON | Jackson (`jackson-databind`), for Twitch REST/EventSub payloads |
| Tests | JUnit 5 |
| Packaging | Docker (multi-stage: `maven:3-eclipse-temurin-25` build → `eclipse-temurin:25-jre-alpine` runtime) |
| CI | GitHub Actions (build, test, JaCoCo coverage) |
| Code quality | SonarCloud |

Package root: `org.hejnaluk.twitchmetrics`.

## Roadmap

Tracked as GitHub issues, grouped by stage:

- **[#6 Stage 0: CI/CD pipeline](../../issues/6)** — GitHub Actions build/test, Sonar quality gate, Docker image publishing, branch protection. Sub-issues: [#14](../../issues/14) Sonar, [#15](../../issues/15) build & test workflow, [#16](../../issues/16) JaCoCo coverage, [#17](../../issues/17) build & push to GHCR, [#18](../../issues/18) branch protection.
- **[#1 Stage 1: Chat-only MVP](../../issues/1)** — IRC reader + core metrics (messages, unique chatters, message length). Sub-issues: [#19](../../issues/19) Maven skeleton, [#20](../../issues/20) IRC WebSocket connection, [#21](../../issues/21) message parsing, [#22](../../issues/22) metrics endpoint, [#23](../../issues/23) core metrics wiring, [#24](../../issues/24) Dockerfile.
- **[#2 Stage 2: Emote & keyword usage counters](../../issues/2)**
- **[#3 Stage 3: EventSub integration](../../issues/3)** — subs, bits, raids.
- **[#4 Stage 4: Grafana dashboard](../../issues/4)**
- **[#5 Stage 5 (exercise): Port to Go](../../issues/5)**

Cross-cutting issues not tied to one stage: [#7](../../issues/7) secrets management, [#8](../../issues/8) IRC/EventSub reconnect & backoff, [#9](../../issues/9) testing strategy, [#10](../../issues/10) deployment, [#11](../../issues/11) health endpoint & JVM metrics, [#13](../../issues/13) Docker Compose for local dev.

See the [full issue tracker](../../issues) for current status.

## Design constraints

- **No per-user Prometheus labels.** `unique_chatters` is a single Gauge computed from a swept sliding-window map, never one time series per user.
- **Bounded label sets.** Emote/keyword metrics use a fixed allow-list plus an `other` bucket.
- **JDK-first dependencies.** HTTP and WebSocket clients come from `java.net.http`; the dependency list is intentionally short (see table above).

## Getting started

Not yet runnable — build, configuration, and usage instructions will be added once [#19 (Maven project skeleton)](../../issues/19) lands.

## Contributing / review

This repo uses a project-scoped Claude Code review subagent (`.claude/agents/code-reviewer.md`) that checks changes against the design constraints above, Maven build hygiene, and CPU/memory performance in the chat message hot path.

## License

Not yet decided.
