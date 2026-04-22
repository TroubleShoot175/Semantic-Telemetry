# Semantic Telemetry

A toolkit for visualizing and measuring how ideas cluster, drift, and evolve in semantic space across brainstorming sessions.

## Description

Semantic Telemetry turns free-form ideas (text) into points in a high-dimensional semantic space using sentence embeddings (SBERT), then uses **t-SNE** to project them into 2D for visualization and a small library of **centroid-based metrics** to quantify originality, divergence, and semantic drift.

The project provides four pipelines that operate on a CSV of ideas:

1. **Full t-SNE Visual** — embeds every idea and plots the complete semantic map.
2. **Group Telemetry** — splits a session into two time phases (0–5 min, 5–10 min) and reports how the group's ideas moved, spread, and introduced novelty between phases.
3. **Individual Telemetry** — plots each participant's trajectory on the shared global semantic map so you can see *where* a person's ideas moved, not just how far.
4. **Single-Idea Originality Score** — scores one candidate idea against a reference dataset and returns its originality percentile relative to the rest of the corpus.

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

The group and individual pipelines split the session into **Time 1 (first 5 minutes)** and **Time 2 (remaining minutes)** based on the earliest timestamp.

## Usage

Edit the execution block at the bottom of `semantic_telemetry.py` and uncomment the pipeline you want to run:

```python
csv_file = 'ideaData.csv'

run_full_tsne_visual(csv_file)
# run_group_telemetry(csv_file)
# run_individual_telemetry(csv_file)
# score_idea_originality("Making bundles all inclusive", csv_file)
```

Then run:

```bash
python semantic_telemetry.py
```

### Pipeline reference

- `run_full_tsne_visual(csv_filepath)` — scatter plot of every idea in 2D t-SNE space, with short labels.
- `run_group_telemetry(csv_filepath)` — prints centroid shift, dispersion (T1 and T2), net expansion, and average novelty; plots T1 (red) vs. T2 (green).
- `run_individual_telemetry(csv_filepath)` — per-participant subplot showing each person's T1→T2 movement against the global semantic map, with a dashed trajectory line between centroids.
- `score_idea_originality(idea, csv_filepath)` — returns the idea's centroid distance and its originality percentile against the reference corpus.

## Implications and Future Work

The most important implication of this project is not any single metric — it is that **ideas can be visualized in semantic space in a way that is legible to facilitators, researchers, and participants**. Once the map exists, many follow-on questions become tractable:

- **Live visualization** during brainstorming sessions, so a group can see its own thinking spread (or collapse) in real time.
- **Cross-session comparison** to measure whether repeated sessions on the same prompt are converging or exploring new territory.
- **Facilitator dashboards** that flag early centroid lock-in and suggest divergence prompts.
- **Richer originality scoring** that combines centroid distance, nearest-neighbor novelty, and cluster membership rather than relying on a single metric.
- **Interactive 3D / UMAP alternatives** to t-SNE for better preservation of global structure.

Future work will focus primarily on improved, interactive visualizations of ideas in semantic space and on refining originality measures against human judgments of novelty.
