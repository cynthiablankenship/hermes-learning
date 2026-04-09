# M01L02 - What Is a Model?

## The Most Important Concept in AI

If you understand what a model is, everything else clicks. So let's be precise.

## A Model Is a File (Or a Set of Files)

When someone says "we're running the Llama 3.1 70B model," what they mean is:
- There is a file (or collection of files) on disk, usually several gigabytes
- That file contains billions of numbers
- A program loads those numbers into GPU memory
- The program then uses those numbers to process input and produce output

The model file is the *learned knowledge*. The program that loads and runs it is separate.

Analogy:
- **The model** is like a textbook — it contains information
- **The inference engine** (like vLLM or TensorRT) is like a student reading the textbook and answering questions
- **The GPU** is like the desk the student sits at

## What's Inside a Model File

Numbers. Lots of them. These numbers are called **weights** (or **parameters**).

When you hear "Llama 3.1 has 70 billion parameters," it means the file contains roughly 70 billion numbers. Each number was adjusted during training to capture some pattern in the training data.

That's why model files are big:
- A 7B parameter model in 16-bit precision ≈ 14 GB on disk
- A 70B parameter model in 16-bit precision ≈ 140 GB on disk
- A 405B parameter model ≈ 810 GB on disk

**This is directly relevant to your job:** If a customer has a DGX with 8 H100 GPUs (80GB each = 640GB total GPU memory), they can run a 70B model but NOT a 405B model (without techniques we'll cover later).

## Model File Formats

Models come in different file formats. You'll see these in logs and on disk:

| Format | Extension | Used By |
|--------|-----------|---------|
| SafeTensors | `.safetensors` | Hugging Face, PyTorch — the modern standard |
| GGUF | `.gguf` | llama.cpp — for running on CPUs/consumer GPUs |
| ONNX | `.onnx` | Cross-framework interchange format |
| Pickle/PyTorch | `.pt`, `.pth`, `.bin` | PyTorch native (older, less secure) |
| TensorRT Engine | `.engine`, `.plan` | Optimized for NVIDIA GPUs |
| HDF5 | `.h5` | TensorFlow/Keras |

When you see a log mentioning a `.safetensors` file, it's loading model weights. When you see `.engine`, it's a model that's been pre-compiled for NVIDIA GPUs.

## Model Architecture vs Model Weights

Two different things:

**Architecture** = the shape/structure of the model (how many layers, what type of layers, how they connect). Think of this as the blueprint of a building.

**Weights** = the actual numbers inside the model (the learned knowledge). Think of this as the building itself, built from that blueprint.

Same architecture can have different weights:
- Meta's Llama 3.1 70B and someone's fine-tuned version of it share the same architecture
- But the weights are different — one is general-purpose, one is specialized

## Where Models Live on a DGX

```bash
# Common locations for model files
/data/models/                    # Often where customers store models
/opt/nvidia/models/              # NVIDIA-provided models
/home/username/.cache/huggingface/  # Hugging Face cache (auto-downloaded)
/tensorrt_engines/               # Pre-compiled TensorRT engines
```

When troubleshooting storage or memory issues, knowing where models are stored matters.

## The Lifecycle of a Model

```
1. TRAINING
   Raw data → Training process → Model weights saved to file
   (This is where GPUs crunch for days/weeks)

2. OPTIMIZATION (optional but common)
   Model file → Quantization, compilation → Optimized model file
   (Make it smaller, faster, more efficient)

3. DEPLOYMENT / INFERENCE
   Model file loaded into GPU memory → Serves predictions
   (This is what most production systems do)
```

For your job:
- **Training** shows up as sustained high GPU utilization, lots of GPU-to-GPU communication, large memory usage
- **Inference** shows up as bursts of GPU activity, lower sustained memory, network traffic from API calls

---

## Exercises

- [ ] Check the size of a model file. If you have access to a DGX, look in `/data/models/` or `~/.cache/huggingface/`
- [ ] Search for model files: `find / -name "*.safetensors" -o -name "*.gguf" -o -name "*.engine" 2>/dev/null | head -20`
- [ ] Calculate: a model with 13 billion parameters in 16-bit (2 bytes per parameter) — how large is the file?
- [ ] Explain to yourself (or Hermes) the difference between model architecture and model weights

## Review Questions

1. What is a model file, physically?
2. Why does a 70B parameter model need more GPU memory than a 7B model?
3. What's the difference between a `.safetensors` file and a `.engine` file?
4. What are the three stages of a model's lifecycle?
5. If a customer has 4 GPUs with 80GB memory each (320GB total) and wants to run a 405B model in 16-bit, will it fit?

---

*Previous: [[M01L01 - What Is AI]] | Next: [[M01L03 - Training vs Inference]]*
