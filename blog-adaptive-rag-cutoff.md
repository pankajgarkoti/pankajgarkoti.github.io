---
layout: default
title: Building an Adaptive RAG Cutoff Before RAG Was a Thing
---

# Building an Adaptive RAG Cutoff Before RAG Was a Thing

*How I used K-Means clustering to solve the retrieval quality problem in 2023 — and how the industry caught up.*

---

In August 2023, I was building a semantic search microservice called MavySearch. The core problem was simple: when you embed a query and search a vector database, how many results should you actually return?

Every tutorial said the same thing: set a threshold. Cosine similarity above 0.7? Keep it. Below? Discard. Pick your number and move on.

That advice was wrong.

## The Problem: Fixed Cutoffs Don't Work

Here's what happens with a fixed distance threshold. Say you set your cutoff at 0.5:

- **Query A** returns results with distances `[0.1, 0.15, 0.2, 0.6, 0.8, 0.95]`. Your 0.5 cutoff works perfectly — three relevant results, three filtered out.
- **Query B** returns `[0.3, 0.35, 0.4, 0.45, 0.48, 0.9]`. Now you keep five results when really the first four are a cluster and the last two are noise.
- **Query C** returns `[0.05, 0.08, 0.52, 0.55]`. You cut half the results, but the real gap is between 0.08 and 0.52 — you should've kept only two.

The distance distribution changes with every query. A fixed threshold is essentially a guess. You either keep noise or miss relevant results, depending on the query.

This problem gets worse with different embedding models. Sentence Transformers might give you Euclidean distances in `[0.2, 2.8]`. Cosine distances land in `[0.0, 1.0]`. A threshold tuned for one model is meaningless for another.

In 2023, most retrieval systems — FAISS, Annoy, Chroma — punted on this. They let you set `top_k` or a distance threshold and called it a day. The implicit assumption was that the developer would tune these values per use case. In practice, nobody did.

## First Attempt: Elbow Point Detection (Aug 21)

My first instinct was borrowed from dimensionality reduction. The "elbow method" finds the knee in a scree plot — the point where adding more principal components stops helping. I adapted it for distance scores.

The idea: plot the first differences of sorted distances, draw a line from the first point to the last, and find the point with the maximum perpendicular distance from that line. That's your cutoff.

```python
def find_cutoff_using_elbow_point(distances: list[float]) -> float:
    if len(distances) == 1:
        return distances[0]

    differences = np.diff(distances)

    n_points = len(differences)
    all_coords = np.vstack((range(n_points), differences)).T
    first_point = all_coords[0]
    line_vector = all_coords[-1] - all_coords[0]
    line_vector_norm = line_vector / np.sqrt(np.sum(line_vector**2))

    vector_from_first = all_coords - first_point
    scalar_product = np.sum(
        vector_from_first * npm.repmat(line_vector_norm, n_points, 1),
        axis=1,
    )
    vector_to_line = vector_from_first - np.outer(
        scalar_product, line_vector_norm
    )

    idx_elbow = np.argmax(np.sqrt(np.sum(vector_to_line**2, axis=1)))
    cutoff = distances[idx_elbow]
    return cutoff
```

It worked on well-behaved distance curves — the kind you see in textbook examples. But real search results aren't textbook examples.

**The problems showed up fast:**

1. For result sets under 5 items, the elbow point was random — sometimes at the start, sometimes at the end.
2. Flat distance curves (all results equally mediocre) produced no meaningful knee.
3. Exponential curves put the elbow at the wrong place.

Seven days of production testing was enough. The elbow method was too fragile for real queries.

## The Solution: K-Means Clustering on Distance Scores (Aug 28)

The replacement was conceptually different. Instead of looking for a shape in the curve, I asked: **can we partition the distances into clusters and keep only the cluster that contains the best results?**

K-Means on a 1D array of distance scores:

```python
def find_cutoff_using_kmeans_clustering(
    distances: list[float], n_clusters: int = 2
):
    data = np.array(distances).reshape(-1, 1)

    kmeans = KMeans(n_clusters=n_clusters, random_state=0)
    kmeans.fit(data)
    labels = kmeans.labels_
    centroids = kmeans.cluster_centers_

    # Find the cluster whose centroid is closest to the best result
    first_distance = distances[0]
    closest_centroid_idx = np.argmin(np.abs(centroids - first_distance))
    cluster_distances = [
        dist
        for label, dist in zip(labels, distances)
        if label == closest_centroid_idx
    ]

    cutoff = max(cluster_distances)
    return cutoff
```

The algorithm finds the cluster of distances that the top result belongs to, then uses the maximum distance in that cluster as the cutoff. Results above that threshold are in a different cluster — they're noise.

### The Adaptive Formula

Using `n_clusters=2` everywhere was too rigid. Two clusters works for 10 results but under-segments 100 results. I needed the number of clusters to scale with result set size.

After testing, the formula that stuck:

```python
n_clusters = math.ceil(math.sqrt(len(distances) / 2))
```

Put into practice:

```python
normalized_distances = [(d / max(distances)) for d in distances]
cutoff = find_cutoff_using_kmeans_clustering(
    normalized_distances,
    n_clusters=math.ceil(math.sqrt(len(distances) / 2)),
)
```

Why `ceil(sqrt(n/2))`? It scales sublinearly with the result set size, matching the diminishing returns in search quality:

- 10 results: `ceil(sqrt(5))` = 3 clusters
- 20 results: `ceil(sqrt(10))` = 4 clusters
- 100 results: `ceil(sqrt(50))` = 8 clusters

This was discovered empirically — I tested various formulas and this one balanced stability (not too many clusters on small sets) with granularity (enough clusters for large sets).

### Distance Normalization

Before feeding distances to K-Means, I normalized them to `[0, 1]`:

```python
normalized_distances = [(d / max(distances)) for d in distances]
```

Simple max-normalization. The worst result maps to 1.0, the best maps close to 0.0. This made K-Means parameters transferable across different embedding models and distance metrics — raw Euclidean distances and cosine distances both become relative relevance scores.

### Query Routing: Single-Word vs Multi-Word

Not every query needs K-Means clustering. Single-word queries like "python" or "auth" are usually entity lookups, not semantic searches. Running clustering on them is overkill.

The routing logic was deliberately simple — check for a space:

```python
if " " in query:
    # Multi-word: semantic search + K-Means cutoff
    results = collection.query(
        n_results=20,
        query_texts=[query],
        where={"user_id": user_id},
    )
    # ... apply K-Means cutoff
else:
    # Single-word: literal substring match, top 6
    results = collection.query(
        n_results=20,
        query_texts=[query],
        where={"user_id": user_id},
        where_document={"$contains": query},
    )
```

Multi-word queries get the full treatment: 20 candidates, normalized distances, K-Means cutoff. Single-word queries get a literal substring filter plus a hard cap of 6 results. The space character as a routing signal was crude but effective — it avoided any ML overhead for query classification while correctly handling the majority of cases.

## Through Today's Lens

It's 2026 now, and the RAG ecosystem has matured significantly. The problems MavySearch solved ad-hoc in 2023 now have established solutions. Here's how they compare.

### Adaptive Retrieval

Modern RAG frameworks have converged on similar ideas:

- **LlamaIndex** has `SimilarityPostprocessor` with configurable thresholds, plus more advanced `MetadataReplacementPostProcessor` and auto-retrieval that adjusts parameters per query.
- **LangChain** offers `ContextualCompressionRetriever` that filters results through an LLM or cross-encoder, and `EnsembleRetriever` that combines multiple strategies.
- **Maximal Marginal Relevance (MMR)** is now standard — it balances relevance with diversity, solving a related problem (redundant results rather than irrelevant ones).

The K-Means approach sits in between these. Like MMR, it's unsupervised and model-agnostic. Like LLM-based filtering, it adapts per query. But it doesn't require a reranking model and adds minimal latency — just a `sklearn.cluster.KMeans` call on a small 1D array.

### Reranking Models

The biggest shift since 2023 is the rise of cross-encoder rerankers:

- **Cohere Rerank** — API-based reranking that scores query-document pairs
- **Cross-encoder models** (ms-marco, BGE-reranker) — run the full query-document pair through a transformer for precise relevance scoring
- **ColBERT** and late-interaction models — token-level matching for fine-grained relevance

These are strictly more powerful than K-Means clustering on distance scores. They understand semantics, not just distance distributions. But they add latency (50-200ms per rerank call) and cost (API calls or GPU inference). The K-Means approach was a zero-cost, zero-latency alternative that worked well enough for the use case.

### Query Routing

The space-based heuristic for single vs. multi-word queries has a modern equivalent:

- **LlamaIndex's RouterQueryEngine** uses an LLM to classify queries and route them to different retrieval strategies
- **Semantic routing** frameworks detect query intent and select the appropriate pipeline
- **Hybrid search** (BM25 + dense retrieval) handles the keyword vs. semantic split without explicit routing

The modern versions are more sophisticated, but the core insight — different query types need different retrieval strategies — was the same.

### What's the Same

The fundamental insight holds: **fixed thresholds are wrong for adaptive retrieval**. Whether you use K-Means clustering, LLM-based filtering, or cross-encoder reranking, you need something that adapts to each query's result distribution. The 2023 approach got the "what" right. The "how" has evolved, trading compute for accuracy.

### What's Different

The 2023 approach was purely statistical — it looked at distance values without understanding content. Modern rerankers actually read the documents and judge relevance semantically. That's a qualitative leap. K-Means can tell you that distance 0.4 is in a different cluster than 0.8, but it can't tell you that a document at distance 0.4 actually answers the question.

The other gap is evaluation. In 2023, I tuned the `sqrt(n/2)` formula by eyeballing results. Modern RAG systems use RAGAS, DeepEval, and other frameworks to measure retrieval quality systematically. Empirical tuning still works, but instrumented evaluation catches edge cases faster.

## What This Shows

MavySearch was a side project, not a research paper. The algorithms weren't novel in a computer science sense — K-Means clustering is decades old. What was novel was applying them to this specific problem at a time when the standard approach was "pick a threshold and hope for the best."

The 7-day iteration from elbow point detection to K-Means reflects a pattern I've found productive: **deploy the first idea fast, observe where it breaks, replace it with something that addresses the specific failure modes.** The elbow method was geometrically elegant but practically fragile. K-Means was less clever but more robust. Production doesn't reward elegance.

The query routing heuristic — a single `if " " in query` check — shows the value of pragmatic shortcuts. It would've been easy to build an ML classifier for query intent. The space check handled 90% of cases with zero complexity. Sometimes the dumbest possible solution is the right one.

Looking at this code three years later, the main thing I'd change is adding evaluation. The algorithms worked well, but "worked well" was based on my subjective judgment. Systematic retrieval benchmarks would have caught edge cases and made the `sqrt(n/2)` formula more defensible. That's the biggest lesson the RAG ecosystem has absorbed since 2023: measure everything, trust nothing.

---

*I build AI systems and retrieval pipelines for businesses. If you need help with RAG architecture, semantic search, or AI agent systems, check out my [services](/services) or [get in touch](/contact).*
