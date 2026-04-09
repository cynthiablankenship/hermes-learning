# M01L01 - What Is AI?

## The Honest Answer

AI is a computer program that learned how to do something by looking at examples, instead of being explicitly programmed with rules.

That's it. Everything else is elaboration.

## Traditional Programming vs Machine Learning

**Traditional programming:**
You write explicit rules. "If the temperature is above 100, trigger an alert." The computer follows your instructions exactly.

```
Input + Rules → Output
```

**Machine learning:**
You give the computer examples of inputs and correct outputs. It figures out the rules on its own.

```
Input + Examples → Learned Rules (a "model")
```

Then later:
```
New Input + Model → Prediction
```

## A Concrete Example

**Traditional approach to detecting spam email:**
A programmer writes rules like:
- If the subject contains "FREE MONEY" → spam
- If the sender is unknown → maybe spam
- If there are more than 5 exclamation marks → spam

Problem: spammers change tactics. You're always chasing new patterns.

**Machine learning approach:**
1. Collect 100,000 emails, labeled "spam" or "not spam"
2. Feed them to a learning algorithm
3. The algorithm finds patterns on its own — maybe patterns no human would think of
4. The result is a **model** — a thing that can now classify new emails

The model isn't a list of rules. It's a mathematical function that was *shaped* by the examples. But you don't need to know the math to use it or understand what it's doing.

## What AI Is NOT

- **Not sentient.** No AI has awareness, feelings, or understanding. It's pattern matching at massive scale.
- **Not magic.** Every AI output is the result of a computation. It can be traced, debugged, and understood (even if it's complex).
- **Not always right.** Models make mistakes. They hallucinate. They have biases from their training data. This is why human oversight matters.
- **Not all the same thing.** "AI" is a broad umbrella. The AI that recommends Netflix shows is completely different from the AI that generates text (like ChatGPT) or the AI that drives cars.

## The AI Family Tree (Simplified)

```
Artificial Intelligence (AI)
├── Machine Learning (ML)       — Learning from data (most of what we call "AI" today)
│   ├── Supervised Learning     — Learning from labeled examples (spam detection, image classification)
│   ├── Unsupervised Learning   — Finding patterns without labels (anomaly detection, clustering)
│   └── Reinforcement Learning  — Learning by trial and reward (game playing, robotics)
└── Deep Learning               — ML using neural networks with many layers
    ├── Computer Vision         — Understanding images and video
    ├── Natural Language Processing (NLP) — Understanding and generating text
    │   └── Large Language Models (LLMs) — ChatGPT, Llama, etc.
    ├── Speech                  — Recognizing and generating speech
    └── Generative AI           — Creating new content (text, images, audio, code)
```

When people say "AI" at Dell, they almost always mean **deep learning** — neural networks running on GPUs. That's what DGX systems are built for.

## Why This Matters for Your Job

You don't need to build AI models. But you need to understand:
- **What the hardware is being asked to do** — so you can tell if it's doing it correctly
- **What "normal" looks like** — so you can spot abnormal behavior in logs
- **What customers mean** when they say "my training job crashed" or "inference latency is too high"

The hardware is the engine. AI workloads are what's driving it. You can't diagnose engine problems if you don't understand what the car is trying to do.

---

## Exercises

- [ ] In your own words, explain the difference between traditional programming and machine learning
- [ ] Think of three examples of AI you interact with regularly (recommendations, voice assistants, etc.) and categorize them using the family tree above
- [ ] Why can't you just write rules to catch all spam emails? What advantage does ML have?

## Review Questions

1. What's the fundamental difference between traditional programming and machine learning?
2. Is a spam filter AI? Why or why not?
3. What does "deep learning" mean and how does it relate to "machine learning"?
4. Why does understanding AI workloads help you troubleshoot hardware?

---

*Next: [[M01L02 - What Is a Model]]*
