# Cell-Routing

A simulation of how mobile phone networks locate subscribers and route calls in
real time, built from scratch in C++ using custom data structures (no STL
containers for the core logic).

When you place a call, the network has to answer two questions in milliseconds:
*where is this person right now*, and *what is the fastest path to reach them*.
CellRoute models both, using classic data structures as the building blocks:

- HashMap (subscriber registry) maps a phone number to its current cell tower.
- Graph (network topology) stores towers as nodes and links as weighted edges.
- Min-Heap (priority queue) powers Dijkstra's shortest-path search.
- Queue powers a BFS hop-count search, for contrast with Dijkstra.

## Build

```
g++ -std=c++17 -o cellroute main.cpp
```

Or with the included Makefile:

```
make
```

## Run

```
./cellroute
```

Then enter commands interactively, or pipe a script in:

```
./cellroute < example.txt
```

## Commands

| Command | Description |
| --- | --- |
| `ADD_TOWER <name>` | Add a tower (single letter A-Z, or a number) |
| `ADD_LINK <a> <b> <weight>` | Add an undirected link between two towers with a delay weight |
| `REGISTER <number> <tower>` | Register a phone number at a tower |
| `MOVE <number> <tower>` | Move an already-registered number to a different tower |
| `CALL <caller> <receiver>` | Route a call using Dijkstra (lowest total delay) |
| `HOPS <caller> <receiver>` | Find fewest hops using BFS (ignores weights) |
| `QUIT` | Exit |

Note: commands use underscores, e.g. `ADD_TOWER`, not `ADD TOWER`.

## Example session

```
ADD_TOWER A
ADD_TOWER B
ADD_TOWER C
ADD_TOWER D
ADD_TOWER E
ADD_LINK A B 9
ADD_LINK A C 2
ADD_LINK B C 4
ADD_LINK B D 7
ADD_LINK C D 3
ADD_LINK D E 5
ADD_LINK C E 8
REGISTER 9876543210 A
REGISTER 9123456789 E
CALL 9876543210 9123456789
HOPS 9876543210 9123456789
QUIT
```

`CALL` reports the lowest-delay route and its total cost. `HOPS` reports the
fewest number of towers in between. The two can differ, which is the point: the
shortest path by hop count is not always the fastest by delay.

## Project structure

```
HashMap.h    subscriber registry (phone number -> tower)
Graph.h      network topology (adjacency list of weighted edges)
MinHeap.h    array-based binary min-heap (Dijkstra's priority queue)
Queue.h      circular queue (BFS frontier)
main.cpp     CLI, Dijkstra routing, BFS routing
```

## Implementation notes

The Dijkstra implementation uses lazy deletion: rather than decreasing a key in
the heap, it pushes a fresh entry for an improved distance and skips stale
entries when they are popped. This avoids needing a decrease-key operation while
keeping the standard O((V + E) log V) complexity.