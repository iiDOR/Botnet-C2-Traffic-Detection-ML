# Project Proposal

**Title:** Botnet / C2 Traffic Detection Using Machine Learning

**Student Name:** Nittin Nobby Mathew

**Roll No.:** AA.SC.P2MCA25077044@ahead.students.amrita.edu

**Topic:** Machine Learning-Based Detection of Botnet Command and Control Communications with Adversarial Robustness Testing

---

## Abstract of the Project

This project trains ML classifiers (Random Forest, XGBoost, Logistic Regression, z-score baseline) on established public datasets (CIC-IDS2017, UNSW-NB15) to benchmark detection of botnet and C2 communications, then builds a custom C2 beacon and server inside an isolated Docker lab to generate live traffic that tests whether public-dataset-trained models generalize to unseen C2 patterns. Flow-level features are extracted using Zeek. The project then measures how evasion techniques — jitter randomization, payload padding, sleep-wake batching, and protocol mimicry — degrade detection, and how much retraining recovers it. Cross-domain evaluation (train on public, test on custom; train on custom, test on public; train on both, test on evasion variants) quantifies real-world generalization. As a novel extension, an Adversarial Reinforcement Learning self-play framework automates this arms race between attacker and defender.

**Research Question:** How robust is an ML classifier that detects C2 beacons based on flow features, and which beacon parameters (jitter, sleep interval, payload size, transport) are most effective at evading it?

---

## Main Objective / Deliverable

The project is structured around six primary technical objectives:

- **Benchmark on public datasets (CIC-IDS2017, UNSW-NB15)** — establish baseline detection performance on the same datasets used in published work, enabling direct comparison with existing approaches.

- **Build a controlled C2 infrastructure from scratch** — a parameterized beacon agent and a logging-only C2 server running inside an isolated Docker network with no internet route, generating live malicious traffic under controlled conditions.

- **Generate a labeled flow dataset** covering 15+ beacon configurations (varying jitter, sleep interval, payload size, transport protocol) and 3+ benign traffic types (software updater, REST poller, browser simulator), captured via tcpdump and processed through Zeek.

- **Evaluate cross-domain generalization** — train on public data and test on custom C2 traffic, train on custom and test on public, train on both and test on evasion variants — measuring whether models trained in one domain transfer to another.

- **Design a flow feature extraction pipeline** that converts raw PCAPs into structured CSV datasets with timing features (inter-arrival time mean, std, coefficient of variation), size features (payload entropy, distinct sizes), connection features (bytes up/down ratio, connection count), and HTTP-level features (URI entropy, POST/GET ratio).

- **Train and evaluate multiple ML detectors** — Random Forest, XGBoost, Logistic Regression, and a statistical z-score baseline — with configuration-based and temporal train/test splits, reporting precision, recall, F1, ROC-AUC, and false-positive rate per traffic type. Use SHAP and permutation importance to identify which features drive detection and which are vulnerable to evasion.

- **Systematically test evasion techniques** — jitter randomization (log-normal, Pareto distributions), payload padding, sleep-wake batching, and protocol mimicry — measuring detection degradation per technique individually and in combination, then executing adversarial retraining to measure detector recovery.

- **Explore Adversarial RL Self-Play** as a novel extension — an offensive RL agent (PPO/SAC) that autonomously discovers evasion strategies by playing against the adaptive detector in a closed-loop Gymnasium environment, replacing hardcoded evasion configurations with learned policies.

**Final Deliverable:** A fully functional adversarial testing framework capable of generating C2 traffic, detecting it, evading detection, and retraining — producing a comprehensive report with detection metrics, evasion results, feature importance analysis, and adversarial retraining findings.

---

## System Architecture

The entire system runs on a single machine using Docker Compose to create an isolated lab network with no internet route.

```
┌─────────────────────────────────────────────────────────────┐
│                    Docker Internal Network                  │
│                  (no internet route)                        │
│                                                             │
│  ┌──────────────┐    HTTPS/HTTP    ┌──────────────────┐     │
│  │ Beacon Agent │  ──────────────> │  Lab C2 Server   │     │
│  │ (Python)     │                  │  (Flask, logging)│     │
│  │ Parameterized│                  └──────────────────┘     │
│  │ jitter/sleep/│                                           │
│  │ payload/proto│                                           │
│  └──────────────┘                                           │
│                                                             │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐       │
│  │ Updater Gen  │  │  REST Poller │  │  Browser Si  │       │
│  │ (benign)     │  │  (benign)    │  │  (benign)    │       │
│  └──────────────┘  └──────────────┘  └──────────────┘       │
│                                                             │
│  ┌──────────────────────────────────────────────────┐       │
│  │         Capture Container (tcpdump / Zeek)       │       │
│  └──────────────────────────────────────────────────┘       │
└─────────────────────────────────────────────────────────────┘
                              │
                         Raw PCAPs
                              │
                              ▼
              ┌──────────────────────────┐
              │  Feature Extraction       │
              │  Zeek conn.log/http.log   │
              │  → Structured CSV         │
              └──────────────────────────┘
                              │
                              ▼
              ┌──────────────────────────┐
              │  ML Detector Pipeline     │
              │  RF / XGBoost / LR /      │
              │  Z-score baseline         │
              │  SHAP feature analysis    │
              └──────────────────────────┘
                              │
                              ▼
              ┌──────────────────────────┐
              │  Evasion Testing Loop     │
              │  Modify beacon params     │
              │  → Recapture → Retest     │
              │  → Retrain → Re-attack    │
              └──────────────────────────┘
                              │
                              ▼
              ┌──────────────────────────┐
              │  Adversarial RL Self-Play │
              │  (Novel Extension)        │
              │  PPO/SAC Offensive Agent   │
              │  vs Adaptive Detector     │
              └──────────────────────────┘
```

### Network Isolation

All containers share a single Docker internal network with **no default gateway and no internet route**. The beacon agent communicates only with the C2 server. The capture container mirrors all traffic via tcpdump or Zeek. No traffic leaves the lab environment at any point during the project.

### Data Flow

1. **Generate:** Beacon agent checks in with C2 server using the current configuration (interval, jitter, payload size, transport).
2. **Capture:** tcpdump records all traffic as PCAP files.
3. **Extract:** Zeek processes PCAPs into conn.log and http.log. A Python pipeline converts these into labeled CSV with flow features.
4. **Detect:** ML models train on labeled flows and report detection metrics.
5. **Evade:** Beacon parameters are modified. New traffic is captured and tested against the trained detector.
6. **Retrain:** Detector retrains on evasion-variant data. The cycle repeats.

---

## Timeline and Milestones

The project is developed across five phases spanning 17 weeks:

| # | Phase | Focus | Weeks | Key Output |
|---|-------|-------|-------|------------|
| P1 | Foundation & Lab Setup | Docker environment, beacon agent, C2 server, benign generators | 1–3 | Lab runs end to end; beacon and benign traffic captured in separate PCAPs |
| P2 | Data Collection | Parameterize beacon (15+ configs), capture grid, Zeek extraction, labeled dataset | 4–6 | Labeled CSV dataset with 15+ beacon configurations and 3+ benign types, versioned in Git |
| P3 | Feature Engineering & Baseline Detector | Feature design, z-score baseline, RF/XGBoost/LR training, SHAP analysis | 7–10 | Working detector with documented metrics, feature importance, confusion matrices by jitter level |
| P4 | Evasion Testing | Evasion hypotheses, implementation, measurement, adversarial retraining, RL self-play exploration | 11–14 | Detection rate table per evasion technique and combination; adversarial retraining results; RL agent prototype |
| P5 | Write-Up & Packaging | Report, repository cleanup, reproducibility script, presentation | 15–17 | Final report, clean repository, dataset, 15-minute presentation |

### Phase 1: Foundation & Lab Setup (Weeks 1–3)

**Week 1 — Scope and Environment:**
- Write the lab-scope and ethics document, obtain department approval.
- Set up Docker Compose with an isolated internal network: C2 container, agent container, capture container running tcpdump/Zeek.
- Verify no route to the internet from the lab network.

**Week 2 — Baseline Beacon:**
- Build a benign beacon agent in Python that checks in at a fixed interval (e.g., 60 seconds) and posts a small JSON heartbeat.
- Build the C2 server (Flask) that logs each check-in and returns a no-op response.
- Capture first traces and confirm they appear in Wireshark.

**Week 3 — Normal Traffic Generators:**
- Build at least three benign generators: periodic API polling, bursty browser-style page loads, and a software updater pattern.
- Confirm each generator's traffic is distinguishable from the beacon at the packet level.

### Phase 2: Data Collection (Weeks 4–6)

**Week 4 — Labels and Capture Plan:**
- Define flow labeling: malicious (beacon) or benign (generator type).
- Define flow boundaries: 5-tuple plus a 60-second time window.

**Week 5 — Vary the Beacon:**
- Parameterize the agent: sleep interval, jitter percentage (0%, 10%, 30%, 50%), payload size distribution, transport (HTTP vs HTTPS), user-agent strings.
- Generate a grid of at least 15 beacon configurations.

**Week 6 — Build the Dataset:**
- Run each configuration for a fixed duration (e.g., 2 hours) and export PCAPs.
- Convert PCAPs to flow records using Zeek (conn.log, http.log) — Zeek is the primary pipeline due to known CICFlowMeter flow reassembly bugs.
- Store the labeled dataset as CSV with a documented schema.

### Phase 3: Feature Engineering & Baseline Detector (Weeks 7–10)

**Week 7 — Feature Design:**
- Timing: mean/std of inter-arrival times, coefficient of variation, autocorrelation at lags 1–5.
- Size: mean, variance, entropy of payload sizes; count of distinct sizes.
- Connection: connections per window, duration, bytes up/down, up/down ratio.
- HTTP-level: URI length, header count, user-agent entropy, POST/GET ratio.

**Week 8 — Baseline Models:**
- Statistical baseline: z-score detector on inter-arrival time mean (±2σ).
- Train Random Forest, XGBoost, Logistic Regression with stratified splits.
- Split by beacon configuration (not random rows) to prevent leakage.
- Temporal split: train on earlier captures, test on later ones.
- Apply class-weight balancing or SMOTE for class imbalance.

**Week 9 — Evaluation:**
- Report precision, recall, F1, ROC-AUC, and false-positive rate per benign traffic type.
- Confusion matrix broken down by jitter level.

**Week 10 — Feature Importance:**
- Permutation importance and SHAP values.
- Identify features a red team can easily manipulate vs. features that are hard to change.

### Phase 4: Evasion Testing (Weeks 11–14)

**Week 11 — Evasion Hypotheses:**
- H1: Increasing jitter above 30% drops recall below 50%.
- H2: Randomized payload padding removes size-based features.
- H3: Long sleep intervals with bursty check-ins evade timing features.

**Week 12 — Evasion Implementation:**
- Jitter randomization (log-normal, Pareto distributions).
- Payload padding to fixed or random size buckets.
- Check-in batching and sleep-wake patterns mimicking human users.
- Protocol mimicry using real update-service request shapes.

**Week 13 — Measurement:**
- Run each evasion variant through the detector.
- Record detection rate per technique, individually and combined.
- Track operational cost of each evasion method (latency, bandwidth, connections).

**Week 14 — Adversarial Retraining & RL Self-Play:**
- Retrain the detector on evasion-variant data and measure recall recovery.
- Document the attack → retrain → re-attack cycle.
- Prototype the Adversarial RL Self-Play: Gymnasium environment where a PPO/SAC agent discovers evasion policies against the adaptive detector.

### Phase 5: Write-Up & Packaging (Weeks 15–17)

**Week 15 — Report:**
- Structure: abstract, introduction, threat model, lab design, dataset, features, detector results, evasion results, RL self-play results, limitations, ethics statement.

**Week 16 — Artifacts:**
- Clean repository: Docker Compose, agent code, C2 server, feature extraction, training notebooks, dataset README.
- Reproducibility script that rebuilds the dataset from a fixed seed.

**Week 17 — Presentation:**
- 15-minute presentation covering the offensive build, detection baseline, evasion results, and RL findings.
- Detection recommendations for defenders.

---

## Tools to be Used for the Project

### Hardware Platform

| Component | Specification | Relevance to Project |
|-----------|--------------|---------------------|
| CPU | Modern multi-core processor | Runs Docker containers, ML training, traffic generation simultaneously |
| RAM | 16 GB+ | Holds multiple Docker containers, ML training datasets, Zeek processing |
| Storage | SSD with 50 GB+ free | PCAP storage (multiple hours × multiple configurations), dataset CSVs |
| GPU (optional) | NVIDIA with CUDA | Accelerates XGBoost training and RL self-play (PPO/SAC); CPU-only fallback supported |

### Software Tools

**Lab Infrastructure:**

| Tool | Purpose in Project |
|------|-------------------|
| Docker & Docker Compose | Container isolation; internal network with no internet route; reproducible lab environment |
| Python 3.10+ | Beacon agent, C2 server, feature extraction, ML training, RL agent |
| Flask | Lab C2 server — receives beacon check-ins, logs heartbeats, returns no-op responses |

**Traffic Capture & Analysis:**

| Tool | Purpose in Project |
|------|-------------------|
| tcpdump | Raw packet capture from Docker internal network to PCAP files |
| Zeek (primary) | Flow extraction: conn.log (connection records), http.log (HTTP metadata). Primary pipeline for PCAP → flow conversion |
| Wireshark | Visual packet inspection; validating beacon patterns; comparing pre/post-evasion traffic |
| CICFlowMeter (sanity check only) | Secondary flow extraction for comparison; known flow reassembly bugs make it unsuitable as primary |

**Machine Learning & Analysis:**

| Tool | Purpose in Project |
|------|-------------------|
| scikit-learn | Random Forest, Logistic Regression, train/test splitting, evaluation metrics, permutation importance |
| XGBoost | Gradient-boosted ensemble classifier; comparison against Random Forest |
| SHAP | Feature importance explanation; identifies which flow features drive detection |
| pandas / NumPy | Data manipulation, feature engineering, statistical computations |
| matplotlib / seaborn | Confusion matrices, ROC curves, detection rate plots, feature importance visualizations |
| Jupyter Notebook | Interactive training, evaluation, and visualization workflows |

**Adversarial RL Extension:**

| Tool | Purpose in Project |
|------|-------------------|
| Gymnasium (OpenAI Gym) | RL environment defining the beacon agent's action/state/reward spaces |
| Stable-Baselines3 | PPO and SAC algorithm implementations for the offensive RL agent |
| PyTorch | Neural network backend for RL policy networks and potential deep learning detector |

**Versioning & Reporting:**

| Tool | Purpose in Project |
|------|-------------------|
| Git / GitHub | Version control for code, dataset schema, and configuration files |
| LaTeX or Markdown | Final report generation |

---

## Learning Involved

| Domain | Skills and Knowledge Gained |
|--------|---------------------------|
| **Offensive Security Engineering** | Designing C2 communication protocols. Building parameterized beacon agents with configurable evasion behaviors (jitter, padding, mimicry). Understanding how real-world C2 frameworks (Cobalt Strike, Sliver) structure their callbacks — and replicating the pattern safely. |
| **Network Traffic Analysis** | Deep packet inspection with Wireshark. Flow-level feature extraction using Zeek. Understanding 5-tuple flow definitions, time-windowed aggregation, and the difference between packet-level and flow-level analysis. PCAP processing at scale. |
| **Machine Learning for Security** | Training classifiers on network flow data. Handling class imbalance (SMOTE, class-weight balancing). Preventing data leakage through configuration-based and temporal train/test splits. SHAP-based model interpretability. Understanding why accuracy is misleading and F1/ROC-AUC matter. |
| **Adversarial Machine Learning** | Systematic evasion testing against trained classifiers. Measuring detection degradation per evasion technique. Adversarial retraining cycles and their limits. Understanding the arms race between attacker adaptation and defender retraining. |
| **Reinforcement Learning** | Formulating security problems as RL environments (action spaces, state spaces, reward functions). Training PPO/SAC agents. Self-play dynamics where offensive and defensive agents co-evolve. |
| **Container Networking & Isolation** | Docker Compose networking. Creating air-gapped internal networks. Ensuring no traffic leakage to the host or internet. Multi-container orchestration for lab environments. |
| **Reproducible Experimentation** | Seed-based reproducibility for traffic generation. Version-controlled datasets. Documented schemas. Configuration-driven experiments that others can replicate. |

---

## Scope, Limitations, and Ethical Boundaries

This project is a controlled academic experiment — not a malware development platform. The following boundaries are fixed throughout the project:

- **All traffic is generated and captured exclusively inside an isolated Docker network with no internet route.** No real-world systems, networks, or third-party infrastructure are targeted at any point.

- **The beacon agent sends only harmless heartbeat data** — timestamps and random tokens. It never executes commands on any host, never persists beyond its container lifecycle, never touches real credentials, and never encrypts or exfiltrates real files.

- **The C2 server is a logging-only Flask application.** It records check-in timestamps and returns no-op responses. It has no command execution capability by design.

- **No real malware is used.** The beacon is purpose-built for this project with intentionally limited functionality. It cannot be repurposed for malicious use without fundamental redesign.

- **The agent must not be deployed on any machine the student does not own**, and must not be tested against any external system.

- **A signed lab-scope document** will be obtained from the department in Week 1 before any traffic generation begins.

- **All ML inference and training runs locally.** No data is sent to external AI services or cloud platforms.

- **The project produces detection recommendations for defenders** alongside the evasion findings, ensuring the work contributes to defensive capability, not just offensive knowledge.

### Limitations

- Detection results are specific to the lab environment and the beacon implementation. Generalization to real-world C2 frameworks (Cobalt Strike, Sliver, Mythic) would require testing against those tools, which is out of scope.
- The RL self-play extension is exploratory. If computational constraints prevent convergence, the manual evasion testing (Weeks 11–13) provides the primary results.
- The dataset is synthetic. While this enables controlled experimentation, it may not capture the full noise and diversity of production network traffic.
- CICFlowMeter is used for comparison only due to known flow reassembly bugs; Zeek is the authoritative pipeline.

---

**Name:** Nittin Nobby Mathew
