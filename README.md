# Semantic Telemetry

A toolkit for visualizing and measuring how ideas cluster, drift, and evolve in semantic space across brainstorming sessions.

## Description

Semantic Telemetry turns free-form ideas (text) into points in a high-dimensional semantic space using sentence embeddings (SBERT), then uses **t-SNE** to project them into 2D for visualization and a small library of **centroid-based metrics** to quantify originality, divergence, and semantic drift.

The project provides five pipelines that operate on a CSV of ideas:

1. **Full t-SNE Visual** — embeds every idea and plots the complete semantic map. When timestamps are present, points are colored by time of appearance (viridis: dark = early, bright = late) with a colorbar, so temporal structure is visible on the map itself.
2. **Group Telemetry** — splits a session into **N configurable time windows** (equal-duration by default, or fixed-size via `window_minutes`) and reports per-window dispersion, consecutive-window centroid shift and novelty, and an overall first-to-last summary. The scatter plot shows all windows on one t-SNE with a dashed line connecting window centroids in order.
3. **Individual Telemetry** — plots each participant's trajectory across N windows on the shared global semantic map, colored by phase, with a dashed trajectory line through each of the participant's phase centroids.
4. **Single-Idea Originality Score** — scores one candidate idea against a reference dataset and returns its originality percentile relative to the rest of the corpus.
5. **Cross-Session Comparison** — projects ideas from multiple CSVs onto a single shared t-SNE map, reports per-session dispersion and pairwise centroid shift and novelty, and draws a trajectory between session centroids so you can see whether repeated sessions are converging or exploring new territory.

### Metrics

- **Centroid Distance** — distance of an idea from the mean of a reference set. Higher = more original relative to the group average.
- **Centroid Shift** — distance between the centroids of two time windows. Captures how far the conversation *drifted*.
- **Dispersion** — average pairwise distance inside a set of ideas. Captures how *spread out* (divergent) the thinking is.
- **Novelty** — for each new idea, the minimum distance to any prior idea, averaged. Captures *true* novelty rather than just drift of the average.

All metrics are derived from cosine similarity on `all-MiniLM-L6-v2` embeddings.

## Motivation

There are few accessible methods for visualizing how ideas cluster in semantic space, or for comparing the semantic structure of repeated brainstorming sessions. Most ideation tools count ideas, tag them, or score them qualitatively — but they rarely show you the *shape* of the conversation or how that shape changes over time.

This project was built to explore two questions:

1. **Can t-SNE be used as a practical means of visualizing ideas in semantic space?** — laying out ideas as points whose distance reflects meaning, so that clusters, outliers, and trajectories become visible at a glance.
2. **Are there better ways to measure the originality of an idea?** — moving beyond frequency or novelty heuristics toward centroid-based and nearest-neighbor measures grounded in sentence embeddings.

## Requirements

- Python 3.8+
- `pandas`
- `numpy`
- `matplotlib`
- `scikit-learn`
- `sentence-transformers`

Install with:

```bash
pip install pandas numpy matplotlib scikit-learn sentence-transformers
```

## Input Format

The tool expects a CSV with the following columns:

| Column           | Required                     | Description                                        |
| ---------------- | ---------------------------- | -------------------------------------------------- |
| `timeStamp`      | yes                          | Parseable timestamp for each idea.                 |
| `idea`           | yes                          | The raw text of the idea.                          |
| `participant_id` | only for individual pipeline | Identifier for who contributed the idea.           |

The group and individual pipelines split the session into N time windows. By default, `n_windows=2` produces equal-duration halves based on the session's earliest and latest timestamps. Pass `n_windows=4` (or any integer) for finer time slicing, or `window_minutes=M` to use fixed-size windows instead (the number of windows is derived from the session's total duration).

## Usage

Edit the execution block at the bottom of `semantic_telemetry.py` and uncomment the pipeline you want to run:

```python
csv_file = 'ideaData.csv'

run_full_tsne_visual(csv_file)                          # colored by timestamp if available
# run_full_tsne_visual(csv_file, color_by_time=False)   # uniform color
# run_group_telemetry(csv_file, n_windows=4)            # 4 equal-duration windows
# run_group_telemetry(csv_file, window_minutes=2)       # fixed 2-minute windows
# run_individual_telemetry(csv_file, n_windows=3)
# score_idea_originality("Making bundles all inclusive", csv_file)
# run_cross_session_comparison(
#     ['sessionA.csv', 'sessionB.csv'],
#     session_names=['Monday brainstorm', 'Friday brainstorm'],
# )
```

Then run:

```bash
python semantic_telemetry.py
```

### Pipeline reference

- `run_full_tsne_visual(csv_filepath, color_by_time=True)` — scatter of every idea in 2D t-SNE space with short labels. Points are colored by timestamp when available.
- `run_group_telemetry(csv_filepath, n_windows=2, window_minutes=None)` — prints per-window dispersion, consecutive-window centroid shift and novelty, and an overall summary when N > 2; plots all windows with a centroid trajectory.
- `run_individual_telemetry(csv_filepath, n_windows=2, window_minutes=None)` — per-participant subplot showing each person's path through N phases on the global semantic map.
- `score_idea_originality(idea, csv_filepath)` — returns the idea's centroid distance and its originality percentile against the reference corpus.
- `run_cross_session_comparison(csv_filepaths, session_names=None)` — projects two or more session CSVs onto a shared t-SNE, prints per-session dispersion and pairwise shift/novelty, and draws a trajectory between session centroids.

## Implications and Future Work

The most important implication of this project is not any single metric — it is that **ideas can be visualized in semantic space in a way that is legible to facilitators, researchers, and participants**. Once the map exists, many follow-on questions become tractable:

- **Live visualization** during brainstorming sessions, so a group can see its own thinking spread (or collapse) in real time.
- **Cross-session comparison** to measure whether repeated sessions on the same prompt are converging or exploring new territory.
- **Facilitator dashboards** that flag early centroid lock-in and suggest divergence prompts.
- **Richer originality scoring** that combines centroid distance, nearest-neighbor novelty, and cluster membership rather than relying on a single metric.
- **Interactive 3D / UMAP alternatives** to t-SNE for better preservation of global structure.

Future work will focus primarily on improved, interactive visualizations of ideas in semantic space and on refining originality measures against human judgments of novelty.
