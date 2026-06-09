# SubgraphRAG — Paper Notes

**Paper:** [SubgraphRAG: Simplifying KG-RAG with Lightweight Subgraph Retrieval](https://arxiv.org/abs/2401.14887)

---

## Core Thesis

> **Graphs retrieve. LLMs reason.**

Most prior KG-RAG systems try to make both the graph and the LLM perform reasoning. This paper argues that is unnecessary. The graph's role is to supply structured evidence; the LLM's role is to reason over it.

```
Traditional KG-RAG          SubgraphRAG

    Question                  Question
        ↓                         ↓
Complex Graph Reasoning    Retrieve Subgraph
        ↓                         ↓
       LLM                   LLM Reasoning
        ↓                         ↓
      Answer                    Answer
```

---

## Why Not Document RAG?

Standard RAG retrieves text chunks, which creates three problems for multi-hop questions:

- Facts can be split across different chunks
- Entity relationships are only implicit
- Multi-hop reasoning is difficult to chain

**Example:**

> *"Who directed the movie that won Best Picture in 1994?"*

The needed facts — `(Forrest Gump, wonBestPicture, 1994)` and `(Forrest Gump, directedBy, Robert Zemeckis)` — may appear in separate chunks. A knowledge graph stores them as explicit, linked triples.

---

## Background

### Knowledge Graph Triples

A KG stores facts as `(head, relation, tail)`:

```
(Barack Obama, presidentOf, USA)
```

### Subgraph

A subgraph is a small, focused portion of the full KG — only the triples relevant to a given query:

```
Forrest Gump ──wonBestPicture──► 1994
Forrest Gump ──directedBy──────► Robert Zemeckis
```

---

## Main Contributions

### 1. Triple Scoring

A lightweight MLP assigns a relevance score to each candidate triple based on how useful it is for answering the query. Only the highest-scoring triples are kept and passed to the LLM.

| Triple | Score |
|---|---|
| `(Forrest Gump, directedBy, Robert Zemeckis)` | High |
| `(Forrest Gump, releaseDate, 1994)` | Lower |

### 2. Parallel Triple Scoring

Previous methods score triples sequentially. SubgraphRAG scores all triples simultaneously, making retrieval faster, more scalable, and cheaper.

### 3. Directional Structural Distance (DSD)

Not all nodes are equally important. Nodes closer to the query entity along the traversal path are generally more relevant. DSD encodes both graph distance and traversal direction as features for the scoring model.

### 4. Adaptive Subgraph Size

Simple questions need fewer triples; complex questions need more. SubgraphRAG dynamically adjusts the number of retrieved triples rather than using a fixed cutoff.

---

## How the LLM Is Used

After retrieval, the subgraph is converted to text and prompted into the LLM:

```
Retrieved Subgraph → Converted to text → LLM prompt → Answer
```

The LLM handles reasoning, inference, and answer generation. The graph does not reason — it only supplies structured evidence.

---

## Explainability

Because the retrieved evidence is a structured subgraph (nodes and edges), the reasoning path is visible and inspectable. This is a significant advantage over document RAG, where it is often unclear why a chunk was retrieved.

Users can trace:
- which entities were retrieved
- which relations connected them
- the full retrieval path

---

## Experimental Results

**Benchmarks:** WebQSP, ComplexWebQuestions (CWQ)

**Key findings:**

1. **Better retrieval → better answers.** Most performance gains came from improving retrieval quality, not from more complex reasoning.
2. **Smaller LLMs are competitive.** Models like Llama 3.1 8B perform surprisingly well when given high-quality retrieved subgraphs.
3. **Less hallucination.** Grounded structured evidence reduces fabricated answers.

> 📌 **Note:** Irrelevant-but-related documents often *hurt* performance, while completely random documents can surprisingly *improve* it.

---

## Limitations

| Issue | Detail |
|---|---|
| KG Construction | Assumes a KG already exists; building large KGs is difficult |
| Scalability | Very large enterprise graphs may still be expensive to handle |
| Contradictions | No explicit mechanism for handling contradictory evidence |
| Faithfulness | The LLM can still misinterpret retrieved evidence |

---

## Relevance to Explainable RAG

The retrieved subgraph acts as a structured **evidence graph**, not just retrieved text. This opens the door to:

- Explicit reasoning path inspection
- Claim and evidence graphs
- Contradiction-aware retrieval
- Faithful reasoning verification

---

## One-Line Summary

SubgraphRAG shows that knowledge graphs are most effective as structured evidence retrieval mechanisms, with LLMs performing the reasoning — achieving better accuracy, efficiency, and explainability than more complex KG-RAG architectures.
