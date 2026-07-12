# Infrastructure: GPUs, Clusters, Networking, Storage

## Contents
1. GPU selection & fleet math
2. Memory hierarchy & what fits where
3. Networking (intra-node & inter-node)
4. Storage architecture for training
5. Cluster software stack
6. Reliability engineering
7. Cost math & procurement
8. Common failure modes & runbook

## 1. GPU selection & fleet math

Decision drivers, in order: (1) memory capacity per GPU, (2) memory bandwidth, (3) interconnect, (4) FLOPs, (5) $/hr. FLOPs are rarely the binding constraint; memory and comms are.

Reference specs (verify current-gen via search — this table ages):
| GPU | HBM | BW | BF16 dense | Interconnect |
|---|---|---|---|---|
| A100 80GB | 80 GB | 2.0 TB/s | 312 TF | NVLink 600 GB/s |
| H100 SXM | 80 GB | 3.35 TB/s | ~990 TF | NVLink 900 GB/s |
| H200 | 141 GB | 4.8 TB/s | ~990 TF | NVLink 900 GB/s |
| B200 | 192 GB | ~8 TB/s | ~2.2 PF | NVLink5 1.8 TB/s |
| MI300X | 192 GB | 5.3 TB/s | ~1.3 PF | IF 896 GB/s |

Fleet sizing recipe:
1. Total FLOPs = 6 × N × D. Example: 7B params × 2T tokens → 8.4e22 FLOPs.
2. Effective per-GPU rate = peak × MFU (0.40 good, 0.30 conservative).
3. GPU-hours = total / rate. 8.4e22 / (990e12 × 0.4) ≈ 2.36e7 s ≈ 6,560 H100-hours per... recompute: 8.4e22/3.96e14 = 2.12e8 s = 58,900 GPU-hours ≈ 307 H100s for 8 days.
4. Add 15–20% for restarts, evals, ablations.

Rules of thumb:
- Fine-tuning ≤13B with LoRA: 1–8 GPUs. Full FT 7B: ≥8×80GB or FSDP on fewer with offload (slow).
- Pretraining <1B: single node fine. 7B-class: 128–512 GPUs typical. 70B-class: 1k–4k GPUs.
- Consumer GPUs (4090/5090): fine for <3B experiments and inference; no NVLink, 24–32GB, PCIe-bound for multi-GPU training.

## 2. Memory hierarchy

Per-GPU consumers during training: weights, grads, optimizer states, activations, comm buffers, fragmentation (~5–10%).
- Mixed-precision Adam full replication: 16 B/param → 7B model = 112 GB (doesn't fit one 80GB GPU → shard).
- Activations ≈ layers × batch × seq × hidden × ~34 bytes (no recompute, BF16, standard transformer). Activation checkpointing cuts to ~2×√ or layer-boundary-only at ~30% recompute cost. Selective (attention-only) recompute is usually the sweet spot.
- 8-bit optimizers (bitsandbytes) cut optimizer states 8B→2B/param. Adafactor cheaper still but touchier.

CPU RAM: ≥2× GPU-HBM-total per node for offload headroom and dataloader workers. Pin memory; use `num_workers` ≈ 4–8/GPU.

## 3. Networking

Intra-node: NVLink/NVSwitch (H100: 900 GB/s bidirectional/GPU) — TP lives here. Never span tensor parallel across nodes on Ethernet.
Inter-node: InfiniBand NDR 400 Gb/s (=50 GB/s) per HCA, typically 8 rails/node on H100 clusters (8×400Gb). RoCEv2 works but needs PFC/ECN tuning; expect more debugging.
- Rule: DP gradient all-reduce moves 2×params bytes per step (ring). For 7B BF16: 28 GB per step across the DP group — overlap with backward or it dominates.
- NCCL env you'll actually touch: `NCCL_IB_HCA`, `NCCL_SOCKET_IFNAME`, `NCCL_DEBUG=INFO` (first debugging step), `NCCL_ALGO`, `NCCL_MIN_CTAS`. Test fabric with `nccl-tests` (all_reduce_perf) BEFORE any training: expect ≥ 0.8× theoretical bus BW.
- GPUDirect RDMA + GDRCopy: verify enabled; 2–3× penalty if traffic bounces through host.

## 4. Storage

Three tiers:
1. **Hot (checkpoints, active shards):** parallel FS (Lustre, GPFS, Weka, VAST) or local NVMe RAID0. Checkpoint write must not stall training: 70B model optimizer ckpt ≈ 1.1 TB (16B/param); at 10 GB/s that's ~2 min — use async/distributed checkpointing (torch.distributed.checkpoint, or DeepSpeed) so each rank writes its shard.
2. **Warm (full corpus, tokenized):** object store (S3/GCS/MinIO). Stream with WebDataset/Mosaic StreamingDataset — random access to shuffled shards, resumable, avoids full local copies.
3. **Cold (raw crawls):** cheapest object/archive tier.

Formats: tokenized fixed-length bins (Megatron .bin/.idx), WebDataset tar shards (~1GB each), or MDS. Avoid millions of small files — metadata ops kill parallel FS.
Throughput target: dataloader must sustain tokens/step × step-rate with 2× headroom; test with a no-model dataloader-only loop.

## 5. Cluster software stack

- Scheduler: **Slurm** (HPC standard; gang scheduling, `srun --gpus-per-node=8`) or **Kubernetes** + Kueue/Volcano (cloud-native; use the NVIDIA GPU Operator). Ray on top for RL/serving hybrids.
- Containers: base on NGC PyTorch images (`nvcr.io/nvidia/pytorch:xx.xx`); pin CUDA/driver/NCCL matrix. enroot/pyxis for Slurm, standard runtime for K8s.
- Env pinning: driver ↔ CUDA toolkit ↔ PyTorch ↔ NCCL ↔ flash-attn versions form a compatibility matrix — record all five in every experiment log.
- Observability: DCGM exporter → Prometheus → Grafana. Watch: GPU util, SM occupancy, HBM used, NVLink/IB throughput, ECC errors, power/thermals. Log per-step: loss, grad-norm, LR, tokens/s, MFU, dataloader wait %.

## 6. Reliability engineering

At scale, hardware fails constantly: plan for ~one GPU/HBM/link failure per few hundred GPU-days.
- Checkpoint interval: balance re-compute loss vs write cost. Optimal ≈ sqrt(2 × MTBF × ckpt_write_time); practically every 30–60 min at 1k-GPU scale, every few hours below 100 GPUs.
- Auto-resume: training script must be idempotent-resumable (data position, RNG states, LR scheduler, optimizer) — test resume-equivalence explicitly (loss curve must continue seamlessly).
- Straggler detection: per-rank step-time logging; one slow GPU (thermal throttle, ECC retirement) drags the whole synchronous job.
- Health checks pre-job: `dcgmi diag -r 3`, nccl-tests, disk BW test. Cordon nodes failing ECC or thermal thresholds.

## 7. Cost math

State assumptions. Ballparks (verify current): H100 cloud $2–3.5/hr reserved–on-demand; A100 $1–2/hr; spot 40–70% off but preemptible → mandatory frequent checkpointing.
- Example: 7B/2T-token pretrain ≈ 59k H100-hr × $2.5 ≈ **$150k** compute alone.
- LoRA fine-tune 8B, 10M tokens: <$50. Full-FT 8B, 1B tokens: 6×8e9×1e9=4.8e19 FLOPs ≈ 34 H100-hr ≈ $100.
- Buy vs rent: break-even ≈ 12–18 months of >60% utilization; ownership adds power (~700W/GPU + cooling ≈ 1.3× PUE), datacenter, and staff.

## 8. Failure runbook

| Symptom | First checks |
|---|---|
| NCCL timeout/hang | `NCCL_DEBUG=INFO`; check a dead rank (OOM on one GPU), firewall on IB ports, mismatched NCCL versions |
| Random CUDA errors, Xid in dmesg | `nvidia-smi -q | grep -i ecc`; Xid 63/64=ECC page retirement, 79=fell off bus → cordon node |
| Slow step time suddenly | thermal throttle (`nvidia-smi dmon`), a straggler rank, storage stall (dataloader wait %) |
| OOM at step N (not step 0) | fragmentation → `PYTORCH_CUDA_ALLOC_CONF=expandable_segments:True`; activation growth from seq-len bucketing |
| Loss NaN after resume | RNG/optimizer state not restored; data position reset causing repeated hot shard |