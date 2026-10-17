# Skills Arsenal

Skills loaded for this project and how they map to the work.

---

## Brainstorming

**Trigger:** Before any creative work — new features, components, architecture decisions, planning.

**Use in project:**
- Designing beacon evasion strategies (jitter distributions, payload padding schemes)
- Planning the C2 protocol (heartbeat format, response structure, command encoding)
- Choosing which novel ML architecture to bolt on (RL self-play vs Mamba vs GNN vs contrastive)
- Designing benign traffic generator profiles that actually mimic real-world patterns
- Feature engineering — deciding which flow features carry the most signal

---

## Systematic Debugging

**Trigger:** Any bug, test failure, or unexpected behavior — before proposing fixes.

**Use in project:**
- Docker network isolation failures (traffic leaking to host)
- Zeek not parsing flows correctly from PCAPs
- ML model producing suspiciously high accuracy (leakage detection)
- Beacon agent connection failures or timing drift
- Feature extraction pipeline producing NaN/missing values
- SMOTE or class-weight balancing producing unexpected distributions

---

## Improve Codebase Architecture

**Trigger:** When the codebase needs structural cleanup or better organization.

**Use in project:**
- Keeping the agent parameterization clean (config-driven, not hardcoded)
- Structuring the feature extraction pipeline (PCAP → Zeek logs → CSV → model input)
- Organizing the evasion toggles so each technique is composable
- Making the adversarial retraining loop modular (swap models, swap evasion configs)
- Ensuring the Docker Compose setup scales when adding new generators or beacon configs

---

## Wireshark Analysis

**Trigger:** Analyzing network traffic, capturing packets, filtering PCAPs, investigating suspicious traffic.

**Use in project:**
- Validating beacon traffic patterns in captured PCAPs
- Confirming benign generators produce distinct traffic profiles
- Following TCP/UDP streams to verify C2 heartbeat format on the wire
- Detecting anomalies in evasion variant captures
- Protocol analysis of HTTP/HTTPS beacon check-ins
- Comparing pre-evasion vs post-evasion traffic side by side
- Verifying Docker network isolation (no unexpected external traffic)

---

## Skill-to-Phase Mapping

| Phase | Skills Used |
|-------|-------------|
| Phase 1: Lab Setup (Weeks 1-3) | Brainstorming, Improve Architecture |
| Phase 2: Data Collection (Weeks 4-6) | Wireshark Analysis, Systematic Debugging |
| Phase 3: Feature Engineering + Detector (Weeks 7-10) | Brainstorming, Systematic Debugging, Improve Architecture |
| Phase 4: Evasion Testing (Weeks 11-14) | Brainstorming, Wireshark Analysis, Systematic Debugging |
| Phase 5: Write-Up (Weeks 15-17) | Improve Architecture |
