# airport-mst

Minimum spanning trees over the global airport network. A dataset of ~9,000 international airports is cleaned into a weighted graph using geodesic distances, and a from-scratch implementation of Prim's algorithm finds the cheapest set of routes connecting a chosen subset — rendered on an interactive world map.

> Previously `Graph-Theory-Final-Project`.

## The question

If you had to connect a set of airports with the least total flight distance, and every airport must be reachable, which routes do you build? That is a minimum spanning tree: `n-1` edges, no cycles, minimum total weight. It is the network-design problem behind airline hub planning, undersea cable routing, and power grid layout.

## Approach

**Data.** `airports-code@public.csv` is a semicolon-delimited open dataset of international airports with IATA codes, city, country and coordinates. Preprocessing drops redundant geo-name-ID columns, splits the packed `coordinates` field into separate latitude and longitude, coerces both to numeric, and audits the remaining nulls before building the graph.

**Graph construction.** For a chosen subset of airports (the notebook uses Tokyo Haneda, Soekarno-Hatta, JFK and Sydney), a complete graph is built in NetworkX with every pair connected. Edge weight is **geodesic** distance via `geopy` — great-circle distance over the WGS-84 ellipsoid, not planar Euclidean distance, because on intercontinental scales treating latitude/longitude as a flat plane is wrong by thousands of kilometres.

**Prim's algorithm, implemented directly.** Starting from one airport, a priority queue holds every edge crossing the visited/unvisited frontier. Repeatedly take the cheapest crossing edge; if its endpoint is already visited, discard it and continue, otherwise add it to the tree. Each edge carries its endpoint coordinates through the queue so the resulting tree can be drawn without a second lookup pass.

The lazy-deletion variant is used — stale edges are left in the queue and skipped on extraction rather than being decreased in place. This costs `O(E log E)` instead of `O(E log V)`, which is irrelevant here and considerably simpler to get right.

**Visualisation.** Both the complete graph and the resulting MST are rendered as Folium maps with markers per airport and polylines per edge, so the reduction from `n(n-1)/2` edges down to `n-1` is visible rather than just reported as a number.

## Results

The notebook prints the MST edge list and total tree weight, contrasted against the total weight of the complete graph. For the four-airport example the tree keeps every airport connected on three edges instead of six.

## Repository layout

```
main.ipynb                   Full pipeline: cleaning → graph → Prim's → maps
airports-code@public.csv     Source dataset (semicolon-delimited)
```

## Running it

```bash
pip install pandas networkx folium geopy requests
jupyter notebook main.ipynb
```

Folium maps render inline in Jupyter; run the notebook top to bottom, since the graph object is built incrementally across cells.

## Design notes

**Why geodesic and not Euclidean.** Sydney to JFK differs by thousands of kilometres between the two measures. An MST built on the wrong metric can select genuinely different edges, so the choice of distance function changes the answer, not just its precision.

**Why implement Prim's when NetworkX ships one.** `nx.minimum_spanning_tree` would be one line. Writing the priority-queue version made the cut property concrete — the reason greedily taking the cheapest frontier edge is globally optimal, rather than merely plausible.

## Known limitations

- Airport subsets are hardcoded by name; a mistyped name silently yields a smaller graph.
- Geodesic distance ignores actual flight paths, airspace restrictions and prevailing winds.
- Complete-graph construction is `O(n²)`, so the notebook is not usable across the full dataset without a spatial index or a k-nearest-neighbour sparsification step first.

## Possible next steps

- Kruskal's with union-find as a comparison, and a check that both produce equal total weight
- Sparsify with k-nearest neighbours to scale past a handful of airports
- Weight edges by real route frequency or cost rather than distance
