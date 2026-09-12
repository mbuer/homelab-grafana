# Agent Notes

## Purpose

This repository contains portable and reproducible Grafana configuration for the Home Lab.

Grafana visualizes metrics collected and stored by Prometheus.

The current deployment runs on `ubuntu-infra`, but the repository must remain portable to another server.

## Primary design goal

Treat the server as replaceable.

Configuration stored in this repository should allow Grafana to be rebuilt on another Linux server with minimal manual configuration.

A normal migration should aim for:

    git clone
    configure environment-specific values
    docker compose up -d

Do not create unnecessary dependencies on the current VM.

## Current architecture

    Proxmox
       │
       ▼
    node_exporter
       │
       ▼
    Prometheus
       │
       ▼
    Grafana

Prometheus collects and stores metrics.

Grafana queries and visualizes them.

Do not move metric-collection logic into Grafana.

## Current dashboard focus

The first dashboard is `Proxmox Health`.

It should answer:

**Is the Proxmox server healthy?**

Primary areas:

- CPU utilization
- CPU temperature
- memory utilization
- system load
- filesystem usage
- network throughput
- network errors and drops
- NVMe temperature
- NVMe SMART health and wear
- battery state and temperature
- AC power state
- Prometheus target availability

## Dashboard design

Prefer a small number of useful panels over displaying every available metric.

Use:

- time-series panels for changing measurements
- stat/status panels for current health conditions
- thresholds where they improve operational understanding

Avoid visual noise.

## Repository structure

- `compose.yaml` — Grafana container definition
- `.env.example` — documented environment-specific variables without secrets
- `provisioning/datasources/` — Grafana data-source provisioning
- `provisioning/dashboards/` — dashboard provisioning
- `dashboards/` — version-controlled dashboard JSON

## Repository rules

- Treat portability as a primary requirement.
- Keep dashboards reproducible in Git.
- Prefer provisioning over undocumented manual setup.
- Store dashboard JSON under `dashboards/`.
- Keep runtime state outside Git.
- Do not commit passwords, tokens, private keys, secrets, databases, logs, or Docker volumes.
- Do not commit `.env`.
- Keep `.env.example` free of real credentials.
- Avoid absolute host paths unless clearly necessary and documented.
- Avoid machine-specific configuration where a portable alternative exists.
- Keep environment-specific values separate from reusable configuration.
- Do not redesign working Prometheus collection when the task only concerns Grafana.
- Add plugins only when they solve a real requirement.
- Keep documentation synchronized with the actual deployment.

## Migration expectation

The repository should remain usable if Grafana is moved from `ubuntu-infra` to another server.

A migration must not require manually rebuilding dashboards from screenshots or memory.

Runtime data may be restored separately if needed, but dashboards and provisioning must remain recoverable directly from Git.
