# 📖 Glossary

Common terms you'll encounter throughout this curriculum.

## A
- **API** — Application Programming Interface. A way for programs to talk to each other.
- **ASR** — Automatic Speech Recognition. Speech to text.
- **Attention** — The mechanism in transformers that lets each token "look at" other relevant tokens. The key innovation behind modern LLMs.

## B
- **Backpropagation** — The algorithm used to adjust model weights during training. Calculates how much each weight contributed to the error.
- **Batch Size** — Number of inputs processed simultaneously. Larger = more efficient but uses more memory.
- **BMC** — Baseboard Management Controller. A mini-computer on the motherboard for remote management.
- **BERT** — Bidirectional Encoder Representations from Transformers. A widely-used embedding/language model.

## C
- **Checkpoint** — A saved snapshot of model state during training. Used to resume if training crashes.
- **CNN** — Convolutional Neural Network. Architecture designed for processing images.
- **Context Window** — Maximum number of tokens a model can process in a single request. Like the model's working memory.
- **CUDA** — NVIDIA's programming platform for running computations on GPUs.
- **cuDNN** — CUDA Deep Neural Network library. Optimized neural network operations.

## D
- **Diffusion Model** — A generative model that starts with noise and gradually refines it into an image (or audio).
- **Docker** — Tool for creating and running containers.
- **DGX** — NVIDIA's pre-built AI supercomputer.

## E
- **ECC** — Error-Correcting Code. GPU memory that can detect and sometimes fix bit errors. Uncorrectable ECC errors = hardware problem.
- **Epoch** — One complete pass through the entire training dataset.
- **Embedding** — A numerical representation (vector) of text, images, or other data that captures meaning.

## F
- **Fine-Tuning** — Taking a pre-trained model and training it further on specialized data.
- **FlashAttention** — An optimized implementation of attention that uses less memory and is faster.

## G
- **GAN** — Generative Adversarial Network. Two networks competing: one generates, one discriminates.
- **Gradient** — The direction and magnitude to adjust weights to reduce errors during training.
- **Gradient Descent** — The optimization process of following gradients to find the best weights.

## H
- **HBM** — High Bandwidth Memory. The type of memory used on datacenter GPUs. Much faster than GDDR.
- **HGX** — NVIDIA's high-performance GPU baseboard. Different form factor from DGX.

## I
- **Inference** — Using a trained model to make predictions. The production phase.
- **InfiniBand** — A very high-speed network technology used between GPU servers.

## K
- **KV Cache** — Key-Value cache used during LLM inference to avoid recomputing attention for previous tokens. Uses GPU memory proportional to context length.

## L
- **LLM** — Large Language Model. A model trained on massive text data to understand and generate language.
- **Loss** — A number measuring how wrong the model's predictions are. Lower = better. Going down during training = learning.
- **Learning Rate** — How large of an adjustment to make to weights each training step.

## M
- **Model** — A file (or set of files) containing learned weights that can make predictions.
- **Multi-modal** — A model that can process multiple types of input (text + images, etc.).

## N
- **NCCL** — NVIDIA Collective Communications Library. Handles GPU-to-GPU communication during distributed training.
- **NVLink** — NVIDIA's high-speed interconnect between GPUs. Up to 900 GB/s bidirectional.
- **NVSM** — NVIDIA System Management. Tool for monitoring DGX health.
- **nvidia-smi** — The primary tool for checking GPU status, memory, utilization, and errors.

## O
- **OOM** — Out of Memory. The GPU ran out of VRAM. One of the most common errors.

## P
- **Parameters** — The learnable numbers (weights) inside a model. "70B parameters" = 70 billion numbers.
- **Pipeline Parallelism** — Splitting a model's layers across different GPUs (like an assembly line).
- **Pre-training** — Training a model from scratch on massive data. Very expensive.
- **PyTorch** — The dominant AI framework. Most models and training scripts use this.

## Q
- **Quantization** — Reducing the precision of model weights (e.g., 16-bit → 4-bit) to use less memory and run faster.

## R
- **RAG** — Retrieval-Augmented Generation. Combining document search with LLM generation.
- **RNN** — Recurrent Neural Network. Processes sequences one step at a time. Largely replaced by transformers.

## S
- **SafeTensors** — Modern, secure file format for storing model weights.
- **Self-Attention** — The mechanism in transformers where each token considers all other tokens in the input.
- **Step/Iteration** — Processing one batch of training data and updating weights once.
- **Tensor** — A multi-dimensional array of numbers. The basic data structure in AI.
- **Tensor Parallelism** — Splitting a single layer's computation across multiple GPUs.
- **TensorRT** — NVIDIA's inference optimization library. Compiles models for faster execution.
- **Token** — A piece of text processed by a model. Roughly 3/4 of an English word.
- **Training** — The process of adjusting model weights using data so the model learns to make good predictions.
- **Transformer** — The neural network architecture behind all modern LLMs. Uses self-attention.
- **Triton Inference Server** — NVIDIA's server software for deploying AI models as APIs.
- **TTFT** — Time To First Token. How quickly an LLM starts generating a response.

## V
- **vLLM** — A popular open-source inference server for LLMs. Handles batching and memory efficiently.

## W
- **Weights** — The learned numbers inside a model that determine its behavior. Same as parameters.

---

*This glossary will grow as you progress through the modules.*
