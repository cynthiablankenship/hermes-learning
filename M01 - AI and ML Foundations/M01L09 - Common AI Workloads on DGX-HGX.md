# M01L09 - Common AI Workloads on DGX-HGX

## What Customers Are Actually Doing

Understanding what workloads run on DGX/HGX systems helps you anticipate problems, ask better questions, and know what "normal" looks like.

## Workload 1: LLM Fine-Tuning

**What it is:** Taking a pre-trained model (like Llama 3.1) and training it further on domain-specific data to make it better at a particular task.

**Example:** A healthcare company fine-tunes Llama on medical records so it can answer clinical questions accurately.

**What you'd see on the hardware:**
- Sustained high GPU utilization (90-100%)
- Heavy NCCL traffic between GPUs
- Model checkpoints being saved periodically (disk I/O spikes)
- Training runs lasting hours to days
- Loss values decreasing over time in logs

**Common issues:**
- NCCL timeout (GPU communication failure)
- CUDA OOM (model + training data too large for GPU memory)
- Checkpoint save failures (disk full)
- GPU memory ECC errors (hardware fault during long runs)

## Workload 2: LLM Inference Serving

**What it is:** Running a trained model as a service that accepts requests and returns responses. Like running your own ChatGPT.

**What you'd see:**
- GPU memory mostly full (model weights loaded), but compute is bursty
- Network traffic from incoming API requests
- Metrics like tokens/second, latency, throughput
- Potentially running 24/7 as a production service

**Common issues:**
- Request timeouts under high load
- OOM when batch size or context length is too large
- Model loading failures (corrupt files, wrong format)
- Latency spikes (KV cache fragmentation, memory management)

## Workload 3: Pre-Training

**What it is:** Training a model from scratch on massive data. This is what OpenAI, Meta, Google do to create foundation models. Very few customers do this — it's enormously expensive.

**What you'd see:**
- Every GPU in the cluster at 100% utilization, 24/7, for weeks
- Massive multi-node communication (hundreds of GPUs)
- Enormous data pipeline (reading TB of training data from storage)
- Power draw near maximum on every node

**Common issues:**
- Any single GPU failure crashes the entire training run
- Network bottlenecks between nodes
- Storage throughput can't keep up with data demand
- Checkpoint recovery after failures (need to roll back to last good state)

## Workload 4: RAG Pipelines

**What it is:** A system that combines document search with LLM generation. The model answers questions based on retrieved documents.

**Architecture:**
```
User question → Embedding model → Vector DB search → Top documents + question → LLM → Answer
```

**What you'd see:**
- Multiple services running (embedding model, vector DB, LLM server)
- Mixed CPU and GPU workloads
- Database I/O (reading document embeddings)
- Lighter GPU usage than pure LLM serving (embedding models are small)

## Workload 5: Image/Video Generation

**What it is:** Running Stable Diffusion, Flux, or similar models to generate or modify images and video.

**What you'd see:**
- Intermittent GPU bursts (one image generated in seconds)
- Moderate memory usage (models are smaller than LLMs)
- Often single-GPU workloads
- Storage I/O for saving generated images

## Workload 6: Speech Processing

**What it is:** Transcription (speech-to-text) or synthesis (text-to-speech).

**What you'd see:**
- Real-time or batch processing
- Moderate GPU usage
- Often runs alongside other workloads

## What to Ask a Customer

When you get a ticket and need to understand their workload:

1. **What type of model are you running?** (LLM, image, custom)
2. **What size is the model?** (7B, 70B, etc.)
3. **Is this training or inference?**
4. **How many GPUs are you using?**
5. **Is this a single node or multi-node?**
6. **What framework are you using?** (PyTorch, TensorFlow)
7. **What inference/training server?** (vLLM, TensorRT, custom)
8. **When did the issue start?** (After a change? Random? Intermittent?)

These questions narrow down the problem space dramatically.

---

## Exercises

- [ ] Match each scenario to a workload type:
  - A: "We're training Llama 3.1 on our internal documents"
  - B: "Our chatbot API is returning 503 errors under load"
  - C: "Training run crashed after 3 days, all 256 GPUs"
  - D: "Generated images have artifacts after driver update"
- [ ] Which workload type would most commonly show NCCL timeout errors? Why?
- [ ] Why does pre-training require so much more hardware than fine-tuning?

## Review Questions

1. What's the difference between pre-training and fine-tuning?
2. Which workload would show sustained 100% GPU utilization for weeks?
3. What makes RAG pipelines architecturally different from simple LLM serving?
4. What are three questions you should ask a customer to understand their workload?
5. Why does a single GPU failure crash a pre-training run but only drop individual inference requests?

---

*Previous: [[M01L08 - The AI Software Stack Overview]] | Next: [[M01L10 - Lab - Reading AI Logs]]*
