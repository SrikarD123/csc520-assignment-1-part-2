# csc520-assignment-1-part-2
This is a repository for the Spring 2026's CSC 520 section submission of Assignment 1 Part 2. This contains the work for both Srikar Desemsetti and Lawrence Arkoh.

# Underwater Submarine Rescue — CSC520 Assignment 1

A state-space search implementation in **SNAP! 11.0.8** featuring BFS, DFS, UCS, and A* in a unified engine. A submarine must collect three emergency beacons and dock at an extraction point while minimising energy cost.

---

## Files

| File | Description |
|------|-------------|
| `UnderwaterRescue_Search.xml` | SNAP! project — import this file |

---

## Quick Start

1. Open **https://snap.berkeley.edu/snap/snap.html** in Chrome or Firefox
2. Click the **SNAP! logo** (top-left) → **Import…** → select `UnderwaterRescue_Search.xml`
3. Press the **Green Flag ▶** to initialise
4. Set `algorithmChoice` to `BFS`, `DFS`, `UCS`, or `AStar` by double-clicking its watcher on the Stage
5. Click the **SearchEngine sprite** (blue rectangle) to run

---

## Scenario

A submarine navigates a network of **26 nodes** — 20 transit states (S0–S19), 3 beacon locations (B1–B3), and 3 docking stations (D1–D3). The mission is to collect all beacons and dock at any station while minimising energy cost.

- **Multiple goals:** any of D1, D2, D3 is a valid end state
- **Multiple objectives:** minimise edge cost AND collect all beacons (encoded as a hard constraint)
- **Forced transitions:** S7 → S8 and S14 → S15 (hazard zones that redirect the submarine)

---

## Infrastructure Design

### Graph representation

The graph lives entirely in **stage-level SNAP! list variables** — no graph data is hardcoded in the search algorithm. To modify the problem without touching the search engine:

| Variable | Contents | How to modify |
|----------|----------|---------------|
| `EDGES` | Strings of `"from,to,cost"` | Add/remove strings to add/remove edges |
| `GOAL_STATES` | List of valid goal nodes | Edit list to change which nodes are goals |
| `BEACON_STATES` | List of beacon nodes | Edit list to change what must be collected |
| `FORCED_FROM` / `FORCED_TO` | Parallel lists for forced transitions | Add matching entries to both lists |

### Search node structure

Each node on the fringe is a 4-element SNAP! list:

```
[ currentNode , pathList , g(n) , beaconsCollectedList ]
```

### Custom block modules

The infrastructure separates **problem definition** (reporter blocks + stage variables) from the **search engine** (`run search` command block).

| Block | Type | Role |
|-------|------|------|
| `neighbors of [st]` | Reporter | Returns successors of a node from `EDGES` |
| `edge cost [fr] to [to]` | Reporter | Looks up a directed edge cost |
| `forced next of [st]` | Reporter | Returns forced successor or empty string |
| `heuristic [st] beacons [bc]` | Reporter | A\* heuristic: `3×uncollected + 5×(not at dock)` |
| `copy list [lst]` | Reporter | Deep-copies a list (prevents aliasing) |
| `list to key [lst]` | Reporter | Concatenates list to string for visited-set hashing |
| `best index in [fringe]` | Reporter | Finds min-cost fringe index for UCS/A\* |
| `run search` | Command | Unified search engine — all four algorithms |

---

## Running Each Algorithm

> **Reset between every run:** press the Green Flag ▶ before each run or results will be incorrect.

### BFS — Breadth-First Search

1. Press **Green Flag ▶**
2. Double-click the `algorithmChoice` watcher → type `BFS` → Enter
3. Double-click the `startState` watcher → type `S0` → Enter
4. Load a test case if needed (see [Test Cases](#-test-cases) below)
5. Click the **SearchEngine sprite**

### DFS — Depth-First Search

1. Press **Green Flag ▶**
2. Double-click `algorithmChoice` → type `DFS` → Enter
3. Double-click `startState` → type `S0` → Enter
4. Load a test case if needed
5. Click the **SearchEngine sprite**

> DFS finds *a* solution but not the cheapest one. Paths may be significantly longer than optimal.

### UCS — Uniform-Cost Search

1. Press **Green Flag ▶**
2. Double-click `algorithmChoice` → type `UCS` → Enter
3. Double-click `startState` → type `S0` → Enter
4. Load a test case if needed
5. Click the **SearchEngine sprite**

> UCS always finds the optimal-cost solution. Expands more nodes than A\* because it uses no heuristic.

### A\* — A-Star Search

1. Press **Green Flag ▶**
2. Double-click `algorithmChoice` → type `AStar` → Enter
   > Exact capitalisation required: capital **A**, capital **S**, no space
3. Double-click `startState` → type `S0` → Enter
4. Load a test case if needed
5. Click the **SearchEngine sprite**

> A\* uses `h(n) = 3 × uncollected_beacons + 5 × (not at dock)`. Finds optimal solutions while expanding ~35% fewer nodes than UCS.

---

## Test Cases

Load a test case **after** pressing the Green Flag and **before** clicking the sprite.

**To fire a test case:** in the SearchEngine sprite's Scripts tab, click directly on the hat block of the `when I receive [setTC1]` / `[setTC2]` / `[setTC3]` script.

| Test Case | Script | Start | Beacons Required | Valid Goals | Description |
|-----------|--------|-------|-----------------|-------------|-------------|
| TC1 | `setTC1` | S0 | B1, B2, B3 | D1, D2, D3 | Full problem from S0 |
| TC2 | `setTC2` | S5 | B1, B2, B3 | D1, D2, D3 | Full problem from S5 |
| TC3 | `setTC3` | S0 | B1, B2 | D1, D2 | Reduced: 2 beacons, 2 goals |

---

## Reading the Output

After a run, the SearchEngine sprite displays a speech bubble:

```
AStar: Cost=34 Steps=9 Expanded=47
```

All detailed output is in the **Stage watchers**:

| Watcher | What it shows |
|---------|--------------|
| `expandedCount` | Nodes popped and processed |
| `finalCost` | Total edge-weight cost of the solution |
| `pathLength` | Number of steps in the solution path |
| `finalPath` | Click to expand — ordered list of nodes from start to goal |
| `searchLog` | Full expansion trace. Last entry: `SOLVED! Alg=... Cost=... Steps=... Expanded=...` |

**`searchLog` entry format:**

```
Expand: S8 g=9 B=B1
Expand: B2 g=12 B=B1B2
SOLVED! Alg=AStar Cost=34 Steps=9 Expanded=47
```

---

## Running All Experiments

To collect data for all 12 combinations (4 algorithms × 3 test cases):

For **each test case** TC1, TC2, TC3:

1. Press **Green Flag ▶**
2. Click the test case hat block (`setTC1`, `setTC2`, or `setTC3`)
3. Set `algorithmChoice` → click sprite → record `expandedCount`, `finalCost`, `pathLength`
4. Repeat steps 1–3 for each of: `BFS`, `DFS`, `UCS`, `AStar`

---

## Heuristic (A\*)

```
h(n, bc) = 3 × |BEACON_STATES − bc| + 5 × (1 if n ∉ GOAL_STATES else 0)
```

- **Admissible:** minimum edge cost to collect a beacon ≥ 3; minimum cost to reach any dock ≥ 5. The heuristic never overestimates.
- **Consistent:** each step either reduces uncollected beacons by 1 (h drops by 3, edge cost ≥ 3) or moves toward a dock (h drops by 5, dock approach cost ≥ 5). Triangle inequality holds.

---

## Assumptions and Limitations

- All edges are **directed**. `S1→B1` does not imply `B1→S1`.
- Beacon collection is **automatic** on arrival — no separate action needed.
- The visited set uses a composite key `state|beaconKey` so the same node can be visited with different beacon sets.
- `algorithmChoice` must be typed **exactly**: `BFS`, `DFS`, `UCS`, or `AStar`.
- SNAP! is single-threaded — the UI will appear frozen during long searches. This is expected.
- There is no graphical drawing of the graph. All output is through watchers and the speech bubble.
