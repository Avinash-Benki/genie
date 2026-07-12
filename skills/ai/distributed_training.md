# Distributed Training

## Contents
1. Parallelism taxonomy & selection
2. Data parallel: DDP, ZeRO, FSDP
3. Tensor parallel
4. Pipeline parallel
5. Context/sequence parallel & expert parallel
6. Composing 3D/4D parallelism
7. Communication analysis
8. Checkpointing & elastic training
9. Debugging distributed jobs

## 1. Taxonomy & selection

Axes: **DP** (replicate/shard states, split batch), **TP** (split individual matmuls), **PP** (split layers into stages), **CP/SP** (split the sequence), **EP** (split MoE experts).

Selection decision tree:
1. Model + optimizer fits one GPU (≤~1B on 80GB)? → plain **DDP**.
2. Doesn't fit, ≤ ~13B? → **FSDP/ZeRO-3** alone across the node(s).
3. 13B–70B multi-node? → **TP=8 within node + FSDP/ZeRO-DP across nodes**; add PP if TP×DP memory still insufficient or inter-node BW weak.
4. ≥70B or thousands of GPUs? → full 3D: TP(intra-node) × PP(inter-node) × DP(outermost); CP for long sequences; EP for MoE.
5. Long context (>32k) training? → add **CP (ring/ulysses attention)**.

Golden rules: TP never crosses node boundary (needs NVLink-class BW). PP tolerates slow links best (only activations at stage boundaries cross). DP gradient traffic overlaps with backward — verify overlap actually happens (profiler).

## 2. Data parallel family

**DDP:** each rank holds full model+optimizer; all-reduce grads (2×P bytes/step ring). Simple, fastest when it fits.

**ZeRO stages (DeepSpeed):**
- ZeRO-1: shard optimizer states (12B/param → 12/N). Comm same as DDP.
- ZeRO-2: + shard grads. Reduce-scatter instead of all-reduce.
- ZeRO-3: + shard params; all-gather params per layer forward AND backward → comm 3×P per step (1.5× DDP) but memory /N.
- Offload (CPU/NVMe): fits huge models on little hardware at 5–20× slowdown — fine for FT experiments, not pretraining.

**FSDP (PyTorch native, prefer FSDP2):**
- ≈ ZeRO-3 semantics: `FULL_SHARD` (Z3), `SHARD_GRAD_OP` (Z2), `HYBRID_SHARD` (shard within node, replicate across — cuts inter-node all-gather; the default choice for 2–32 node jobs).
- Wrap policy: per-transformer-block auto-wrap; enables comm/compute overlap via prefetch (`forward_prefetch=True`, `backward_prefetch=BACKWARD_PRE`).
- Use `use_orig_params=True` (FSDP1) / FSDP2 DTensor for torch.compile compatibility and per-param FQNs.
- `MixedPrecision(param_dtype=bf16, reduce_dtype=fp32)` — fp32 grad-reduce avoids precision loss at large DP degree.

Memory/GPU (P params, N-way sharding, mixed precision): full-shard ≈ 16P/N + activations + one layer's gathered params (peak).

## 3. Tensor parallel

Megatron-style: attention heads and MLP split column-then-row so each block needs 2 all-reduces fwd + 2 bwd. Comm per layer ≈ 8 × batch × seq × hidden bytes (BF16) — this is why TP demands NVLink.
- TP degree: keep heads % tp == 0; ffn % tp == 0. TP=8 max on standard HGX; TP>8 needs NVSwitch domains (NVL72-class).
- Sequence parallelism (Megatron-SP): shards the LayerNorm/dropout activations along sequence within the TP group, replacing all-reduce with reduce-scatter+all-gather — same comm volume, big activation memory savings. Always enable with TP.
- Async TP / overlap (`tp_comm_overlap`): hides all-gather behind GEMMs via P2P chunks; worth it ≥70B.

## 4. Pipeline parallel

Split layers into S stages; microbatches (m) flow through. Bubble fraction = (S−1)/(m+S−1) → need m ≥ 4×S for <20% bubble.
- Schedules: GPipe (fill-drain, high memory), **1F1B** (standard, memory = S× less activation than GPipe), interleaved 1F1B/virtual stages (bubble /v at cost of more comm), zero-bubble (ZB-H1) splits backward into B/W for near-zero bubble.
- PP comm: only activations at boundaries — batch×seq×hidden per microbatch per boundary; friendly to Ethernet.
- Balance stages by profiled time, not layer count (embedding + first/last stages are heavier; put lm_head with fewer layers).

## 5. Context & expert parallel

**CP:** Ring Attention (P2P KV rotation, overlap-friendly) or DeepSpeed-Ulysses (all-to-all on heads). Use when seq × hidden activation per GPU explodes (≥32–64k tokens). Loss/logits also need seq-sharding awareness.
**EP (MoE):** experts sharded across EP group; two all-to-alls per MoE layer. Keep EP within nodes when possible; token-dropping vs dropless (capacity factor 1.0–1.25). Load-balancing aux loss ~1e-2; monitor expert utilization entropy — collapse is the classic failure.

## 6. Composing 3D/4D

World = TP × CP × PP × DP (× EP interleaved). Layout order (innermost=fastest comms): TP → CP → EP → PP → DP.
Example 70B on 512 H100 (64 nodes): TP=8 (intra-node), PP=4, DP=16, microbatch to hit global batch ~4M tokens. Memory check: params/stage = 70B/4 = 17.5B; per GPU after TP: 2.2B → weights+grad+opt (ZeRO-1 across DP) ≈ 2.2×(2+2+12/16) ≈ 10.5 GB + activations — comfortable.
Tuning order: (1) fit memory (raise TP/PP or recompute), (2) maximize MFU (lower TP if comm-bound, raise microbatch), (3) then scale DP.

## 7. Communication analysis

Per step, P params, DP degree d, ring collectives:
- DDP all-reduce: 2P(d−1)/d bytes/GPU.
- ZeRO-3/FSDP: all-gather P (fwd) + all-gather P (bwd) + reduce-scatter P = 3P.
- Time = bytes / bus_BW; compare to step compute time; require overlap coverage >80%.
- Global batch scaling: comm per GPU is constant with d, but exposed comm grows if compute/step shrinks (smaller per-GPU batch) — the reason strong scaling dies.
- Gradient accumulation trades comm frequency for memory: k accumulation steps = 1 reduce per k micro-steps (DDP `no_sync`).

## 8. Checkpointing & elasticity

- Use `torch.distributed.checkpoint` (DCP) or DeepSpeed universal checkpoints: rank-sharded, topology-independent resume (reshard TP/PP/DP on load).
- Save: model shards, optimizer shards, LR scheduler, dataloader state (sample index / shard cursor), RNG per rank, config hash. Async save (DCP async_save) to hide write latency.
- Elastic: torchrun `--max-restarts`, TorchElastic rendezvous; on failure, resume from last ckpt at possibly different world size — verify batch-size invariance of your LR schedule (token-based schedules resume cleanly, epoch-based don't).

## 9. Debugging distributed

1. Reproduce small: same script, 2 GPUs, tiny model — logic bugs first.
2. Hangs: `py-spy dump --pid` per rank / `TORCH_NCCL_TRACE_BUFFER_SIZE` + flight recorder; usually one rank diverged (OOM, exception swallowed) or collective-order mismatch (conditional collective).
3. Numerical divergence between parallel configs: compare with TP=1 fp32 fixed batch; check RNG (dropout) seeding per TP rank, grad-reduce dtype, loss-scale.
4. Perf: torch profiler + HTA / nsys — look at NCCL kernel gaps, dataloader gaps, unoverlapped all-gathers. MFU = 6NDtokens_per_s/(GPUs×peak). Report MFU in every experiment.