# Logistics_Path_Finding
https://share.synthesia.io/c5980d6b-a55b-4179-bae8-b7ff82845e40 
Graph analysis of the London Underground using Neo4j GDS — Dijkstra shortest paths, Betweenness Centrality, PageRank, and Louvain community detection on an 80-station property graph. Identifies structural bottlenecks, optimal routes, and network resilience under targeted station closures.
# London Underground: Public Transit Pathfinding and Bottleneck Analysis
### Using Neo4j Graph Data Science (GDS)

---

## Project Overview

This project constructs and analyzes a real-world geospatial graph database representing the London Underground (the "Tube") transit network using Neo4j's Cypher query language and the Neo4j Graph Data Science (GDS) library.

The analysis applies four core graph algorithms to answer operational and structural questions about one of the world's oldest and most complex metro systems:

- **Dijkstra's Algorithm** — weighted shortest-path routing between stations
- **Betweenness Centrality** — identification of structural bottlenecks
- **PageRank** — measurement of station influence and prestige
- **Louvain Community Detection** — discovery of geographic and operational clusters

A structured network resilience simulation evaluates the consequences of removing high-criticality stations from the network.

---

## Dataset

The dataset is the London Tube Network originally compiled by [Nicola Greco](https://github.com/nicola/tubemaps) and comprises three CSV files:

| File | Description | Key Columns |
|---|---|---|
| `london_stations.csv` | 80 Tube stations | id, name, latitude, longitude, zone, total_lines |
| `london_connections.csv` | 179 track connections | station1, station2, line, time (minutes) |
| `london_lines.csv` | 11 Tube lines | line, name, colour, stripe |

**Network scope:** 80 stations, 11 lines, 338 directed CONNECTION relationships, 165 SERVED_BY relationships.

---

## Graph Model

| Element | Type / Label | Key Properties |
|---|---|---|
| Node | `:Station` | id, name, latitude, longitude, zone, total_lines |
| Node | `:Line` | id, name, colour, stripe |
| Relationship | `:CONNECTION` | time (float, minutes), line (int) |
| Relationship | `:SERVED_BY` | none — connects Station to Line |

CONNECTION relationships are bidirectional: each CSV row produces two directed edges.

---

## Project Structure

```
├── notebooks/
│   └── london_underground_analysis.ipynb   # Main Jupyter Notebook deliverable
├── data/
│   ├── london_stations.csv
│   ├── london_connections.csv
│   └── london_lines.csv
└── README.md
```

---

## Requirements

### Software
- Neo4j Desktop or Neo4j Server (version 5.x recommended)
- Neo4j Graph Data Science (GDS) plugin (version 2.x)
- Python 3.8+
- Jupyter Notebook

### Python Dependencies
```
neo4j
pandas
jupyter
```

Install with:
```bash
pip install neo4j pandas jupyter
```

---

## Setup Instructions

### 1. Start Neo4j

Launch Neo4j Desktop and create a new database, or start your Neo4j server instance. Ensure the GDS plugin is installed and enabled.

### 2. Load the Dataset

Place the three CSV files in your Neo4j import directory:

| Setup | Import folder path |
|---|---|
| Neo4j Desktop | `~/.config/Neo4j Desktop/Application/relate-data/dbmss/<your-db>/import/` |
| Neo4j Server (Linux) | `/var/lib/neo4j/import/` |
| Docker | Mount a volume to `/var/lib/neo4j/import/` |

Alternatively, load directly from the GitHub raw URLs:
```
https://raw.githubusercontent.com/nicola/tubemaps/master/distances/london.stations.csv
https://raw.githubusercontent.com/nicola/tubemaps/master/distances/london.connections.csv
https://raw.githubusercontent.com/nicola/tubemaps/master/distances/london.lines.csv
```

### 3. Run the Notebook

```bash
jupyter notebook notebooks/london_underground_analysis.ipynb
```

Execute cells in order. Each section builds on the graph state established by the previous one.

---

## Analysis Sections

### Part 1: Graph Construction
- Schema constraints and indexes on Station.id and Station.name
- Station node ingestion from london_stations.csv
- Line node ingestion from london_lines.csv
- CONNECTION relationship ingestion (bidirectional)
- SERVED_BY relationship enrichment derived from connection topology

### Part 2: Exploratory Data Analysis
- Graph size overview (91 nodes, 503 relationships)
- Node labels and relationship types
- Node property schema verification
- Station distribution across fare zones
- Top 10 most physically connected stations (degree centrality)
- Average inter-station travel time per line
- Major interchange stations served by 3 or more lines
- Station count per Tube line
- Network diameter analysis
- Terminal station identification
- Local clustering coefficient computation
- Parallel route (redundant track) identification
- Fare zone boundary crossing analysis
- Fastest Zone 1 access from each outer zone

### Part 3: Advanced Cypher Analysis
- Zone boundary crossing connections
- Terminal stations and resilience implications
- Local clustering coefficient with corrected undirected pair counting
- Parallel routes and redundant track segments
- Fastest access to Zone 1 from each outer zone

### Part 4: GDS Algorithm Application

#### 4.1 Weighted Shortest Path — Dijkstra's Algorithm
- Graph projection: `tube-weighted` (UNDIRECTED, time-weighted)
- Point-to-point routing: Euston to Hammersmith, Canary Wharf to Paddington, Putney Bridge to Finsbury Park
- Single-source Dijkstra from Kings Cross St. Pancras to all reachable stations
- Discussion of model limitations as a real-world journey planner

#### 4.2 Betweenness Centrality
- Graph projection: `tube-weighted`
- Mathematical definition and justification over degree centrality and PageRank
- Top 20 stations by betweenness score
- Zone distribution analysis
- Dual top-10 overlap with degree centrality

#### 4.3 PageRank
- Graph projection: `tube-pagerank` (UNDIRECTED, damping factor 0.85, min 50 iterations)
- Conceptual distinction from betweenness centrality
- Top 15 stations by PageRank score
- Composite criticality score: normalized betweenness × normalized PageRank
- Top 10 stations by composite criticality
- Practical applications for transit authorities

#### 4.4 Community Detection — Louvain Algorithm
- Graph projection: `tube-louvain` (UNDIRECTED, time-weighted)
- Justification over Label Propagation and Weakly Connected Components
- 16 communities detected, modularity score: 0.5669
- Community profile analysis: size, dominant zone, lines, sample stations
- Multi-zone community identification
- Operational utility for Transport for London

### Part 5: Network Resilience Simulation (Bonus)
- **5.1** Targeted disruption: Kings Cross St. Pancras closure — adjacent stations and alternative path availability
- **5.2** Cascading failure analysis: longest shortest paths through top 5 betweenness stations and minimum detour times
- **5.3** Redundancy assessment: alternative routes for top 3 composite criticality stations, detour time penalties, and parallel line investment justification

---

## Key Findings

| Finding | Detail |
|---|---|
| Most connected station | Kings Cross St. Pancras (8 lines, 7 neighbors) |
| Highest betweenness | Green Park (763.54) |
| Highest PageRank | Green Park (2.3463) |
| Highest composite criticality | Green Park (1.0 — perfect dual dominance) |
| Network diameter | Aldgate East to Clapham South, 43 minutes, 10 hops |
| Communities detected | 16 (6 meaningful, 10 singletons from isolated stations) |
| Louvain modularity | 0.5669 (strong community structure) |
| Isolated stations | 10 stations absent from connections CSV |
| Terminal stations | Canary Wharf (Jubilee), Wembley Central (Bakerloo) |
| Redundant track segments | 32 station pairs served by more than one line |

---

## Graph Projections Used

| Projection Name | Algorithm | Orientation | Weight |
|---|---|---|---|
| `tube-weighted` | Dijkstra, Betweenness | UNDIRECTED | time |
| `tube-pagerank` | PageRank | UNDIRECTED | none (unweighted) |
| `tube-louvain` | Louvain | UNDIRECTED | time |

---

## Known Data Limitations

- The dataset covers 80 of the real 272 London Underground stations
- 10 stations are present in london_stations.csv but absent from london_connections.csv and are structurally isolated
- Victoria and Waterloo & City lines have no CONNECTION relationships in the dataset
- Missing intermediate stations on the District, Piccadilly, and Central line corridors cause artificial routing detours in shortest path results
- The model does not account for line transfer times, service frequency, or real-time operational conditions

---

## Node Properties Written by GDS Algorithms

After running all GDS sections, each Station node carries the following computed properties:

| Property | Algorithm | Description |
|---|---|---|
| `betweenness` | Betweenness Centrality | Raw betweenness score |
| `pagerank` | PageRank | PageRank prestige score |
| `community_id` | Louvain | Community membership id |

---

## References

- Greco, N. (2014). London Tube Maps Dataset. https://github.com/nicola/tubemaps
- Neo4j Graph Data Science Library Documentation. https://neo4j.com/docs/graph-data-science/
- Brandes, U. (2001). A faster algorithm for betweenness centrality. Journal of Mathematical Sociology.
- Blondel, V.D. et al. (2008). Fast unfolding of communities in large networks. Journal of Statistical Mechanics.
- Page, L. et al. (1999). The PageRank Citation Ranking: Bringing Order to the Web. Stanford InfoLab.

---

## Author
Lucia Shumba
Graduate student project — Neo4j Graph Data Science certification track.
