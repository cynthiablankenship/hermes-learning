# M01L04 - Neural Networks Conceptually

## No Math. Promise.

You don't need equations to understand what a neural network is doing. Here's the mental model.

## The Basic Idea

A neural network is a system that transforms input into output through a series of processing steps called **layers**.

```
Input → [Layer 1] → [Layer 2] → [Layer 3] → ... → [Layer N] → Output
```

Each layer takes what the previous layer gave it, transforms it in some way, and passes it to the next layer.

## What's a Layer?

Think of a layer as a filter or a transformation step. Each layer looks for different patterns:

- **Early layers** find simple patterns (edges in images, basic word relationships in text)
- **Middle layers** combine simple patterns into complex ones (shapes, phrases)
- **Later layers** recognize high-level concepts (objects, meaning, intent)

In an image recognition network:
- Layer 1: "I see lines and edges"
- Layer 5: "I see circles and rectangles"
- Layer 20: "I see a face"
- Layer 50: "I see a specific person"

## What's Inside a Layer?

Each layer has a set of **weights** — numbers that determine what patterns it looks for. These weights start random and get adjusted during training.

Think of each weight as a dial that can be turned:
- Before training: all dials are random → garbage output
- During training: the dials are gradually adjusted → output gets better
- After training: dials are frozen → consistent, useful output

The "70 billion parameters" in Llama 3.1 70B? Those are 70 billion individual dials across all the layers.

## How Training Works (Conceptually)

1. Feed the network an example (e.g., a picture of a cat)
2. The network makes a prediction ("I think this is a dog" — wrong)
3. Compare to the correct answer ("cat")
4. Calculate how wrong it was (the **loss**)
5. Adjust all the weights slightly to be less wrong next time (**backpropagation**)
6. Repeat trillions of times

The adjustment process is called **gradient descent** — imagine you're blindfolded on a mountain and trying to find the valley (lowest error). You feel which direction is downhill and take a step. Repeat until you're at the bottom.

## Common Neural Network Architectures

You don't need to know the details of these, just recognize the names:

| Architecture | What It's For | Key Idea |
|-------------|---------------|----------|
| **CNN** (Convolutional Neural Network) | Images, video | Scans for patterns spatially (like looking through a magnifying glass) |
| **RNN** (Recurrent Neural Network) | Sequences (text, time series) | Remembers previous inputs to understand context |
| **Transformer** | Text, multi-modal | The architecture behind all modern LLMs (ChatGPT, Llama, etc.) |
| **GAN** (Generative Adversarial Network) | Generating images | Two networks competing: one creates, one judges |
| **Diffusion** | Generating images (DALL-E, Stable Diffusion) | Starts with noise, gradually refines into an image |

**Transformers are the most important one for your job.** Almost everything running on DGX systems today is transformer-based.

## The Transformer — What Makes It Special

The transformer architecture (introduced in the "Attention Is All You Need" paper in 2017) revolutionized AI. The key innovation is called **self-attention**:

- Instead of processing words one at a time (like RNNs), transformers look at ALL words simultaneously
- Each word "pays attention" to relevant other words in the input
- This allows much better understanding of context and relationships
- It also parallelizes extremely well on GPUs — which is why GPU demand exploded

When people talk about "attention heads" in model specs, they're talking about how many parallel attention calculations the model does.

## Model Size — What the Numbers Mean

When you see model specs like "Llama 3.1 8B" vs "Llama 3.1 70B" vs "Llama 3.1 405B":

| Size | Parameters | Model File Size (16-bit) | Min GPU Memory | Typical Use |
|------|-----------|------------------------|---------------|-------------|
| 8B | 8 billion | ~16 GB | 1 GPU (24GB+) | Edge, local, development |
| 70B | 70 billion | ~140 GB | 2-4 GPUs (80GB each) | Production inference |
| 405B | 405 billion | ~810 GB | 8+ GPUs (80GB each) | High-quality production |
| 1T+ | 1 trillion+ | ~2 TB | Multiple DGX nodes | Frontier models (GPT-4 class) |

**For your job:** When a customer says they're running a 70B model, that's a medium-sized model. It needs at least 2 H100s for inference, more for training. If they say 405B, that's a big model requiring serious hardware.

---

## Exercises

- [ ] Explain the concept of neural network layers to someone (or to Hermes in chat) without using any math
- [ ] Why do you need more GPUs for a 405B model than a 70B model?
- [ ] Look up "Attention Is All You Need" — what year was it published and why was it important?
- [ ] If a customer tells you they're running a transformer model, what type of workload is it most likely?

## Review Questions

1. What is a "layer" in a neural network? What does it do?
2. What are "weights" and what happens to them during training?
3. What's the key innovation of the transformer architecture?
4. A 70B parameter model — roughly how large is the file on disk?
5. Why are transformers better suited to GPUs than older architectures?

---

*Previous: [[M01L03 - Training vs Inference]] | Next: [[M01L05 - Why GPUs]]*
