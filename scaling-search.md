---
marp: true
footer: '[@skeptrune](htttps://x.com/skeptrune) - nick.k@trieve.ai'
transition: fade
theme: gaia
---

<!-- _footer: 'https://github.com/skeptrunedev/scaling-search' -->

![bg left](https://cdn.trieve.ai/scaling-search-presentation/sharded-cubism.jpeg)

# Scaling Retrieval With Sharding

### 1.2B Chunks at <100ms

---

![bg right:40%](https://avatars.githubusercontent.com/u/15804464?v=4)

## Nicholas Khami
### Founder/CEO </br>[trieve](https://trieve.ai)

<i class="fa-brands fa-twitter"></i> Twitter: @skeptrune
<i class="fa-brands fa-mastodon"></i> Mastodon: @skeptrune
<i class="fa-brands fa-linkedin"></i> LinkedIn: - [nicholas-khami](https://www.linkedin.com/in/nicholas-khami-5a0a7a135/)
<i class="fa-brands fa-github"></i> GitHub: [github.com/skeptrunedev](https://github.com/skeptrunedev)

---

![bg left](https://cdn.trieve.ai/scaling-search-presentation/cubism-clock)

# Agenda

- Latency by layer in retrieval
- Fast Typo Detection
- Optimizing vector inference
- Scaling ingest
- Sharding your search index
  
---

![bg right:40%](https://cdn.trieve.ai/scaling-search-presentation/server-timing-list-small.png)

# Latency by layer

1. Inference vectors for query ~15ms
2. Typo detection+correction ~1ms
3. Score docs in search index ~10ms
4. Re-rank (LTR/cross-encode) ~15ms
5. First LLM token (TTFT) ~250ms

--- 

# Fast Typo Correction

<!-- _footer: '[trieve.ai/building-blazingly-fast-typo-correction-in-rust](https://trieve.ai/building-blazingly-fast-typo-correction-in-rust)' -->

![bg left:40%](https://cdn.trieve.ai/scaling-search-presentation/cubism-crab.jpeg)

- Many vector db's don't correct typos
    - Harder when multi-tenant
- BK-Tree vs. SymSpell Data Structures
    - [SymSpell is 100x faster](https://seekstorm.com/blog/symspell-vs-bk-tree/)

--- 

# PSA: Vector/Re-ranker Latency

<!-- _footer: '[docs.trieve.ai/vector-inference](https://docs.trieve.ai/vector-inference)' -->

![bg right:30%](https://cdn.trieve.ai/scaling-search-presentation/sand-timer.jpeg)

- Cloud API's take ~150ms, but 15ms possible w/ Candle
- Ingesting 1,000,000 chunks (33k batches)
    - ~80mins using OpenAI
    - ~8mins on Candle
- LTR plugins usually <10ms

--- 

# Scaling Ingest

<!-- _footer: '[github.com/trieve/ingestion-worker](https://github.com/devflowinc/trieve/blob/main/server/src/bin/ingestion-worker.rs)' -->

![bg left:30%](https://cdn.trieve.ai/scaling-search-presentation/katy-fwy-ingest.jpeg)

- You almost certainly need a queue+worker architecture
- Using a worker for updates and deletions enables chunking experiments at scale
- Consider an id system which allows you to upsert easily

--- 


<img width="100%" height="100%" src="https://trieve.b-cdn.net/scaling-search-presentation/trieve-queue-diagram.png" />

---

# Sharding your search index Pt. 1

## TLDR: think about it

### What is a shard?

- A shard is an instance of the search lib (Lucene, FAISS, etc.)
- Shards are made up of segments
- Segments are indices (IDF or HNSW)
- Small segments should be merged off-peak hours

---

# Sharding your search index Pt. 2

### How to optimize?

- Usually best to think about sharding before HNSW params
- Always have `shards > 2*nodes` such that you can horizontally scale CPU if needed
- Replicate shards for more read throughput
- Different db's map their thread pools to shards differently and you should work with your vendor to address it

