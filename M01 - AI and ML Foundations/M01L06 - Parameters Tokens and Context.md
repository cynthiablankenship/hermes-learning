# M01L06 - Parameters, Tokens, and Context

## The Three Concepts Behind Every AI Workload

When you read logs, specs, or customer tickets, these three terms come up constantly. Understanding them tells you a lot about what's happening on the hardware.

## Parameters — The Size of the Model

A **parameter** is a single number inside the model. Each parameter is a weight that was adjusted during training.

When someone says "Llama 3.1 70B":
- 70B = 70 billion parameters
- Each parameter is a number (typically stored as a 16-bit float = 2 bytes)
- 70 billion × 2 bytes = ~140 GB just to store the model in memory

**Parameter count tells you:**
- How much GPU memory is needed
- How much computation is needed per forward pass
- Roughly how "capable" the model is (more parameters = more capacity to learn, generally)

**The trend:** Models are getting bigger. Early GPT models were ~100M parameters. GPT-3 was 175B. Modern frontier models are estimated at 1-2 trillion parameters.

## Tokens — The Unit of AI Text Processing

Models don't read text character by character or word by word. They read **tokens**.

A token is a piece of text — typically a common word, part of a word, or a punctuation mark:

```
"Hello, how are you today?"
```
Tokens: `["Hello", ",", " how", " are", " you", " today", "?"]` — about 7 tokens.

**Rules of thumb:**
- 1 token ≈ 3/4 of an English word
- 100 tokens ≈ 75 words ≈ a short paragraph
- A typical page of text ≈ 500-1,000 tokens
- This entire lesson ≈ 3,000-4,000 tokens

**Why tokens matter for your job:**
- **Throughput is measured in tokens/second.** Training logs say "1250 tokens/s." Inference logs say "generated 256 tokens in 1.2s."
- **Memory usage scales with tokens.** More tokens in a request = more GPU memory needed to process it.
- **Pricing is per token.** API services charge $X per million tokens.
- **Context windows are measured in tokens.** A model with "128K context" can handle about 128,000 tokens of input + output.

## Context Window — How Much the Model Can "Remember"

The **context window** (or context length) is the maximum number of tokens a model can process in a single request.

Think of it as the model's working memory:
- Input text uses tokens from the context window
- The model's response also uses tokens from the context window
- Once the context window is full, the model can't take more input

| Context Size | What It Fits |
|-------------|-------------|
| 4K tokens | A few pages of text |
| 32K tokens | A long article or short chapter |
| 128K tokens | A short book (~300 pages) |
| 1M tokens | A very long book or many documents |

**Why this matters for hardware:**
- Attention computation scales quadratically with context length — processing 128K tokens takes ~1,000x more compute than 4K tokens
- Memory for attention scales linearly with context length
- A customer running long-context models will show very different GPU usage patterns than short-context

## Quantization — Making Models Smaller

If a 70B model needs 140 GB in 16-bit, how do people run it on less hardware? **Quantization** — reducing the precision of the numbers.

| Precision | Bits per Parameter | 70B Model Size | Quality Impact |
|-----------|-------------------|----------------|---------------|
| FP32 (32-bit float) | 4 bytes | 280 GB | Full precision (training default) |
| FP16 (16-bit float) | 2 bytes | 140 GB | Negligible loss (standard for inference) |
| INT8 (8-bit integer) | 1 byte | 70 GB | Small loss, widely used |
| INT4 (4-bit integer) | 0.5 bytes | 35 GB | Noticeable loss, but runs on much less hardware |

**For your job:** When a customer says they're running "Llama 3.1 70B in 4-bit quantization," the model only needs ~35 GB instead of ~140 GB. That's the difference between needing 2 H100s and fitting on 1.

## Batch Size — Processing Multiple Requests at Once

**Batch size** = how many inputs the GPU processes simultaneously.

- Batch size 1: Process one request at a time (low latency, low throughput, wastes GPU capacity)
- Batch size 32: Process 32 requests at once (higher latency per request, but much higher throughput)

**For your job:**
- Training: Large batch sizes (64, 256, 1024+) → more efficient training
- Inference: Batch size is tuned for the right balance of latency vs throughput
- If GPU memory is low → can't fit large batches → lower throughput or OOM errors

---

## Exercises

- [ ] Calculate: A 13B parameter model in FP16. How many GB?
- [ ] Calculate: A 70B model in INT4. How many GB?
- [ ] Estimate: This sentence "The quick brown fox jumps over the lazy dog" — roughly how many tokens?
- [ ] If a model has a 32K context window, roughly how many pages of text can it process at once?
- [ ] Why does a longer context window need more GPU memory?

## Review Questions

1. What is a "parameter" in a model? What does the number tell you?
2. What is a token? Roughly how many tokens are in 1,000 words?
3. What does "128K context window" mean for the model and for the hardware?
4. What is quantization and why would someone use it?
5. A customer is running a 70B model in FP16 on a single H100 (80 GB). Will it fit? What could they do to make it fit?

---

*Previous: [[M01L05 - Why GPUs]] | Next: [[M01L07 - Types of AI Models]]*
