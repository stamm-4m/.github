# STAMM

**S**oft sensor moni**T**oring **a**nd **m**aintenance framework for **M**achine learning **M**odels

[![License: Apache 2.0](https://img.shields.io/badge/License-Apache_2.0-blue.svg)](https://www.apache.org/licenses/LICENSE-2.0)
[![Documentation](https://img.shields.io/badge/docs-stamm.inrae.fr-green)](https://stamm.inrae.fr)

STAMM is an open-source MLOps framework dedicated to the deployment, monitoring, and maintenance of machine-learning soft sensors in industrial environments. Unlike general-purpose MLOps platforms, which assume single-language pipelines monitored through near-synchronous prediction error, STAMM is purpose-built for the regime in which industrial soft sensors actually operate: ground-truth labels arrive offline hours or days after the prediction, processes show slow non-stationary dynamics, and model code is written across heterogeneous languages.

Although developed and validated in the context of an industrial-scale fed-batch fermentation, the framework is **domain-agnostic** and applies to a wide range of industrial contexts including chemical production, environmental monitoring, and other data-intensive processes.

---

## What STAMM gives you

- A **language-agnostic REST model registry** that serves Python, R, and other models behind a single `/predict` endpoint. Supported export formats include `pkl`, `joblib`, `ONNX`, `h5`, `keras` (Python) and `rds`, `rdata`, `pmml` (R).
- An **extensible drift-detection package** with six reference implementations (ADWIN, PSI, Kolmogorov–Smirnov, PCA-CD, KDQ-tree, MMD) and a uniform interface for adding new ones.
- An **output-level model-divergence metric** (MSE, Pearson, Spearman) that substitutes for residual-based monitoring when fresh ground-truth labels are unavailable.
- A **dashboard with a human-in-the-loop labelling interface** so that process experts can annotate anomalies and approve retraining without writing code.
- A **FAIR-aligned YAML metadata schema** for soft-sensor models that links to FAIRDOM-SEEK catalogues such as the IBISBA Knowledge Hub.
- A **containerised reference deployment** (Docker Compose) that you can stand up in one command.

---

## Architecture

STAMM is organised around five loosely coupled components connected by three pipelines (data ingestion, real-time inference, and maintenance with human-in-the-loop corrections).

| # | Component | Repository |
|---|-----------|------------|
| I | Time-series database (InfluxDB 2.7) + relational backbone (PostgreSQL/TimescaleDB) | [`data-backbone`](https://github.com/stamm-4m/data-backbone) |
| II | Workflow orchestrator (Apache Airflow 3.0.6) | [`workflow-orchestrator`](https://github.com/stamm-4m/workflow-orchestrator) |
| III | Language-agnostic model registry (FastAPI) | [`model-registry`](https://github.com/stamm-4m/model-registry) |
| IV | Dashboard (Plotly-Dash) | [`dashboard`](https://github.com/stamm-4m/dashboard) |
| V | Drift detectors (Python package) | [`drift_detectors_pack`](https://github.com/stamm-4m/drift_detectors_pack) |
| – | Node-RED bioreactor emulator (for the IndPenSim demo) | [`Node-RED-emulator`](https://github.com/stamm-4m/Node-RED-emulator) |

Each repository ships its own `README.md`, `Dockerfile`, and `docker-compose.yml`.

---

## Installation

### Requirements

- Linux, macOS, or Windows with **Docker ≥ 24** and **Docker Compose**
- Python ≥ 3.12 (for local development of Python components)
- R ≥ 4.1 (for local development of R-based soft sensors)
- ≥ 8 GB RAM recommended

Once the stack is healthy, the services are available at:

| Service | URL |
|---------|-----|
| Dashboard | `http://localhost:8050` |
| Model registry (Swagger UI) | `http://localhost:8000/docs` |
| Airflow UI | `http://localhost:8080` |
| InfluxDB UI | `http://localhost:8086` |
| Node-RED emulator | `http://localhost:1880` |

### Component-only installation

Each component repository can also be deployed independently. See the per-repository READMEs for component-specific instructions.

---

## Usage

A full walk-through with screenshots is available in the [STAMM documentation](https://stamm.inrae.fr). The typical workflow is:

1. **Stream raw measurements** into the time-series database (`stamm_raw` bucket) — either through native REST hooks of the equipment, through MQTT applications or more advanced frameworks such as [LEAF](https://leaf-framework.org/), or, for the demo, through the Node-RED emulator that replays IndPenSim batches.
2. **Register a soft sensor** in the model registry by uploading the model artefact (any supported format) together with its YAML metadata file.
3. **Schedule inference** through the Airflow `deployment_soft_sensors` DAG, which discovers registered soft sensors, calls `/predict`, and writes predictions into the `stamm_predictions` bucket.
4. **Monitor drift** through the dashboard's *Monitoring* view, which shows training-vs-live density comparisons per input variable and pairwise model-divergence plots.
5. **Annotate anomalies and approve retraining** through the dashboard's *Simulation Assessment* view; labels are persisted to the `stamm_metadata` bucket and feed back into the maintenance pipeline.

---

## Citation

If you use STAMM in your research, please cite the  paper:

```bibtex
@article{SUAREZ2026102783,
title = {STAMM: Soft sensor moniToring and mAintenance framework for Machine learning Models},
journal = {SoftwareX},
volume = {35},
pages = {102783},
year = {2026},
issn = {2352-7110},
doi = {https://doi.org/10.1016/j.softx.2026.102783},
url = {https://www.sciencedirect.com/science/article/pii/S235271102600275X},
author = {Carlos Suarez and Alexander Astudillo and Brett Metcalfe and Matthew Crowther and Jasper J. Koehorst and Esteban Castillo and Ariane Bize and David Camilo Corrales},
keywords = {Machine learning operations (MLOps);Soft sensors;Drift detection;Digital twins;Industry 4.0;Bioprocess monitoring},
abstract = {STAMM (Soft sensor moniToring and mAintenance framework for Machine learning Models) is an open-source MLOps framework for industrial machine-learning soft sensors. Unlike general-purpose MLOps platforms (MLflow, Kubeflow, Metaflow, ClearML), STAMM targets the regime where ground-truth labels arrive offline hours or days late, processes exhibit slow non-stationary dynamics, and models are multi-language. The framework comprises five loosely coupled components: a time-series database, workflow orchestrator, language-agnostic REST model registry, dashboard with human-in-the-loop labelling, and an extensible drift-detection package. STAMM was validated on an industrial-scale fed-batch penicillin fermentation (IndPenSim) with seven coexisting R and Python soft sensors served through the model registry.}
}
```

The ML soft sensors deployed in the IndPenSim case study are described in:

```bibtex
@article{metcalfe2025mlops,
  title   = {Towards a machine learning operations (MLOps) soft sensor for real-time predictions in industrial-scale fed-batch fermentation},
  author  = {Metcalfe, Brett and Acosta-Pavas, Juan Camilo and Robles-Rodriguez, Carlos Eduardo and Georgakilas, George K. and Dalamagas, Theodore and Aceves-Lara, Cesar Arturo and Daboussi, Fayza and Koehorst, Jasper J. and Corrales, David Camilo},
  journal = {Computers \& Chemical Engineering},
  volume  = {194},
  pages   = {108991},
  year    = {2025},
  doi     = {10.1016/j.compchemeng.2024.108991}
}

@article{acostapavas2024soft,
  title   = {Soft sensors based on interpretable learners for industrial-scale fed-batch fermentation: Learning from simulations},
  author  = {Acosta-Pavas, Juan Camilo and Robles-Rodriguez, Carlos Eduardo and Griol, David and Daboussi, Fayza and Aceves-Lara, Cesar Arturo and Corrales, David Camilo},
  journal = {Computers \& Chemical Engineering},
  volume  = {187},
  pages   = {108736},
  year    = {2024},
  doi     = {10.1016/j.compchemeng.2024.108736}
}
```

---

## License

STAMM is released under the **Apache License 2.0**. See [`LICENSE.txt`](./LICENSE.txt) for the full text.

---


## Contact

 David Camilo Corrales — INRAE, Toulouse Biotechnology Institute (TBI), 135 Avenue de Rangueil, 31077 Toulouse, France. <David-Camilo.Corrales-Munoz@inrae.fr>

---

## Acknowledgements

This research was conducted within the EU Horizon 2020 [Bioindustry 4.0](https://www.bioindustry4.eu/) project (Grant Agreement No. 101094287). 
