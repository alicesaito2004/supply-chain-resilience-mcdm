# Supply Chain Resilience MCDM Framework

A multi-criteria decision analysis framework for evaluating supply chain configurations under cyber disruption risk.

## Research Question

**How should organizations configure their supply chains to balance cost efficiency with resilience against cyber disruptions?**

---

## Key Findings

| Condition | CFO Recommendation | COO Recommendation | CMO Recommendation |
|-----------|-------------------|-------------------|-------------------|
| Normal | C3 (Dual/Lean/None) | C3 (Dual/Lean/None) | C3 (Dual/Lean/None) |
| Disrupted | C3 (Dual/Lean/None) | **C7 (Multi/Lean/Partial)** | C3 or C7 |

### Headline Insight
> "The optimal supply chain configuration depends on stakeholder priorities: cost-focused executives prefer lean dual-sourcing (C3), while operations-focused executives should adopt multi-sourcing with partial IT redundancy (C7) when disruption risk is considered."

### Key Takeaways
1. **Dual-sourcing (C3) is the baseline** for cost-conscious organizations
2. **Multi-sourcing with partial IT (C7)** is preferred when operational continuity matters
3. **IT redundancy alone (C5) is insufficient** — supply diversification matters more
4. **Recommendations are robust** to cyber probability assumptions (1-20%)
5. **Cost priority is the key variable** — thresholds at 5%, 30%, 40% weight

---

## Supply Chain Network

```
Suppliers          Factories       Warehouses       Retailers
   S1 ─────┬──────► F1 ──────┬────► W1 ──────┬────► R1
   S2 ─────┼──────► F2 ──────┼────► W2 ──────┼────► R2
   S3 ─────┘                 └────► W3 ──────┼────► R3
                                             ├────► R4
                                             └────► R5
```

- **3 Suppliers**: S1 (primary, 120 units), S2 (secondary, 100 units), S3 (tertiary, 60 units)
- **2 Factories**: F1, F2 (150 unit capacity each)
- **3 Warehouses**: W1, W2, W3 (100-120 unit capacity)
- **5 Retailers**: R1-R5 (total demand: 200 units)

---

## The 8 Configurations

| Config | Sourcing | Inventory | IT Redundancy | Description | Supply |
|--------|----------|-----------|---------------|-------------|--------|
| C1 | Single | Lean | None | Cost-minimized | 120 |
| C2 | Single | Buffered | None | Inventory hedge | 120 |
| C3 | Dual | Lean | None | Supplier hedge | 220 |
| C4 | Dual | Buffered | None | Hedged operations | 220 |
| C5 | Single | Lean | Full | Cyber-resilient | 120 |
| C6 | Dual | Buffered | Full | Maximum resilience | 220 |
| C7 | Multi | Lean | Partial | Diversified | 280 |
| C8 | Multi | Buffered | Partial | Balanced | 280 |

### Configuration Dimensions

**Sourcing Strategy:**
- Single: Only S1 active (120 units) — lowest cost, highest risk
- Dual: S1 + S2 active (220 units) — balanced
- Multi: All suppliers active (280 units) — highest resilience

**Inventory Policy:**
- Lean: No buffer stock — lower holding cost, less shock absorption
- Buffered: Safety stock at warehouses — higher cost, better demand surge handling

**IT Redundancy:**
- None: No backup systems — vulnerable to cyberattack
- Partial: Some redundancy — moderate protection
- Full: Complete backup — best cyber protection, highest cost

---

## Disruption Scenarios

| Scenario | Probability | Effect | Mitigation |
|----------|-------------|--------|------------|
| **Cyberattack** | 5% | Warehouse capacity reduced to 10% | IT redundancy |
| **Supplier Failure** | 5% | Primary supplier (S1) goes offline | Multi-sourcing |
| **Demand Surge** | 4% | 50% demand increase | Buffer inventory |
| **Compound** | 1% | Cyber + supplier failure together | All mitigations |
| Normal | 85% | No disruption | — |

### Real-World Motivation
- **Asahi Group (Jan 2025)**: Ransomware paralyzed ordering systems across 30+ factories
- **Toyota/Kojima (2022)**: Supplier cyberattack halted 14 assembly plants for days
- **Change Healthcare (2024)**: $25M ransom, months of healthcare claims disruption
- **CDK Global (2024)**: 3,000+ auto dealerships offline for weeks

---

## SCOR Evaluation Criteria

| Criterion | Measure | Direction | Source |
|-----------|---------|-----------|--------|
| **Cost** | Total cost to serve ($) | Lower is better | Flow solver + fixed costs |
| **Reliability** | Fill rate (%) | Higher is better | Flow solver (satisfied/demand) |
| **Recovery Time** | Days to 95% service | Lower is better | Disruption model |
| **Financial Exposure** | Expected loss ($) | Lower is better | Shortfall × penalties |
| **Asset Efficiency** | Throughput / cost | Higher is better | Computed ratio |

### Sample Results (Normal Condition)

| Config | Cost | Reliability | Recovery | Fin. Exposure | Asset Eff. |
|--------|------|-------------|----------|---------------|------------|
| C1 | $720 | 60.0% | 0 days | $1,200 | 0.167 |
| C3 | $1,280 | 100.0% | 0 days | $0 | 0.156 |
| C6 | $1,918 | 100.0% | 0 days | $0 | 0.104 |
| C7 | $1,550 | 100.0% | 0 days | $0 | 0.129 |

---

## MCDM Methods

### 1. WSM (Weighted Sum Model)
- **Type**: Compensatory, additive
- **Formula**: Score = Σ(weight × normalized_value)
- **Behavior**: High score on one criterion can offset low score on another

### 2. WPM (Weighted Product Model)
- **Type**: Compensatory, multiplicative
- **Formula**: Score = Π(value^weight)
- **Behavior**: More punishing of weak criteria (multiplying by small numbers)

### 3. TOPSIS (Technique for Order Preference by Similarity to Ideal Solution)
- **Type**: Distance-based
- **Process**: 
  1. Normalize using vector normalization
  2. Find ideal (best on all) and anti-ideal (worst on all)
  3. Compute Euclidean distance to both
  4. Closeness = d_anti / (d_ideal + d_anti)
- **Behavior**: Susceptible to rank reversal when alternatives added/removed

### 4. PROMETHEE II (Preference Ranking Organization Method)
- **Type**: Outranking, pairwise comparison
- **Process**:
  1. Compare each pair of alternatives on each criterion
  2. Apply preference function (V-shape with thresholds)
  3. Compute positive flow (dominance) and negative flow (dominated)
  4. Net flow = positive - negative
- **Behavior**: Considers pairwise consistency, not just distance to ideal

### Method Agreement
When all 4 methods agree → **robust recommendation**
When methods disagree → investigate structural differences

---

## Stakeholder Profiles

| Criterion | CFO | COO | CMO |
|-----------|-----|-----|-----|
| Cost | **40%** | 15% | 10% |
| Reliability | 15% | **30%** | **35%** |
| Recovery | 10% | **25%** | 15% |
| Fin. Exposure | **25%** | 15% | 15% |
| Asset Efficiency | 10% | 15% | **25%** |

**CFO (Chief Financial Officer)**: Prioritizes cost minimization and limiting financial risk

**COO (Chief Operating Officer)**: Prioritizes operational reliability and fast recovery

**CMO (Chief Marketing Officer)**: Prioritizes customer experience (reliability) and brand protection

---

## Sensitivity Analysis

### Probability Sensitivity
**Question**: Does the recommendation change if cyber risk is 1% vs 20%?

**Finding**: No. C7 remains optimal for COO across entire range tested.

→ **Robust to probability assumptions**

### Weight Sensitivity
**Question**: How does cost prioritization affect the recommendation?

**Finding** (COO, disrupted condition):
| Cost Weight | Optimal Config |
|-------------|----------------|
| < 5% | C8 (Balanced) |
| 5-30% | C7 (Diversified) |
| 30-40% | C3 (Dual Lean) |
| > 40% | C1 (Cost-minimized) |

→ **Sensitive to cost priorities** — the 30% threshold is critical

---

## Visualizations

| File | Description |
|------|-------------|
| `network_topology.png` | Supply chain network structure |
| `configuration_comparison.png` | Single vs dual vs multi sourcing |
| `all_configurations.png` | All 8 configurations in grid |
| `disruption_impact.png` | C3 vs C6 under cyberattack and supplier failure |
| `criteria_comparison.png` | Bar chart of criteria scores by configuration |
| `best_config_summary.png` | Summary table of best config by stakeholder × condition |
| `ranking_heatmap.png` | Full 24 rankings (4 methods × 3 stakeholders × 2 conditions) |
| `ranking_shift.png` | How rankings change from normal to disrupted |
| `probability_sensitivity.png` | COO scores vs cyber probability |
| `probability_sensitivity_cfo.png` | CFO scores vs cyber probability |
| `weight_sensitivity_cost.png` | COO scores vs cost weight |
| `weight_sensitivity_reliability.png` | COO scores vs reliability weight |

---

## Project Structure

```
supply-chain-resilience-mcdm/
│
├── README.md                 # This file
├── requirements.txt          # Python dependencies
├── PRESENTATION_GUIDE.md     # Slide-by-slide presentation guide
│
├── models.py                 # Network, Node, Edge data structures
├── configurations.py         # C1-C8 configuration definitions
├── disruptions.py            # Disruption scenario functions
├── flow_solver.py            # Min-cost flow LP solver
│
├── criteria.py               # SCOR criteria computation
├── mcdm.py                   # WSM, WPM, TOPSIS, PROMETHEE II
├── stakeholders.py           # CFO, COO, CMO weight profiles
│
├── run_analysis.py           # Main analysis pipeline (24 rankings)
├── visualize.py              # Heatmaps and ranking charts
├── sensitivity.py            # Probability and weight sensitivity
├── network_viz.py            # Network topology diagrams
│
└── visuals/                  # Generated visualizations
    ├── network_topology.png
    ├── configuration_comparison.png
    ├── ...
    └── sensitivity_report.txt
```

---

## Installation & Usage

### Requirements
```
numpy
scipy
pandas
matplotlib
seaborn
```

### Install
```bash
pip install -r requirements.txt
```

### Run Full Analysis
```bash
python run_analysis.py
```

Output: 24 rankings (4 methods × 3 stakeholders × 2 conditions)

### Generate Visualizations
```bash
python visualize.py
python sensitivity.py
python network_viz.py
```

### Run Criteria Evaluation
```bash
python criteria.py
```

Output: Decision matrices for all configurations under all scenarios

---

## Methodology Flow

```
1. Build Network (models.py)
         ↓
2. Apply Configuration (configurations.py)
         ↓
3. Apply Disruption (disruptions.py)
         ↓
4. Solve Min-Cost Flow (flow_solver.py)
         ↓
5. Compute SCOR Criteria (criteria.py)
         ↓
6. Build Decision Matrix (criteria.py)
         ↓
7. Apply Stakeholder Weights (stakeholders.py)
         ↓
8. Run MCDM Methods (mcdm.py)
         ↓
9. Generate Rankings (run_analysis.py)
         ↓
10. Sensitivity Analysis (sensitivity.py)
```

---

## Limitations

- **Synthetic data**: Network parameters are illustrative, not from a real company
- **Simplified topology**: Real supply chains have more tiers and complexity
- **Static probabilities**: Disruption probabilities don't change over time
- **Recovery estimation**: Recovery times are heuristic, not simulated
- **Single objective**: Doesn't consider multi-period or dynamic optimization

---

## Future Work

- Apply framework to real company data
- Dynamic disruption modeling (time-varying probabilities)
- Include additional criteria (sustainability, lead time, flexibility)
- Multi-period stochastic optimization
- Integration with enterprise risk management systems

---

## Team

- **Alice** - Criteria functions, report lead
- **Sean** - MCDM methods (TOPSIS, PROMETHEE), sensitivity analysis
- **Meiqi** - Stakeholder profiles, presentation, WSM/WPM
- **Yuantong** - Network model, flow solver, code integration
- **Sekito** - Configurations, disruptions, network visualizations

---

## References

- APICS/ASCM SCOR Model documentation
- Ghanbari et al. (2023) - Supply chain cyber risk assessment
- Crosignani et al. (2023) - Cyberattack propagation in supply networks
- Hwang & Yoon (1981) - TOPSIS method
- Brans & Vincke (1985) - PROMETHEE method

---

## License

MIT License - Academic use permitted with attribution.
