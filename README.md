# Facility Location MILP

An interactive tool for the capacitated facility location problem. Enter your own customers, facilities, costs and capacities, and it finds the lowest-cost set of facilities to open and how much to ship from each one.

**Live demo:** [https://Kaartikchugh.github.io/Facility-Location-MILP/](https://kaartikchugh18.github.io/Facility-Location-MILP/)

## What it does

- **Single facility mode:** each facility has one capacity level and one fixed cost.
- **Dual facility mode:** each facility can open at low capacity, open at high capacity, or stay closed.
- Editable tables for unit transportation costs, customer demand, fixed costs and capacities.
- Generates the full mathematical formulation for your data.
- Draws the facility-to-customer network and highlights the optimal flows.
- Solves to optimality and reports total, fixed and transport cost.

## Model

Minimize total fixed cost plus transportation cost, subject to:

- Every customer's demand is met exactly.
- Each facility ships no more than its capacity, and only if it is open.
- Open indicators are binary (Y ∈ {0,1}) and flows are non-negative.

## How it solves

The solver checks every possible opening plan. For each plan it finds the cheapest shipping flow using a min-cost flow algorithm, then keeps the best plan. It handles up to 8 facilities and 8 customers.

## Run it locally

Download `index.html` and open it in any browser. No installation is needed.
