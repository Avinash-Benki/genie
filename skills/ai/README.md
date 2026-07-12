
**What's inside:**

The SKILL.md acts as a router with a deliberately "pushy" trigger description (fires on GPUs, FSDP, RLHF, quantization, OOM debugging, capacity questions — even without the words "AI engineering") plus 7 operating principles that apply to every task, headlined by *napkin math first*: the 6ND training-compute identity, 16 bytes/param memory rule, and MFU assumptions baked in so any model using this skill starts every answer with arithmetic, not vibes.

The 7 expert references, each self-contained with a table of contents:

1. **infrastructure.md** — GPU spec tables, fleet-sizing recipes with worked examples, NCCL debugging, storage tiering, checkpoint-interval math, Xid-error runbook, buy-vs-rent break-evens
2. **distributed-training.md** — the parallelism decision tree (DDP → FSDP → 3D), ZeRO stage-by-stage memory math, TP/PP/CP/EP composition rules, communication-volume formulas, resharding checkpoints, hang-debugging
3. **pretraining.md** — Chinchilla vs inference-aware overtraining, the full hyperparameter table (β2=0.95, z-loss, WSD schedules), loss-spike response ladder, mixture/annealing strategy, MoE and long-context extension recipes
4. **post-training.md** — SFT→DPO→GRPO pipeline, the DPO pathology catalog (SimPO/IPO/KTO alternatives), GRPO/DAPO specifics, RLVR verifier design, LoRA targeting rules, and a failure-modes table (template mismatch, length inflation, reward hacking)
5. **data-curation.md** — full pipeline ordering, MinHash dedup parameters, decontamination protocol, tokenizer fertility, and 5 synthetic-data generation patterns with collapse safeguards
6. **evals.md** — harness rigor checklist (the silent score-swingers), binomial error bars and paired tests, LLM-judge bias catalog with calibration protocol, contamination detection on models, CI regression gates
7. **inference-optimization.md** — prefill-vs-decode performance model, KV-cache capacity math with worked 8B example, quantization decision table, speculative-decoding tradeoffs, the 8-step tuning playbook, $/1M-token cost formula

Everything is written to age well: hard-won principles and formulas inline, with explicit instructions to search for current-generation specs, prices, and benchmark rotations rather than trusting static numbers.
