# Post-training

## Contents
1. Pipeline overview & when to use what
2. SFT: data, recipe, hyperparameters
3. Chat templates & special tokens
4. Preference optimization: DPO family
5. RLHF with PPO/GRPO; reward models
6. RLVR: verifiable rewards & reasoning training
7. PEFT: LoRA/QLoRA decision guide
8. Continued pretraining & domain adaptation
9. Merging, distillation
10. Safety tuning & regression control
11. Failure modes catalog

## 1. Pipeline & selection

Modern stack: **SFT → preference optimization (DPO/variants) → RL (GRPO/PPO) where verifiable or reward-model signal exists**, with eval gates between stages.
Choose by goal: style/format/instruction-following → SFT (+DPO); subjective quality/harmlessness → RM + PPO or DPO on preferences; math/code/agentic correctness → RLVR (GRPO) with programmatic verifiers; knowledge injection → continued pretrain (SFT alone can't add knowledge reliably, it mostly elicits).

## 2. SFT

Data > algorithm. 1k–10k excellent, diverse, deduplicated examples beat 100k mediocre (LIMA effect); production assistants use 100k–1M+ curated across capabilities with explicit per-capability quotas.
Recipe:
- Loss on completion tokens only (mask prompt); packing with per-sample masking OK.
- LR: full-FT 1e-5–2e-5 (7–8B), 5e-6 (70B); cosine, warmup 3%; **1–3 epochs** (watch eval, not train loss — SFT overfits fast); batch 64–512 sequences; bf16.
- NEFTune (embed noise) sometimes +; sample-level length-normalized loss for long outputs.
- Mixture care: include some "identity/refusal/format" data every stage or later stages erase earlier behaviors (catastrophic forgetting of behaviors, not just facts). Replay 1–5% of previous-stage data in each new stage.

## 3. Chat templates

- Single source of truth (tokenizer_config jinja); train and serve MUST byte-match — the #1 silent quality killer in deployments. Test: render a multiturn convo in trainer and in server, diff token ids.
- Special tokens (<|im_start|> etc.): added to tokenizer, embeddings resized; new tokens' embeddings init = mean of existing (not random-large).
- Loss masking per role: user/system masked, assistant unmasked; tool-call results usually masked, tool-call generation unmasked.

## 4. DPO family

DPO loss: -log σ(β[(logπ(yw|x)−logπref(yw|x)) − (logπ(yl|x)−logπref(yl|x))]). β 0.05–0.3 (lower = stronger drift from ref). LR 5e-7–1e-6 (10–20× below SFT!), 1 epoch typical.
- Data: pairs from your OWN policy's samples (on-policy-ish) work far better than off-the-shelf preference sets; 5k–100k pairs.
- Known pathology: DPO drives down BOTH logprobs (chosen too), can cause length inflation & off-distribution drift. Mitigations: NLL anchor on chosen (RPO/CPO-style +λ·SFT-loss), length-regularized variants (R-DPO), or use **SimPO** (reference-free, length-normalized, β≈2–2.5, γ≈0.3–1.4) / **IPO** (bounded, less overfit) / **KTO** (unpaired thumbs-up/down data).
- Iterative DPO (sample → judge → pair → train, 2–4 rounds) ≈ cheap RLHF; strong default when you lack RL infra.

## 5. PPO/GRPO & reward models

**Reward model:** init from SFT model, Bradley-Terry pairwise loss on preferences; 100k–1M pairs for general RM. Evaluate on RewardBench-style held-out + adversarial probes (length bias, sycophancy, format bias). RMs saturate/generalize poorly off-distribution — refresh with on-policy samples each RL round.
**PPO:** 4 models in memory (policy, ref, RM, value) — heavy. Key hps: KL coef (target KL 0.05–0.2/token or adaptive), clip ε 0.2, GAE λ 0.95, batch 512–1024 prompts, LR 1e-6, minibatch epochs 1–4. Watch: reward↑ while KL modest; if reward↑ + eval↓ = reward hacking.
**GRPO:** drops the value model; advantage = (r − mean_group)/std_group over G samples/prompt (G=4–16). Standard for reasoning-RL; cheaper, stabler. Variants: DAPO (clip-higher, dynamic sampling — resample groups with zero advantage variance), token-level vs sequence-level KL, remove std-normalization (Dr.GRPO) to fix length bias.
Infra: vLLM for rollouts + FSDP training loop (open frameworks: TRL, verl, OpenRLHF); rollout↔train weight-sync cadence is the throughput crux.

## 6. RLVR (verifiable rewards)

Reward = programmatic check: math answer match (sympy-normalized), unit tests pass, JSON schema valid, agent task completion. This is the engine behind reasoning models.
- Recipe: strong base + light SFT on long-CoT seeds → GRPO with binary/graded verifier reward + format reward (small) + length shaping (careful: causes verbosity if positive).
- Curriculum by difficulty (pass-rate bands 10–70%: too easy/hard = zero gradient signal — dynamic filtering).
- Watch: response length drift, entropy collapse (add entropy bonus or temperature in rollouts), verifier exploits (regex-gameable checkers) — red-team the verifier first.
- Evals: pass@1 with temp 0.6–1.0 multi-sample mean, AND pass@k trend (RLVR can raise pass@1 while shrinking pass@k diversity).

## 7. LoRA/QLoRA guide

- LoRA: r=16–64, α=2r, dropout 0.05, target ALL linear layers (q,k,v,o,gate,up,down) — not just q,v; LR 1e-4–2e-4 (10× full-FT). Capacity: great for style/format/task; weaker for large knowledge shifts & reasoning-RL.
- QLoRA: NF4 base + bf16 adapters; ~0.5–1 pt quality tax typically; enables 70B FT on 2×80GB.
- Full-FT vs LoRA: full-FT still measurably better for aggressive distribution shifts and RL; LoRA ≈ parity for narrow SFT. rsLoRA/DoRA marginal gains, use if in stack.
- Merging adapters for serving: merge into bf16 weights, re-quantize after (never merge into quantized weights).

## 8. Continued pretraining (CPT)

For domain/language adaptation: 1–50B domain tokens mixed with 10–30% original-distribution replay (else general-capability collapse); LR ≈ 0.1× pretrain peak with re-warmup; then re-run the SFT/DPO stack (CPT damages instruction behavior).

## 9. Merging & distillation

- Merging (mergekit): SLERP/TIES/DARE for combining specialists — cheap, surprisingly strong for style blends; always re-eval safety after merges.
- Distillation: (a) black-box: SFT on teacher outputs (most "distillation" today); (b) logit KD (forward KL, temperature 1–2) needs teacher logits & same tokenizer; (c) on-policy distillation (student samples, teacher scores/relabels) fixes exposure bias — best quality per FLOP for reasoning transfer.

## 10. Safety & regression control

Every stage gates on: capability suite + safety suite (refusal correctness both directions: harmful-comply AND benign-overrefusal) + format/regression tests (JSON validity, tool-call syntax, multilingual spot checks). Keep a frozen "canary set" never used in training. Diff behavior vs previous release on fixed prompts (win/loss/tie judge) before shipping.

## 11. Failure modes

| Symptom | Likely cause → fix |
|---|---|
| Model answers well but ignores system prompt | system-turn masked wrong / template mismatch |
| Sudden verbosity after DPO/RL | length bias in RM/judge or positive length shaping → length-normalize, penalize |
| "As an AI..." over-refusals up | preference data skew → add benign-hard prompts with helpful completions |
| Repetition at inference post-SFT | over-epoched SFT; LR too high; check no EOS-loss bug (EOS must be trained!) |
| Great evals, bad vibes | contaminated/narrow evals; run human/arena-style checks |
| RL reward ↑, everything else ↓ | reward hacking → inspect top-reward samples manually every run |