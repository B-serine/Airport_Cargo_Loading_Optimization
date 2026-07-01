# Airport Cargo Loading Optimization

Two mathematical formulations, one exact solver, and two metaheuristics compared on the 3D bin-packing problem of loading heterogeneous cargo into aircraft compartments.

## Overview

Airlines must assign every cargo item to an aircraft compartment before departure, minimizing total handling cost while respecting weight, volume, safety, and balance constraints. Forward compartments are cheaper to load than rear ones, so cost grows with distance from the loading door.

**Project scope:** 13 benchmark instances · 2 mathematical formulations · 4 algorithms tested · NP-hard problem complexity.

## Problem Constraints

| ID | Name | Description |
|----|------|-------------|
| C1 | Assignment | Every item goes to exactly one compartment |
| C2 | Weight Cap | Total weight ≤ compartment limit |
| C3 | Volume Cap | Total volume ≤ compartment capacity |
| C4 | Heavy Items | Heavy cargo restricted to the lower deck |
| C5 | Incompatibility | Some item pairs cannot share a bin |
| C6 | Co-location | Some item pairs must share a bin |
| C7–C8 | Balance | Center of mass within safety tolerance |

## Dataset

- **13 benchmark instances** (3dBPP_1 → 3dBPP_test)
- **6 feasible** (≥2 bins, sufficient capacity)
- **7 infeasible** (single bin or volume exceeded)

Feasible instances range from 38–53 items across 2 bins, with varying incompatibility and co-location pair counts.

## Mathematical Formulations

### 1. Classical MILP
- Binary decision variables: `x_ij` (item i → compartment j), `y_j` (compartment j used)
- Objective: minimize handling cost + a small bin-usage penalty
- Encodes constraints C1–C9 as linear (in)equalities, including weighted center-of-mass balance

### 2. Graph-Based CP Model
- Builds a conflict graph where edges connect items that cannot share a bin (explicit incompatibilities, weight-infeasible pairs, volume-infeasible pairs)
- Integer variable `x_i ∈ {0,…,m-1}` per item
- Solves via domain propagation and arc-consistency (no LP relaxation)
- Chromatic lower bound gives the minimum bins required instantly

**Trade-off:** MILP is stronger for finding provably optimal solutions on small instances; Graph-CP is faster for pruning and pre-processing.

## Algorithms

All algorithms share an **integer assignment vector** encoding (`x_i` = compartment for item i) and a common set of operators: `move_item`, `swap_items`, `or_opt(k)`, and `perturbation`, plus a 6-pass repair heuristic (heavy items → incompatibility → co-location → weight overload → balance → cost-improving).

| Method | Type | Key Idea |
|--------|------|----------|
| **Branch & Bound** | Exact | Most-constrained-item branching, admissible lower bound, greedy warm start; guaranteed optimal but scales poorly beyond ~100 items |
| **Genetic Algorithm (GA)** | Population-based | Tournament-3 selection, adaptive uniform/single-point crossover, adaptive mutation (0.08→0.02), diversity-triggered immigration, 60% post-crossover repair |
| **Simulated Annealing (SA)** | Local search | Violation-aware neighborhood (swap when incompatibilities are active, move otherwise), geometric cooling (α=0.9985), up to 3 reheats and 3 restarts |
| **MILP Solver** | Exact | Direct solve of the classical formulation |

## Experimental Results

**Protocol:** 13 instances, 4 methods, 60s time budget for exact methods / 30s for metaheuristics, fixed seeds.

| Method | Family | Avg Rank | Mean Gap% | Wins (of 6) | Feasibility | Mean Time |
|--------|--------|----------|-----------|-------------|-------------|-----------|
| MILP | Exact | 1.17 | 0.00% | 5 | 46% | < 1 s |
| SA | Local Search | 1.50 | 0.00% | 5 | 46% | 30 s |
| GA | Population | 1.83 | 0.56% | 3 | 44% | 30 s |
| B&B | Exact | 3.50 | 5.5% | 2 | 38% | 60 s |

**Key findings:**
- MILP is the strongest overall performer on this dataset.
- SA is nearly as strong and matches MILP exactly on 5 of 6 feasible instances (0.00% gap).
- GA is solid but trails SA/MILP slightly in rank.
- B&B is the weakest performer, mainly due to lower feasibility rates within the time limit.

## Conclusions

1. **Exact methods** (MILP, B&B) are reliable and optimal for small instances (n ≤ 60) but not suitable for large-scale or real-time use.
2. **Simulated Annealing** is the most robust metaheuristic — best default choice for larger or real-time scenarios, thanks to auto-calibrated cooling/reheating and violation-aware moves.
3. **Genetic Algorithm** performs well on multi-bin instances with maintained diversity, but is less competitive on tightly constrained single-bin cases.
4. **The repair heuristic is critical** — multiple repair passes substantially improve feasibility on tight instances.

---
*NMO Group Project — Airport Cargo Loading Optimization*
