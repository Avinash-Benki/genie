# Pre-training

## Contents
1. Scaling laws & budget planning
2. Architecture defaults that work
3. Hyperparameters (the numbers)
4. Data mixture & curriculum
5. Training stability: spikes, divergence, remedies
6. Monitoring & mid-run interventions
7. Long-context extension
8. MoE specifics
9. Ablation methodology

## 1. Scaling laws & budget

- Chinchilla-optimal: **D ≈ 20 × N** tokens for compute-optimal loss (7B → 140B tokens). But *inference-aware* training goes far beyond: modern small models train 100–500× N (8B on 1.5–15T tokens) because inference cost dominates lifetime spend — "overtrained" is usually right for deployed models.
- Compute: C ≈ 6ND. Fix budget C, choose N vs D by deployment constraint (target latency/VRAM → N; then D = C/6N).
- Loss prediction: L(N,D) = E + A/N^α + B/D^β (α≈0.34, β≈0.28 Chinchilla fits) — fit your own on small runs (isoFLOP sweeps at 1e18–1e20) before committing the big run; never extrapolate someone else's constants across a different data distribution.
- µP (maximal update parametrization) / µTransfer: tune LR & init on a small proxy width, transfer to target width — de-risks HP choice for big runs. Verify with a coord-check.

## 2. Architecture defaults (dense decoder)

Start from the boring recipe; deviate one thing at a time:
- Pre-norm RMSNorm; SwiGLU MLP (ffn ≈ 8/3 × hidden, round to hardware-friendly multiples of 256); RoPE (θ=10k base; 500k+ if long-context planned); GQA (kv_heads = heads/4 to /8) — free inference win, negligible quality cost; no biases; tied embeddings below ~3B, untied above.
- Depth/width: aspect ratio hidden/layers ≈ 64–128; deeper is better per-param up to trainability limits.
- Vocab: 32k (small models) to 128k–256k (multilingual/frontier); vocab params = 2×V×hidden (untied) — meaningful at small N.
- Init: normal(0, 0.02) scaled; residual projections scaled by 1/√(2L) (GPT-2 style) — or µP.
- QK-norm and z-loss (1e-4 × log²Z) both cheap and materially improve stability at scale — include by default ≥7B.

## 3. Hyperparameters

| Item | Default | Notes |
|---|---|---|
| Optimizer | AdamW β=(0.9, 0.95), eps 1e-8 | β2=0.95 (not .999) at scale — spike resistance |
| Peak LR | ~3e-4 (1B), 3e-4→1.5e-4 (7–13B), ~1e-4 (70B) | Scale ~1/√width if not µP; sweep ±2× on proxy |
| Schedule | Warmup 0.1–1% of steps (≥1–2k) → cosine to 10% peak, or WSD (warmup-stable-decay) | WSD lets you extend training/decay late — good when D undecided |
| Weight decay | 0.1 (decoupled) | Skip on norms/embeddings optionally |
| Grad clip | 1.0 global-norm | Watch clip-rate: >5% steps clipping = LR too high |
| Batch size | Ramp 0.25M→4M tokens (7B-class); critical batch grows with training | Token-based schedule, not step-based |
| Precision | BF16 compute, FP32 grad-reduce & optimizer master | FP8 (Transformer Engine) ~1.3–1.6× speedup ≥H100 — adopt after BF16 baseline works |
| Dropout | 0.0 for single-epoch web-scale | >1 epoch or small data: 0.1 |

## 4. Data mixture & curriculum

- Mixture is the highest-leverage knob after scale. Typical strong open mixes: heavily filtered CommonCrawl/FineWeb-class web 55–70%, code 10–20%, math 3–8%, papers/books 5–10%, wiki/reference 2–5%, multilingual to taste. Weight by *quality-adjusted* sampling, not raw size; cap epochs/source (≤4 epochs before measurable degradation; unique > repeated).
- Curriculum: mostly uniform, but a **decay-phase upgrade** (last 10–20%: upsample highest-quality + instruction-dense + math/code, "midtraining"/annealing) reliably lifts benchmarks — this is where cheap wins live.
- Sequence packing: pack documents to full seq-len with EOD tokens; block cross-document attention (attention mask per doc or intra-doc masking) — measurable quality gain, prevents cross-doc leakage.
- Shuffle at shard AND intra-shard level; a bad shuffle shows up as periodic loss oscillation matching shard boundaries.

## 5. Stability: spikes & divergence

Loss spike taxonomy & response ladder:
1. Transient spike, self-recovers < few hundred steps → log it, continue; inspect the data window (usually a pathological doc run: garbage, repeats, exotic unicode).
2. Spike + grad-norm explosion, recovers slowly → resume from pre-spike ckpt, **skip the offending data batches** (proven effective at frontier scale), optionally lower LR 10–20%.
3. Repeated spikes same region → data problem; audit + filter that source.
4. Slow divergence (loss climbing over thousands of steps) → LR too high for this phase, or β2 too high, or fp16 (switch to bf16), or missing z-loss/qk-norm.
Preventives: bf16 (never fp16 at scale without loss-scaling expertise), β2=0.95, z-loss, qk-norm, grad-clip 1.0, embedding LR possibly lower, warmup long enough, EOD-aware packing.
Track: grad-norm (per-step), max logit/attention entropy, per-source loss, weight & update RMS (µP coord check style).

## 6. Monitoring & interventions

Dashboard minimum: token loss (+ per-domain), grad norm, clip rate, LR, tokens/s & MFU, eval-suite every N tokens (perplexity on held-out slices + few-shot lite set: HellaSwag, ARC, MMLU-lite, HumanEval-lite), spike counter, dataloader wait.
Mid-run levers (safe): extend D with WSD; adjust mixture weights at a checkpoint boundary (log it); batch ramp. Unsafe mid-run: architecture changes, tokenizer changes, optimizer swaps.
Divergence insurance: keep last K=3 checkpoints + one "golden" every 50–100B tokens.

## 7. Long-context extension

Do it as continued pretrain, not from scratch: raise RoPE θ (ABF: 10k→500k–8M) or use YaRN/NTK scaling; train 20–100B tokens at target length with upsampled long documents (books, code repos, concatenated threads); keep some short data (≥50%) to avoid short-context regression. Needs CP/ring-attention infra beyond ~32k. Validate with needle-in-haystack AND real long-doc tasks (RULER-class), not NIAH alone.

## 8. MoE specifics

- Typical: 8–64 experts, top-2 (or top-1 fine-grained + shared expert), active params ~1/4–1/8 total. Same loss ≈ dense at ~1/3–1/2 active FLOPs; costs: memory = total params, comms (all-to-all), infra complexity.
- Aux losses: load-balance (~1e-2), router z-loss (~1e-3). Monitor per-expert token share; entropy collapse → raise balance coef or add jitter.
- LR ~30% lower than dense equivalents; router in fp32.

## 9. Ablation methodology

- Proxy scale: 10–100× smaller compute, SAME data pipeline & eval suite; 2–3 seeds for anything <2× noise floor.
- Change one variable; measure at matched compute AND matched tokens (both, or conclusions mislead).
- Beware small-scale inversions (things that help at 100M hurt at 10B: e.g., heavy dropout, some curricula) — confirm top-2 candidates at an intermediate scale before the flagship run.