---
trigger: always_on
description: fast-plscan implements a density-based clustering algorithm based on HDBSCAN*.
---

# fast-plscan — Project Guidelines

fast-plscan implements a density-based clustering algorithm based on HDBSCAN*.

## Architecture

The library is split into a Python layer and a C++ core:

- `src/fast_plscan/__init__.py` — Public API; all exported symbols listed in `__all__`.
- `src/fast_plscan/sklearn.py` — Scikit-learn-compatible `PLSCAN` estimator (`BaseEstimator`, `ClusterMixin`).
- `src/fast_plscan/api.py` — Functional Python API of the main algorithm.
- `src/fast_plscan/prediction/` — Additional functional Python API for specialized prediction tasks.
- `src/fast_plscan/plots/` — Additional visualization classes for the main data structures.
- `src/fast_plscan/_helpers.py` — Internal utilities for the Python layer.
- `src/fast_plscan/_api/` — C++ extension compiled with **nanobind** and OpenMP (C++23).

Public entry-point: `from fast_plscan import PLSCAN` (sklearn interface) or the functional API functions.

### Algorithm overview

PLSCAN computes cluster labels through a fixed conversion pipeline:

1. Mutual-reachability MST/forest
2. Linkage tree
3. Condensed tree
4. Leaf tree
5. Persistence trace
6. Selected clusters and point labels/probabilities

A Python layer orchestrates these stages (`compute_mutual_spanning_tree` or `extract_mutual_spanning_forest`, then `clusters_from_spanning_forest`). A C++ layer performs the core conversions.

#### 1) MST/forest -> Linkage tree

- Input: sorted spanning-tree edges `(parent_point, child_point, distance)`.
- Output: linkage rows `(parent_node, child_node, child_count, child_size)` in merge order.
- Main structure: single forward sequential loop over MST edges with a disjoint-set/union-find state (`find` + path compression + `link`).
- Relation to indices:
	- Point indices are `0..num_points-1`.
	- Merge nodes are introduced as `num_points + row_index`.
	- Each linkage row corresponds to one merge event at the same edge order as MST distance order.

#### 2) Linkage tree -> Condensed tree

- Input: full linkage hierarchy plus MST distances and `min_cluster_size`.
- Output: condensed rows `(parent, child, distance, child_size)` and `cluster_rows`.
- Main structure: reverse sequential scan over linkage rows (bottom-up), with branch pruning and delayed row placement for pruned subtrees.
- Relation to indices:
	- Condensed parents use cluster labels starting at `num_points`.
	- `num_points` acts as a phantom root label for forest support.
	- `child < num_points` means a point row.
	- `child >= num_points` means a cluster-segment row.
	- `cluster_rows` stores row indices of cluster-segment rows (used by later stages).

#### 3) Condensed tree -> Leaf tree

- Input: condensed tree rows and `cluster_rows`.
- Output: per-segment arrays: `parent`, `[min_distance, max_distance]`, `[min_size, max_size]`.
- Main structure: a few linear passes (no recursive traversal):
	- Pass over all condensed rows to fill segment birth/min distance.
	- Pass over `cluster_rows` to fill parent and max distance.
	- Reverse paired pass over `cluster_rows` to propagate size intervals.
- Relation to indices:
	- Leaf segment index is the condensed cluster label offset: `segment_idx = condensed_label - num_points`.
	- Therefore `condensed_tree.parent[row] - num_points` and `condensed_tree.child[row] - num_points` index leaf-tree segments for cluster rows.
	- Segment `0` is reserved for the phantom root.

#### 4) Leaf tree -> Persistence trace

- Input: leaf segment size/distance lifetimes (and condensed point rows for distance/density variants).
- Output: trace arrays `min_size[]`, `persistence[]`.
- Main structure:
	- Build candidate size thresholds from leaf intervals, sort + unique.
	- Accumulate each segment's persistence over its active size interval via interval fill on the trace.
	- For distance/density and bi-persistence, use a reverse scan of condensed point rows and walk ancestors via `leaf_tree.parent` to collect contributions.
- Result semantics:
	- Standard traces sum one persistence value per active segment.
	- Bi-persistence traces integrate distance/density persistence contributions over size.

#### 5) Persistence trace -> selected clusters -> point labelling

- Input: persistence trace + leaf/condensed trees.
- Output: selected segment ids, per-point labels, and probabilities.
- Main structure:
	- Select the best cut size by maximizing total trace persistence (bounded by `max_cluster_size`).
	- Compute selected segments for that cut with a linear pass over leaf segments.
	- Build segment labels in one forward pass over leaf segments (inherit parent label unless selected).
	- Label points in one pass over condensed point rows (`child < num_points`).
- Relation to indices:
	- Selected cluster ids are leaf segment indices (`uint32`) compatible with leaf-tree indexing.
	- Point-row parent mapping uses `parent_segment = condensed_tree.parent[row] - num_points`.
	- Probability is normalized within selected segment persistence (distance to segment death / segment persistence span).

### Additional functionality

PLSCAN also exposes additional functionality beyond the core clustering pipeline:

#### Cluster layers (`PLSCAN.cluster_layers`)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JelmerBot/fast_plscan](https://github.com/JelmerBot/fast_plscan) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
