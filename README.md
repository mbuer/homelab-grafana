# homelab-grafana

Portable and reproducible Grafana dashboards and provisioning for the Home Lab.

The goal of this repository is to keep Grafana configuration independent from any single server so the deployment can be moved, rebuilt, or restored with minimal manual work.

The initial focus is Proxmox health monitoring using metrics stored by Prometheus on `ubuntu-infra`.

## Goals

- provide a clear view of Home Lab health
- keep dashboards reproducible in Git
- provision data sources and dashboards automatically
- avoid undocumented manual Grafana configuration
- keep runtime state separate from configuration
- make the deployment portable to another server
- expand gradually as the Home Lab grows

## Architecture

    Proxmox
       │
       │ node_exporter :9100
       ▼
    Prometheus
       │
       │ time-series data
       ▼
    Grafana
       │
       ▼
    Dashboards and alerts

Prometheus is responsible for collecting and storing metrics.

Grafana queries Prometheus and provides visualization, dashboards, thresholds, and later alerting.

Metric collection logic should remain outside Grafana.

## Initial dashboard

The first dashboard is `Proxmox Health`.

It should answer one main question:

**Is the Proxmox server healthy?**

Initial panels should include:

- CPU utilization
- CPU temperature
- memory utilization
- system load
- root filesystem usage
- network receive and transmit rate
- network drops and errors
- NVMe temperature
- NVMe wear
- NVMe SMART health
- battery charge
- battery temperature
- AC power state
- Prometheus target availability

Time-series panels should be used for measurements that change over time.

Stat or status panels should be used for health states such as:

- AC connected
- NVMe critical warning
- NVMe media errors
- target availability

## Repository layout

    homelab-grafana/
    ├── README.md
    ├── AGENTS.md
    ├── compose.yaml
    ├── .env.example
    ├── .gitignore
    ├── provisioning/
    │   ├── datasources/
    │   │   └── prometheus.yml
    │   └── dashboards/
    │       └── dashboards.yml
    └── dashboards/
        └── proxmox-health.json

### compose.yaml

Defines the Grafana container and persistent runtime volume.

### provisioning/datasources/

Contains Grafana data-source provisioning.

The initial data source is Prometheus running on the Home Lab infrastructure server.

### provisioning/dashboards/

Tells Grafana where provisioned dashboards are stored.

### dashboards/

Contains dashboard JSON definitions that are version-controlled in Git.

## Portability

Portability is a primary design goal.

The Grafana deployment should not depend on the current server beyond a small number of environment-specific settings.

A future migration should ideally require only:

    git clone <repository>
    cp .env.example .env
    edit environment-specific values
    docker compose up -d

The following should remain portable through Git:

- Docker Compose configuration
- Grafana provisioning
- Prometheus data-source configuration
- dashboard definitions
- repository structure
- deployment documentation

Machine-specific configuration should be kept to a minimum.

Avoid absolute paths or undocumented manual configuration when a portable alternative exists.

## Persistent data

Grafana runtime data is intentionally kept outside Git.

This can include:

- Grafana database
- users
- sessions
- preferences
- plugins
- runtime-generated state

Persistent Docker volumes may be backed up separately if this state needs to be preserved during migration.

The monitoring configuration and dashboards should remain reconstructable from this repository even without the runtime volume.

## Secrets

Do not commit:

- passwords
- API tokens
- private keys
- `.env`
- certificates containing private material
- Grafana databases
- logs
- Docker volumes

`.env.example` may document required variables but must never contain real credentials.

## Prometheus data source

Grafana initially uses the Prometheus instance running on `ubuntu-infra`.

The Prometheus endpoint is treated as an environment-specific value so Grafana can later move to another server without rewriting dashboards.

The Docker networking between Grafana and Prometheus will be defined explicitly rather than assuming both repositories automatically share a Docker network.

## Deployment philosophy

Keep the stack simple.

Prefer:

- Docker Compose
- Grafana provisioning
- dashboard JSON in Git
- explicit documentation
- portable configuration

Avoid:

- manual-only dashboard configuration
- hidden server state
- unnecessary plugins
- hard-coded machine-specific paths
- storing secrets in Git

## Current status

Repository structure prepared.

Next steps:

1. configure Docker networking
2. configure Docker Compose
3. provision the Prometheus data source
4. configure dashboard provisioning
5. deploy Grafana
6. verify Prometheus connectivity
7. build the Proxmox Health dashboard
8. commit the dashboard JSON to Git
