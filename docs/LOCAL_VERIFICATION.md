# Local verification

Checked November 5, 2026 using Python 3.12, Node 24, pnpm 11.25, PostgreSQL 16, and Docker Compose.

- Frozen uv and pnpm installs completed.
- Ruff, mypy, frontend TypeScript, both chart helper tests, and the production dashboard build passed.
- With a separate disposable PostgreSQL test database: 33 Python tests passed; the Snowflake account test was skipped.
- Built and started a separate Docker stack from this checkout with fresh PostgreSQL, Kafka, and data volumes.
- Ran the real historical baseline through Kafka and Spark: 4,320 unique bars and 4,320 window bars reconciled, with zero late-window drops and zero backlog at completion.
- The API health and overview endpoints returned HTTP 200.
- Browser walkthrough: populated dashboard, ETH symbol selection, one-hour chart, and streaming operations. No console errors were observed.

The baseline took 54.21 seconds on this local run, including JVM startup. This is a small replay measurement, not a production throughput claim. Other failure scenarios and the optional Snowflake account integration were not rerun during this check.
