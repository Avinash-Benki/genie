# Evals

## Contents
1. Eval strategy & hierarchy
2. Benchmark landscape by capability
3. Implementation rigor (the silent killers)
4. Statistical methodology
5. LLM-as-judge done right
6. Contamination detection
7. Agentic & tool-use evals
8. Safety evals
9. Production/online evals & regression gates
10. Building custom evals

## 1. Strategy & hierarchy

Layered pyramid — cost up, frequency down:
1. **Per-checkpoint (automatic):** perplexity slices + lite few-shot set (15-min budget).
2. **Per-candidate:** full academic suite + domain suites + safety.
3. **Pre-release:** judge-based head-to-head vs current prod, human evals on frontier cases, red-team.
4. **Online:** A/B, thumbs, task-completion telemetry.
An eval is a decision instrument: define BEFORE running what number triggers what action (ship/block/investigate). Evals without decision rules are dashboards, not evals.

## 2. Benchmark landscape (verify current SOTA sets via search — this rotates fast)

- Knowledge/reasoning: MMLU(-Pro), GPQA(-Diamond), ARC-C, AGIEval.
- Math: GSM8K (saturated—use as smoke test), MATH-500, AIME/HMMT-style comps, OlympiadBench.
- Code: HumanEval+/MBPP+ (EvalPlus - base versions are gameable), LiveCodeBench (rolling, contamination-resistant), SWE-bench(-Verified) for repo-level.
- Long context: RULER, NIAH-multikey (NIAH alone is trivially gamed), LongBench-v2.
- Instruction following: IFEval (verifiable constraints), MT-Bench/Arena-Hard (judge-based), WildBench.
- Multilingual: MGSM, Global-MMLU/BELEBELE.
- Prefer "live"/rolling and functional-correctness sets over static multiple choice wherever stakes are high.

## 3. Implementation rigor

Same model, same benchmark can swing many points on harness details. Lock and log ALL of:
- Prompt format & chat template (base vs chat scoring differs fundamentally: loglikelihood-choice vs generative-parse), few-shot count & exact shots, answer-extraction regex (test it on 50 outputs manually!), max_new_tokens (truncation silently zeroes long-CoT scores), temperature (greedy for single-shot claims; temp+n-sample mean for reasoning models), stop sequences, system prompt, tokenizer version, seed.
- Use a standard harness (lm-eval-harness, evalchemy, HELM) + pin its commit; publish configs with results. Never compare numbers across different harnesses/settings — reproduce baselines yourself under YOUR harness before claiming wins.

## 4. Statistics

- Every score gets an error bar: binomial SE = √(p(1−p)/n); n=500 → ±~2.2pts at p=.5 (95% CI). MMLU-lite subsets are noisier than teams assume.
- Compare models on the SAME items: paired tests (McNemar / bootstrap over items); report win-loss on item level, not just aggregate deltas.
- Multi-sample metrics: pass@k unbiased estimator (Chen et al.) — never compute pass@k from k samples naively; sample n>k. Report pass@1 (mean over samples) AND pass@k when diversity matters (RL can trade one for the other).
- Multiple comparisons: sweeping 20 checkpoints × 15 evals WILL produce spurious "best" cells; hold out a confirmation suite for the final pick.
- Variance sources to control: shot sampling, prompt paraphrase (report sensitivity), sampling seed.

## 5. LLM-as-judge

- Known biases: position (A-vs-B order — always swap & average), length (verbosity wins — length-control or penalize), self-preference (judge favors own family — use a judge from a different family or an ensemble), style-over-substance, sycophancy toward confident tone.
- Design: pairwise > absolute scoring (more reliable); rubric-anchored prompts with reference answers where possible; force structured verdict then parse; temperature 0; sample judge 3× on ties.
- Calibrate: 100–300 human-labeled comparisons; report judge-human agreement (target ≥80% on clear cases) and re-calibrate whenever judge model version changes. A judge is a model artifact with a version — pin it.
- Never let the same model family be: generator of training data AND judge of the eval that gates it, without a human-anchored audit (circular inflation).

## 6. Contamination

Detection on the MODEL (complementing data-side decontamination):
- Exact/near-recall probes: prompt with question prefix, check verbatim continuation of answers/options.
- Perplexity asymmetry: benchmark items vs paraphrased items (suspicious if original ≪ paraphrase).
- N-gram overlap report from data pipeline (13-gram standard).
- Time-split checks: performance cliff on post-cutoff versions (LiveCodeBench monthly slices) is the tell.
Treat "trained-on-test" as severity-1; always report contamination methodology with results.

## 7. Agentic evals

SWE-bench-Verified, terminal/browser task suites (WebArena-class), tool-use (BFCL for function calling), multiturn agent traces.
Specifics: environment version pinning (a docker image IS part of the eval), flaky-test quarantine, timeout policy, max-steps budget reported, pass^k (all k succeed) for reliability claims vs pass@k for capability claims; cost-per-solve alongside solve-rate (a 100×-cost +2pt agent is often a regression).

## 8. Safety evals

Two-sided measurement always: harmful-request compliance rate (red suites, jailbreak batteries — static sets go stale, refresh adversarially) AND over-refusal on benign-lookalike prompts (XSTest-style). Add: prompt-injection resistance for tool-using deployments, PII-regurgitation probes, bias suites relevant to domain. Gate releases on BOTH directions with explicit thresholds.

## 9. Online & regression

- Canary set: frozen, secret, never-trained-on prompt set run on every release; alert on any item flip.
- A/B: define primary metric pre-launch (task completion, retention, thumbs ratio); guard metrics (latency, cost, refusals); sequential-testing or fixed-horizon — no peeking-and-stopping.
- Telemetry evals: sample real traffic → judge/human score weekly; drift dashboards per intent cluster.

## 10. Building custom evals

1. Collect 50–200 REAL failure/success cases from actual usage (not imagined).
2. Write graders: prefer programmatic (exact/schema/execution) > judge-with-rubric > vibes. Test the grader itself (grade 20 known-good/bad by hand first).
3. Balance difficulty (target 30–70% baseline pass — ceilinged/floored evals give zero signal), tag by category for slice reporting.
4. Version items; quarantine any item once its exact text might have entered training; refresh 10–20%/quarter.
5. Wire into CI: threshold gates + slice-level regression alarms; every model change PR shows the eval diff.