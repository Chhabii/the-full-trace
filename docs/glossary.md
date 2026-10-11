# Glossary

One term per concept, defined once, used everywhere.
The "Do not use" column lists synonyms that must not appear for the same concept, so the text never drifts.
Seeded 2026-10-09 for Phase 2. Writers add a term the first time it is defined in the manuscript and fill in "First defined in".

## Requests, users and conversations

| Term | Definition | Do not use | First defined in |
|---|---|---|---|
| request | One user turn sent to the system: the HTTP call from the browser and all the work it causes, until the final token is streamed. | query (except in retrieval), job | |
| reference request | The single example request that the whole manuscript follows (`constants.json` → `hypothetical.reference_request`). | example request, sample request | part-00 §0.2 |
| your request | The reference request, seen from the viewer's side. Drawn in amber in the video. |  | part-00 §0.2 |
| user | The person who presses Send. | customer, client (for the person) | |
| tenant | The organization or account that owns data and quotas; the unit of isolation. | customer, org (in prose) | part-03 §3.1 |
| conversation | The ordered list of stored messages between a user and the assistant. | chat (as a noun for stored data), thread (for stored messages) | |
| turn | One user message plus the assistant's answer to it. | round (for a turn) | |
| message | One stored entry in a conversation (user, assistant or tool role). | | |
| client | The browser or app code that builds and sends the request. | frontend (except "frontend application" in 2.1) | part-02 §2.1 |
| idempotency key | A client-chosen random ID that names one intent; the server uses it to recognize a repeat and run the operation at most once. | retry token | part-02 §2.1 |
| conversation store | The distributed SQL store that holds conversations, messages, settings and quota counters, with a home region per user. | Conversation DB, conversation database | part-03 §3.4 |
| daily active users (DAU) | Users who send at least one request in a day (hypothetical: 250,000,000). | | part-14 §14.1 |
| think time | The time between the end of one response and the next request of the same user. | idle time (for this) | part-14 §14.1 |
| duty cycle | The fraction of time that a user has a generation in flight (hypothetical: 4% at the peak). | | part-14 §14.1 |

## Tokens and text

| Term | Definition | Do not use | First defined in |
|---|---|---|---|
| token | One unit of the model's vocabulary, identified by a token ID. | word piece (except when explaining BPE) | part-04 §4.2 |
| token ID | The integer index of a token in the vocabulary. | token number | part-04 §4.2 |
| tokenizer | The program that turns text into token IDs and back. | encoder (reserved for the vision encoder) | part-04 §4.2 |
| vocabulary | The fixed set of tokens a model knows; its size is `vocab_size`. | dictionary | part-04 §4.2 |
| special token | A token with a control meaning (role, end of turn, tool call, image placeholder). | control character | part-04 §4.2 |
| chat template | The rule that turns structured messages into one token sequence. | prompt format | part-04 §4.2 |
| prompt tokens | The tokens the model reads for one inference pass (input). | input length (in prose), ISL (only in tables, defined) | |
| output tokens | The tokens the model generates in one inference pass. | completion tokens | |
| cached prompt tokens | The prompt tokens that the model server serves from the prefix cache in one pass; the invoice prices them lower (the price list calls them cached input tokens). | | part-16 §16.4 |
| context | The full token sequence given to the model for one inference pass. | prompt (for the whole sequence) | part-05 §5.6 |
| context window | The maximum number of tokens the model accepts in one context. | context length (for the limit), max length | part-05 §5.6 |
| context budget | The number of tokens the orchestrator lets one prompt use (the cap of 131,072 in the reference system, RA-68), shared among the parts of the context; part 5.6 counts it for your request. Not the scheduler's per-step token budget. | token budget (for the prompt), prompt budget | part-05 §5.6 |
| detokenization | Turning token IDs back into UTF-8 text. | decoding (reserved for the decode phase) | |
| UTF-8 | A byte encoding of Unicode text that uses 1 to 4 bytes per character; the first byte says how many follow. |  | part-10 §10.11 |
| incremental detokenizer | The detokenizer that decodes a short window of recent tokens at every step and sends only the new, complete characters. |  | part-10 §10.11 |
| replacement character | U+FFFD, the character a decoder writes for an incomplete or invalid byte sequence; at the end of a window it means the next token may complete the character. |  | part-10 §10.11 |
| byte-level BPE | Byte-pair encoding over the 256 byte values: it merges the most frequent adjacent pairs, so any byte string has a tokenization and no unknown token is needed. | wordpiece (for this) | part-04 §4.2 |
| visual token | A vector produced by the vision encoder and projector that takes one position in the LLM context (15,552 for the reference photo). | image token (reserved for discrete image codes in an early-fusion model) | part-04 §4.3 |
| patch | A 14 x 14 pixel block of an image that the vision encoder reads as one input (62,208 patches for the reference photo). | tile (for an image patch) | part-04 §4.3 |
| vision encoder | The model that turns the pixels of an image into feature vectors (MoonViT-V2 in `M-frontier`). | image encoder | part-04 §4.1 |
| projector | The small network that maps visual features to the LLM hidden size so that visual tokens take positions in the text sequence. | adapter (except for cross-attention adapters) | part-04 §4.3 |
| XTML | Kimi K3's chat format: XML-like markup in which three reserved special tokens replace the angle brackets; a tool call is a `call` element in the `tools` channel. | K3 tool syntax | part-06 §6.1 |
| Matryoshka representation learning (MRL) | A training method that makes one embedding carry information from coarse to fine granularity, so its first d dimensions still work as a smaller embedding. | nested embedding, dimension cutting (for the training) | part-05 §5.1 |

## The inference pass

| Term | Definition | Do not use | First defined in |
|---|---|---|---|
| inference pass | One run of the model over a context until a stop condition: one prefill plus a sequence of decode steps. | generation call, model call (in prose) | |
| prefill | The phase that processes all prompt tokens in parallel and writes their K and V into the KV cache. | prompt processing, context encoding | part-11 §11.3 |
| decode | The phase that generates one new token per sequence per step. | generation phase | part-11 §11.3 |
| decode step | One iteration of decode for every sequence in the batch. | iteration (except "scheduler iteration", see below) | |
| scheduler iteration | One loop of the engine's scheduler: choose the batch, run one model step, sample, update state. | engine step (in prose) | part-00 §0.6 |
| sequence | One request's token stream inside the engine; it owns its own KV blocks. | stream (reserved for the network stream) | |
| batch | The set of sequences that one scheduler iteration runs together. |  | part-00 §0.3 |
| continuous batching | Choosing the batch again at every scheduler iteration, so finished sequences leave and new ones join. | in-flight batching (mention once as TensorRT-LLM's name), dynamic batching | part-11 §11.5 |
| token budget | The maximum number of tokens one scheduler iteration may process (`max_num_batched_tokens`). For the limit on one prompt, use "context budget". | batch size (for tokens) | part-00 §0.6 |
| token quota | A tenant's allowance of tokens over a long period (for example per month), enforced on counters in the conversation store before inference (RA-10, RA-17). | token budget (for a tenant) | part-03 §3.1 |
| chunked prefill | Splitting a long prefill into chunks that fit the token budget, mixed with decode steps. | | part-11 §11.7 |
| static batching | Forming a batch, running it until its longest sequence ends, and only then forming the next batch; finished sequences leave their slots empty meanwhile. | request-level batching (for static batching) | part-11 §11.5 |
| stall-free batching | Sarathi-Serve's rule for one token budget: decodes first, then the partial prefill chunk, then new requests, so a long prefill never pauses the decodes. | decode-priority batching (for this rule) | part-11 §11.6 |
| generation stall | The gap in which the decodes of a batch get no token because one step runs a whole long prefill. | decode stall (for this gap) | part-11 §11.7 |
| preemption | Removing a running sequence from the batch and freeing its KV blocks so others can run. | eviction (reserved for caches) | part-13 §13.8 |
| logits | The model's raw score for every vocabulary entry at one position. | scores (alone) | part-10 §10.9 |
| sampling | Choosing the next token from the logits (temperature, top-k, top-p, min-p, greedy). | | part-10 §10.9 |
| stop condition | A rule that ends decode: EOS, stop sequence, max tokens, tool call, cancellation, timeout, policy stop. | | part-10 §10.14 |
| speculative decoding | A draft proposes several tokens and the target model verifies them in one forward pass. | | part-11 §11.10 |
| draft | The small model or head that proposes tokens in speculative decoding. | drafter, assistant model | part-11 §11.10 |
| acceptance rate (alpha) | The probability that one drafted token passes the acceptance test; the tokens per target pass are (1 − α^(γ+1)) ÷ (1 − α). | hit rate (for acceptance) | part-11 §11.10 |
| constrained decoding | Removing, before each sampling step, every token that would break a grammar, so the output is valid by construction. | guided decoding, structured outputs (that is vLLM's feature name) | part-06 §6.1 |
| grammar bitmask | One bit per vocabulary token for one sequence and step: 1 if the token keeps the output valid; the engine sets the logits of 0-bit tokens to minus infinity. | token filter, logit mask | part-06 §6.1 |
| forward pass | One run of all layers over a batch of token positions, from token IDs to logits; prefill runs one over all prompt tokens, each decode step runs one over one token per sequence. | pass (alone, reserved for inference pass) | part-10 intro |
| B, L, d_model | The three dimensions of every activation tensor: B sequences in the batch, L tokens of one sequence in this pass, d_model values per token (the hidden size). |  | part-10 §10.1 |
| embedding table | The matrix E of shape [vocab_size, d_model] whose row t is the starting vector of token ID t; the lookup copies rows and does no multiplication. | embedding matrix | part-10 §10.1 |
| decoder layer | One block of the model: an attention sub-block and an MLP sub-block, each with its own norm and each adding its result to the residual stream. | transformer block, transformer layer | part-10 §10.2 |
| residual stream | The running hidden state X that every sub-block reads (after a norm) and adds its result to; it passes through no norm until the final norm after the last layer. | skip connection (for the stream) | part-10 §10.2 |
| pre-norm | Placing the norm inside each residual branch, X + f(RMSNorm(X)), so the residual stream is never rescaled between layers. |  | part-10 §10.3 |
| RMSNorm | Root-mean-square normalization: divides each token's vector by its RMS and multiplies it by a learned scale; it subtracts no mean and has no bias. | layer norm (for RMSNorm) | part-10 §10.3 |
| query, key, value | The three per-head vectors that each normalized token vector is projected into: queries score against keys, and the scores weight the values. |  | part-10 §10.4 |
| causal mask | The mask that adds minus infinity to the score of every key after the query position, so a token never attends to a later token. | attention mask (for the causal mask) | part-10 §10.6 |
| softmax | The function that turns a vector of scores into weights between 0 and 1 that sum to 1: exponentiate each score, then divide by the sum. |  | part-10 §10.6 |
| online softmax | Computing a softmax block by block with a running maximum and sum, rescaling the earlier partial results when the maximum grows, so the result is exact without holding the whole row. | streaming softmax | part-10 §10.7 |
| FlashAttention | An exact attention kernel that splits Q, K and V into blocks and applies the online softmax, so the L x L score matrix never reaches HBM. |  | part-10 §10.7 |
| varlen attention | An attention kernel that takes all sequences of a batch as one flat token tensor plus an array of sequence boundaries, so no sequence is padded. | padded batch (for varlen input) | part-10 §10.1 |
| RoPE | Rotary position embedding: rotates each pair of query and key dimensions by an angle that grows with position, so a score depends only on the distance between two tokens. |  | part-10 §10.5 |
| YaRN | A long-context scaling rule for RoPE that divides the angles of slowly turning pairs by the context factor and blends the pairs in between. |  | part-10 §10.5 |
| NoPE | No positional encoding: the layers rotate nothing, as in the layers of `M-frontier`. |  | part-10 §10.5 |
| SwiGLU | The gated MLP of a decoder layer: SiLU of x W_gate, multiplied element by element with x W_up, then projected back by W_down. | FFN (for the SwiGLU MLP) | part-10 §10.8 |
| LM head | The final linear layer that turns the last hidden state of a sequence into one logit per vocabulary entry. | output head, unembedding matrix | part-10 §10.9 |
| weight tying | Reusing the embedding table, transposed, as the LM head matrix; neither reference model does this. |  | part-10 §10.9 |
| temperature | The number T that divides the logits before the softmax; T below 1 sharpens the distribution and T above 1 flattens it. |  | part-10 §10.9 |
| top-k, top-p, min-p | Filters that keep only the k most probable tokens, the smallest set whose probability reaches p, or the tokens at least min_p times as probable as the best one. |  | part-10 §10.9 |
| greedy decoding | Taking the token with the highest logit at every step, with no random draw. |  | part-10 §10.9 |
| seed | The number that fixes the random draws of the sampler for one request. | random seed (alone) | part-10 §10.12 |
| batch-invariant kernel | A kernel that uses one reduction order for every batch size, so a request's logits no longer depend on the other sequences in the batch. | deterministic kernel (for this) | part-10 §10.12 |
| reasoning token | A hidden output token that the model writes before its visible answer; it costs one decode step and one token of KV like any other output token. | thinking token (for the concept) | part-10 §10.13 |
| test-time compute | Spending more inference FLOPs per request to get a better answer, through reasoning tokens, several sampled answers or tool loops. | inference-time compute | part-10 §10.13 |
| finish reason | The field that says why a sequence ended: stop, length, repetition or tool call. | stop reason (alone; vLLM's field is `finish_reason`) | part-10 §10.14 |
| end-of-sequence token | The special token whose sampling ends a sequence. |  | part-10 §10.14 |
| repetition detection | A check after every token that ends the sequence when the same pattern of tokens has repeated a set number of times at the end of the output. | loop detection | part-10 §10.14 |

## Latency, throughput and load

| Term | Definition | Do not use | First defined in |
|---|---|---|---|
| TTFT | Time to first token: from the click on Send until the first token the model generates (reasoning tokens included) reaches the client as a stream event. The engine reports its own share, from arrival at the engine to its first sampled token, as `vllm:time_to_first_token_seconds`. | first-token latency | part-00 §0.1 |
| first visible text | The moment the first answer text (not a reasoning status) appears on screen; for a reasoning model it comes after the reasoning tokens, so it is much later than TTFT. | time to first visible token (as a name for TTFT) | part-00 §0.1 |
| TPOT | Time per output token after the first, averaged over one request. | | |
| ITL | Inter-token latency: the time between two consecutive tokens of one request. | | |
| end-to-end latency | From the click on Send to the last token. | total time (in prose), end-to-end time (for this) | |
| throughput | Work completed per second (requests/s or tokens/s; the unit is always stated). | capacity (for a rate) | part-11 §11.12 |
| goodput | Requests completed within their SLO per second. | | part-11 §11.12 |
| utilization | The fraction of time a resource is busy. | load (for this) | part-00 §0.4 |
| queue depth | The number of requests waiting, not yet running. | backlog (except "token backlog", defined) | part-00 §0.4 |
| pool saturation | One number for a whole pool: the mean over its endpoints of the larger of queue_score = (waiting + credit) / 5 and kv_score = KV usage / 0.8; a band dispatches only while it is below the band's ceiling. | pool load | part-09 §9.4 |
| in-flight tokens | The EPP's own count of uncached prompt tokens dispatched to an endpoint and not yet prefilled (added at dispatch, removed at the first response chunk). | pending tokens | part-09 §9.2 |
| token load | An endpoint's in-flight tokens plus the uncached tokens of the request being placed; the token-load scorer prefers the lowest. | | part-09 §9.3 |
| Little's Law | L = λ × W: for a stable system, the mean number of items inside equals the mean arrival rate times the mean time each item stays. | | part-09 §9.5 |
| in-flight generation | A generation that has started and not yet ended; the mean count equals the arrival rate times the mean decode time. | active generation (for this), live generation (for this) | part-14 §14.1 |
| peak-to-average ratio | The peak request rate divided by the mean request rate over the day (hypothetical: 1.45 globally). | peak factor (for this) | part-14 §14.1 |
| headroom | Capacity kept above the forecast peak, to cover forecast error and bursts shorter than the autoscaler's reaction time (hypothetical: 15%). | slack (for capacity), spare capacity (for this) | part-14 §14.6 |
| thundering herd | A crowd of clients that act in the same second, for example after an outage; jittered retries, admission control, caches and warm floors spread the spike. | retry storm (for this), stampede (for clients) | part-14 §14.7 |
| SLO | A target for a measured service level, for example p95 TTFT ≤ 1.5 s. | SLA (a contract, different) | part-19 §19.3 |
| p50, p95, p99 | The 50th, 95th, 99th percentiles of a distribution. | median (only as the p50 synonym once) | part-19 §19.2 |
| counter | A number that a component updates as it works; vLLM exposes its counters on an HTTP `/metrics` endpoint in Prometheus format. |  | part-00 §0.4 |
| active sequences | Requests that the scheduler runs in the current batch (`vllm:num_requests_running`). | running requests (in prose) | part-00 §0.4 |
| batched tokens | Tokens that one scheduler iteration processes (`vllm:iteration_tokens_total`). |  | part-00 §0.4 |
| tokens per second | Output tokens the engine produces each second (the rate of `vllm:generation_tokens`). |  | part-00 §0.4 |
| RTT | Round-trip time: the time from sending a packet to receiving the reply. | ping time, latency (for RTT) | part-02 §2 intro |
| RPM | A limit on requests per minute; each request costs one bucket token. |  | part-03 §3.2 |
| TPM | A limit on model tokens per minute; the cost of a request is a number of model tokens. |  | part-03 §3.2 |
| token bucket | A rate limiter with capacity B and refill rate r that admits a request only if enough bucket tokens remain, so it caps both the sustained rate and the burst. | leaky bucket (a different algorithm) | part-03 §3.2 |
| bucket token | A unit of rate-limit budget; not a model token. Each sentence in Part 03 says which sense it means. |  | part-03 §3.2 |
| TPM reservation | The `prompt_tokens + max_tokens` that the model router takes from the tenant TPM bucket before a pass; after the pass it charges the tokens used and refunds the rest (RA-10). | token reservation (for this) | part-16 §16.4 |

## KV cache and memory

| Term | Definition | Do not use | First defined in |
|---|---|---|---|
| KV cache | The stored K and V tensors of earlier tokens of a sequence, kept in GPU memory so they are read, not recomputed. | model memory, attention memory | part-13 §13.1 |
| KV block | A fixed-size group of tokens' K and V (16 tokens in vLLM's default), the unit of KV allocation. | page (except when comparing with OS paging), slot (for a block) | part-13 §13.2 |
| KV slot | The position of one token's K and V inside the block pool: block ID × block size + offset in the block. vLLM's `slot_mapping` holds one slot per new token. | | part-13 §13.2 |
| block table | The per-sequence map from logical block numbers to physical KV block numbers. | page table (except in the OS comparison) | part-13 §13.2 |
| block pool | All physical KV blocks on a GPU, with a free queue of blocks that no running sequence uses. | | part-13 §13.2 | part-01 §1.3 |
| prefix cache | Keeping KV blocks of finished requests so a later request with the same prefix reuses them. | prompt cache, context cache | part-01 §1.3 |
| prefix-cache hit | A request whose first blocks match cached blocks. | cache hit (alone, ambiguous) | part-13 §13.3 |
| cache salt | A per-tenant value mixed into block hashes so tenants never share cached blocks. | | part-13 §13.5 |
| hot prefix | A long prefix that a large share of all requests starts with, such as one big tenant's system prompt; a pure prefix score sends all of them to the endpoint that cached it first. | popular prefix | part-09 §9.3 |
| HBM | The GPU's high-bandwidth memory. | VRAM (except once, as a synonym), GPU RAM | part-11 §11.1 |
| latent KV | The compressed per-token vector that MLA caches instead of full K and V. | compressed cache | part-13 §13.1 |
| KV head | One K and V pair that several query heads read; `M-dense` has 8 KV heads for 96 query heads. | key-value head | part-10 §10.4 |
| recurrent state | The fixed-size per-sequence state that a linear-attention layer keeps instead of a growing KV cache. | hidden state (reserved for the residual stream) | part-13 §13.1 |
| hidden state | The `[B, L, d_model]` activations that flow between layers (the residual stream). | embedding (after layer 0) | part-00 §0.3 |
| MHA | Multi-head attention: every query head has its own K and V. |  | part-13 §13.1 |
| MQA | Multi-query attention: one K and V is shared by all query heads. |  | part-13 §13.1 |
| GQA | Grouped-query attention: groups of query heads share one KV head (12 query heads per KV head in `M-dense`); it sits between MHA and MQA. |  | part-13 §13.1 |
| MLA | Multi-head latent attention: stores one compressed vector per token per layer (the latent KV) instead of separate K and V per head. |  | part-13 §13.1 |
| linear attention | An attention layer that keeps a fixed-size recurrent state per sequence instead of a growing cache. |  | part-13 §13.1 |
| KDA | Kimi Delta Attention: the linear-attention layer of `M-frontier`; it keeps one fixed-size recurrent state per sequence. |  | part-13 §13.1 |
| sliding-window attention | An attention layer that keeps only the last W tokens, so its cost per layer is min(T, W) tokens. |  | part-13 §13.1 |
| sparse attention | Attention that reads only a selected subset of earlier tokens per query; every token's KV is still stored, so the cache does not shrink. |  | part-13 §13.1 |
| KV quantization | Storing K and V in FP8 (1 byte) instead of BF16 (2 bytes) to halve the bytes per token. |  | part-13 §13.1 |
| reference count | The number of block tables that point to one physical KV block; a block with count 0 returns to the free queue. |  | part-13 §13.2 |
| copy-on-write | Copying a shared partial block when one of its sequences writes into it; vLLM V1 shares only full blocks, so ordinary prefix sharing needs no copy. |  | part-13 §13.2 |
| free queue | The queue of free KV blocks in least-recently-used order; the engine takes new blocks from its head. | free list (the block-pool entry uses it once; prefer "free queue") | part-13 §13.2 |
| block hash | The hash of a full KV block: it covers the previous block's hash, this block's token IDs and the extra keys, so it identifies the entire prefix up to the end of that block. |  | part-13 §13.3 |
| radix tree | SGLang's prefix store: each edge holds a run of token IDs and each node points to the KV slots of its tokens. |  | part-13 §13.3 |
| recurrent-state checkpoint | A saved full copy of one sequence's recurrent state at a chosen token boundary, so a hybrid model can reuse a prefix. | checkpoint (alone) | part-13 §13.3 |
| KV tier | One level of the KV storage hierarchy: G1 HBM, G2 pinned CPU DRAM, G3 local NVMe, G4 a remote store. |  | part-13 §13.6 |
| GPUDirect RDMA | RDMA in which the NIC reads GPU memory directly, with no copy through CPU memory. |  | part-13 §13.7 |
| layer-wise streaming | Sending layer i's KV blocks while computing layer i+1, so only the last layer's transfer adds to the time. |  | part-13 §13.7 |
| free KV blocks | KV blocks that no running sequence holds; derived from `vllm:kv_cache_usage_perc` (usage = 1 - free blocks / (all blocks - 1)). | KV headroom | part-00 §0.4 |
| HBM used | Bytes of GPU memory in use, read from the GPU driver; it stays almost flat because the engine allocates the whole KV block pool at startup. |  | part-00 §0.4 |

## Distributed inference

| Term | Definition | Do not use | First defined in |
|---|---|---|---|
| replica | One complete copy of the model weights that can serve requests on its own. It can span one GPU, an 8-GPU node, or many nodes. | instance, copy (for a replica), deployment (for a replica) | part-01 §1.3 |
| model server | One running inference-engine process group that serves one replica and exposes one endpoint. | server (alone), pod (only for the Kubernetes object) | part-01 §1.3 |
| inference engine | The software that runs the model and schedules sequences (vLLM, SGLang, TensorRT-LLM). | runtime, framework | |
| worker | The group of GPUs that serves one replica, or one phase of it, as one unit that is started, routed to and drained together. For `M-frontier` a prefill worker has 8 GPUs (TEP8) and a decode worker has 16 GPUs (DEP16) (RA-55). | rank (for the whole group), instance (for a worker) | part-01 §1.3 |
| GPU process | The one process that drives one GPU inside a model server; vLLM calls it a worker process. This text says "rank" when it speaks of collectives and "GPU process" otherwise. | worker (for one GPU) | part-01 §1.3 |
| rank | The index of one GPU process inside a collective-communication group, also used for that process (a DEP16 decode worker has ranks 0 to 15). | worker (for one GPU) | part-01 §1.3 |
| endpoint | The network address of a model server that the inference gateway can pick. For `M-frontier` in this reference system, one address per DEP16 rank (RA-71), so one decode worker exposes 16 endpoints. | backend (except "backend" for the whole CPU tier), target (for an endpoint) | part-01 §1.9 |
| TP | Tensor parallelism: one layer's matrices split across GPUs. | model parallelism (ambiguous) | part-01 §1.3 |
| PP | Pipeline parallelism: layers split into stages on different GPUs. | | part-12 §12.2 |
| DP | Data parallelism: independent replicas (or attention DP ranks) that process different requests. | | part-12 §12.3 |
| EP | Expert parallelism: MoE experts spread across GPUs. | | part-12 §12.4 |
| collective | A communication operation that all ranks of a group take part in (AllReduce, AllGather, ReduceScatter, all-to-all). | | part-12 §12.1 |
| NVLink domain | The set of GPUs that NVLink connects directly through NVLink switches (8 in an HGX H100 node, 72 in a GB300 NVL72 rack). | NVLink island | part-12 §12.8 |
| prefill pool, decode pool | The groups of model servers dedicated to one phase in disaggregated serving. | prefill cluster, decode cluster | part-13 §13.7 |
| disaggregation | Running prefill and decode on different workers and moving the KV blocks between them. | split serving | part-13 §13.7 |
| KV transfer | Moving KV blocks between workers or tiers. | KV migration | part-13 §13.7 |
| expert | One MoE feed-forward network. | | part-12 §12.4 |
| expert router | The small layer that scores experts for each token and picks the top-k. | gate (except "gate" as the formula name), MoE router | part-12 §12.4 |
| MoE | Mixture of experts: a model whose MLP is replaced by many experts, of which a router picks a few for each token. | sparse model (for MoE) | part-01 §1.6 |
| AllReduce | The collective after which every rank holds the element-wise sum of all ranks' buffers; it equals a ReduceScatter followed by an AllGather. | all-reduce (for the collective) | part-12 §12.1 |
| ReduceScatter | The collective after which each rank holds one slice of the element-wise sum of all ranks' buffers. | reduce-scatter (spelling) | part-12 §12.1 |
| AllGather | The collective after which every rank holds the concatenation of every rank's slice. | all-gather (spelling) | part-12 §12.1 |
| all-to-all | The collective in which every rank sends a different piece to every other rank; MoE dispatch and combine use it. | alltoall (in prose) | part-12 §12.1 |
| custom all-reduce | vLLM's own kernel for small AllReduce messages (under 8 MiB, at most 8 fully NVLink-connected GPUs): each GPU reads its peers' buffers directly. Written this way only as the name of the kernel. | | part-12 §12.1 |
| NCCL | NVIDIA's collective communication library; for each call it picks an algorithm (ring, tree, NVLS) and a protocol from the topology. | | part-01 §1.3 |
| stage | One group of consecutive layers on its own GPUs in pipeline parallelism. Part 9 uses "stage" for a step of the dispatch order and Part 11 for a row of the stage card; those are different things. Part 21 uses "stage A to J" for one of the ten forms of your request, and Part 24 uses "evolution stage" (below) for a step of system growth. | layer group (for a stage) | part-12 §12.2 |
| microbatch | One slice of work that enters a pipeline on its own: a prefill chunk or a decode batch. | micro-batch (spelling) | part-12 §12.2 |
| bubble | The idle time of pipeline stages while the pipeline fills and drains; with S stages and M microbatches its fraction is (S − 1) / (M + S − 1). | pipeline stall (for the bubble) | part-12 §12.2 |
| TEP8 | The expert layout of one prefill worker: attention tensor-parallel over 8 GPUs, experts spread over the same 8 GPUs. | | part-12 §12.4 |
| DEP16 | The expert layout of one decode worker: attention data-parallel over 16 GPUs, experts spread over the same 16 GPUs. | | part-12 §12.4 |
| attention DP rank | One data-parallel copy of the attention layers inside a DEP worker; it holds its own sequences, its own full KV cache and a full copy of the attention weights, and joins every MoE layer in step with the other ranks. | DP rank (alone) | part-12 §12.3 |
| dispatch (MoE) | The all-to-all that sends each token's latent to the GPUs of its chosen experts. | scatter (for dispatch) | part-12 §12.4 |
| combine (MoE) | The all-to-all that returns each expert's output to the token's GPU, where the weighted sum forms. | gather (for combine) | part-12 §12.4 |
| EPLB | vLLM's Expert Parallel Load Balancer: it records expert load over a window of steps and rearranges experts, with redundant copies of hot experts, every set number of steps. | | part-12 §12.4 |
| context parallelism (CP) | Splitting one long sequence along its tokens across GPUs, so a long prefill ends sooner and a long KV cache fits; ring attention is one way to do it. | sequence parallelism (for CP) | part-12 §12.5 |

## The request path

| Term | Definition | Do not use | First defined in |
|---|---|---|---|
| PoP | Point of Presence: an edge site close to users. | edge location | part-00 §0.1 |
| edge proxy | The L7 proxy at a PoP that terminates TLS and runs edge checks. | CDN server (when talking about the proxy) | |
| region | A geographic group of data centers that the global traffic manager can choose. | site, data center (for a region) | part-01 §1.1 |
| availability zone | An isolated failure domain inside a region. | AZ (only after defined) | part-01 §1.1 |
| API gateway | The regional service that authenticates, enforces quotas and routes requests to application services. | front door | part-03 §3.1 |
| orchestrator | The service that builds the context, calls models and tools, runs the tool loop and streams the answer. | agent (for this service), controller | part-03 §3.6 |
| model router | The component that picks which model deployment serves a request. | router (alone) | part-03 §3.7 |
| inference gateway | The component that picks one endpoint of the chosen model (InferencePool + endpoint picker in this reference system). | load balancer (for this), router (alone) | part-00 §0.1 |
| endpoint picker (EPP) | The llm-d / Gateway API Inference Extension service that scores endpoints and returns one. |  | part-01 §1.9 |
| scheduler | The inference engine's loop that chooses the batch at each scheduler iteration. | dispatcher | part-00 §0.6 |
| admission control | The decision to admit, queue, redirect or reject a request before it uses GPU memory. | | part-09 §9.4 |
| InferencePool | The Gateway API Inference Extension resource (`inference.networking.k8s.io/v1`) that selects the model-server pods of one pool by label and names the endpoint picker that serves them. | server pool (for this resource) | part-09 §9.1 |
| flow control | The EPP feature that holds passes in per-flow queues inside priority bands before scheduling and dispatches them only while pool saturation is below the band's ceiling; it is off by default in llm-d. | admission queue | part-09 §9.4 |
| priority band | The set of all flows with one priority value; flow control serves a higher band first, and each band has its own saturation ceiling. | priority class | part-09 §9.4 |
| fairness ID | The identifier that groups passes into one flow for fair scheduling inside a band; in this reference system it is the tenant ID, set by the model router. | flow key | part-09 §9.4 |
| wait limit | The longest a pass waits in flow control (60 s by default); after it the EPP returns 429. | queue timeout, TTL (except the header name `rejected-ttl-expired`) | part-09 §9.4 |
| sticky endpoint | An endpoint whose prefix match for the request reaches the affinity filter's threshold (0.8); the filter keeps only sticky endpoints unless one is too far behind. | pinned endpoint | part-09 §9.3 |
| deficit round robin | A fair-queueing rule that gives each queue a token quantum per round and dispatches its head request while the cost fits the queue's counter. | weighted round robin (a different rule) | part-09 §9.6 |
| attained service | The service one tenant received recently, as input tokens plus 2 × output tokens decayed by half every 60 s; least attained service serves the tenant with the least. | used service | part-09 §9.6 |
| starvation | A request waiting without limit while the scheduler serves others. | | part-09 §9.6 |
| aging | Raising a waiting request's or tenant's priority with its waiting time, so starvation has a bound (the head-wait term in LAS). | | part-09 §9.6 |
| tool call | Structured model output that names a tool and its arguments. | function call (mention once as the API name) | part-06 §6.1 |
| tool result | The data the orchestrator appends to the context after running a tool. | tool output | part-06 §6.2 |
| tool executor | The trusted service that runs one tool. | | part-06 §6.1 |
| sandbox | The isolated environment (Firecracker microVM in this reference system) that runs model-requested code. | container (unless it is one) | part-08 §8.1 |
| chunk | One retrievable piece of a document, with metadata. | passage (except in citations), split (as a noun for a chunk) | part-04 §4.4 |
| embedding | A dense vector that represents a chunk or a query. | vector (alone, when the meaning is "embedding") | part-04 §4.4 |
| vector index | The ANN data structure over embeddings (HNSW in this reference system). | vector database (for the index itself) | part-05 §5.2 |
| reranker | A cross-encoder model that rescores query–chunk pairs. | | part-05 §5.3 |
| long-term memory | Extracted per-user records stored outside the conversation. | memory (alone) | part-05 §5.5 |
| SSE event | One `event:`/`data:` record in a Server-Sent Events stream. | message (for this) | |
| stream | The open HTTP response that carries SSE events to the client. | connection (for the HTTP stream) | |
| deadline | The absolute time by which a request or call must finish. | timeout (except for a single call's duration) | part-03 §3.1 |
| tool schema | A tool's name, description and JSON Schema for its arguments; the chat template writes it into the context as text. | tool definition, tool declaration (except the `M-frontier` message type `tool-declare`) | part-06 §6.1 |
| tool-call parser | The code in the model server that finds a call in the generated text and turns it into a tool name and arguments. | tool parser, call decoder | part-06 §6.1 |
| tool-call ID | The random ID (`chatcmpl-tool-` plus a UUID) that vLLM gives a parsed call and that ties the result to it; the model never writes it. | call ID, tool_call_id (except as an API field) | part-06 §6.1 |
| tool-status event | The SSE event the orchestrator sends the client instead of the call's tokens (RA-50). | tool event | part-06 §6.1 |
| tool loop | The orchestrator's loop of inference passes: after each pass it validates, authorizes and runs the calls, appends the results and calls the model again, up to the loop limit. | agent loop | part-06 §6.2 |
| execution record | The orchestrator's stored record of one tool run: execution ID, tool-call ID, conversation, pass number, tool, argument hash and state. | tool log, job record | part-06 §6.2 |
| human approval | A pause in the tool loop in which the user approves or denies one side-effecting call with its exact arguments. | confirmation | part-06 §6.3 |
| credential broker | The trusted service that issues short-lived tokens from vault leases to the tool executor, so no secret enters the context. | token broker, secret manager (for this service) | part-06 §6.4 |
| connection ID | The logical ID of a user's connected account (for example `conn_mail_01`); the only thing about a connector that the model sees. | connector ID, account ID | part-06 §6.4 |
| MCP | Model Context Protocol: an open protocol (JSON-RPC 2.0) for discovering and calling tools on other servers; optional in this reference system. | MCP protocol (after first use) | part-06 §6.5 |
| MCP client, MCP server | The MCP client is the connector inside the host (here inside the tool executor); the MCP server offers tools, resources and prompts. | client (alone, for the MCP client: `client` means the browser or app) | part-06 §6.5 |
| search provider | The external service that returns ranked results for the model's queries (Option A, RA-28). | search API (for the service), search engine (for the provider) | part-07 §7.2 |
| egress service | The separate service that fetches every web page: it resolves each name once, checks every address, connects only to a checked address and checks every redirect (RA-29). | fetcher (for the service), web fetcher, proxy (alone, for this service) | part-07 §7.4 |
| canonical URL | The one normalized form of a URL (https, lowercase host, no default port, fragment or tracking parameters, no trailing slash) that merges duplicate results. | normalized URL (for this) | part-07 §7.2 |
| source ID | The per-turn number the orchestrator gives each source (workspace chunk, snippet or page); the only thing the model writes to cite it. | citation ID, source number, chunk number (for sources in general) | part-07 §7.6 |
| provenance record | The stored record of one source: source ID, URL, kind, dates, HTTP metadata, text hash and passage offsets, written before the model sees the text. | citation record, metadata (alone) | part-07 §7.6 |
| citation validator | The orchestrator code that checks each citation marker in the stream (ID exists, text hash intact, claim supported, span found) before it sends a citation event. | citation checker, verifier (reserved for a verifier model) | part-07 §7.6 |
| URL frontier | The crawler's structure that decides which URL to fetch next (front queues for priority, back queues for politeness). | crawl queue, URL queue | part-07 §7.3 |
| inverted index | A map from each term to its sorted list of document IDs (postings). | term index, reverse index | part-07 §7.3 |
| prompt injection | Text in data (a retrieved chunk, a web page, a tool result) that tries to instruct the model; a jailbreak comes from the user's own text. | indirect injection (use only when contrasting with a jailbreak) | part-03 §3.8 |
| instruction hierarchy | Training that makes the model rank instructions by role: system, then developer, then user, then tool and web data. | priority training | part-07 §7.5 |
| deterministic enforcement | A control implemented as code that gives the same answer whatever the model writes (permissions, sandbox, allowlists, approval, sanitizer, CSP). | guardrail (for this) | part-07 §7.5 |
| RAG (retrieval-augmented generation) | Giving the model text that a retriever found at request time; in this reference system application code runs the search and puts the found chunks into the prompt. | retrieval (alone, for the whole pipeline) | part-05 §5.1 |
| hybrid retrieval | Running BM25 and vector search on the same query, fusing the two rankings with RRF and reranking the best candidates. | hybrid search (except in an engine's own name), mixed retrieval | part-05 §5.3 |
| BM25 | A lexical scoring function: it scores a chunk by the query terms it contains, with rare terms weighted up and long chunks weighted down. | keyword search (for BM25 alone), TF-IDF (a different formula) | part-05 §5.3 |
| HNSW | Hierarchical Navigable Small World: a layered neighbour graph over the vectors; a search descends the layers greedily and then explores layer 0 with a candidate list of size `ef`. | ANN graph, graph index (alone) | part-05 §5.2 |
| IVFFlat | An index that clusters the vectors with k-means and, per query, scans exactly only the `nprobe` closest lists. | IVF index (alone), cluster index | part-05 §5.2 |
| RRF (Reciprocal Rank Fusion) | A fusion rule that scores a chunk as the sum of 1 / (k + rank) over the rankings it appears in; this reference system uses k = 60. | rank fusion (alone), score fusion (a different method) | part-05 §5.3 |
| bi-encoder, cross-encoder | A bi-encoder embeds the query and each chunk separately, so chunk vectors exist before the query; a cross-encoder reads the query and one chunk together in one forward pass and scores the pair. | pair scorer, two-tower (for bi-encoder) | part-05 §5.3 |
| embedding cache | The store of chunk and query embeddings, keyed by tenant, model ID, dimensions and text hash, so an unchanged chunk never reaches the GPU again. | vector cache | part-05 §5.1 |
| ACL principal | A user or group name stored on a chunk as metadata, such as `group:finance` or `user:u_93`; a query keeps only chunks whose principals intersect the user's. | permission tag, role (for a principal), label (for an ACL principal) | part-05 §5.4 |
| post-filter | A permission filter that runs after the vector search and removes forbidden results, which can leave the top k short or empty; this reference system filters inside the index instead. | after-filter, late filter | part-05 §5.4 |
| memory record | One short fact or preference in plain text, with its embedding, stored per user in long-term memory. | memory item, memory entry, memory (alone) | part-05 §5.5 |
| compaction | Replacing the oldest turns of a conversation with a summary, so later prompts are shorter at the cost of one prefix-cache miss. | summarization (for the technique), truncation (a different technique) | part-05 §5.7 |
| recall@k, precision@k | The share of the relevant chunks that appear in the top k (recall) and the share of the top k that is relevant (precision). | hit rate (for recall@k) | part-05 §5.8 |
| MRR, nDCG | Mean reciprocal rank (the mean of 1 / rank of the first relevant chunk) and normalized discounted cumulative gain (graded relevance, discounted by log2 of the rank). |  | part-05 §5.8 |
| groundedness | Whether each claim in the answer is supported by a retrieved chunk it cites. | faithfulness, hallucination rate (for the inverse) | part-05 §5.8 |
| code executor | The stateless tool executor for `run_python`: it asks the sandbox manager for the session's VM and returns the normalized tool result. | code runner, sandbox service | part-08 §8.1 |
| sandbox manager | The daemon on each sandbox host that keeps the host's warm pool, binds VMs to sessions, starts each Firecracker process through the jailer, and is the only party that talks to the VM and to the object store. | VM manager, sandbox controller | part-08 §8.1 |
| microVM | One Firecracker process with its own guest kernel, two vCPUs, its own memory and a few virtual devices. | container (it is not one), VM (alone, after the first use) | part-08 §8.1 |
| guest agent | The small program inside the microVM that talks to the host only over vsock: it writes the uploaded files, starts the code as an unprivileged user, collects the output and reports the result. | in-VM agent, agent (alone) | part-08 §8.1 |
| warm pool | A set of already restored, idle microVMs that a new session can claim in milliseconds; each host keeps its own pool, refilled from a template snapshot. | VM pool, pre-warmed pool | part-08 §8.1 |
| sandbox session | The sandbox state of one conversation: live (the VM stays for 120 s after a call) or files only (scratch files and outputs in the object store, no VM). | sandbox instance, kernel (for Python state) | part-08 §8.1 |
| artifact | A file that the code writes to `/mnt/output` and the system returns; the model sees only its file ID, and the user fetches it through the file service with a presigned URL. | attachment (for a sandbox output), output file (except for the directory) | part-08 §8.1 |
| computer-use tool | A tool that lets the model drive a browser in its own microVM, one screenshot and one action per pass. | browser agent, GUI agent | part-08 §8.2 |
| accessibility tree | The browser's structured list of roles, names and states of the visible controls, sent as text so the model picks an element by ID instead of by coordinates. | DOM (a different structure), page structure (for the accessibility tree) | part-08 §8.2 |

## Network and edge

| Term | Definition | Do not use | First defined in |
|---|---|---|---|
| global traffic manager | The component that tells each PoP which region should receive new requests, weighing health, free capacity, latency, data residency and model availability. | traffic manager (alone), GTM (unless defined) | part-01 §1.1 |
| anycast | Many PoPs announce the same address prefix over BGP, and the internet routing system picks the PoP that receives the packets. |  | part-01 §1.1 |
| stub resolver | The DNS resolver on the user device; it forwards questions to a recursive resolver. |  | part-02 §2.3 |
| recursive resolver | The DNS resolver that answers for many users: it queries the root, TLD and authoritative servers and caches every answer. |  | part-02 §2.3 |
| authoritative server | The DNS server that holds the zone data for a domain and gives the final answer. |  | part-02 §2.3 |
| TTL | Time to live: the unsigned 32-bit count of seconds for which a DNS record may be cached (zero means do not cache). |  | part-02 §2.3 |
| BGP | Border Gateway Protocol (BGP-4): carries reachability information between autonomous systems and picks routes by policy and path length, not by latency or distance. |  | part-02 §2.4 |
| 0-RTT | Resumed TLS 1.3 or QUIC connection that sends application data in its first flight, encrypted with keys from a session ticket; replayable, so the reference system accepts it only for idempotent GET paths. | early data (mention once as the RFC 8470 name) | part-02 §2.5 |
| head-of-line blocking | A lost packet that holds back other data behind it: in TCP it blocks all later bytes, in QUIC only the streams whose data was in the lost packet. |  | part-02 §2.5 |
| WAF | Web application firewall: matches the request against rules for known attack patterns at the edge. |  | part-02 §2.6 |
| presigned URL | A time-limited URL that carries the signer permission for one operation on one object; anyone who holds it can use it, so it acts as a bearer token. | signed link | part-02 §2.2 |
| SSRF | Server-side request forgery: an attack in which a server fetches a URL that an attacker chose and so reaches a target the attacker cannot reach directly. | request forgery (alone) | part-07 §7.4 |
| DNS rebinding | An attack on the gap between check and use: a name answers a public address to the check and an internal address to the later connection. | DNS spoofing | part-07 §7.4 |
| Content Security Policy (CSP) | A response header that lists the sources from which the browser may load scripts, images and connections; `img-src` limits images. | CSP header (after first use) | part-07 §7.5 |
| Streamable HTTP | The MCP transport for a remote server: one HTTPS endpoint, each request one POST, answered with one JSON object or an SSE stream. | HTTP transport | part-06 §6.5 |
| vsock | A socket address family for host-to-guest communication; Firecracker provides it as a virtio device that maps to Unix sockets on the host, and it is the only channel between a microVM and the sandbox manager. | VM socket, serial channel | part-08 §8.1 |
| capability | One unit of root's privileges in Linux; the code in the sandbox runs as an unprivileged user with none of them and with `no_new_privs` set. | permission (for a capability), privilege flag | part-08 §8.1 |
| jailer | The Firecracker helper that starts each VMM in a chroot, in new mount, PID and network namespaces, under its own cgroup, and then drops it to an unprivileged uid and gid. | launcher, VMM wrapper | part-08 §8.1 |
| RDMA | Remote direct memory access: one machine's NIC reads or writes another machine's registered memory without the remote CPU. | | part-12 §12.7 |
| InfiniBand | A fabric built for RDMA; the DGX H100 compute NICs run NDR InfiniBand at 400 Gb/s. | | part-12 §12.7 |
| RoCE v2 | RDMA over Converged Ethernet: the RDMA transport inside Ethernet, IPv4 and UDP packets, with priority-based flow control to avoid packet loss. | | part-12 §12.7 |
| fat-tree | A topology in which leaf switches join spine switches with many links, so any half of the hosts reaches the other half at full rate. | | part-12 §12.7 |
| rail-optimized fabric | A GPU-cluster fabric in which NIC i of every node connects to the same leaf switch i, so a node's eight NICs form eight rails. | | part-12 §12.7 |
| backpressure | A slow receiver makes the sender stop instead of making buffers grow; a proxy passes it on by no longer reading from upstream. | back pressure, back-pressure | part-02 §2.5 |
| transport flow control | The window mechanism of HTTP/2 (`WINDOW_UPDATE`) and QUIC (`MAX_STREAM_DATA`, `MAX_DATA`) that limits how much a sender may send on one stream and on one connection. | flow control (for the transport windows) | part-15 §15.4 |
| stall limit | How long a stream buffer may stay full before the edge resets the stream: 30 s in this reference system (RA-74). | stall timeout | part-15 §15.4 |
| keep-alive comment | An SSE comment line (`: keep-alive`) that the server sends after 15 s without an event, so that proxies do not close a quiet stream (RA-47). | heartbeat | part-15 §15.2 |
| delta event | The `output_text.delta` SSE event: it carries the text of one token, about 49 bytes on the wire. | text event | part-15 §15.2 |
| merged output | One output object that vLLM builds from several outputs that wait in a request queue when the reader is slow; the next chunk then carries several tokens. | coalesced output, coalescing | part-15 §15.2 |
| Nagle's algorithm | The TCP rule that holds small writes while earlier data is not yet acknowledged; the socket option `TCP_NODELAY` turns it off (RA-47). |  | part-15 §15.2 |
| replay buffer | A per-stream store of recent events that would let a server resend events after a reconnect; this reference system has none (RA-48). | resume buffer | part-15 §15.6 |
| fetch by ID | The client step after a broken stream: it fetches the stored assistant message by its ID and acts on its status (RA-48). | stream resume (for this) | part-15 §15.6 |

## Application backend and data

| Term | Definition | Do not use | First defined in |
|---|---|---|---|
| concurrency | Many tasks in progress during the same period. | parallelism (a different concept) | part-00 §0.6 |
| parallelism | Many tasks executing at the same instant on separate hardware. | concurrency (a different concept) | part-00 §0.6 |
| event loop | The loop that takes one task or ready event at a time and runs it to the end; a browser page and a CPU service both use one. |  | part-02 §2.1 |
| connection pool | Open connections to one upstream that are lent to operations, because a new connection per operation is expensive. |  | part-03 §3.3 |
| MVCC | Multi-version concurrency control: each PostgreSQL statement sees a snapshot, and an update writes a new row version instead of changing the old one. |  | part-03 §3.4 |
| WAL | Write-ahead log: PostgreSQL changes a data file only after the log records that describe the change reach durable storage. |  | part-03 §3.4 |
| quorum | A majority of replicas; a change commits when a quorum has it, so 3 replicas is the smallest group with a quorum. |  | part-01 §1.1 |
| hot standby | A PostgreSQL standby that answers read-only queries. | read replica (mention once as the other name) | part-03 §3.4 |
| home region | The region that holds a user conversation data with synchronous replication inside it. |  | part-03 §3.4 |
| task queue | A queue for work that need not finish before the answer; separate consumer processes run it later. |  | part-03 §3.3 |
| circuit breaker | A per-dependency state machine (closed, open, half-open) that fails calls at once while a dependency is failing (RA-13). | breaker (alone), fuse | part-07 §7.2 |
| full jitter | A backoff rule in which a client sleeps a random time between 0 and the capped exponential backoff: random(0, min(cap, base × 2^attempt)). | | part-14 §14.7 |
| freshness lifetime | How long an HTTP response (or a cache entry) counts as fresh: from `s-maxage`, `max-age`, `Expires` minus `Date` or a heuristic; the query class caps it. | expiry (for a cache entry), shelf life | part-07 §7.8 |
| negative cache | A short-lived cache entry for an empty result, so a burst of one failing query does not hit the provider again. | empty cache | part-07 §7.7 |
| sharded counter | A counter stored as several rows (32 per tenant and period) so that no single row limits the update rate; a check sums all rows. | split counter | part-16 §16.1 |
| primary key | The columns that identify one row and, in a distributed SQL store, decide which range stores it. |  | part-16 §16.1 |
| range | A contiguous slice of the sorted key space of the conversation store, with its own replica group (3 voting replicas). | tablet | part-16 §16.1 |
| leaseholder | The replica of a range that answers reads that need the latest write and proposes writes. |  | part-03 §3.4 |
| hotspot | One range that receives all new rows, for example when keys grow with time. | hot spot (for a range) | part-16 §16.1 |
| interleaving | A layout that stores child rows (messages) under their parent row (conversation) in the same range. |  | part-16 §16.1 |
| event log | The Kafka-class replicated log that receives usage, audit, analytics and `turn_completed` events and never waits for a reader (RA-20); the title of section 16.2 calls it the event stream. | event stream (for the log), message bus | part-16 §16.2 |
| topic | A named, durable log of events inside the event log; this reference system has `usage`, `audit`, `analytics` and `turn_completed`. |  | part-16 §16.2 |
| partition | One ordered part of a topic; events with the same key go to the same partition, and order holds only inside one partition. |  | part-16 §16.2 |
| partition key | The key of an event; it decides the partition, so events with one key stay in order and the key also spreads the load. | routing key | part-16 §16.2 |
| offset | The position of an event in its partition; a consumer position is the offset of the next event to read. |  | part-16 §16.2 |
| consumer group | A set of consumer processes that share the work of one topic: each partition goes to exactly one consumer of the group at a time, and each group keeps its own offsets. |  | part-16 §16.2 |
| consumer lag | The distance between the last offset written to a partition and the offset that a consumer has reached. | consumer backlog | part-16 §16.2 |
| in-sync replicas (ISR) | The replicas of a partition that have all committed events; an event commits when all of them have it. |  | part-16 §16.2 |
| idempotent consumer | A consumer that gives the same result when it sees an event twice: it drops event IDs it has seen in the last 24 h and stores its offsets with its results. | deduplicating consumer | part-16 §16.2 |
| usage event | The record of one inference pass or one tool call that the model router or orchestrator appends to the `usage` topic: counts, never prices, with an ID that hashes request ID, pass and kind. | usage record, metering event | part-16 §16.4 |
| ledger | The billing table of hourly usage sums per tenant, hour, model, kind and partition, written by the aggregator; invoices are built from its final rows. | usage ledger, billing table | part-16 §16.4 |
| reconciliation | The nightly comparison of engine usage, TPM charges and the ledger per tenant and day; each difference becomes an adjustment row. |  | part-16 §16.4 |
| LRU | Least recently used: under memory pressure a cache evicts the entry that has been idle longest. |  | part-16 §16.3 |
| LFU | Least frequently used: under memory pressure a cache evicts the entry with the fewest recent hits. |  | part-16 §16.3 |
| content-addressed key | A cache key that contains a hash of the content or a version, so a change creates a new key and no old entry needs invalidation. |  | part-16 §16.3 |
| memory pressure | The state of a cache that has reached its memory limit and must evict an entry before it can write. |  | part-16 §16.3 |
| semantic response cache | A cache that returns an earlier answer for a question that is only similar, found by embedding similarity; safe only for public, non-personal, time-insensitive questions. |  | part-16 §16.3 |

## Deployment, startup and control plane

| Term | Definition | Do not use | First defined in |
|---|---|---|---|
| cold start | The work that turns an idle GPU into a replica that can accept requests (eight steps for vLLM on Kubernetes). | boot (for this) | part-01 §1.2 |
| warm replica | A set of running processes that already hold weights in HBM, compiled kernels, recorded CUDA Graphs, an allocated KV block pool and open communication channels. |  | part-01 §1.3 |
| warm floor | The minimum number of ready replicas that every pool keeps, so a pool never drops to zero. | minimum replicas (in prose) | part-01 §1.2 |
| N+1 reserve | One extra unit of a failure domain per region (here one rack) that covers the loss of one such unit. | | part-14 §14.6 |
| follow-the-sun | Placement that sends work that can move to the regions where it is night. | | part-14 §14.8 |
| background work | Preemptible work that fills idle capacity (batch and flex-tier requests, embedding backfills, offline evaluations) and runs in the `batch` priority band. | offline work (for this) | part-14 §14.8 |
| CUDA Graph | A recorded sequence of kernel launches that is replayed with one launch. |  | part-01 §1.3 |
| control plane | The part that decides what runs where and how traffic flows (autoscaler, service discovery, certificates, rollout); no user byte passes through it. |  | part-01 §1.8 |
| data plane | The part that carries the work itself: user traffic, model requests, tool calls, token streaming and KV transfer. |  | part-01 §1.8 |
| service discovery | The mechanism that tells a client which network addresses currently serve a name. |  | part-01 §1.9 |
| EndpointSlice | The Kubernetes object that lists the addresses of a Service pods, with the conditions ready, serving and terminating per endpoint. |  | part-01 §1.9 |
| xDS | The APIs through which an Envoy proxy receives clusters, endpoints, listeners and routes from a management server over a long-lived stream. |  | part-01 §1.9 |
| readiness probe | The Kubernetes check that decides whether a pod joins the set of endpoints. |  | part-01 §1.9 |
| canary | A new model version or deployment that first receives a small weighted share of traffic. |  | part-01 §1.4 |

## Reliability and failure handling

| Term | Definition | Do not use | First defined in |
|---|---|---|---|
| timeout | The longest time that one call may take before its caller gives up; a duration, where a deadline is a clock time. | time limit (for a call) | part-17 §17.1 |
| timeout budget | The split of one deadline into the timeouts of the calls inside it. | time budget (for this) | part-17 §17.1 |
| child deadline | The time that one inner call gets: the smaller of its cap and the time left in the parent minus the reserve the parent keeps (RA-11). | child time (for this), child timeout (for this) | part-17 §17.1 |
| cap (timeout) | The largest timeout that a call ever gets, however much time its parent has left. |  | part-17 §17.1 |
| reserve (timeout) | The time that a parent keeps for itself after a child call returns, for example to write the partial answer. |  | part-17 §17.1 |
| cancel | The signal that stops the work of one request: each hop forwards it down to the scheduler, which frees the KV blocks (RA-49); vLLM names the engine step `abort`. | kill (for a request) | part-17 §17.2 |
| RTO | Recovery time objective: the longest time that a service may stay unavailable after a failure. | recovery time (for RTO) | part-17 §17.4 |
| RPO | Recovery point objective: the largest window of committed data, measured in time, that a failure may lose. | data-loss window (for RPO) | part-17 §17.4 |
| failover | Moving the traffic of a failed region to another region through global traffic management (RA-06). | fail-over | part-02 §2.3 |
| fallback | The model router's rule for a deployment that is saturated or failing: the same model in the next-nearest healthy region first, a smaller model last (RA-57). | failover (for the router's rule) | part-03 §3.7 |
| failure domain | The unit that one fault takes out as a whole: for `M-frontier` one rack, which is one NVLink domain. | fault domain | part-01 §1.1 |
| blast radius | The number of GPUs that one fault takes out: 8 for a TP=8 replica or a TEP8 worker, 16 for a DEP16 worker, 72 for one NVLink domain. |  | part-17 §17.11 |
| warm spare | An idle GPU with its weights loaded that replaces a failed GPU in a worker (8 per `M-frontier` rack, RA-55). | hot spare | part-01 §1.2 |
| Xid | The code of a fault message that NVIDIA's driver writes to the kernel log; the catalog gives one immediate action per code. |  | part-17 §17.11 |
| correctable error | A memory error that ECC fixed; the data is right. |  | part-17 §17.11 |
| uncorrectable error | A memory error that ECC cannot fix; the data is corrupt, and containment limits the harm to the applications that hit it. |  | part-17 §17.11 |
| retry budget | A counter per client or dependency that allows a retry only while retries stay below a share (10%) of recent requests, so a failing dependency sees about 1.1 times its normal load (RA-12). | retry quota, retry limit | part-14 §14.7 |
| full jitter | A retry delay drawn at random between 0 and the capped exponential value min(cap, base x 2^attempt). | equal jitter (for full jitter) | part-14 §14.7 |
| circuit breaker states | Closed (calls pass and failures are counted in a window), open (calls fail at once while a timer runs) and half-open (a few trial calls pass; enough successes close it, one failure opens it again). | tripped (for open) | part-07 §7.2 |
| outlier detection | Envoy's passive health check: it ejects an endpoint after consecutive errors for a base ejection time, up to a maximum share of the hosts. | passive ejection | part-01 §1.9 |
| load shedding | Rejecting the excess requests at once with 429, so that a bounded queue keeps the other requests on time. |  | part-17 §17.7 |
| traffic weight | A proportional share of requests that goes to one of several backends; weights 90 and 10 send 90% and 10%. | traffic percentage (for a weight) | part-17 §17.10 |
| draining | Removing a worker from new routing, letting its active requests finish within the drain bound, then stopping it. | graceful shutdown (for draining) | part-17 §17.10 |
| drain bound | The longest time that a draining engine waits for in-flight requests before it aborts the rest (vLLM `--shutdown-timeout`; 300 s in this reference system, hypothetical). | drain timeout (for the bound) | part-17 §17.10 |

## Security, privacy and safety

| Term | Definition | Do not use | First defined in |
|---|---|---|---|
| threat model | A list of the assets, the actors, the entry points on a path and the control at each one. |  | part-18 §18.1 |
| asset (security) | Any data or artifact whose loss, change or disclosure harms a user, a tenant or the operator. |  | part-18 §18.1 |
| trust boundary | A point where data passes between components with different levels of trust; every control of Part 18 sits on one. |  | part-18 §18.1 |
| least privilege | Each identity holds only the rights that its task needs. | minimal access (for this) | part-18 §18.1 |
| mTLS | Mutual TLS: both sides present a certificate, so each internal call is authenticated in both directions (RA-16). |  | part-03 §3.1 |
| SPIFFE ID | A URI of the form `spiffe://<trust domain>/<workload identifier>` that names one workload; the workload API hands the workload a short-lived X.509 certificate for it. | workload name (for the ID) | part-01 §1.8 |
| KMS | Key management service: the service that holds the top-level keys and wraps and unwraps data keys; its keys never leave its hardware security modules unencrypted. | key server | part-18 §18.1 |
| envelope encryption | A data key encrypts the data and another key encrypts the data key; the top key stays in the KMS. | double encryption (for this) | part-18 §18.1 |
| data key | The key that encrypts one object or storage range; it is stored wrapped (encrypted) under a key-encryption key. |  | part-18 §18.1 |
| key-encryption key (KEK) | The key that wraps data keys: one per tenant per region (RA-76); deleting it makes that tenant's data unreadable in every copy. | master key (for the KEK) | part-18 §18.1 |
| exfiltration | Data that leaves through a channel the user did not intend; every output of the model is a channel. | data leak (for exfiltration) | part-07 §7.5 |
| cache salt | A per-tenant string mixed into the hash of the first KV block, so two tenants never share prefix-cache blocks (RA-40). | cache key salt | part-13 §13.5 |
| hold-back | Text kept on the server until a condition holds (a tool call completes, a citation resolves, an output check passes) before the client sees it. | holdback | part-06 §6.1 |
| stream-first | An output-check design that sends tokens at once and stops or retracts them later; the exposed tokens are W plus the tokens that arrive during one check. |  | part-18 §18.3 |
| output-check window | The group of new tokens (32 in this reference system, with a 16-token overlap) that the classifier checks together before the hold-back releases them. | chunk (for a check window) | part-18 §18.3 |
| fail closed | On an error of a safety or policy control, refuse or stop instead of letting the request pass. | fail secure, fail-closed | part-18 §18.3 |
| log redaction | Removing, masking or replacing sensitive values in a log record before it is stored: schema allowlist, identifier transforms, pattern scrub and escaping (RA-77). | scrubbing (for the whole step) | part-18 §18.4 |
| pseudonym | A keyed hash (HMAC-SHA256 with a KMS key, truncated) that replaces the user ID in a log record. | alias (for the hash) | part-18 §18.4 |
| baggage | The W3C list of key-value pairs that travels with a trace on every outgoing call; it must not cross trust boundaries (RA-52). |  | part-03 §3.1 |
| data residency | The rule that limits the jurisdiction in which content may be stored and processed. | data sovereignty (for residency) | part-18 §18.5 |
| residency-bound tenant | A tenant whose contract or law ties its content to one jurisdiction; its requests go only to regions inside it, and it fails closed when none is available (RA-78). | pinned tenant, bound tenant | part-18 §18.5 |

## Hardware

| Term | Definition | Do not use | First defined in |
|---|---|---|---|
| SM | Streaming multiprocessor: the GPU's unit that runs thread blocks. | core (for an SM) | part-00 §0.6 |
| Tensor Core | The SM unit that performs small matrix multiply-accumulate operations. | | part-11 §11.1 |
| FLOPs | A count of floating-point operations. |  | part-00 §0.3 |
| FLOP/s | Floating-point operations per second (a rate). Written "TFLOP/s" or "PFLOP/s". | FLOPS (ambiguous) | part-00 §0.3 |
| arithmetic intensity | FLOPs performed per byte moved from memory. | | part-11 §11.4 |
| roofline | The bound min(peak FLOP/s, arithmetic intensity × memory bandwidth). | | part-11 §11.4 |
| ridge point | The arithmetic intensity where the two roofline bounds meet. | | part-11 §11.4 |
| model checkpoint | The set of weight files of a trained model, as stored and distributed (sharded safetensors files plus config). Written "checkpoint" alone only in Part 01 contexts about weights. | | part-01 §1.4 |
| KDA checkpoint | A saved copy of a sequence's KDA recurrent state at a prefix boundary, used for prefix-cache reuse in hybrid models. | checkpoint (alone, in KV-cache contexts) | part-13 §13.3 |
| block scaling | Giving each small block of consecutive values its own scale, so a 4-bit code covers whatever range the block needs. |  | part-01 §1.5 |
| MXFP4 | OCP block format: FP4 E2M1 elements, blocks of 32, one E8M0 (power-of-two) scale per block, 4.25 bits per value. |  | part-01 §1.5 |
| NVFP4 | NVIDIA block format: FP4 E2M1 elements, blocks of 16, an FP8 E4M3 scale per block plus one FP32 scale per tensor, 4.5 bits per value. |  | part-01 §1.5 |

| HBM bandwidth | The rate at which the GPU moves bytes between HBM and the chip (3.35 TB/s on an H100 SXM); it bounds every memory-bound kernel. | memory speed (for this) | part-11 §11.1 |
| thread block | The group of threads that one kernel launch places on one SM; a grid of thread blocks is one launch. | CTA (in prose) | part-11 §11.1 |
| warp | A group of 32 threads of one thread block that issue the same instruction at the same time; an H100 SM holds at most 64 resident warps. | wavefront (for a warp) | part-11 §11.1 |
| Tensor Memory Accelerator (TMA) | The Hopper unit that copies a whole tile between HBM and shared memory from one thread's request. | | part-11 §11.1 |
| GEMM | General matrix multiplication: the operation Y = X × W that the linear layers run, with X of shape [M, K] and W of shape [K, N]. | matmul (in prose) | part-11 §11.2 |
| wave quantization | The effect that the GPU runs output tiles in whole waves of SMs, so a partial wave costs as much as a full one. | tile quantization (for waves) | part-11 §11.2 |
| MFU | Model FLOPs utilization: the achieved FLOP/s of a step divided by the peak FLOP/s; unlike utilization, it measures the work done. | FLOPs efficiency (for MFU) | part-11 §11.12 |
| kernel fusion | Merging consecutive kernels into one, so the intermediate result stays in registers or shared memory instead of HBM and one launch replaces several. | | part-11 §11.11 |
| full graph, piecewise graph | The two CUDA Graph kinds of vLLM's default mode: a full graph replays the whole forward pass of a decode-only step; piecewise graphs cover the pieces between attention calls in prefill and mixed steps. | | part-11 §11.11 |
| NVSwitch | The switch chip that joins every GPU of an NVLink domain to every other GPU at full NVLink rate (4 chips in a DGX H100 node). | | part-12 §12.6 |
| peer-to-peer transfer | A move of bytes from one GPU's HBM to another's without a copy through CPU memory. | GPU-to-GPU copy (for this) | part-12 §12.6 |
| NUMA node | One CPU socket with its local memory; a core that touches memory of the other socket crosses the slower inter-socket link. | | part-12 §12.6 |
| hardware pool | A set of identical nodes (the same GPU, network and software image); a replica never spans two pools. | GPU pool (for the hardware set) | part-12 §12.9 |

## Other branches: images, voice and long-running tasks

| Term | Definition | Do not use | First defined in |
|---|---|---|---|
| image deployment | The separate pool of GPU workers, with its own queue, autoscaler and model files, that runs image-generation tool calls. | image cluster | part-20 §20.1 |
| denoising step | One pass of an image model's network over all positions of the image latent; a fixed number of steps turns noise into an image. | iteration (for a denoising step) | part-20 §20.1 |
| image latent | The small grid of vectors on which a diffusion or flow-matching model works instead of pixels; the VAE decodes it to pixels. | | part-20 §20.1 |
| VAE | Variational autoencoder: the network that maps pixels to latent positions and decodes the final latent back to pixels. | | part-20 §20.1 |
| diffusion model | An image model that starts from random noise and removes a little noise per step. | | part-20 §20.1 |
| flow matching | An image-model method that learns a velocity field from noise to data; rectified flow is the variant with a straight path, which needs fewer steps. | | part-20 §20.1 |
| autoregressive image model | An image model that turns an image into discrete codes and predicts them one by one, as a text model predicts tokens. | | part-20 §20.1 |
| classifier-free guidance (CFG) | A method that runs a second pass per step with a negative prompt and mixes the two outputs; a guidance-distilled model needs only one pass. | | part-20 §20.1 |
| prompt blocking | The check by classifiers on the image prompt that can stop the tool call before any GPU work. | | part-20 §20.1 |
| output blocking | The check by classifiers on the decoded image; no pixel leaves the worker before it passes. | | part-20 §20.1 |
| C2PA manifest | A signed record of assertions about a file, hard-bound to the file's bytes by a hash, that the image worker attaches to every image. | | part-20 §20.1 |
| artifact event | The SSE event that carries the signed URL of a file to the client, so file bytes never travel through the token stream (RA-63, RA-74). | | part-20 §20.1 |
| ASR | Automatic speech recognition: the model that turns the user's audio into text. | speech-to-text (after the first mention) | part-20 §20.2 |
| TTS | Text-to-speech: the model that turns the answer text into audio, starting on the first phrase. | | part-20 §20.2 |
| VAD | Voice activity detection: a small model that marks each audio chunk as speech or silence; server VAD ends a turn after silence, semantic VAD when the sentence is complete. | | part-20 §20.2 |
| cascaded voice pipeline | The voice design in which streaming ASR, the text model and streaming TTS run one after another. | | part-20 §20.2 |
| native speech model | One model that reads audio tokens and writes audio tokens, with text alongside. | | part-20 §20.2 |
| voice gateway | The service in the serving region that ends the browser's WebRTC media and talks to the speech deployments over gRPC streams. | | part-20 §20.2 |
| jitter buffer | A buffer that holds a few packets and releases them at the pace of the source, to hide uneven packet arrival. | | part-20 §20.2 |
| playout buffer | The browser's buffer that smooths downlink audio packets before they play. | | part-20 §20.2 |
| end of turn | The moment the system decides that the user has finished speaking; the silence before it is the largest stage of a spoken turn. | | part-20 §20.2 |
| barge-in | The user starts to speak while the assistant's audio plays; the system stops the answer and the GPU work and keeps only what the user heard. | interruption (for barge-in) | part-20 §20.2 |
| research task | A long background task of dozens of inference passes whose state lives in a durable task record, not in one request or stream. | agent run (for this), job (for a task) | part-20 §20.3 |
| task record | The stored row of one research task: identity, status, plan, cursor, notes, sources, artifacts, budget and lease. | | part-20 §20.3 |
| task worker | An orchestrator process (not a GPU group) that takes task IDs from a queue and runs the passes of a research task. | worker (alone, for this process) | part-20 §20.3 |
| task lease | The right of one task worker to run a task until an expiry time; when it expires, another worker takes the task. | | part-20 §20.3 |
| task checkpoint | The state that a task worker writes after every pass (cursor, notes, sources, budget used) so that another worker can resume. | checkpoint (alone, for task state) | part-20 §20.3 |
| task budget | The limits that bound a research task: a spending cap, a wall-clock limit and the tenant's quotas; it replaces the chat loop limit. | | part-20 §20.3 |
| progress stream | The SSE stream of progress events from the event log that a client opens on a task ID; closing it does not stop the task. | | part-20 §20.3 |
| task compaction | One extra pass in which the model summarizes the old tool results of a task and the summary replaces them in the working context. | | part-20 §20.3 |
| task graph | The steps of a research task as a graph: independent sub-questions can run in parallel, and merging the findings depends on all of them. | | part-20 §20.3 |

## Observability, SLOs and autoscaling

| Term | Definition | Do not use | First defined in |
|---|---|---|---|
| span | One timed operation in a trace: a name, a start and end time, attributes, events and a status; its span ID is unique. |  | part-03 §3.1 |
| trace ID | The ID that all spans of one request share; it travels between services in the `traceparent` header. |  | part-03 §3.1 |
| traceparent | The W3C Trace Context header `{version}-{trace-id}-{parent-id}-{trace-flags}` that carries the trace ID and the caller's span ID to the next service. |  | part-03 §3.1 |
| trace | The set of spans that share one trace ID; the parent links make it a tree with one root. |  | part-19 §19.1 |
| parent span ID | The span ID of the span that caused this span. |  | part-19 §19.1 |
| span event | A timestamped point inside one span. |  | part-19 §19.1 |
| metric | A number that a process updates and exports over time. |  | part-19 §19.1 |
| log | One timestamped record of one event, written by one process. |  | part-19 §19.1 |
| gauge | A metric value that rises and falls. |  | part-19 §19.1 |
| histogram | A metric that counts observations per bucket (`_bucket{le="…"}`) and keeps `_sum` and `_count`. |  | part-19 §19.1 |
| exemplar | A reference from a metric sample to the trace of one real request; it sits on a histogram bucket or a counter sample. |  | part-19 §19.1 |
| tail-based sampling | Sampling in which the collector decides after the trace ends, so it can keep every trace with an error or an SLO miss. | tail sampling (alone) | part-19 §19.1 |
| SLI | Service level indicator: a measured ratio of good events to valid events. |  | part-19 §19.3 |
| error budget | The share of events that may fail: 1 − SLO. |  | part-19 §19.3 |
| burn rate | The error fraction divided by the budget fraction; a burn rate of 1 spends exactly the whole budget in the SLO window. |  | part-19 §19.3 |
| Horizontal Pod Autoscaler (HPA) | The Kubernetes controller that sets the replica count of a workload from a metric: desiredReplicas = ceil[current × (current metric ÷ desired metric)]. | pod autoscaler (alone) | part-19 §19.4 |
| KEDA | A Kubernetes autoscaler that reads external metrics, for example a Prometheus query, and creates and owns an HPA for each `ScaledObject`. |  | part-19 §19.4 |
| online eval | A score of a sample of production answers after they reach the user; it never delays the answer (RA-80). |  | part-19 §19.5 |
| grader model | A model that rates a sample of answers against a written rubric. | judge model (for this) | part-19 §19.5 |
| shadow traffic | A copy of real requests sent to a new version whose answers are discarded. | mirror traffic (alone) | part-19 §19.5 |
| saturation (of a resource) | The amount of work that waits for a resource; it differs from pool saturation (part 9.4), which is one number for a whole pool. |  | part-19 §19.6 |
| profiler | A tool that records what happens inside one process, for example NVIDIA Nsight Systems (CPU threads, CUDA calls, kernels, NVTX ranges, NCCL). |  | part-19 §19.6 |

## Timeline, state and flow (Part 21)

| Term | Definition | Do not use | First defined in |
|---|---|---|---|
| super trace | The one timeline that joins the times of the earlier parts for your request, from the key press (R0) to the last pixel (R25), with the data form at each boundary. | master trace (for this) | part-21 §21.1 |
| critical path | The chain of steps whose sum sets the end-to-end latency; a step is on it only when the next step needs its output. | hot path (for this) | part-21 §21.4 |
| side work | Work that records, counts or improves later turns. Nothing in the answer reads it, so it never blocks a token. It differs from background work (part 14.8), which fills idle capacity. | off-path work (for this) | part-21 §21.4 |
| control flow | The set of decisions that pick where work runs and when it stops. It differs from the control plane (part 1.8), which is the autoscaler, service discovery and rollout side. | | part-21 §21.3 |
| data flow | The bytes that the decisions of the control flow move. | | part-21 §21.3 |
| stateless component | A component that keeps nothing between requests that another replica would need, so any replica can take the next request. | | part-21 §21.5 |
| stateful component | A component that keeps data that only it holds, so the next request must find it. | | part-21 §21.5 |
| affinity | A rule that sends a request to the component that holds its state: the home region, the existing KV, the active stream or the warm session. | stickiness (for this) | part-21 §21.5 |
| soft state | State that the system can compute again from durable state; losing it costs time, never correctness (prefix-cache blocks, visual tokens, search results). | cache state (for this) | part-21 §21.5 |
| durable state | State that lives in a replicated store with an owner who deletes it on request or by a retention rule (messages, uploads, ledger rows). | persistent state (for this) | part-21 §21.7 |
| ephemeral state | State that the system deletes at once or on a timer after the response (the request object, active KV blocks, the sandbox VM, stream buffers). | transient state (for this) | part-21 §21.8 |

## The miniature (Part 24)

| Term | Definition | Do not use | First defined in |
|---|---|---|---|
| miniature | The runnable small version of the reference system in `sim/mini/`: real asyncio code around a fake model, a toy tokenizer and a virtual clock. | mini-system (for this) | part-24 intro |
| stand-in | In the miniature, a small piece of code that keeps the interface of a real component and replaces its inside. | mock (for this), stub (for this) | part-24 §24.1 |
| fake model | A deterministic script that takes the place of the language model in the miniature: at each inference pass it reads the context and returns fixed text. It has no weights. | mock model (for this), dummy model (for this) | part-24 intro |
| toy tokenizer | The miniature's tokenizer: one token per word, number or punctuation mark, with the token ID taken from a CRC-32 checksum of the token text. | | part-24 intro |
| virtual clock | The event loop clock of the miniature: its `time()` returns a counter and its `select(timeout)` adds the timeout to the counter at once, so a wait costs no wall time. | simulated clock (for this), fake clock (for this) | part-24 intro |
| reduced request | The reference request at a smaller size, used in the miniature so that every table fits on a page (1 earlier turn, 4 chunks, a 448 x 336 photo). | small request (for this) | part-24 §24.2 |
| evolution stage | One of the five steps of growth of a chat system: one machine, several GPU replicas, Kubernetes, multi-region, frontier-scale distributed inference. Each stage keeps the interfaces of the one before. | | part-24 §24.1 |
| waiting queue | The engine's queue of requests that wait for admission, in arrival order. | | part-11 §11.6 |
| running list | The engine's list of admitted sequences, in admission order. | | part-24 §24.3 |
| prefill queue | In the engine's scheduler, the waiting queue plus each running sequence whose prompt is not complete. It differs from the prefill work queued at an endpoint (part 9.3). | | part-24 §24.3 |
| decode queue | In the engine's scheduler, each running sequence whose prompt is complete. | | part-24 §24.3 |

## The workbook, the atlas and the synthesis (Parts 22, 23 and 25)

| Term | Definition | Do not use | First defined in |
|---|---|---|---|
| weight memory | The parameter count times the bytes per parameter; the number format decides it. | | part-01 §1.6 |
| active parameters | The parameters that one token multiplies in a forward pass. For a mixture-of-experts model: the non-expert weights plus the routed experts that the router picks. They set the FLOPs per token, while the total parameters set the memory. | activated parameters (for this) | part-01 §1.6 |
| resident memory | The weight bytes that the GPUs of one worker hold after the parallel layout copies some weights. On a DEP16 worker every rank also keeps the non-expert weights, so the total exceeds the checkpoint size. | | part-22 §22.1 |
| latency budget | The split of one end-to-end time, or of one TTFT, into hops. Each hop has a p50 and a tail; the sum of the p50s is the time of the trace. | | part-20 §20.2 |
| tail assumption | The hypothetical p95/p50 ratio (log-normal) that the latency budget gives to one kind of hop, because the reference system has no measured tail. | | part-22 §22.5 |
| cost center | One line of the cost model: a resource (GPUs, search provider, sandbox, storage, network, telemetry) with its quantity per request, unit price and cost per million requests. | cost line (for this) | part-22 §22.6 |
| serving unit | In the overload simulation, a group of 8 GPUs that serves traffic as one unit. The 8 spare GPUs of a rack join as one serving unit (RA-86). | worker (for this group), unit (alone) | part-22 §22.8 |
| counterfactual | A number that the `counterfactuals` simulation computes for the reference system with one layer removed (no prefix cache, no KV cache). It uses the same models as part 14.4. | | part-25 intro |
| sizing question | One of the eleven questions whose answers fix a deployment (lengths, rate, objectives, attention design, number formats, GPU, parallelism, hit rate, tool share, search share, regions). | | part-25 §25.2 |
| request router | In the GPU path of 23.3, the first step: the inference gateway plus the endpoint picker. It differs from the model router. | router (alone) | part-23 §23.3 |
| streamer | In the GPU path of 23.3, the last step: the API server's detokenizer and output parsers, which turn token IDs into text deltas and hold back tool-call text. | | part-23 §23.3 |
| reasoning-status event | The SSE event that tells the client the model is reasoning, in place of the raw reasoning (RA-72). For the reference request it reaches the screen at 0.97 s, the moment that ends the TTFT clock. | reasoning summary (for this event) | part-10 §10.13 |
