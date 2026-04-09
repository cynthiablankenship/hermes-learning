# M01L10 - Lab: Reading AI Logs

## Lab Overview

This lab is all about pattern recognition — reading log entries and understanding what's happening. These are based on real patterns you'll encounter on the job.

---

## Part 1: Identify the Workload Type

Read each log snippet and identify: **Is this training, inference, or something else?**

### Snippet 1
```
[2026-04-09 10:15:32] INFO: Loading model weights from /data/models/llama-3.1-70b/safetensors
[2026-04-09 10:15:45] INFO: Model loaded successfully. 72.3 GB in GPU memory.
[2026-04-09 10:15:46] INFO: Starting vLLM server on 0.0.0.0:8000
[2026-04-09 10:16:02] INFO: Received request: 256 tokens
[2026-04-09 10:16:04] INFO: Generated 512 tokens in 1.8s (TTFT: 0.3s)
```

- [ ] Workload type: _________
- [ ] What clues gave it away?

### Snippet 2
```
[2026-04-09 10:15:32] Epoch 2/5 | Step 15234/100000 | loss=1.234 | lr=3e-5
[2026-04-09 10:15:33] Epoch 2/5 | Step 15235/100000 | loss=1.231 | lr=3e-5
[2026-04-09 10:15:34] Epoch 2/5 | Step 15236/100000 | loss=1.228 | lr=3e-5
[2026-04-09 10:15:35] NCCL error: Call to ncclAllReduce failed: unhandled system error
[2026-04-09 10:15:35] Training run interrupted. Last checkpoint: step 15000
```

- [ ] Workload type: _________
- [ ] What went wrong?
- [ ] What would you check first?

### Snippet 3
```
[2026-04-09 10:15:32] INFO: Processing image batch (32 images, 512x512)
[2026-04-09 10:15:34] INFO: Step 12/50 | noise_level=0.23
[2026-04-09 10:15:36] INFO: Step 13/50 | noise_level=0.19
[2026-04-09 10:15:38] INFO: Step 14/50 | noise_level=0.15
[2026-04-09 10:16:12] INFO: Generated 32 images in 40.2s
```

- [ ] Workload type: _________
- [ ] What model type is this?

### Snippet 4
```
[2026-04-09 10:15:32] INFO: Embedding query: "What is the warranty coverage for DGX H100?"
[2026-04-09 10:15:32] INFO: Query embedding: 768-dimensional vector
[2026-04-09 10:15:32] INFO: Searching vector DB... found 5 relevant documents
[2026-04-09 10:15:33] INFO: Sending context + query to LLM (context: 2048 tokens)
[2026-04-09 10:15:35] INFO: LLM response generated: 256 tokens in 1.2s
```

- [ ] Workload type: _________
- [ ] What system architecture is this?

---

## Part 2: Diagnose the Problem

Read each scenario and think about what you'd investigate.

### Scenario 1
```
$ nvidia-smi
+------------------------------------------------------+
| GPU  Name        Memory-Usage | GPU-Util  Compute M. |
|   0  H100        78234MiB/81920MiB |  98%      Default |
|   1  H100        78234MiB/81920MiB |  98%      Default |
|   2  H100        78234MiB/81920MiB |  98%      Default |
|   3  H100        78234MiB/81920MiB |  98%      Default |
+------------------------------------------------------+
```

Customer says: "Training is running but very slow."

- [ ] What's the memory situation?
- [ ] What's the compute situation?
- [ ] Is the problem likely GPU-related? What else could it be?

### Scenario 2
```
$ nvidia-smi
| GPU  Name        Memory-Usage | GPU-Util  Compute M. |
|   0  H100            5MiB/81920MiB |   0%      Default |
|   1  H100        81915MiB/81920MiB |  95%      Default |
|   2  H100            5MiB/81920MiB |   0%      Default |
|   3  H100            5MiB/81920MiB |   0%      Default |
```

Customer says: "Only GPU 1 is being used."

- [ ] Why might only one GPU be in use?
- [ ] Is this necessarily a problem?
- [ ] What questions would you ask the customer?

### Scenario 3
```
[2026-04-09 10:15:35] ERROR: CUDA error: out of memory
[2026-04-09 10:15:35] ERROR: Tried to allocate 2.50 GiB. GPU 0 has 45.00 MiB free
[2026-04-09 10:15:35] WARNING: KV cache full, evicting sequences
```

- [ ] What workload type is this? (training or inference?)
- [ ] What does "KV cache full" mean?
- [ ] What would you suggest to the customer?

### Scenario 4
```
$ dmesg | grep -i nvidia
[  512.345678] NVRM: Xid (PCI:0000:07:00): 79, pid=12345, GPU has fallen off the bus
[  512.345680] NVRM: GPU 0000:07:00.0: GPU fell off the bus
```

- [ ] What does "GPU has fallen off the bus" mean?
- [ ] Is this a software or hardware problem?
- [ ] What would you check?

### Scenario 5
```
[2026-04-09 10:15:32] nvidia-smi
|   0  H100     5MiB/81920MiB | 0% |  N/A   42C   P0   85W / 700W |
|   1  H100     5MiB/81920MiB | 0% |  N/A   41C   P0   82W / 700W |
|   2  H100     5MiB/81920MiB | 0% |  N/A   82C   P0   85W / 700W |
|   3  H100     5MiB/81920MiB | 0% |  N/A   43C   P0   84W / 700W |
```

- [ ] Anything unusual here? Which GPU?
- [ ] Why is this suspicious?
- [ ] What could cause this?

---

## Part 3: Quick Calculations

- [ ] A customer wants to run a 70B model in FP16 on H100s (80 GB each). How many GPUs minimum just to hold the model weights?
- [ ] The same model in INT4 quantization — how many GPUs?
- [ ] A training run processes 2000 tokens per step, 1000 steps per epoch, 3 epochs. Total tokens processed?
- [ ] A model generates at 50 tokens/second. How long to generate a 1000-token response?

---

## Lab Completion

- [ ] All snippets correctly identified by workload type
- [ ] All scenarios analyzed with reasonable diagnoses
- [ ] All calculations completed
- [ ] Can explain your reasoning for each answer

**If you're stuck on any of these, ask Hermes.** The goal is understanding, not getting every answer right on the first try.

---

*Congratulations! Module 1 complete. Next up: [[M02 - The NVIDIA AI Software Stack]]*
