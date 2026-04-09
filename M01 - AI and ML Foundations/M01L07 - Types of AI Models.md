# M01L07 - Types of AI Models

## What's Running on the Hardware You Support

Not all AI models are the same. Different types have different hardware requirements, different failure modes, and show up differently in logs. Here are the main categories you'll encounter.

## Large Language Models (LLMs)

**What they do:** Generate and understand text. ChatGPT, Llama, Mistral, Phi — these are all LLMs.

**What they look like in the wild:**
- Model sizes: 7B to 1T+ parameters
- Usually transformer architecture
- Served via an inference server (vLLM, TensorRT-LLM, Triton)
- APIs that accept text input, return text output

**Hardware signature:**
- High memory usage (model weights loaded into GPU)
- Compute-heavy during generation, memory-bandwidth-bound during prompt processing
- Often split across multiple GPUs (tensor parallelism)

**Log clues:**
```
Loading model weights... 100%
Started inference server on port 8000
Received request: 128 tokens
Generated 512 tokens in 2.3s
```

**Examples:** GPT-4, Llama 3.1, Mistral, Phi, Qwen, Gemma

## Computer Vision Models

**What they do:** Analyze images and video — classification, object detection, segmentation.

**What they look like:**
- Smaller than LLMs typically (100M to 10B parameters)
- CNN-based or Vision Transformer (ViT)
- Process images (224x224, 512x512, 1024x1024 pixels)

**Hardware signature:**
- Lower memory than LLMs (models are smaller)
- Very compute-intensive (lots of convolutions)
- Often single-GPU workloads

**Examples:** ResNet, YOLO, Stable Diffusion (generative), SAM (Segment Anything)

## Diffusion Models (Image Generation)

**What they do:** Generate images from text descriptions. DALL-E, Midjourney, Stable Diffusion.

**How they work:** Start with random noise, gradually denoise it into a coherent image over many steps (20-50 iterations).

**Hardware signature:**
- Each step is a full forward pass through the model
- 20-50 passes per image → very compute-intensive
- Memory depends on image resolution and model size
- Often single-GPU but heavy compute

**Examples:** Stable Diffusion, DALL-E, Flux

## Speech Models

**Two main types:**
- **ASR (Automatic Speech Recognition):** Speech → Text (Whisper)
- **TTS (Text-to-Speech):** Text → Speech

**Examples:** Whisper (ASR), ElevenLabs, Bark (TTS)

## Embedding Models

**What they do:** Convert text into numerical vectors (lists of numbers) that capture meaning. Used for search, similarity, and retrieval.

**These are small and fast** — typically 100M-1B parameters. Often CPU-only workloads. You won't see these stressing hardware much.

**Examples:** BERT, sentence-transformers, NV-Embed

## Multimodal Models

**What they do:** Process multiple types of input — text + images, text + audio, etc.

**These are increasingly common.** A model that can look at a photo and answer questions about it. A model that can process video and generate a summary.

**Examples:** GPT-4V, LLaVA, Gemini

## Retrieval-Augmented Generation (RAG)

Not a model type, but a pattern you'll see constantly:

```
User asks question
   ↓
System searches a database for relevant documents
   ↓
Relevant documents + question sent to LLM
   ↓
LLM generates answer based on retrieved context
```

RAG systems combine:
- An embedding model (to search documents)
- A vector database (to store document embeddings)
- An LLM (to generate answers)

**For your job:** When a customer says "we have a RAG pipeline," they're running at least two models — an embedding model and an LLM — plus a database. Troubleshooting means checking each component.

## What Dell Customers Are Actually Running

On DGX and HGX systems, the most common workloads in 2025-2026:

| Workload | Frequency | Typical Hardware |
|----------|-----------|-----------------|
| LLM fine-tuning (training) | Very common | 4-8 GPUs, multi-node |
| LLM inference/serving | Very common | 2-8 GPUs per instance |
| Stable Diffusion / image gen | Common | 1-4 GPUs |
| Custom model training | Common | Full DGX nodes |
| RAG systems | Very common | Mixed CPU + GPU |
| Speech processing | Moderate | 1-2 GPUs |

---

## Exercises

- [ ] For each log snippet below, identify what type of model is likely running:
  - A: `Processing image 512x512, step 15/50, loss=0.023`
  - B: `Token 512/1024 generated, TTFT=0.3s`
  - C: `Embedding 768-dimensional vector for document chunk 47`
- [ ] Why are LLMs more likely to need multiple GPUs than embedding models?
- [ ] What is RAG and why does it involve more than just an LLM?

## Review Questions

1. Name three types of AI models and what they do.
2. Which model type is most likely to cause NCCL errors? Why?
3. What's the difference between an LLM and an embedding model in terms of hardware requirements?
4. A customer says "our image generation pipeline is slow." What model type are they likely running?
5. What makes diffusion models compute-intensive?

---

*Previous: [[M01L06 - Parameters, Tokens, and Context]] | Next: [[M01L08 - The AI Software Stack Overview]]*
