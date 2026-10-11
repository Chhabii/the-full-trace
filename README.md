# The Full Trace

This repository holds the text, the data and the sources behind the film **How One ChatGPT-Style Request Is Served, End to End, at Global Scale** and its book.

- **Watch the film** (5 h 26 min, 35 chapters): https://www.youtube.com/watch?v=W4LuFlwm8i4
- **Read the book** (PDF): [the-full-trace.pdf](https://github.com/Chhabii/the-full-trace/releases/latest/download/the-full-trace.pdf)
- **Check the numbers**: every number in the film and the book comes from the data and sources in this repository.

The film follows one message to an AI chat service. It goes from the browser through the network edge, a region, the gateways and the model router, into a GPU cluster, through the KV cache token by token, and back to the screen. The system in the film is a **reference system**: a realistic design built from public sources. It does not describe the real system of any one company.

## Start here

1. **To learn the subject**, watch the film or read the book. Both follow the same order.
2. **To read one topic in full**, open its part in [`manuscript/`](manuscript/). The map below links each film chapter to its part.
3. **To check a number**, follow its source ID (for example `[SRC-748]`) to [`docs/sources.md`](docs/sources.md), or its claim ID (for example `CL-2273`) to [`docs/claims.md`](docs/claims.md). The input values are in [`constants/`](constants/), and the computed results are in [`data/`](data/).

## Map: film chapter, time and manuscript part

| # | Time | Film chapter | Manuscript |
|---|---|---|---|
| 0 | [0:00](https://www.youtube.com/watch?v=W4LuFlwm8i4&t=0s) | Cold open: one Send, the fleet behind it, the question | [Part 00](manuscript/part-00-the-question-and-the-reference-request.md), [Part 25](manuscript/part-25-misconceptions-and-synthesis.md) |
| 1 | [3:32](https://www.youtube.com/watch?v=W4LuFlwm8i4&t=212s) | T−∞: what is already running | [Part 01](manuscript/part-01-t-minus-infinity.md), [Part 22](manuscript/part-22-quantitative-workbook.md) |
| 2 | [15:11](https://www.youtube.com/watch?v=W4LuFlwm8i4&t=911s) | Pressing Send: client, request envelope, upload path | [Part 02](manuscript/part-02-client-and-global-network.md) |
| 3 | [20:04](https://www.youtube.com/watch?v=W4LuFlwm8i4&t=1204s) | Packets: DNS, BGP and anycast, TCP and QUIC, TLS, the edge | [Part 02](manuscript/part-02-client-and-global-network.md) |
| 4 | [28:27](https://www.youtube.com/watch?v=W4LuFlwm8i4&t=1707s) | Getting to a region: the routing hierarchy | [Part 02](manuscript/part-02-client-and-global-network.md) |
| 5 | [33:38](https://www.youtube.com/watch?v=W4LuFlwm8i4&t=2018s) | The backend: ingress, rate limits, concurrency, state, persistence | [Part 03](manuscript/part-03-application-backend.md), [Part 00](manuscript/part-00-the-question-and-the-reference-request.md) |
| 6 | [43:05](https://www.youtube.com/watch?v=W4LuFlwm8i4&t=2585s) | The orchestrator, model routing, safety checks before inference | [Part 03](manuscript/part-03-application-backend.md), [Part 24](manuscript/part-24-build-a-miniature.md) |
| 7 | [48:51](https://www.youtube.com/watch?v=W4LuFlwm8i4&t=2931s) | Turning inputs into tokens: text, image, PDF | [Part 04](manuscript/part-04-multimodal-ingestion.md) |
| 8 | [58:53](https://www.youtube.com/watch?v=W4LuFlwm8i4&t=3533s) | Retrieval: chunks, vectors, HNSW, BM25, fusion, rerank, ACLs, memory | [Part 05](manuscript/part-05-retrieval-memory-context.md), [Part 24](manuscript/part-24-build-a-miniature.md) |
| 9 | [1:11:02](https://www.youtube.com/watch?v=W4LuFlwm8i4&t=4262s) | Building the context: budget, history, four kinds of memory | [Part 05](manuscript/part-05-retrieval-memory-context.md) |
| 10 | [1:17:23](https://www.youtube.com/watch?v=W4LuFlwm8i4&t=4643s) | Into the inference system: router, admission, queue, Little's Law | [Part 09](manuscript/part-09-inference-routing-admission-queueing.md), [Part 23](manuscript/part-23-diagram-atlas.md) |
| 11 | [1:26:10](https://www.youtube.com/watch?v=W4LuFlwm8i4&t=5170s) | Prefill: the request meets the GPU | [Part 11](manuscript/part-11-gpu-execution-and-batching.md) |
| 12 | [1:33:15](https://www.youtube.com/watch?v=W4LuFlwm8i4&t=5595s) | Inside one layer: the Transformer math | [Part 10](manuscript/part-10-transformer-internals.md) |
| 13 | [1:46:12](https://www.youtube.com/watch?v=W4LuFlwm8i4&t=6372s) | The GPU doing the math: GEMMs, roofline, FlashAttention, kernels | [Part 11](manuscript/part-11-gpu-execution-and-batching.md), [Part 10](manuscript/part-10-transformer-internals.md), [Part 22](manuscript/part-22-quantitative-workbook.md) |
| 14 | [1:57:30](https://www.youtube.com/watch?v=W4LuFlwm8i4&t=7050s) | The KV cache: size, pages, prefix reuse, tiers, privacy | [Part 13](manuscript/part-13-kv-cache.md) |
| 15 | [2:10:49](https://www.youtube.com/watch?v=W4LuFlwm8i4&t=7849s) | Many GPUs as one model: TP, PP, DP, EP, rack-scale NVLink | [Part 12](manuscript/part-12-distributed-inference.md) |
| 16 | [2:27:06](https://www.youtube.com/watch?v=W4LuFlwm8i4&t=8826s) | Disaggregation: KV moves from prefill to decode | [Part 13](manuscript/part-13-kv-cache.md), [Part 22](manuscript/part-22-quantitative-workbook.md) |
| 17 | [2:33:44](https://www.youtube.com/watch?v=W4LuFlwm8i4&t=9224s) | The first token: logits, sampling, detokenization, determinism | [Part 10](manuscript/part-10-transformer-internals.md) |
| 18 | [2:42:54](https://www.youtube.com/watch?v=W4LuFlwm8i4&t=9774s) | The decode loop and continuous batching, with the fleet in view | [Part 11](manuscript/part-11-gpu-execution-and-batching.md), [Part 13](manuscript/part-13-kv-cache.md), [Part 22](manuscript/part-22-quantitative-workbook.md), [Part 24](manuscript/part-24-build-a-miniature.md) |
| 19 | [2:56:57](https://www.youtube.com/watch?v=W4LuFlwm8i4&t=10617s) | A tool call leaves the GPU: tool calling and MCP | [Part 06](manuscript/part-06-tool-calling.md), [Part 15](manuscript/part-15-streaming-back-to-pixels.md), [Part 24](manuscript/part-24-build-a-miniature.md) |
| 20 | [3:07:38](https://www.youtube.com/watch?v=W4LuFlwm8i4&t=11258s) | Web search: deciding, searching, crawling and indexing, fetching safely, injection, citations | [Part 07](manuscript/part-07-web-search-and-browsing.md) |
| 21 | [3:19:10](https://www.youtube.com/watch?v=W4LuFlwm8i4&t=11950s) | Code sandbox and computer use | [Part 08](manuscript/part-08-code-and-computer-use-sandboxes.md) |
| 22 | [3:25:41](https://www.youtube.com/watch?v=W4LuFlwm8i4&t=12341s) | The second inference pass and the reasoning mode | [Part 06](manuscript/part-06-tool-calling.md), [Part 09](manuscript/part-09-inference-routing-admission-queueing.md), [Part 10](manuscript/part-10-transformer-internals.md), [Part 24](manuscript/part-24-build-a-miniature.md) |
| 23 | [3:33:51](https://www.youtube.com/watch?v=W4LuFlwm8i4&t=12831s) | Streaming back: gRPC, SSE, buffering, backpressure, browser rendering to pixels | [Part 15](manuscript/part-15-streaming-back-to-pixels.md) |
| 24 | [3:43:41](https://www.youtube.com/watch?v=W4LuFlwm8i4&t=13421s) | Why the first token was fast: the TTFT breakdown and the timeline | [Part 22](manuscript/part-22-quantitative-workbook.md) |
| 25 | [3:49:08](https://www.youtube.com/watch?v=W4LuFlwm8i4&t=13748s) | Scale: a million generations, capacity math, the curves, demand and troughs | [Part 14](manuscript/part-14-concurrency-and-capacity.md), [Part 11](manuscript/part-11-gpu-execution-and-batching.md), [Part 22](manuscript/part-22-quantitative-workbook.md), [Part 25](manuscript/part-25-misconceptions-and-synthesis.md) |
| 26 | [4:05:24](https://www.youtube.com/watch?v=W4LuFlwm8i4&t=14724s) | Storage, caches, events, billing | [Part 16](manuscript/part-16-storage-caches-events.md) |
| 27 | [4:13:48](https://www.youtube.com/watch?v=W4LuFlwm8i4&t=15228s) | When things break | [Part 17](manuscript/part-17-reliability.md), [Part 14](manuscript/part-14-concurrency-and-capacity.md) |
| 28 | [4:27:22](https://www.youtube.com/watch?v=W4LuFlwm8i4&t=16042s) | Security, isolation, privacy, safety | [Part 18](manuscript/part-18-security-privacy-safety.md), [Part 07](manuscript/part-07-web-search-and-browsing.md) |
| 29 | [4:37:41](https://www.youtube.com/watch?v=W4LuFlwm8i4&t=16661s) | Watching and scaling the fleet: traces, metrics, SLOs, autoscaling, cold start, quality | [Part 19](manuscript/part-19-observability-slos-autoscaling.md), [Part 22](manuscript/part-22-quantitative-workbook.md) |
| 30 | [4:51:13](https://www.youtube.com/watch?v=W4LuFlwm8i4&t=17473s) | Other branches: image generation, voice, long-running agents | [Part 20](manuscript/part-20-other-branches.md) |
| 31 | [4:58:11](https://www.youtube.com/watch?v=W4LuFlwm8i4&t=17891s) | The second turn, the state map, cleanup | [Part 21](manuscript/part-21-super-trace-and-system-state.md), [Part 23](manuscript/part-23-diagram-atlas.md) |
| 32 | [5:04:53](https://www.youtube.com/watch?v=W4LuFlwm8i4&t=18293s) | The full super trace at speed, with the data transformation chain | [Part 21](manuscript/part-21-super-trace-and-system-state.md), [Part 23](manuscript/part-23-diagram-atlas.md) |
| 33 | [5:12:08](https://www.youtube.com/watch?v=W4LuFlwm8i4&t=18728s) | Misconceptions corrected | [Part 25](manuscript/part-25-misconceptions-and-synthesis.md) |
| 34 | [5:19:27](https://www.youtube.com/watch?v=W4LuFlwm8i4&t=19167s) | Synthesis: why it is fast at this scale | [Part 25](manuscript/part-25-misconceptions-and-synthesis.md), [Part 22](manuscript/part-22-quantitative-workbook.md), [Part 23](manuscript/part-23-diagram-atlas.md), [Part 24](manuscript/part-24-build-a-miniature.md) |

## The manuscript

The manuscript is the full text of the subject in 26 parts. GitHub shows its diagrams directly.

- [Part 00: Part 00: The question, the reference request, and how to read this](manuscript/part-00-the-question-and-the-reference-request.md)
- [Part 01: Part 01: T−∞, the system before a request arrives](manuscript/part-01-t-minus-infinity.md)
- [Part 02: Part 02: The client and the global network](manuscript/part-02-client-and-global-network.md)
- [Part 03: Part 03: The application backend and orchestration](manuscript/part-03-application-backend.md)
- [Part 04: Part 04: Multimodal ingestion](manuscript/part-04-multimodal-ingestion.md)
- [Part 05: Part 05: Retrieval, memory and context construction](manuscript/part-05-retrieval-memory-context.md)
- [Part 06: Part 06: Tool calling](manuscript/part-06-tool-calling.md)
- [Part 07: Part 07: Web search and browsing](manuscript/part-07-web-search-and-browsing.md)
- [Part 08: Part 08: Code and computer-use sandboxes](manuscript/part-08-code-and-computer-use-sandboxes.md)
- [Part 09: Part 09: Inference routing, admission and queueing](manuscript/part-09-inference-routing-admission-queueing.md)
- [Part 10: Part 10: Transformer internals, one forward pass](manuscript/part-10-transformer-internals.md)
- [Part 11: Part 11: GPU execution and batching](manuscript/part-11-gpu-execution-and-batching.md)
- [Part 12: Part 12: Distributed inference](manuscript/part-12-distributed-inference.md)
- [Part 13: Part 13: KV-cache systems](manuscript/part-13-kv-cache.md)
- [Part 14: Part 14: Concurrency and capacity at huge scale](manuscript/part-14-concurrency-and-capacity.md)
- [Part 15: Part 15: Streaming back to pixels](manuscript/part-15-streaming-back-to-pixels.md)
- [Part 16: Part 16: Storage, caches and events](manuscript/part-16-storage-caches-events.md)
- [Part 17: Part 17: Reliability](manuscript/part-17-reliability.md)
- [Part 18: Part 18: Security, privacy and safety](manuscript/part-18-security-privacy-safety.md)
- [Part 19: Part 19: Observability, SLOs, autoscaling and quality](manuscript/part-19-observability-slos-autoscaling.md)
- [Part 20: Part 20: Other branches](manuscript/part-20-other-branches.md)
- [Part 21: Part 21: The super trace and system state](manuscript/part-21-super-trace-and-system-state.md)
- [Part 22: Part 22: The quantitative workbook](manuscript/part-22-quantitative-workbook.md)
- [Part 23: Part 23: Diagram atlas](manuscript/part-23-diagram-atlas.md)
- [Part 24: Part 24: Build a miniature version](manuscript/part-24-build-a-miniature.md)
- [Part 25: Part 25: Misconceptions, diagnostics and synthesis](manuscript/part-25-misconceptions-and-synthesis.md)

## What is in this repository

```
manuscript/   the full text in 26 parts, with diagrams
data/         the results that the calculations produced, one JSON file per topic
constants/    the model and hardware facts with their sources, and the hypothetical values
docs/
  reference-architecture.md   the reference system: each choice (RA-n) with its reason
  sources.md                  the sources (SRC-n) with their access dates
  claims.md                   each claim (CL-n), its evidence class and its source
  glossary.md                 the technical terms
LICENSE       Creative Commons Attribution 4.0 International
CITATION.cff  how to cite this work
```

## How to read the numbers

- Every important claim has a truth class. **A** is a publicly documented mechanism, with a source link or a code citation. **B** is a common production design. **C** is a choice made for this reference system. **D** is proprietary or unknown, so the text never states it as fact. [`docs/claims.md`](docs/claims.md) lists each claim with its class.
- All traffic and fleet numbers are **hypothetical**. The text labels them "hypothetical" or "illustrative" each time. They live in [`constants/hypothetical.json`](constants/hypothetical.json).
- Some evidence pointers name private research notes (for example `research/notes/03-engines.md §2.5`). Those notes are not in this repository. Each pointer also names the public source that the note quotes.
- The calculation code and the film's scene code are not in this repository yet.

## Credits

The film uses these tools and assets: [Remotion](https://www.remotion.dev/) for the animation, an [ElevenLabs](https://elevenlabs.io/) voice for the narration, [Kenney Interface Sounds](https://kenney.nl/assets/interface-sounds) (CC0) for the sound effects, [Natural Earth](https://www.naturalearthdata.com/) (public domain) through [world-atlas](https://github.com/topojson/world-atlas) for the map, [KaTeX](https://katex.org/) for the formulas, [three.js](https://threejs.org/) and [d3](https://d3js.org/) for drawing, and the fonts Inter and JetBrains Mono (SIL Open Font License 1.1).

This is an independent explainer. It is not affiliated with, or endorsed by, any company or project that it names.

## Licence and citation

© 2026 Chhabii. The text, the diagrams and the data are licensed under [CC BY 4.0](LICENSE): you can share and adapt them, also for commercial use, if you give credit. Cite this work with [`CITATION.cff`](CITATION.cff).

## Errors

If you find an error, open an issue. Give the chapter or part, the section, the claim and, if you have one, a better source.

---

The Full Trace on YouTube: [@thefulltrace](https://www.youtube.com/@thefulltrace)
