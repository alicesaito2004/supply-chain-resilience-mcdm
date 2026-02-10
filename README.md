# Supply Chain Resilience MCDM Framework

A multi-criteria decision analysis framework for evaluating supply chain configurations under cyber disruption risk.

## Overview

This framework helps organizations answer: **How should we configure our supply chain to balance cost efficiency with resilience against cyber disruptions?**

The pipeline evaluates 8 strategic configurations across 5 SCOR performance criteria using 4 MCDM methods from 3 stakeholder perspectives under normal and disrupted conditions.

---

## Quick Start

```bash
# Install dependencies
pip install -r requirements.txt

# Run full analysis (24 rankings)
python run_analysis.py

# Generate visualizations
python visualize.py
python sensitivity.py
python network_viz.py
```

---

## Architecture

```
┌─────────────────────────────────────────────────────────────────────────┐
│                           DATA LAYER                                    │
├─────────────────────────────────────────────────────────────────────────┤
│  models.py              Network, Node, Edge data structures             │
│  configurations.py      C1-C8 strategic configuration definitions       │
│  disruptions.py         Cyberattack, supplier failure, demand surge     │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                         COMPUTATION LAYER                               │
├─────────────────────────────────────────────────────────────────────────┤
│  flow_solver.py         Min-cost flow LP solver (scipy)                 │
│  criteria.py            SCOR criteria computation from flow results     │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                          ANALYSIS LAYER                                 │
├─────────────────────────────────────────────────────────────────────────┤
│  mcdm.py                WSM, WPM, TOPSIS, PROMETHEE II methods          │
│  stakeholders.py        CFO, COO, CMO weight profiles                   │
│  sensitivity.py         Probability and weight sensitivity sweeps       │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                          OUTPUT LAYER                                   │
├─────────────────────────────────────────────────────────────────────────┤
│  run_analysis.py        Main pipeline - generates 24 rankings           │
│  visualize.py           Heatmaps, ranking charts                        │
│  network_viz.py         Network topology diagrams                       │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## Module Reference

### Data Layer

#### `models.py`
Core data structures for supply chain network representation.

| Class | Description |
|-------|-------------|
| `Node` | Network node with id, type, supply/demand, attributes |
| `Edge` | Directed edge with capacity, cost, flow constraints |
| `Network` | Container for nodes and edges with query methods |

#### `configurations.py`
Defines the 8 strategic supply chain configurations.

| Function | Description |
|----------|-------------|
| `build_configuration(config_id)` | Returns configured Network for C1-C8 |
| `get_config_properties(config_id)` | Returns sourcing/inventory/IT settings |

**Configurations:**
- **C1-C2**: Single source (S1 only)
- **C3-C4**: Dual source (S1 + S2)
- **C5-C6**: Single/Dual with full IT redundancy
- **C7-C8**: Multi source (all suppliers) with partial IT

#### `disruptions.py`
Disruption scenario functions that modify network state.

| Function | Description |
|----------|-------------|
| `apply_disruption(network, type)` | Returns disrupted network copy |
| `apply_cyberattack(network)` | Reduces warehouse capacity |
| `apply_supplier_failure(network)` | Disables primary supplier |
| `apply_demand_surge(network)` | Increases retailer demand 50% |
| `apply_compound(network)` | Cyber + supplier failure |
| `estimate_recovery_time(config_id, type)` | Days to 95% service level |

---

### Computation Layer

#### `flow_solver.py`
Linear programming solver for min-cost network flow.

| Function | Description |
|----------|-------------|
| `solve_min_cost_flow(network)` | Returns optimal flow, cost, fill rate |

**Features:**
- Handles partial demand satisfaction (slack variables)
- Handles excess supply (sink variables)
- Uses scipy.optimize.linprog with HiGHS method

**Returns:**
```python
{
    'success': bool,
    'total_cost': float,
    'fill_rate': float,  # satisfied_demand / total_demand
    'satisfied_demand': float,
    'flows': dict  # edge flows
}
```

#### `criteria.py`
Computes SCOR criteria from flow solver output.

| Function | Description |
|----------|-------------|
| `evaluate_configuration(config_id, scenario)` | Returns CriteriaScores object |
| `build_decision_matrix(configs, scenario)` | Returns (n_configs × 5) numpy array |
| `build_expected_matrix(configs, probs)` | Probability-weighted average matrix |

**SCOR Criteria:**
| Criterion | Direction | Source |
|-----------|-----------|--------|
| Cost | Minimize | Flow cost + fixed infrastructure |
| Reliability | Maximize | Fill rate from flow solver |
| Recovery Time | Minimize | `estimate_recovery_time()` |
| Financial Exposure | Minimize | Shortfall × penalties |
| Asset Efficiency | Maximize | Throughput / cost |

---

### Analysis Layer

#### `mcdm.py`
Four multi-criteria decision making methods.

| Function | Description |
|----------|-------------|
| `wsm(matrix, weights, types)` | Weighted Sum Model |
| `wpm(matrix, weights, types)` | Weighted Product Model |
| `topsis(matrix, weights, types)` | Distance to ideal solution |
| `promethee_ii(matrix, weights, types)` | Outranking with preference flows |

**All methods return:**
```python
MCDMResult(
    scores: np.ndarray,    # Raw scores per alternative
    rankings: List[int],   # 1 = best, n = worst
    method: str
)
```

#### `stakeholders.py`
Stakeholder weight profiles for criteria.

| Function | Description |
|----------|-------------|
| `get_stakeholder(name)` | Returns StakeholderProfile with weights |
| `STAKEHOLDERS` | Dict of all profiles (CFO, COO, CMO) |

**Weight Profiles:**
| Criterion | CFO | COO | CMO |
|-----------|-----|-----|-----|
| Cost | 40% | 15% | 10% |
| Reliability | 15% | 30% | 35% |
| Recovery | 10% | 25% | 15% |
| Fin. Exposure | 25% | 15% | 15% |
| Asset Efficiency | 10% | 15% | 25% |

#### `sensitivity.py`
Sensitivity analysis for probability and weight parameters.

| Function | Description |
|----------|-------------|
| `run_probability_sensitivity(stakeholder, probs, method)` | Vary cyber probability |
| `run_weight_sensitivity(stakeholder, criterion, range, method)` | Vary criterion weight |
| `plot_probability_sensitivity(df, path)` | Generate probability sweep plot |
| `plot_weight_sensitivity(df, path)` | Generate weight sweep plot |

---

### Output Layer

#### `run_analysis.py`
Main pipeline that generates all 24 rankings.

```bash
python run_analysis.py
```

**Output:** Rankings for 4 methods × 3 stakeholders × 2 conditions

#### `visualize.py`
Generates ranking and criteria visualizations.

```bash
python visualize.py
```

**Outputs:**
- `ranking_heatmap.png` - Full 24 rankings
- `ranking_shift.png` - Normal → Disrupted changes
- `criteria_comparison.png` - Criteria by configuration
- `best_config_summary.png` - Summary table

#### `network_viz.py`
Generates network topology diagrams.

```bash
python network_viz.py
```

**Outputs:**
- `network_topology.png` - Supply chain structure
- `configuration_comparison.png` - Single/dual/multi sourcing
- `disruption_impact.png` - Before/after disruption
- `all_configurations.png` - All 8 configs grid

---

## Pipeline Flow

```
1. Build base network
   └── models.py: Network()

2. Apply configuration (C1-C8)
   └── configurations.py: build_configuration()

3. Apply disruption scenario
   └── disruptions.py: apply_disruption()

4. Solve min-cost flow
   └── flow_solver.py: solve_min_cost_flow()

5. Compute SCOR criteria
   └── criteria.py: evaluate_configuration()

6. Build decision matrix
   └── criteria.py: build_decision_matrix()

7. Apply stakeholder weights
   └── stakeholders.py: get_stakeholder()

8. Run MCDM methods
   └── mcdm.py: wsm(), wpm(), topsis(), promethee_ii()

9. Generate rankings
   └── run_analysis.py

10. Sensitivity analysis
    └── sensitivity.py
```

---

## Project Structure

```
supply-chain-resilience-mcdm/
│
├── README.md                 # This file
├── RESULTS.md                # Results summary and findings
├── PRESENTATION_GUIDE.md     # Slide-by-slide guide
├── requirements.txt          # Python dependencies
│
├── models.py                 # Network data structures
├── configurations.py         # C1-C8 definitions
├── disruptions.py            # Disruption scenarios
├── flow_solver.py            # LP solver
├── criteria.py               # SCOR criteria
├── mcdm.py                   # MCDM methods
├── stakeholders.py           # Weight profiles
├── sensitivity.py            # Sensitivity analysis
├── run_analysis.py           # Main pipeline
├── visualize.py              # Result charts
├── network_viz.py            # Network diagrams
│
└── visuals/                  # Generated outputs
    ├── network_topology.png
    ├── ranking_heatmap.png
    ├── ...
    └── sensitivity_report.txt
```

---

## Requirements

```
numpy
scipy
pandas
matplotlib
seaborn
```

Install: `pip install -r requirements.txt`

---

## Usage Examples

### Run single configuration evaluation
```python
from configurations import build_configuration
from disruptions import apply_disruption, DisruptionType
from flow_solver import solve_min_cost_flow

net = build_configuration("C3")
disrupted = apply_disruption(net, DisruptionType.CYBERATTACK)
result = solve_min_cost_flow(disrupted)
print(f"Fill rate: {result['fill_rate']:.1%}")
```

### Build decision matrix
```python
from criteria import build_decision_matrix, Scenario

configs = ["C1", "C2", "C3", "C4", "C5", "C6", "C7", "C8"]
matrix = build_decision_matrix(configs, Scenario.NORMAL)
print(matrix)  # Shape: (8, 5)
```

### Run MCDM analysis
```python
from mcdm import topsis, CriterionType
from stakeholders import get_stakeholder

weights = get_stakeholder("COO").weights
types = [CriterionType.COST, CriterionType.BENEFIT, 
         CriterionType.COST, CriterionType.COST, CriterionType.BENEFIT]

result = topsis(matrix, weights, types)
print(f"Rankings: {result.rankings}")
```

---

## Team

| Member | Responsibility |
|--------|----------------|
| Alice | Criteria functions, report |
| Sean | MCDM methods, sensitivity |
| Meiqi | Stakeholders, presentation |
| Yuantong | Network model, flow solver |
| Sekito | Configurations, disruptions, visuals |

---

## License

MIT License - Academic use permitted with attribution.
