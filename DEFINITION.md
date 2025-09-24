# RAPTOR: Risk‑Adaptive Pointer‑Typed Oscillatory Recurrent Network

A single, integrated, near‑linear system that streams, remembers, retrieves, reasons, and self‑checks—with hard bounds, calibration, and abort paths. It replaces dense attention with a keyed surrogate + event‑driven pointers, replaces KV caches with transactional memory, replaces linear CoT with typed, budgeted graph‑of‑thought, and puts a FLOP‑aware controller over everything.

Why it usurps Transformers
- Sequence scaling: O(n) compute and memory; no n^2 attention; bounded KV‑like state; streaming updates O(1) per token.
- Long context: content‑aware events with false‑negative control; ECC‑guarded pointers; episodic DAG memory; type‑aware addressing; rare global transforms.
- Recurrence/memory: stable SSM state + global keyed memory + transactional WORM tiers with typed ECC; real persistent, content‑addressable memory.
- Reasoning/verification: typed constraints, unit algebra, ECC, submodular GoT‑lite, certified abstention; not just soft traces.
- Position/time: rationally‑independent oscillators with anchor resets and phase‑augmented addressing; extrapolates beyond training lengths.
- Compute thrift: constrained MDP budgeter (AEGIS) with safety floors; ACT halting; event caps; background compaction; rare fallback transforms.
- Stability: passivity‑constrained SSM + reversible, contractive residual stacks; queueing‑stable event pipeline.

High‑level dataflow (per token)
1) Flow core updates SSM state; ORBIT phases update/smooth; HAWK reads/updates global keys; ACT halting decides extra micro‑steps if needed.
2) CHED hazard checks for an event; if triggered and admitted under budget, propose k anchor candidates via multi‑hash; filter with ECC parity, type/phase checks, hyperbolic 2‑hop pruning; optionally read from memory tiers/DAG along vetted edges.
3) If high‑risk span: AEGIS may allow a tiny GoT‑lite expansion with typed constraints and submodular greedy; verifiers run a cheap cascade; if residual risk remains above threshold, abstain or allocate one refinement iteration.
4) Memory writes: transactional reserve‑verify‑commit into SPINE tiers and HELIX episode DAG; compaction runs at fixed cadence in the background.
5) AEGIS adjusts per‑span budgets using dual variables; safety floors ensure integrity/verification minimums.

Architecture definition

1) Flow core: Stable SSM + reversible contractive residuals (SANDGLASS)
- State per head h: x_t^h ∈ R^s. Update x_t^h = A_h(g_t)x_{t-1}^h + B_h(g_t)u_t; y_t = Σ_h C_h(g_t)x_t^h.
- Constraints: spectral radius ρ(A_h) ≤ 1; passivity via Lyapunov parametrization; residual blocks have Lipschitz ≤ 1 − ε; reversible additive coupling to keep activation memory O(1).
- Optional rare global mix: budgeted Performer‑style random features on flagged spans (never full attention).
- Complexity: O(n·M·s), streaming O(1)/token; activation memory O(1) per layer.

2) Global keyed surrogate (HAWK) for attention‑like exchange
- Keys: K global keys partitioned into banks: frequent‑mode and rarity. Keys carry type and phase tags.
- Reads/writes: per token update top‑r keys (r≪K) chosen by relevance; kernel approximation via Nyström with streaming coreset/sensitivity sampling; kernel herding prevents collapse.
- Safety: ACT halting for micro‑steps; optional one‑shot FAVOR+ feature pass only on AEGIS‑flagged spans.
- Complexity: O(n·r·d); K,r fixed → O(n).

3) Event detection (CHED) with hard caps and drift handling
- Two‑tier trigger: fast per‑token score vs covariate‑conditional conformal quantile qα’(z_t); candidates confirmed by bounded‑window GLR threshold η to catch bursts.
- Budgets: sliding window W with m_max admissions; largest margins admitted; M/G/1 priority queue with queue length cap.
- Drift: monitor calibration residuals; adjust α’, margins, and micro‑steps online; fall back to more local compute on OOD.
- Guarantees: FN ≤ α + ε under smooth shift; ARL0 and delay per GLR; m_per_window ≤ m_max ⇒ linear compute.

4) Pointers: ECC‑guarded, phase‑typed, hyperbolic multi‑hop
- Proposals: k independent minhash projections over canonicalized content + ORBIT phase bins; sample diversely (DPP‑lite).
- Parity: CRC‑32C base plus extra random projection bits to reach p parity bits; accept only parity‑consistent anchors.
- 2‑hop prune: hyperbolic Poincaré embeddings for anchors; reject u→w→v if triangle inequality violated beyond slack ε; maintain a low‑stretch spanner over anchors for connectivity.
- Union bound: set p≥ceil(log2(Q/δ)) for Q pointer checks/sequence to keep undetected false link ≤ δ.
- Complexity: O(k) per event; constants small.

5) Memory: SPINE WORM tiers + HELIX episodic DAG (typed, ECC, CRDT)
- Tiers: fixed capacities C_i; two‑choice hashing + small stash; typed Bloom filters per tier; deterministic compaction cadence with small caps.
- Writes: transactional reserve‑verify‑commit; CRDT join (value, timestamp, type_tag) resolves within type; cross‑type conflicts abstain; ECC on entries and merges.
- Episodes: nodes summarizing spans with type‑tagged edges (coref, citations, entailment); TTL/decay; depth‑capped retrieval walks.
- Complexity: amortized O(1) write; O(1) expected read (bounded T, filters); background compaction budgeted.

6) Types, units, and selective enforcement (LATTICE)
- Type head: outputs risk‑controlling prediction set T(x) (RAPS). Hard‑enforce only if |T|=1; else soft penalties.
- Units: abstract interpretation for arithmetic expressions; interval + unit algebra; hard reject inconsistent operations.
- Typed addressing: include type bits in keys/hashes to cut collisions and focus retrieval.
- Guarantees: hard enforcement false‑reject ≤ δ; unit mismatch errors impossible in covered ops.

7) Reasoning and verification (GoT‑lite + cascaded verifiers)
- Frontier: tiny (e.g., up to 8 nodes), typed subgoals drawn from vetted pointers/memory; budgeted expansions.
- Objective: submodular coverage F(S) over detected violations with calibrated fix probabilities; greedy selection gives ≥(1−1/e) of optimal coverage under budget.
- Heuristic: admissible lower bound on residual violations via conformal quantiles; prunes safely; verification cascade (types→regex→unit→symbolic eval) early‑exits; certified abstention if residual > τ after B expansions/refinements.
- Complexity: O(1) per invocation; only on flagged spans.

8) Governance (AEGIS): constrained optimization of compute
- Constrained MDP: minimize task loss under FLOP budget B and per‑module floors b_i. Dual ascent via mirror descent updates λ,μ_i; per‑window enforcement and priority queues to control tail latency.
- Signals: entropy, CHED margins, pointer parity/confidence, type set size, unit checks, memory backlog; outputs budgets and admissions.
- Guarantees: average budget satisfaction; floors never zero; module queues kept with ρ≤ρ* to bound tail latency.

9) Position/time (ORBIT)
- Oscillators: bank of quasi‑incommensurate frequencies; cosine/sine phases; Kalman smoothing; 2‑of‑3 anchor voting at boundaries; use relative phases Δφ to compose keys and pointers.
- Guarantees: injective within anchored windows; phase drift bounded; collisions rare and caught by parity.

Complexity, stability, and guarantees (sketches)

- End‑to‑end sequence complexity: SSM+HAWK O(n); events capped m≤(n/W)·m_max; per event O(k) pointer ops and O(1) memory ops; GoT‑lite invoked on a small fraction with constant work. Total O(n) with small constants; streaming memory O(1) w.r.t. n.
- Gradient stability: passivity‑constrained SSM and contractive reversible residuals imply global BIBO stability and bounded gradients; small‑gain composition keeps overall Lipschitz ≤ 1.
- Event FN control: covariate‑conditional conformal quantiles yield P(miss) ≤ α + ε under smooth shift; GLR adds burst detection with target ARL0; admission caps guarantee bounded compute.
- Pointer correctness: parity bits p sized so undetected collision ≤ δ via union bound; hyperbolic triangle checks prune spurious 2‑hops; spanner keeps multi‑hop costs bounded.
- Memory integrity: CRDT semantics + ECC + typed merges ensure no silent corruption; conflicts abstain; compaction costs amortized O(1).
- GoT effectiveness: with submodular F, greedy expansion achieves ≥(1−1/e) of optimal violation coverage under expansion budget; admissible heuristic avoids over‑pruning.
- Position extrapolation: rationally independent frequencies + anchors produce quasi‑periodic codes with injectivity on windows; phase noise reduced by smoothing; parity guards remaining collisions.
- Queueing: event pipeline as M/G/1 with ρ<1 (AEGIS‑controlled) and capped queue size → exponential tail on backlog; main path prioritized to bound latency.

Training, calibration, and curriculum

- Stage 1: Flow + HAWK
  - Train SSM with spectral/Lipschitz penalties; pretrain HAWK keys via kernel herding; add sensitivity sampling; ACT halting warmup. Optional occasional FAVOR+ to stabilize approximation.
- Stage 2: ORBIT + CHED
  - Train oscillator phases with boundary detection; fit CHED quantile models per partition; calibrate with Mondrian conformal + weighted residuals; wire GLR confirm; set m_max and service rates; add OOD detector to inflate α on shift.
- Stage 3: Pointers + hyperbolic embeddings
  - Train Poincaré embeddings for anchors; implement minhash+CRC parity; calibrate p by target δ and measured Q; add DPP‑lite diversity; fit triangle slack ε quantiles.
- Stage 4: Memory (SPINE+HELIX)
  - Start with soft writes; introduce transactional RVC; ECC injection during training to learn abstention; enforce CRDT merges; tune compaction cadence; add typed Bloom filters.
- Stage 5: Types/units
  - Supervise type head with weak labels; train RAPS prediction sets; enable hard enforcement only for |T|=1 or in math/code; integrate unit algebra and interval analysis.
- Stage 6: GoT‑lite + verifiers
  - Train fix‑probabilities p_{g,v}; shape submodular F; calibrate admissible heuristics via conformal bounds; enable certified abstention; tie MDL penalty to verifier runtime.
- Stage 7: AEGIS governance
  - Warm‑start with simple heuristics; switch to dual ascent with per‑window constraints; set module safety floors; couple to queue states and measured service rates.

Diagnostics and ablations
- Event: FN/FP vs α; GLR ARL0/delay; queue length tails; OOD triggers.
- Pointer: collision rate vs p; diversity coverage; 2‑hop prune accuracy; spanner stretch.
- HAWK: kernel approximation error (Nyström); key diversity; rarity hit‑rate; ACT step histograms.
- Memory: write acceptance, compaction backlog, Bloom FP, conflict abstentions.
- Types/units: RAPS coverage δ, unit violation zero rate.
- GoT‑lite: F(S) gain curves, greedy vs optimal on small cases, abstention frequency vs error.
- AEGIS: budget adherence, tail latency, safety floor spend.
- End‑to‑end: perplexity vs length (128k–1M); streaming decode tokens/s at tiny memory; needle‑in‑haystack; math/code acceptance; retrieval QA conflict resolution.

Systems and hardware

- Fused kernels: single per‑token kernel updates SSM state, HAWK reads/writes, halting; keep head states and key banks in SRAM/registers.
- Hash/parity: SIMD minhash sketches; CRC‑32C hardware intrinsics; random projections with fused XOR/fma.
- Memory tiers: contiguous buckets, two‑choice selection; vectorized Bloom; compaction on a background stream with rate limiting; small stashes per bucket.
- Event streams: separate CUDA streams for events and memory; AEGIS throttles to keep main path latency tight; batch pointer checks across tokens.
- Optional: small MoE with yield‑aware routing if more capacity is needed; still budgeted by AEGIS.

What to measure head‑to‑head vs Transformers
- Scaling: FLOPs/token and memory vs context length; decode throughput with bounded KV.
- Long‑context QA: precision/recall of pointers; adverse distractor robustness; episode DAG retrieval success.
- Math/code: unit/type violation rate (target ~0), verifier acceptance, abstention rate vs accuracy.
- Reasoning: F(S) improvement vs step budget; certified abstention correlation with errors.
- Reliability: undetected pointer collision (target ≤ δ); memory conflict abstention rate and accuracy on conflicts.
- Calibration: CHED FN vs α under domain shift; OOD fallback behavior.
- Efficiency: tokens/s at fixed FLOPs; FLOPs per accepted answer on hard tasks.

Limitations and honest edges
- Type coverage beyond math/code remains partial; enforcement is selective to avoid overconstraint.
- Conformal guarantees assume smooth covariate shift; severe shifts require online recalibration and conservative fallbacks.
- Hyperbolic embedding/pruning depends on good training; we use slack ε and abstain liberally to avoid errors.
- Budget controller is nonconvex; we guarantee feasibility and tails, not global optimality.
- Systems complexity is nontrivial; we keep defaults minimal (SSM+HAWK) and make precision modules strictly opt‑in and budgeted.

Glossary (modules)
- SSM: stable selective state‑space model with passivity and reversible residuals.
- HAWK: global keyed memory approximating attention via Nyström; keys updated/read sparsely with ACT halting.
- CHED: covariate‑conditional conformal hazard detector + GLR confirm + admission caps and queueing.
- ECC pointer: parity‑guarded anchors from minhash+CRC with diversity, phase tags, and hyperbolic 2‑hop pruning.
- SPINE: transactional WORM tiers with two‑choice hashing, Bloom filters, ECC, deterministic compaction, and CRDT merges.
- HELIX: episodic DAG overlay with typed edges, TTL, and depth‑capped walks.
- LATTICE: type/units system with risk‑controlling prediction sets (RAPS) and hard unit algebra checks.
- GoT‑lite: typed, budgeted frontier with submodular coverage objective, admissible heuristic, and certified abstention.
- AEGIS: constrained‑optimization budget controller with mirror‑descent duals and per‑window constraints.
- ORBIT: rationally‑independent oscillators with Kalman smoothing and anchor voting for phase‑augmented addressing.

How RAPTOR resolves Transformer pain points (delta vs RAVEN)
- Quadratic blow‑up: no dense attention; HAWK keys approximate it in O(1); events/pointers budgeted; worst‑case O(n).
- Long‑context filtering: FN‑controlled events + parity‑guarded pointers + episodic DAG; stronger, cheaper than cross‑encoder + buffer.
- Recurrence/memory: stable SSM and transactional memory with ECC; KV cache replaced by bounded, content‑addressable tiers.
- Reasoning: typed constraints + submodular GoT‑lite + verifier cascade + certified abstention; leaner and more principled than generic GoT.
- Positions: anchored oscillators integrated into addressing; robust extrapolation far beyond training lengths.
- Static compute: AEGIS enforces FLOP budgets with floors and tail control; ACT and admissions shape per‑span compute.
- Gradient stability: passivity + contractive reversible stacks; explicit small‑gain guarantees.
- KV cache: bounded tiers and keys; no unbounded KV growth; on‑device and ECC‑guarded.
- IO/bandwidth: fused scans and key ops; compact memory tiers and Bloom filters; background compaction avoids spikes.

Typical hyperparameters (starting points)
- SSM heads M=8–16, state s=64–128; residual ε≈0.05.
- HAWK keys K=64–128; updates r=4–8; rarity bank 1/4 of K.
- Events: W=512, m_max=32; k=6 proposals.
- Parity bits p≥32 for δ≤1e−8 and Q≈1e4; adjust with measured Q.
- Memory: T=3–4 tiers; small stash=2; Bloom FP per tier ≈1e−4; compaction every 256 tokens with small cap.
- Types: RAPS δ=0.05 for hard enforcement; units checker enabled on numeric spans.
- GoT‑lite: frontier F≤8; expansions B≤16 per span; abstention threshold τ tuned to target error/abstain tradeoff.
- Governance: per‑window budgets over 2k tokens; target ρ*≈0.7 for event queues.

Bottom line
RAPTOR is a lean, budget‑honest, verifiable alternative to Transformers: linear in sequence length, stable for deep/long horizons, selective over long context, with explicit memory and checks. It improves on RAVEN by replacing heuristics and cross‑encoders with calibrated gates, parity‑guarded addressing, transactional memory, typed constraints, and a principled budget controller—while keeping the default path simple and fast.
