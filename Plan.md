# Full Project Plan: Botnet / C2 Traffic Detection Using Machine Learning

**Duration:** 17 weeks  
**Difficulty:** Intermediate to Advanced  
**Track:** Offensive build, with the detector as the measured target

Your college project is a detection project, so this plan builds the offensive side first (a benign C2 beacon you control), then uses the classifier as the target you test evasion against. The output is a red-team study that also meets the detection requirement.

---

## Project Concept

A botnet agent checks in with its command server on a timer. Real C2 traffic often looks normal: regular HTTPS requests with small payloads, similar to software updaters or chat apps. The project asks whether a classifier can separate beacon traffic from normal traffic, and how much an operator can change the beacon before the classifier fails.

**Research question:** How robust is an ML classifier that detects C2 beacons based on flow features, and which beacon parameters (jitter, sleep interval, payload size, transport) are most effective at evading it?

---

## Ethics and Containment (Non-Negotiable)

- All traffic stays inside an isolated Docker network or VM host-only network with no internet route.
- The beacon agent sends only harmless heartbeat data (timestamps and random tokens). It never runs commands on any host, never persists, never touches real credentials, and never encrypts or exfiltrates real files.
- The C2 server is your own lab Flask or Python server that only logs check-ins.
- Do not deploy the agent on any machine you do not own, and do not test against any external system.
- Keep a written lab-scope document and get your department's sign-off in Week 1.

---

## Architecture

```
[Beacon Agent (Docker)] ---HTTPS/HTTP---> [Lab C2 Server (Docker)]
          |                                         |
          +------ Capture point (tcpdump/Zeek) -----+
                              |
                    [Feature extraction]
                              |
                    [ML Detector (scikit-learn)]
                              |
              [Evasion variants -> retest loop]
```

Normal traffic comes from a separate set of Docker containers that simulate benign behavior (a software updater, a REST polling client, a browser-like HTTP client doing page loads).

---

## Phase 1: Foundation and Lab Setup (Weeks 1–3)

**Week 1: Scope and environment**

- Write the lab-scope and ethics document, get approval.
- Set up Docker Compose with an isolated internal network, a C2 container, an agent container, and a capture container running tcpdump or Zeek.
- Confirm there is no route to the internet from the lab network.

**Week 2: Baseline beacon**

- Build a benign beacon agent in Python that checks in every fixed interval (for example, 60 seconds) and posts a small JSON heartbeat.
- Build the C2 server (Flask) that logs each check-in and returns a no-op response.
- Capture your first traces and confirm they appear in Wireshark.

**Week 3: Normal traffic generators**

- Build at least three benign generators: periodic API polling, bursty browser-style page loads, and a software updater pattern.
- Confirm each generator's traffic looks different from the beacon at the packet level.

**Milestone:** Lab runs end to end. Beacon and benign traffic are captured in separate PCAP files.

---

## Phase 2: Data Collection (Weeks 4–6)

**Week 4: Define labels and capture plan**

- Label every flow: malicious (beacon) or benign (generator type).
- Decide on flow definition: 5-tuple plus a time window (for example, 60-second windows per flow).

**Week 5: Vary the beacon**

- Parameterize the agent: sleep interval, jitter percentage (0%, 10%, 30%, 50%), payload size distribution, transport (HTTP vs HTTPS), and user-agent strings.
- Generate a grid of beacon configurations. Aim for at least 15 configurations.

**Week 6: Build the dataset**

- Run each configuration for a fixed duration (for example, 2 hours) and export PCAPs.
- Convert PCAPs to flow records using Zeek (conn.log, http.log) — prefer Zeek over CICFlowMeter (CICFlowMeter has known flow reassembly bugs).
- Store the labeled dataset as CSV with a documented schema and a README.

**Milestone:** A labeled dataset with at least 15 beacon configurations and 3 benign traffic types, versioned in Git.

---

## Phase 3: Feature Engineering and Baseline Detector (Weeks 7–10)

**Week 7: Feature design**

Extract features per flow window:

- **Timing:** mean and std of inter-arrival times, coefficient of variation, autocorrelation at lags 1–5 (drop if time is tight — mean/std/CoV is where the signal is).
- **Size:** mean, variance, and entropy of packet and payload sizes; count of distinct sizes.
- **Connection:** number of connections per window, duration, bytes up/down, up/down ratio.
- **HTTP-level (if available):** URI length, header count, user-agent entropy, POST vs GET ratio.

**Week 8: Baseline models**

- **Statistical baseline first:** z-score detector on inter-arrival time mean (±2σ). If ML can't beat this, that's a finding.
- Train Random Forest, XGBoost, and logistic regression with stratified train/test splits.
- Split by beacon configuration, not by random rows, to avoid leakage between near-identical windows.
- Also do a temporal split: train on earlier captures, test on later ones.
- Apply class-weight balancing or SMOTE — benign traffic will outnumber beacon traffic.

**Week 9: Evaluation**

- Report precision, recall, F1 per class, ROC-AUC, and false-positive rate per normal traffic type.
- Build a confusion matrix broken down by jitter level. Jitter is expected to be the main factor.

**Week 10: Feature importance**

- Use permutation importance and SHAP values to find which features matter most.
- Identify features that a red team can easily change versus features that are hard to change (this matters for Phase 4).

**Milestone:** A working baseline detector with documented metrics and feature importance results.

---

## Phase 4: Evasion Testing (Weeks 11–14)

This phase is the red-team core of the project.

**Week 11: Evasion hypotheses**

Write down testable hypotheses, for example:

- H1: Increasing jitter above 30% drops recall below 50%.
- H2: Randomized payload padding removes size-based features.
- H3: Long sleep intervals with bursty check-ins evade timing features.

**Week 12: Evasion implementation**

Modify the agent to implement each evasion technique as a toggle:

- Jitter randomization with a distribution such as log-normal or Pareto.
- Payload padding to fixed or random size buckets.
- Check-in batching and sleep-wake patterns that mimic a human user.
- Protocol mimicry, such as using a real update-service request shape.

**Week 13: Measure**

- Rerun the captured traffic for each evasion variant through the detector.
- Record detection rate per technique, individually and in combination.
- Track which evasion methods cost the most in operational terms (higher latency, more bandwidth, more connections).

**Week 14: Adversarial retraining test**

- Retrain the detector on evasion-variant data and measure how much recall recovers.
- Document the cycle: attack → retrain → re-attack. This shows how fragile the detector is under adaptation.

**Milestone:** A table showing detection rate for each evasion technique and combination, with the cost of each.

---

## Phase 5: Write-Up and Packaging (Weeks 15–17)

**Week 15: Report**

Structure: abstract, introduction, threat model, lab design, dataset, features, detector results, evasion results, limitations, ethics statement.

**Week 16: Artifacts**

- Clean up the repository: Docker Compose, agent code, C2 server, feature extraction scripts, training notebooks, dataset README.
- Include a reproducibility script that rebuilds the dataset from a fixed seed.

**Week 17: Presentation and review**

- Prepare a 15-minute presentation covering the offensive build, the detection baseline, and the evasion results.
- Prepare a short list of detection recommendations for defenders, since the college rubric likely expects them.

**Milestone:** Submitted report, code repository, dataset, and presentation.

---

## Deliverables Checklist

- [ ] Signed lab-scope and ethics document
- [ ] Docker lab with isolated network
- [ ] Benign beacon agent with parameterized evasion toggles
- [ ] Lab C2 server (logging only)
- [ ] Three benign traffic generators
- [ ] Labeled flow dataset (15+ configurations)
- [ ] Feature extraction pipeline
- [ ] Statistical baseline detector (z-score on IAT)
- [ ] ML baseline detectors with metrics
- [ ] Evasion test results table
- [ ] Adversarial retraining results
- [ ] Final report and presentation
- [ ] Reproducibility script and README

---

## Tools

| Purpose | Tool |
|---------|------|
| Lab and isolation | Docker Compose, internal Docker networks |
| Agent and C2 | Python, Flask, requests |
| Capture | tcpdump, Wireshark, Zeek |
| Flow extraction | Zeek conn.log / http.log (primary), CICFlowMeter (sanity check only) |
| ML | pandas, NumPy, scikit-learn, XGBoost, SHAP |
| Visualization | matplotlib, seaborn |
| Versioning | Git |

---

## Risks and Mitigation

| Risk | Mitigation |
|------|------------|
| Dataset too clean, detector looks unrealistically good | Add more benign generators and noise traffic; report results per benign type |
| Data leakage between training and test windows | Split by configuration AND by time window, not by row |
| Class imbalance (benign >> malicious) | Apply class-weight balancing or SMOTE; report F1 per class, not overall accuracy |
| Scope creep into real malware behavior | Keep the agent limited to heartbeats; review scope document weekly |
| Results depend on one lab setup | Document the setup; rerun with a second seed or traffic mix |
| CICFlowMeter flow reassembly errors | Use Zeek as primary pipeline; CICFlowMeter for comparison only |
