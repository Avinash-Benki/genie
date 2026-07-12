# Inference & Optimization

## Contents
1. Performance model: where time goes
2. Memory math & KV cache
3. Serving engines & config
4. Batching & scheduling
5. Quantization decision guide
6. Attention & kernel optimizations
7. Speculative decoding & multi-token
8. Long context & caching strategies
9. Multi-GPU & disaggregated serving
10. Latency/throughput tuning playbook
11. Cost engineering & capacity planning

## 1. Performance model

Two regimes:
- **Prefill:** compute-bound; time ≈ 2·N·L_prompt / (FLOPs·MFU). Parallel across tokens.
- **Decode:** memory-bandwidth-bound; each token reads all weights + KV: time/token ≈ (weight_bytes + kv_bytes) / HBM_BW. A 8B BF16 model on H100 (3.35TB/s): 16GB/3.35TBps ≈ 4.8ms/token floor at batch 1 → batching is how you buy throughput (weights read once per batch).
- Metrics: TTFT (prefill), TPOT/ITL (decode), tokens/s/GPU (throughput), goodput (requests meeting SLO). Optimize the one your product feels: chat = TTFT+TPOT; batch jobs = throughput; agents = end-to-end trajectory latency.

## 2. Memory & KV cache

KV bytes = 2 · layers · kv_heads · head_dim · dtype · seq · batch.
Example 8B (32L, 8 kv-heads GQA, 128 dim, FP16): 2·32·8·128·2 = 131 KB/token → 32k-ctx request = 4.3 GB. This — not weights — caps concurrency.
Levers: GQA (arch), **PagedAttention** (near-zero fragmentation — table stakes), KV quantization FP8 (≈free) / INT4 (small quality tax), sliding-window/hybrid layers, prefix sharing.
Capacity: concurrent_seqs ≈ (HBM − weights − activations) / (kv_per_token × mean_ctx).

## 3. Serving engines

- **vLLM:** default general choice; PagedAttention, continuous batching, prefix caching, spec decode, wide model support.
- **SGLang:** RadixAttention (automatic prefix-tree KV reuse — shines for agents/few-shot/heavy shared prefixes), structured output fast paths.
- **TensorRT-LLM:** peak NVIDIA perf, FP8/FP4 paths, more build friction.
- llama.cpp/ollama/MLX: edge/local.
Config that matters: `max_num_batched_tokens` (prefill chunk budget), `gpu_memory_utilization` (0.90–0.95), `enable_prefix_caching`, tensor_parallel_size, chunked prefill ON (smooths TTFT vs TPOT), scheduler policy.

## 4. Batching & scheduling

- Continuous (in-flight) batching: sequences join/leave per step — 2–10× throughput vs static batching.
- Chunked prefill: split long prompts into chunks interleaved with decode steps → protects TPOT of running streams from new long-prompt stalls; budget = max_num_batched_tokens (2k–8k typical).
- Priority/SLO scheduling: separate interactive vs batch queues; admission control before saturation (queueing collapses goodput past ~80% util).
- Throughput scales ~linearly with batch until KV-capacity or compute crossover; find the knee with a load sweep (vary concurrency, plot tokens/s and p95 ITL).

## 5. Quantization

| Method | Bits | Quality | When |
|---|---|---|---|
| FP8 (W8A8, TE/vLLM) | 8 | ~lossless | Default on H100+; ~1.5–1.8× decode |
| INT8 SmoothQuant | 8 | ~lossless | A100-era W8A8 |
| AWQ / GPTQ | 4 (W4A16) | −0.5–2% typical | Memory-bound serving, single-GPU fits |
| FP4/NVFP4 | 4 | small tax, HW-native | Blackwell+ |
| GGUF K-quants | 2–6 | varies | llama.cpp/local |
Rules: quantize weights first (decode is weight-BW-bound → W4 ≈ up to 2–3× tok/s when memory-bound); KV-quant next; activations last. ALWAYS re-run your task evals post-quant (generic MMLU deltas hide task-specific damage, esp. math/code & long-tail languages). Calibration set = your real traffic distribution, 128–512 samples. Never merge LoRA into quantized weights.

## 6. Kernels & attention

FlashAttention-2/3 (prefill), fused RoPE/RMSNorm/SwiGLU, CUDA graphs for decode (kernel-launch overhead at small batch), torch.compile (max-autotune) for custom stacks. FP8 GEMM via Transformer Engine. On serving engines these are mostly built-in — your job is to not disable them and to keep versions current.

## 7. Speculative decoding

Draft proposes k tokens, target verifies in one pass; exactness preserved. Speedup = f(acceptance rate α, draft cost): typically 1.5–3×.
- Options: separate small draft model (same tokenizer!), **EAGLE-2/3 heads** (best acceptance, needs trained head), Medusa, n-gram/prompt-lookup (free, great for RAG/code-edit with copy-heavy outputs), self-spec (layer-skip).
- Works best: low temperature, predictable domains (code), small batch (latency-motivated). Under heavy batching, spec-decode can HURT throughput (wasted verify FLOPs) — measure at your operating batch size.

## 8. Long context & caching

- Prefix/prompt caching: cache KV of shared system prompts/few-shots/docs; RadixAttention (SGLang) or vLLM APC; hit-rate is the metric — restructure prompts to maximize shared prefixes (static parts FIRST, per-request parts last; keep tool schemas byte-identical).
- Agent loops: KV reuse across steps is the single biggest agent-latency lever; append-only prompt design (never edit earlier turns mid-trajectory).
- Context length vs cost: KV linear in ctx, prefill quadratic-ish in practice — summarize/compact trajectories; RAG top-k discipline beats stuff-everything.

## 9. Multi-GPU & disaggregation

- TP for latency (splits BW reads) — TP=2–8 intra-node; PP for capacity across nodes; replicate+route beyond that.
- Disaggregated prefill/decode (P/D split, e.g., 2 prefill GPUs feeding 6 decode GPUs via KV transfer): isolates TTFT from TPOT, better hardware matching; adopt when scale ≥ dozens of GPUs and SLOs are tight (vLLM-disagg / Dynamo-class routers).
- MoE serving: EP across GPUs + wide-EP with DP attention; all-to-all becomes the bottleneck — co-locate experts by routing stats.

## 10. Tuning playbook (in order)

1. Measure baseline: load-sweep → TTFT/TPOT/thruput curves + GPU util + KV occupancy.
2. Right-size model (distill/smaller variant) — the 10× lever nobody wants to hear.
3. Quantize weights (FP8 or AWQ) → re-eval.
4. Enable prefix caching + restructure prompts for hit-rate.
5. Chunked prefill + batch-token budget tuned to SLO.
6. Speculative decoding if latency-bound at low batch.
7. Scale out with router (session-affinity for cache hits) / P-D disaggregation.
8. Re-run the load sweep after EVERY change; keep a perf regression CI (fixed traffic replay).

## 11. Cost engineering

$/1M output tokens ≈ GPU_$/hr ÷ (tok/s × 3600) × 1e6 ÷ utilization. Example: H100 $2.5/hr @ 2,500 tok/s effective, 60% util → $0.46/M. Compare vs API pricing including idle risk. Utilization is the whole game: batch-job backfill for off-peak, autoscale on queue depth not GPU util, spot for stateless replicas.