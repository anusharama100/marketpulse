# MarketPulse

A historical market-data replay lab built with Kafka, Spark Structured Streaming, PostgreSQL, and a Next.js dashboard. It follows market bars through ingestion, validation, stream windows, and warehouse writes so you can see what happens when deliveries arrive late, repeat, or fail.

The default dataset is one day of official Binance Vision archives for BTCUSDT, ETHUSDT, and SOLUSDT: 4,320 one-minute OHLCV bars. Every view labels the data as historical replay. Prices are quoted in USDT.

![MarketPulse dashboard](docs/screenshots/overview.png)

[Case study](portfolio/case-study.md) · [Architecture](docs/architecture.md) · [Verification](docs/verification.md) · [CI runs](https://github.com/anusharama100/marketpulse/actions)

## What to explore

- A provider adapter that checks downloaded ZIP files against published SHA-256 checksums.
- Kafka raw and curated topics with versioned envelopes and fixed symbol partitions.
- A Spark audit query that validates messages and writes facts, decisions, and an outbox in one PostgreSQL transaction.
- A second Spark query for event-time windows, watermarks, and bounded duplicate suppression.
- Replay scenarios for duplicates, late and reordered messages, invalid records, and interruption after a sink commit.
- Six dashboard views with chart inspection, CSV exports, source provenance, and replay comparisons.

## Run the demo

Install Docker with Compose and `make`:

```sh
git clone https://github.com/anusharama100/marketpulse.git
cd marketpulse
make demo
```

The first build downloads runtime dependencies and may take several minutes. No market-data key or cloud account is required. The command starts the stack, enables the bounded local replay form, and waits for a completed or failed baseline run.

Open the dashboard at [localhost:8031](http://127.0.0.1:8031) and API documentation at [localhost:8030/docs](http://127.0.0.1:8030/docs).

Without `make`:

```sh
MP_LAB_ENABLED=true docker compose up -d --build
docker compose exec -T api marketpulse demo
```

Source archives and checkpoints stay in local volumes. The default source date is January 6, 2025. Data and derived screenshots have separate [source-data terms](docs/data-license.md).

## Try a failure scenario

```sh
make replay SCENARIO=duplicate
make replay SCENARIO=late
make replay SCENARIO=restart
```

Compare expected counts, observed counts, audit decisions, and retained event-ID fingerprints in the replay dashboard. Late bars remain in the fact table even when the watermark excludes them from closed windows.

The `restart` and `warehouse_retry` scenarios exercise the same commit-boundary interruption. Each run has independent checkpoints. The lab caps source datasets at 10,000 events and runs one worker; it is intended for bounded experiments.

## How it is built

**Python 3.12, Kafka, Spark Structured Streaming, PostgreSQL, FastAPI, Next.js, React, and TypeScript.** The container stack uses Java 17 and matching Spark/Kafka connectors.

```text
Historical archives -> Kafka raw topic -> Spark validation
  -> PostgreSQL facts, audit decisions, and outbox
  -> Kafka curated topic -> Spark event-time windows
  -> PostgreSQL window results -> FastAPI -> dashboard
```

| Directory | Contents |
| --- | --- |
| `marketpulse/` | Contracts, database, CLI, and replay orchestration |
| `services/` | Ingestion, Spark queries, warehouse loader, and API |
| `apps/dashboard/` | Next.js interface |
| `warehouse/` | Migrations, models, and data checks |
| `scripts/`, `tests/` | Replay verification and automated checks |
| `docs/` | Contracts, recovery notes, screenshots, and evidence |

## Delivery and analytical limits

The services do not share one atomic transaction, so the project does not claim end-to-end exactly-once delivery. PostgreSQL identities and batch records handle sink retries. Outbox publication is at least once; identical curated messages are suppressed within each watermarked window. Conflicting identities are quarantined.

Prices and volumes use decimals in storage and strings in API responses. Returns require adjacent observed bars; rolling measures require contiguous history. Missing intervals are not filled. Exploratory anomaly flags are not trading signals.

This is historical replay, with no order execution or forecasting feature. The three selected crypto symbols do not represent a whole market. Live subscriptions, cloud deployment, high availability, and sustained throughput are outside the verified scope. See [streaming semantics](docs/streaming-semantics.md) and [data contracts](docs/data-contracts.md).

## Tests and recorded results

```sh
make test
make verify
make benchmark
```

Native development uses Python 3.12 with uv, Node 24, and pnpm 11.25. Database tests need a separate disposable database; browser checks need a populated running application. Setup details are in the [runbook](docs/runbook.md).

The checked-in [verification](docs/verification.md) records 33 Python tests with one Snowflake account test skipped, two chart tests, eight desktop/mobile browser journeys, and eight Kafka/Spark experiments. A separate clean-source stack processed all 4,320 bars.

The recorded baseline took 29.68 seconds including JVM startup, with p50/p95 curation delays of 12.22/20.79 seconds. These are small local replay measurements, not production capacity or live-feed latency. See [benchmarks](docs/benchmarks.md).

## Optional integrations and cleanup

A Snowflake exporter is implemented but account-level execution is unverified. It needs an existing account and an explicit execution flag; the local demo does not provision or connect to Snowflake. See [warehouse strategy](docs/warehouse-strategy.md).

Use `make logs` to inspect services and `make stop` to stop them while retaining volumes. `docker compose down -v` deliberately discards this project's stored data and checkpoints.

Original project code uses the [MIT license](LICENSE). Market data retains its own [license and attribution requirements](docs/data-license.md).

## Latest local check

See [local verification](docs/LOCAL_VERIFICATION.md) for the November 5, 2026 setup, runtime checks, and remaining test boundaries.
