---
title: 🇮🇹🔍 ItColBERT - An Italian Late-Interaction Retriever
date: 2026-08-26 10:00 +0200
categories: [Personal Projects, ItColBERT - Italian Late-Interaction Retriever]
tags: [NLP, Information Retrieval, ColBERT, PyLate, Hugging Face, RAG]
---

<div align="center">
  <img src="../assets/img/posts/it_colbert/banner.jpg" alt="itcolbert_banner" style="width:100%; max-width:800px;">
</div>

to my knowledge, there was no **monolingual Italian ColBERT** out there — plenty of multilingual late-interaction models include Italian among dozens of languages, and plenty of strong Italian dense embedders exist, but nothing combined the two: an Italian-*specialized* model that keeps token-level matching. that gap is what made me want to try building one myself.

the result is **ItColBERT**, trained end to end on a single RTX 3090 at home, with every design decision, failed experiment and dead end documented along the way rather than hidden.

## 🧩 What is Late Interaction?

most retrieval embedding models compress a whole passage into **one vector**. ColBERT-style **late interaction** models keep **one small vector per token** instead, and score a document by matching each query token against its best document token (**MaxSim**) rather than comparing two averaged blobs.

that keeps fine-grained detail — exact names, numbers, phrasing — that gets blurred away by single-vector compression, at the cost of a larger index. it's the retrieval approach behind [ColBERT / ColBERTv2](https://github.com/stanford-futuredata/ColBERT) from Stanford, and the one [PyLate](https://github.com/lightonai/pylate) (the library this project is built on) makes practical to train and serve.

<details markdown="1">
<summary><strong>🔎 curious how late interaction actually works, step by step? (click to expand)</strong></summary>

<br>

**Single vector vs. multi-vector representations**

a "normal" dense retriever squeezes an entire passage — however long — into **one fixed-size vector**, a single point that's supposed to summarize its whole meaning. a multi-vector representation instead keeps **one vector per token**: the sentence "AI improves web search" doesn't become one blob, it becomes four small vectors, one per word, each preserving that word's own meaning in context.

<div align="center">
  <img src="../assets/img/posts/it_colbert/single_vs_multivector.png" alt="single vector vs multivector representation" style="width:100%; max-width:750px;">
  <br><em>source: <a href="https://qdrant.tech/documentation/tutorials-search-engineering/using-multivector-representations/">Qdrant docs — multivector representations</a></em>
</div>

<br>

**No interaction vs. early interaction vs. late interaction**

the three retrieval families differ in **when** the query and the document actually "meet":

- **no interaction** (plain dense retrieval / bi-encoders): query and document are each squeezed into a single vector, completely independently, and only compared at the very end via a cheap similarity function (cosine/dot product). every document can be encoded **offline, once**, and at query time only the query needs encoding — extremely fast, but the model never sees query and document together, so a lot of nuance gets averaged away.
- **early interaction** (cross-encoders): query and document text are concatenated and pushed through the transformer **together**, so every document token can attend to every query token through several self-attention layers. this is the most accurate scoring approach — but nothing about the document can be precomputed, so the full model has to rerun **per document, per query**. too slow to search a whole corpus with; used only to rerank a short candidate list.
- **late interaction** (ColBERT-style): documents are still encoded independently and fully precomputable offline — but as **many** token vectors, not one. at query time only the query needs fresh encoding; the actual query↔document comparison happens at scoring time via MaxSim. this recovers much of a cross-encoder's token-level precision while keeping the expensive part (encoding documents) something done once, offline — not per query.

<div align="center">
  <img src="../assets/img/posts/it_colbert/interaction_types.png" alt="no interaction vs early interaction vs late interaction diagram" style="width:100%; max-width:900px;">
  <br><em>source: <a href="https://qdrant.tech/course/multi-vector-search/module-1/late-interaction-basics/">Qdrant course — late interaction basics</a></em>
</div>

<br>

**The MaxSim operator, in words**

for every single query token, MaxSim looks across **all** of a document's token vectors and finds the one that matches best — then adds up these best-matches, one per query token, into the document's final score. so a document doesn't need to match the query's *overall* meaning; for every individual query token, it just needs *somewhere* in the text a close match to that specific word. that's exactly what lets late interaction preserve exact names, numbers and phrasing — the things a single averaged vector blurs away.

<div align="center">
  <img src="../assets/img/posts/it_colbert/maxsim_operator.jpg" alt="ColBERT MaxSim operator diagram" style="width:100%; max-width:650px;">
  <br><em>source: <a href="https://www.linkedin.com/posts/shubhendu-sharma_rag-retrievalaugmentedgeneration-colbert-activity-7440947799581499392-pOzw/">Shubhendu Sharma, LinkedIn</a></em>
</div>

<br>

**A worked example**

illustrative numbers below, not real model output (the "Try it" section further down has that) — just enough to make the difference concrete. take one query and two candidate documents:

- **query:** *"capitale Italia"* → tokens: `capitale`, `Italia`
- **doc A:** *"Roma è la capitale d'Italia"* (Rome is the capital of Italy) → tokens (stopwords dropped for readability): `Roma`, `capitale`, `d'`, `Italia`
- **doc B:** *"Milano è la capitale economica del Paese"* (Milan is the economic capital of the country) → tokens: `Milano`, `capitale`, `economica`, `Paese`

doc A is obviously the right answer to a human. here's what each architecture actually receives as input, and what it hands back as output.

**No interaction (bi-encoder)**

```
OFFLINE — before any query exists
  doc A text ──[Encoder]──► doc A embedding  (1 vector, stored in an index)
  doc B text ──[Encoder]──► doc B embedding  (1 vector, stored in an index)

AT QUERY TIME
  query text ──[Encoder]──► query embedding  (1 vector)

  cosine(query, doc A) = 0.82
  cosine(query, doc B) = 0.79
```

**input:** the query text alone, and each document text alone — the model never sees a query and a document together, ever. **output:** one similarity number per document, compared only after both sides are already reduced to a single vector each.

doc A still edges ahead here, but barely — squeezing "Roma è la capitale d'Italia" down to one vector mixes the fact "capital of Italy" together with everything else in the sentence, so the model can't specifically tell the query "yes, *Italia* is mentioned" — it can only compare two blurry overall impressions.

**Early interaction (cross-encoder)**

```
AT QUERY TIME — repeated for every candidate document, nothing precomputed
  [query + doc A] ──[Cross-Encoder]──► relevance score = 9.1
  [query + doc B] ──[Cross-Encoder]──► relevance score = 3.4
```

**input:** the query **and** the document *together*, fed into the model as one combined sequence, so every word on both sides can directly attend to every other word. **output:** one relevance score per document — but it took a full model pass, per document, to get it, since nothing about doc A or doc B could be computed ahead of time.

much more decisive gap than "no interaction" — the model can directly see "capitale" sitting right next to "d'Italia" in doc A, and that's missing in doc B. the cost: this has to be redone from scratch for every single candidate document, every single query, which is why cross-encoders only rerank a short list rather than search a whole corpus.

**Late interaction (ColBERT / MaxSim)**

```
OFFLINE — before any query exists
  doc A tokens (Roma, capitale, d', Italia)   ──[Encoder]──► 4 token embeddings, stored
  doc B tokens (Milano, capitale, economica, Paese) ──[Encoder]──► 4 token embeddings, stored

AT QUERY TIME
  query tokens (capitale, Italia) ──[Encoder]──► 2 token embeddings

  MaxSim(query, doc A) — compare every query token against every doc A token, keep only the best:
    "capitale" vs Roma=0.74, capitale=1.00, d'=0.02, Italia=0.15  → best = 1.00 (capitale)
    "Italia"   vs Roma=0.77, capitale=0.13, d'=0.02, Italia=1.00  → best = 1.00 (Italia)
    score = 1.00 + 1.00 = 2.00

  MaxSim(query, doc B) — compare every query token against every doc B token, keep only the best:
    "capitale" vs Milano=0.35, capitale=0.96, economica=0.23, Paese=0.68  → best = 0.96 (capitale)
    "Italia"   vs Milano=0.12, capitale=0.12, economica=0.12, Paese=0.74  → best = 0.74 (Paese)
    score = 0.96 + 0.74 = 1.70

  ranking: doc A (2.00) > doc B (1.70)
```

**input:** query and document encoded completely separately, same as "no interaction" — but instead of one vector each, every token keeps its own, and document token vectors are computed and stored offline, before any query exists. **output:** not one comparison, but many — every query token is compared against *every* document token (4 comparisons per query token per document above), and only the single **best** one counts toward that query token's contribution to the final score.

both documents match `capitale` almost perfectly (1.00 and 0.96), since both literally contain that exact word — MaxSim doesn't get fooled or blurred by that, it just correctly scores it as a strong match on both sides. the ranking is decided by the *other* query token: `Italia` finds a near-perfect match inside doc A (the word "Italia" itself, 1.00), but the best match it finds anywhere in doc B is `Paese` ("country", 0.74) — related, clearly weaker. that's the level of detail a single pooled vector throws away, and that a cross-encoder would also catch — but only by paying the full per-document cost that late interaction avoids by precomputing document tokens once, offline.

</details>

## 🎯 Why an Italian-Specialized Model?

before this project, the Italian options were:

- multilingual late-interaction models that *include* Italian among many languages ([`jina-colbert-v2`](https://huggingface.co/jinaai/jina-colbert-v2), [`mLateOn`](https://huggingface.co/lightonai/mLateOn), [`ColBERT-XM`](https://huggingface.co/antoinelouis/colbertxm), [`SauerkrautLM-Multi-ModernColBERT`](https://huggingface.co/VAGOsolutions/SauerkrautLM-Multi-ModernColBERT))
- strong Italian dense embedders that drop late interaction entirely

not "the first Italian ColBERT" — the models above already cover Italian — but the first one **specialized** on it.

## 🛠️ How It Was Built

**backbone:** [`nickprock/Italian-ModernBERT-base-embed-mmarco-mnrl`](https://huggingface.co/nickprock/Italian-ModernBERT-base-embed-mmarco-mnrl) → PyLate `ColBERT` (token vectors, dim 128, MaxSim). starting from a checkpoint that already retrieves, rather than a raw language model, follows the [ColBERT-Zero](https://huggingface.co/blog/lightonai/colbert-zero) efficiency result: supervised contrastive + distillation from a retrieval-capable init reaches ~99% of full multi-vector pretraining at roughly **10× lower cost**.

| stage | what it does |
|---|---|
| **phase 1 — contrastive** | supervised contrastive training on mMARCO-it triples, reranker-mined hard negatives, Italian Wikipedia retrieval pairs, MIRACL-ita and SQuAD-ita — ~2.4M triplets |
| **phase 2 — distillation** | KL distillation from a single cross-encoder teacher (`mxbai-rerank-large-v2`, via LightOn's dataset), sample budget spread proportionally across all 8 dataset splits |
| **checkpoint selection** | on retrieval metrics (nDCG@10 / MRR@10), never on hold-out KD loss alone |

## 🗺️ The Journey (Including What Didn't Work)

three rounds of work, and two of them were rejected after fair testing — which is arguably the more useful part to write down:

1. **round 1 — broaden the data, then distil.** the starting checkpoint only knew machine-translated mMARCO. added more varied Italian sources, then a distillation stage from a cross-encoder teacher. this became the model that shipped — strongest Italian-specialized late-interaction model tested, weakest on long documents.
2. **found the long-document weakness was mostly mechanical.** the hardest benchmark's documents run a few thousand words; they were being truncated at 512 tokens, discarding ~80% of the average document. chunking documents at query time — **no retraining at all** — recovered most of the gap (+0.06 nDCG@10, the largest single gain in the whole project).
3. **round 2 — mined harder negatives** from the model's own prediction mistakes. no real improvement, and a measurable regression in generalization. rejected.
4. **round 3 — trained the model to natively read twice as much text per document** (1024 vs 512 tokens), instead of relying on query-time chunking. ran a controlled A/B on identical data. statistically indistinguishable from just chunking a normally-trained model, once compared fairly on held-out queries. rejected.
5. **what shipped:** since neither training round beat "train normally, then chunk long documents at query time," that's exactly what's in the released model.

the full numbers behind every one of these steps — including the significance testing, not just the headline deltas — are in [`TODO.md`](https://github.com/enricollen/it-colbert/blob/main/TODO.md) in the repo.

## 📊 Results

real numbers, paired-bootstrap significance tested. **†** = not statistically distinguishable from ItColBERT (p > .05) — read those as ties regardless of which number is higher.

| Model | MLDR-it (nDCG@10) | mMARCO-it (MRR@10) | MIRACL-ita (nDCG@10) | SQuAD-ita (nDCG@10) |
|---|---|---|---|---|
| **ItColBERT** | **0.4008** (0.4610 chunked) | **0.7196** | **0.7194** | **0.9026** |
| mLateOn | 0.4623 | 0.8207 | 0.7880 | 0.9480 |
| jina-colbert-v2 | 0.3858 † | 0.8389 | 0.7755 | 0.8849 |
| bge-m3 (dense) | 0.4531 | 0.7812 | 0.7566 | 0.8247 |
| multilingual-e5-large (dense) | 0.4310 † | 0.8239 | 0.7653 | 0.8513 |
| SauerkrautLM-Multi-ModernColBERT | 0.3122 | 0.5342 | 0.5996 | 0.8338 |
| ColBERT-XM | 0.2734 | 0.6654 | 0.6260 | 0.8558 |
| BM25 | 0.4850 (vs. 0.4610 chunked: †) | 0.5715 | 0.5516 | 0.8262 |

the strongest **Italian-specialized** late-interaction model tested here, beating every general-purpose late-interaction alternative except one (mLateOn, the strongest multilingual late-interaction model found). behind large multilingual dense embedders on most benchmarks — matching those was never the goal, they're a different model class.

## 🪶 Small but Capable

one point worth calling out on its own: **ItColBERT is the smallest model in the whole comparison** — and it isn't close.

| Model | Parameters | MLDR-it (nDCG@10) |
|---|---|---|
| **ItColBERT** | **~135M** | **0.4008** (0.4610 chunked) |
| SauerkrautLM-Multi-ModernColBERT | 149M | 0.3122 |
| ColBERT-XM | 277M | 0.2734 |
| mLateOn | 307M | 0.4623 |
| multilingual-e5-large (dense) | 560M | 0.4310 † |
| bge-m3 (dense) | 568M | 0.4531 |
| jina-colbert-v2 | ~0.6B | 0.3858 † |

built on ModernBERT-**base** rather than one of the ~560M-parameter multilingual giants, ItColBERT still comes out ahead of `SauerkrautLM-Multi-ModernColBERT` (roughly the same size class), `ColBERT-XM` (2× the parameters), and statistically ties `jina-colbert-v2` (~4.4× the parameters) on the primary out-of-domain benchmark. mLateOn is the one model that beats it outright while also being smaller than the dense giants — worth being upfront about rather than glossing over.

being 4× smaller than the multilingual giants and still landing in the same neighborhood matters in practice: smaller index footprint, cheaper inference, and it's realistic to fine-tune or run fully offline on a single consumer GPU — which is exactly how the whole thing was built and evaluated in the first place.

## 🚀 Try It

```bash
pip install -U pylate
```

```python
from pylate import rank, models

model = models.ColBERT(model_name_or_path="enricollen/ItColBERT")

queries = ["Qual è la capitale d'Italia?"]
documents = [[
    "Roma è la capitale d'Italia.",
    "Milano è la capitale economica del Paese.",
]]
documents_ids = [[1, 2]]

queries_embeddings = model.encode(queries, is_query=True)
documents_embeddings = model.encode(documents, is_query=False)

reranked = rank.rerank(
    documents_ids=documents_ids,
    queries_embeddings=queries_embeddings,
    documents_embeddings=documents_embeddings,
)
print(reranked)
# [[{'id': 1, 'score': 31.682}, {'id': 2, 'score': 31.552}]]
# one list per query, sorted highest score first — "Roma" wins, as expected.
```

model card, indexing recipe for full corpora, and the long-document chunking trick are all on the [Hugging Face page](https://huggingface.co/enricollen/ItColBERT).

## 💻 Hardware

everything — every training run, every benchmark, all significance testing — was done on **one consumer machine**, not a cluster:

- **CPU**: Intel Core i7-14700K
- **GPU**: NVIDIA RTX 3090 (24GB)
- **RAM**: 32GB (27GB usable under WSL2)

no multi-GPU, no cloud compute. that ceiling shaped quite a few decisions along the way — mini-batch sizes, chunking's ~26GB host-RAM cost, a couple of OOM kills documented in the repo.

## ⚠️ Limitations & What's Next

- MIRACL-ita and SQuAD-ita are **community machine translations**, not official benchmark resources — labelled as such everywhere they're used
- behind large multilingual dense embedders on most axes; the goal here was an Italian-*specialized* late-interaction model, not beating general-purpose giants
- next candidates on the list: RRF fusion with BM25 (looks like the cheapest real gain left), a dim-64 variant for smaller indexes, and pinning dataset revisions for full reproducibility

## 🔗 Links

- 🐙 GitHub repository: **[github.com/enricollen/it-colbert](https://github.com/enricollen/it-colbert)** — training code, benchmark suite, and the full development history (if you find it useful, a star ⭐ is always appreciated)
- 🤗 Model on Hugging Face: **[huggingface.co/enricollen/ItColBERT](https://huggingface.co/enricollen/ItColBERT)** — ready-to-use weights, model card, and usage examples

## 🎬 Conclusion

this was as much a project about **measurement discipline** as it was about training a model. two separate training-based attempts (harder negatives, training at longer document length) both failed to beat a training-free trick (chunking) that only surfaced because I bothered to check where the truncation was actually happening. that's the actual lesson from this project — not every improvement needs more GPU hours, sometimes it needs a better look at the data first.

if you work with Italian text search or RAG and want token-level retrieval without going multilingual, give it a try and let me know how it performs on your data.
