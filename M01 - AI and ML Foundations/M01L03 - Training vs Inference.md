# M01L03 - Training vs Inference

## The Two Things GPUs Do in AI

Every AI workload falls into one of two categories:

1. **Training** — Teaching a model by processing massive amounts of data
2. **Inference** — Using a trained model to make predictions

Understanding the difference is critical for your job because they stress the hardware differently, fail differently, and show up differently in logs.

## Training — Building the Model

**What happens:** The model looks at billions of examples and adjusts its internal numbers (weights) to get better at its task. Each adjustment is tiny, but after trillions of adjustments across weeks of computation, the model becomes capable.

**What it looks like on the hardware:**
- Sustained 90-100% GPU utilization for hours, days, or weeks
- GPU memory almost entirely full (the model + training data + intermediate calculations)
- Heavy GPU-to-GPU communication (NCCL traffic) because training is distributed across multiple GPUs
- Lots of disk I/O at the start (loading training data) and periodic checkpoints (saving model state)
- Network traffic between nodes if using multi-node training

**Key terms you'll see in training logs:**
- `epoch` — One complete pass through the training dataset
- `step` or `iteration` — Processing one batch of data
- `loss` — A number measuring how wrong the model is (going down = learning)
- `checkpoint` — A saved snapshot of the model's current state
- `gradient` — The direction to adjust weights to reduce errors
- `learning rate` — How big of an adjustment to make each step

## Inference — Using the Model

**What happens:** A trained model receives input (text, image, audio) and produces output. No learning happens. The model's weights are frozen.

**What it looks like on the hardware:**
- Bursty GPU utilization (spikes when a request comes in, idle between requests)
- GPU memory holds the model weights but less temporary data than training
- Much less GPU-to-GPU communication (unless the model is too big for one GPU)
- Network traffic from API requests coming in and responses going out
- Lower overall power draw than training

**Key terms you'll see in inference logs:**
- `request` or `prompt` — Input sent to the model
- `token` — A piece of text (roughly 3/4 of a word in English)
- `latency` — How long it takes to produce a response
- `throughput` — How many requests per second the system handles
- `batch` — Processing multiple requests together for efficiency
- `TTFT` — Time To First Token (how quickly the first word appears)

## Side by Side Comparison

| Aspect | Training | Inference |
|--------|----------|-----------|
| Purpose | Build/teach the model | Use the model |
| GPU utilization | Sustained 95-100% | Bursty, varies with load |
| GPU memory | Nearly full, constant | Holds model, varies with batch size |
| Duration | Days to weeks per run | Continuous service (weeks/months) |
| GPU-to-GPU traffic | Very heavy (NCCL) | Light (unless model is sharded) |
| Failure impact | Lose progress since last checkpoint | Drop individual requests |
| What customer cares about | Loss going down, training finishing on time | Latency, throughput, availability |
| Common errors | NCCL timeout, OOM, checkpoint corruption | Request timeouts, model loading failures |

## How This Shows Up in Logs

### Training log example:
```
[Step 1500/100000] loss=2.3456 lr=0.0001 throughput=1250 tokens/s GPU mem: 78.2/80.0 GB
[Step 1501/100000] loss=2.3421 lr=0.0001 throughput=1248 tokens/s GPU mem: 78.2/80.0 GB
NCCL error: timeout in allReduce
```
→ This is training. Loss is being tracked. NCCL error means GPUs couldn't communicate.

### Inference log example:
```
INFO: Received request (128 tokens)
INFO: Generated 512 tokens in 2.3s (TTFT: 0.4s)
INFO: Throughput: 450 requests/min, avg latency: 1.8s
ERROR: CUDA out of memory. Failed to allocate 2.1 GB
```
→ This is inference. Tracking latency and throughput. OOM means too many simultaneous requests.

## Why This Matters for Troubleshooting

- **If training crashes**, the customer lost time. They need to know if the hardware caused it (GPU failure, NVLink down, memory error) or the software (bad config, data issue). You check dmesg, nvidia-smi, NCCL logs.
- **If inference is slow**, the customer's users are waiting. You check GPU utilization, memory fragmentation, batch sizes, network latency.
- **Different failure modes require different fixes.** A training NCCL timeout might be a network issue. An inference OOM might be a configuration issue. Knowing which workload type you're dealing with narrows the problem space dramatically.

---

## Exercises

- [ ] Look at these two log snippets and identify which is training and which is inference:
  - Snippet A: `Generated 256 tokens in 1.2s | Throughput: 320 req/s | P99 latency: 2.1s`
  - Snippet B: `Epoch 3/10 Step 45000/150000 loss=1.234 lr=3e-5 grad_norm=0.45`
- [ ] Explain in your own words why training uses more GPU memory than inference
- [ ] Why does training cause heavy GPU-to-GPU traffic but inference typically doesn't?

## Review Questions

1. What's the fundamental difference between training and inference?
2. What does "loss" mean in a training log? Is a lower number good or bad?
3. What does "TTFT" stand for and what workload type cares about it?
4. If you see "NCCL timeout" in logs, is this more likely training or inference? Why?
5. A customer says their model is "serving requests slowly." Is this a training problem or an inference problem?

---

*Previous: [[M01L02 - What Is a Model]] | Next: [[M01L04 - Neural Networks Conceptually]]*
