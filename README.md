# Semantic Telemetry

A toolkit for visualizing and measuring how ideas cluster, drift, and evolve in semantic space across brainstorming sessions.

## Description

Semantic Telemetry turns free-form ideas (text) into points in a high-dimensional semantic space using sentence embeddings (SBERT), then uses **t-SNE** to project them into 2D for visualization and a small library of **centroid-based metrics** to quantify originality, divergence, and semantic drift [1].

The project provides four pipelines that operate on a CSV of ideas:

1. **Full t-SNE Visual** — embeds every idea and plots the complete semantic map.
2. **Group Telemetry** — splits a session into two time phases (0–5 min, 5–10 min) and reports how the group's ideas moved, spread, and introduced novelty between phases.
3. **Individual Telemetry** — plots each participant's trajectory on the shared global semantic map so you can see _where_ a person's ideas moved, not just how far.
4. **Single-Idea Originality Score** — scores one candidate idea against a reference dataset and returns its originality percentile relative to the rest of the corpus.

## Metrics

| Metric | Description |
|--------|-------------|
| **Centroid Distance** | Distance of an idea from the mean of a reference set. Higher = more original relative to the group average. |
| **Centroid Shift** | Distance between the centroids of two time windows. Captures how far the conversation _drifted_. |
| **Dispersion** | Average pairwise distance inside a set of ideas. Captures how _spread out_ (divergent) the thinking is. |
| **Novelty** | For each new idea, the minimum distance to any prior idea, averaged. Captures _true_ novelty rather than just drift of the average. |

All metrics are derived from cosine similarity on `all-MiniLM-L6-v2` embeddings [1].

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

| Column | Required | Description |
|--------|----------|-------------|
| `timeStamp` | yes | Parseable timestamp for each idea |
| `idea` | yes | The raw text of the idea |
| `participant_id` | only for individual pipeline | Identifier for who contributed the idea |

The group and individual pipelines split the session into **Time 1 (first 5 minutes)** and **Time 2 (remaining minutes)** based on the earliest timestamp [1].

## Usage

Edit the execution block at the bottom of `semantic_telemetry.py` and uncomment the pipeline you want to run [1]:

```python
csv_file = 'ideaData.csv'

run_full_tsne_visual(csv_file)
# run_group_telemetry(csv_file)
# run_individual_telemetry(csv_file)
# score_idea_originality("Making certain bundles be all inclusive,", csv_file)
```

Then run:

```bash
python semantic_telemetry.py
```

## Pipeline Reference

### `run_full_tsne_visual(csv_filepath)`
Scatter plot of every idea in 2D t-SNE space with short labels [1].

### `run_group_telemetry(csv_filepath)`
Prints centroid shift, dispersion (T1 and T2), net expansion, and average novelty; plots T1 (red) vs. T2 (green) [1].

### `run_individual_telemetry(csv_filepath)`
Per-participant subplot showing each person's T1→T2 movement against the global semantic map, with a dashed trajectory line between centroids [1].

### `score_idea_originality(idea, csv_filepath)`
Returns the idea's centroid distance and its originality percentile against the reference corpus [1].

## Core Functions

| Function | Purpose |
|----------|---------|
| `get_centroid_distance()` | Calculates semantic distance of a single idea from the centroid of reference embeddings |
| `get_centroid_shift()` | Calculates semantic drift between two sets of embeddings |
| `get_dispersion()` | Calculates internal spread (divergence) of a set of embeddings |
| `get_novelty()` | Calculates minimum distance to past ideas (true novelty) |
| `ingest_and_split()` | Standardized data intake; splits session at 5-minute midpoint |

---
