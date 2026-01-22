# Bioethanol Supply Chain Network Optimization

A mixed-integer linear programming (MILP) model for designing an optimal bioethanol supply chain network in Texas. The model determines facility locations and material flows to minimize total cost while meeting demand constraints.

![Python](https://img.shields.io/badge/Python-3.8+-blue.svg)
![PuLP](https://img.shields.io/badge/Solver-PuLP%20(CBC)-green.svg)
![License](https://img.shields.io/badge/License-MIT-yellow.svg)

## Problem Overview

The bioethanol industry requires efficient logistics to transport biomass from agricultural suppliers to processing plants. This project optimizes:

- **Where to build hubs** (consolidation points for biomass)
- **Where to build plants** (bioethanol production facilities)  
- **How much to ship** along each route
- **How much to buy** from third-party suppliers

### Network Structure

```
Suppliers (254) ──[Truck]──> Hubs (33) ──[Train]──> Plants (167)
                                                         │
                                                         ▼
                                              Third-Party Supply
```

## Mathematical Model

### Decision Variables

| Variable | Type | Description |
|----------|------|-------------|
| yⱼ | Binary | 1 if hub j is opened, 0 otherwise |
| zₖ | Binary | 1 if plant k is opened, 0 otherwise |
| Sᵢⱼ | Continuous | Biomass flow from supplier i to hub j (Mg) |
| Sⱼₖ | Continuous | Biomass flow from hub j to plant k (Mg) |
| Sₜ | Continuous | Third-party supply quantity (Mg) |

### Objective Function

Minimize total cost:

```
min Z = Σⱼ(fⱼ·yⱼ) + Σₖ(gₖ·zₖ) + Σᵢⱼ(Cᵢⱼ·Sᵢⱼ) + Σⱼₖ((Cⱼₖ + lⱼₖ)·Sⱼₖ) + G·Sₜ
```

Where:
- fⱼ = Hub investment cost ($3.48M)
- gₖ = Plant investment cost ($130.96M)
- Cᵢⱼ = Truck transportation cost from i to j
- Cⱼₖ = Train transportation cost from j to k
- lⱼₖ = Loading/unloading cost ($9.07/Mg)
- G = Third-party price (sensitivity parameter)

### Constraints

1. **Supply Limit**: Flow from supplier cannot exceed available supply
   ```
   Σⱼ Sᵢⱼ ≤ Oᵢ  for all i
   ```

2. **Hub Capacity**: Inflow to hub limited by capacity (only if open)
   ```
   Σᵢ Sᵢⱼ ≤ qⱼ·yⱼ  for all j
   ```

3. **Plant Capacity**: Inflow to plant limited by capacity (only if open)
   ```
   Σⱼ Sⱼₖ ≤ pₖ·zₖ  for all k
   ```

4. **Flow Balance**: What enters a hub must leave
   ```
   Σᵢ Sᵢⱼ ≥ Σₖ Sⱼₖ  for all j
   ```

5. **Train Capacity**: Each rail link has maximum throughput
   ```
   Sⱼₖ ≤ tⱼ  for all j,k
   ```

6. **Demand Satisfaction**: Must meet total demand
   ```
   Σⱼₖ Sⱼₖ + Sₜ ≥ D
   ```

## Parameters

| Parameter | Value | Description |
|-----------|-------|-------------|
| fⱼ | $3,476,219 | Hub investment cost |
| gₖ | $130,956,797 | Plant investment cost |
| qⱼ | 300,000 Mg | Hub capacity |
| pₖ | 655,447 Mg | Plant capacity |
| tⱼ | 338,000 Mg | Train capacity per route |
| lⱼₖ | $9.07/Mg | Loading/unloading cost |
| D | 6,363,408 Mg | Total demand |

## Key Findings

### Supply Gap Analysis

| Metric | Value |
|--------|-------|
| Total Demand | 6,363,408 Mg |
| Local Supply Available | 3,053,378 Mg |
| **Supply Gap** | **3,310,030 Mg (52%)** |

**Critical insight**: Local suppliers can only provide 48% of required demand. Third-party purchases are mandatory.

### Sensitivity Analysis Results

The model was solved for third-party prices ranging from $50 to $2,000/Mg:

| Third-Party Price | Total Cost | Own Production | Third-Party | Hubs | Plants |
|-------------------|------------|----------------|-------------|------|--------|
| $50/Mg | $0.32B | 0% | 100% | 0 | 0 |
| $250/Mg | $1.59B | 0% | 100% | 0 | 0 |
| $500/Mg | $2.46B | 48% | 52% | 11 | 5 |
| $1,000/Mg | $4.12B | 48% | 52% | 11 | 5 |
| $2,000/Mg | $7.43B | 48% | 52% | 11 | 5 |

**Break-even point**: Around $250-300/Mg. Below this, 100% third-party is cheapest. Above this, maximize local production.

## Installation

```bash
# Clone the repository
git clone https://github.com/mknishat/SC_Network_gurobi.git
cd SC_Network_gurobi

# Install dependencies
pip install pulp pandas numpy matplotlib folium
```

## Usage

1. Open `Main .ipynb` in Jupyter Notebook or VS Code
2. Run all cells sequentially
3. View results:
   - `sensitivity_analysis.png` - Cost and network design charts
   - `supply_chain_map.html` - Interactive map of optimal network

## Project Structure

```
SC_Network_gurobi/
├── Main .ipynb              # Main optimization notebook
├── Testing.ipynb            # Testing and validation
├── TX_suppliers.csv         # Supplier locations and capacities
├── TX_hubs.csv              # Potential hub locations
├── TX_plants.csv            # Potential plant locations
├── TX_roads.csv             # Truck routes and costs
├── TX_railroads.csv         # Train routes and costs
├── TX_network.csv           # Network topology
├── sensitivity_analysis.png # Output: sensitivity charts
├── supply_chain_map.html    # Output: interactive map
├── model formulation.pdf    # Mathematical formulation
├── Case Study Descriptions.pdf
└── README.md
```

## Data Files

| File | Records | Description |
|------|---------|-------------|
| TX_suppliers.csv | 254 | Biomass suppliers (counties) with supply quantities |
| TX_hubs.csv | 33 | Potential hub locations with coordinates |
| TX_plants.csv | 167 | Potential plant locations with coordinates |
| TX_roads.csv | ~8,000 | Truck routes with transportation costs |
| TX_railroads.csv | ~5,500 | Train routes with costs and loading fees |

## Outputs

### 1. Sensitivity Analysis Charts

Four-panel visualization showing:
- Total cost vs third-party price
- Supply mix (own vs purchased)
- Cost breakdown by component
- Number of facilities opened

### 2. Interactive Map

HTML map showing:
- 🔵 Selected hub locations
- 🔴 Selected plant locations
- ⚫ Potential (not selected) locations

## Solver

This project uses **PuLP** with the **CBC (COIN-OR Branch and Cut)** solver:

- Free and open-source
- No license required
- Handles MILP problems with ~200 binary variables and ~14,000 continuous variables
- Typical solve time: 3-5 minutes per scenario

## Assumptions

1. Single time period (annual planning horizon)
2. Deterministic demand and supply
3. Homogeneous hub and plant capacities
4. Transportation costs are linear in quantity
5. No inventory holding between periods

## Limitations

- Does not account for demand uncertainty
- Single-period model (no multi-year planning)
- Assumes unlimited third-party supply availability
- Does not model detailed facility construction timelines

## Future Work

- [ ] Stochastic programming for demand uncertainty
- [ ] Multi-period planning with capacity expansion
- [ ] Carbon footprint optimization
- [ ] Detailed facility siting with land-use constraints
- [ ] Integration with real-time market prices

## References

1. Texas biomass supply chain data
2. COIN-OR CBC Solver Documentation
3. PuLP Linear Programming Library

## Author

**Md. Mahbubul Nishat**  
GitHub: [@mknishat](https://github.com/mknishat)

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
