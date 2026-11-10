# CKDu Well-Water Safety Network

A multi-agent simulation platform for monitoring well-water quality indicators and coordinating scarce safe-water resources across communities in Sri Lanka.

Built as MSc Data Science and AI Multi-Agent Systems coursework at the University of Sri Jayewardenepura, this project combines autonomous agents, water-quality scoring, resource negotiation, and an interactive dashboard. It explores how communities can coordinate access to reverse-osmosis (RO) units and water bowsers using shared risk indicators.

This is a research and education simulation. Water-quality indicators do not diagnose Chronic Kidney Disease of Unknown Origin (CKDu) or establish that water chemistry causes CKDu. Simulation readings are synthetic; published aggregate data and modelling assumptions are documented separately.

## Group and academic details

- **University:** University of Sri Jayewardenepura
- **Programme:** MSc in Data Science and Artificial Intelligence
- **Module:** Multi-Agent Systems
- **Faculty:** Faculty of Computing
- **Department:** Department of Knowledge Engineering and Communication
- **Submitted to:** Dr. Budditha Hettige
- **Submission date:** 26 October 2026

| Member | Name | Responsibility |
| --- | --- | --- |
| A | K.K.N.C Perera | Data layer, Clock Agent, Well Monitor Agent |
| B | R.K Wanigasooriya | Water Quality Index, Risk Scoring Agent |
| C | M.M.S.D Wijerathna | Hotspot Agent, Village Agent, Resource Coordinator Agent, Contract Net |
| D | U.H.N.K Siriwardhana | Alert Agent, evaluation, dashboard |

## Features

- **Well monitoring:** advance synthetic well readings through simulated months.
- **Risk scoring:** calculate a Water Quality Index (WQI), village-level tiers, and contributing parameters.
- **Hotspot screening:** evaluate risk history against explicitly configured, provisional screening rules.
- **Resource coordination:** run Contract Net negotiation between village agents and a resource coordinator.
- **Alerts:** process risk and hotspot evidence, with delivery acknowledgements from a simulated local inbox.
- **Interactive dashboard:** explore risk maps, simulation results, agent activity, message exchanges, and resource awards.
- **Simulation Lab:** validate synthetic CSV inputs, configure runs, and inspect their progress from the dashboard.
- **Evaluation:** report supported metrics and explicitly identify metrics whose required evidence is unavailable.

## Technology stack

| Area | Technologies |
| --- | --- |
| Language and concurrency | Python, asyncio |
| Multi-agent system | SPADE, XMPP, FIPA ACL-style messages, Contract Net Protocol |
| Dashboard and maps | Streamlit, Folium, OpenStreetMap |
| Data and statistics | pandas, NumPy, SciPy |
| Testing | pytest |
| Packaging and development | Docker, Docker Compose, pip, Git |
| Data and configuration | CSV, JSON, JSONL, GeoJSON, YAML |

See [requirements.txt](requirements.txt) for the complete pinned dependency list.

## Agent roles

| Agent | Responsibility |
| --- | --- |
| Clock | Broadcast simulated month ticks |
| Well Monitor | Load the current month's well readings |
| Risk Scoring | Calculate well and division risk indicators |
| Village | Maintain local risk state and respond to resource proposals |
| Hotspot | Screen division risk history using the configured rule |
| Resource Coordinator | Request bids, select awardees, and collect confirmations |
| Alert Processing / Inbox | Apply alert policy and acknowledge simulated receipt |

The simulation runner synchronizes stages and records their outcomes. The dashboard reads persisted artifacts and can launch isolated runs through Simulation Lab.

## Quick start with Docker

Run these commands from the repository root with Docker and Docker Compose installed:

```bash
docker compose build
docker compose run --rm demo-runner
docker compose up -d dashboard
```

Open **http://localhost:8501**. The dashboard exposes the generated local demo and Simulation Lab views.

The `demo-runner` service generates an illustrative local demo without starting SPADE agents. Use **Simulation Lab** for live SPADE simulations. The local demo requires a new or empty output directory. If `demo_output/` already contains a bundle, preserve it and choose a fresh directory:

```bash
CKDU_DEMO_HOST_DIR=./demo_output_new docker compose run --rm demo-runner
CKDU_DEMO_HOST_DIR=./demo_output_new docker compose up -d dashboard
```

Stop the dashboard with:

```bash
docker compose down
```

## Local setup

Python 3.12 matches the Docker runtime. From the repository root:

```bash
python3.12 -m venv .venv
source .venv/bin/activate
python -m pip install -r requirements.txt
python -m streamlit run dashboard/app.py
```

On Windows, create the environment with `py -3.12 -m venv .venv` and activate it with `.venv\Scripts\Activate.ps1` in PowerShell.

### Generate an illustrative local demo

Choose a new or empty output directory:

```bash
python -m src.sim.run_local_demo \
  --output-dir ./demo_output_new \
  --months 12 \
  --start-month 10 \
  --resource RO_unit \
  --pool-size 3

CKDU_DEMO_OUTPUT_DIR=./demo_output_new python -m streamlit run dashboard/app.py
```

Select **Generated local demo** in the dashboard. This path reuses scoring, screening, alert, and illustrative allocation logic; it does not execute the live agent negotiation protocol.

### Run a live SPADE simulation

This example runs three simulated months with explicit calendar labels and a provisional two-month hotspot rule:

```bash
python -m src.sim.runner \
  --start-month-index 0 \
  --end-month-index 2 \
  --calendar-months 10,11,12 \
  --output-dir ./simulation_run_example \
  --resource RO_unit \
  --resource-pool 3 \
  --hotspot-trigger-tier Poor \
  --hotspot-trigger-tier "Very Poor" \
  --hotspot-trigger-tier Unsuitable \
  --hotspot-consecutive-months 2 \
  --embedded-server

CKDU_SIMULATION_RUN_DIR=./simulation_run_example python -m streamlit run dashboard/app.py
```

Use a fresh output directory for each run. Select **Live Coordination** or **Agent Monitor** to inspect the artifacts. Alternatively, configure and launch a run directly in **Simulation Lab**.

For runner options, output contracts, and stage/evaluation availability, see [Simulation runner](docs/simulation_runner.md).

### Dashboard configuration

| Environment variable | Purpose |
| --- | --- |
| `CKDU_DEMO_OUTPUT_DIR` | Generated local-demo directory |
| `CKDU_SIMULATION_RUN_DIR` | An isolated SPADE simulation run to inspect |
| `CKDU_SIMULATION_LAB_DIR` | Persistent Simulation Lab inputs, process logs, and runs |
| `CKDU_DATA_ROOT` | Alternative dashboard data directory |

The environment-variable examples above use macOS/Linux shell syntax. In PowerShell, set variables with `$env:CKDU_SIMULATION_RUN_DIR = "./simulation_run_example"` before launching the dashboard.

## Testing

With the virtual environment activated, run:

```bash
python -m pytest
```

The suite covers core scoring and message schemas, allocation and negotiation, agent behaviour, simulation outputs, evaluation, and dashboard helpers.

## Project structure

```text
ckdu-mas/
├── dashboard/                # Streamlit dashboard and animated risk map
├── data/
│   ├── raw/                  # Published aggregates and method/source tables
│   └── processed/            # Derived datasets, including synthetic wells
├── src/
│   ├── agents/               # SPADE agent behaviours
│   ├── core/                 # Scoring, bidding, allocation, and message schemas
│   └── sim/                  # Runners, evaluation, and artifact readers
├── tests/                    # Automated tests and illustrative fixtures
├── docs/                     # Methods, assumptions, and implementation details
├── logs/                     # Team output logs
├── demo_output/              # Illustrative local-demo artifacts
├── Dockerfile
├── compose.yaml
└── requirements.txt
```

## Data and interpretation

- Synthetic well readings retain `source=synthetic`; the live runner rejects real or mixed-source inputs.
- Seasonal readings are repeated across their assigned months, rather than representing independently observed monthly samples.
- Map points use approximate division locations; they do not establish individual well locations.
- Hotspot rules are provisional screening settings, not validated epidemiological models.
- Alert acknowledgements confirm receipt by a simulated inbox, not a real health officer.
- Completed execution does not imply scientific validation. Evaluation metrics remain unavailable when required reference labels, baselines, or provenance are missing.

## Documentation

- [Data sources](data/raw/README_data_sources.md)
- [Data and modelling assumptions](docs/data_assumptions.md)
- [WQI method](docs/wqi_method.md)
- [Simulation runner and Simulation Lab](docs/simulation_runner.md)
- [Contract Net coordinator](docs/contract_net_coordinator.md)
- [Local demo contracts](docs/local_demo_contract.md)
- [Known issues](docs/known_issues.md)
- [Project plan](docs/project_plan.md)
