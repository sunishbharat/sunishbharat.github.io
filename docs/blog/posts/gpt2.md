---
date: 2026-03-01
categories:
  - AI
  - Deep Learning
authors:
  - sunish
description: What I learned building GPT-2 from scratch in PyTorch - tokenization, causal attention, training loops, and why a program manager with a signal processing background decided to do it at all.
---

# Implementing GPT-2 from Scratch in PyTorch

My career has been in software delivery, embedded systems, automotive, satellite. AI was never part of my job description. I came to it entirely on my own, out of curiosity, during evenings and weekends, reading papers and watching the field move faster than anything I had seen before. At some point that curiosity stopped being enough. I could read about transformers and follow the concepts, but I had never actually built one. I knew the vocabulary without knowing the substance. That bothered me enough to do something about it.

So I built GPT-2 from scratch. Not as a tutorial to follow along to, but as a personal project to genuinely understand what I was managing. The implementation follows Sebastian Raschka's excellent book *Build a Large Language Model (From Scratch)*, which I'd recommend to anyone who wants the same, rigorous without being inaccessible. The code is on [GitHub](https://github.com/sunishbharat/llm-from-the-ground-up){ target="_blank" } and runs on Google Colab.

A big part of the foundation came from [Andrej Karpathy's](https://karpathy.ai){ target="_blank" } videos. I watched almost all of them, spending hours and days going through each one line by line, pausing, rewinding, running the code alongside. It sounds like a lot of time and it was, but it was worth every minute. Karpathy has a way of making things that feel intimidating on paper become genuinely clear when you see them built step by step. His videos made my fundamentals solid in a way that reading alone never quite does.

There was also something personal about it for me. My M.Sc. was in Signal Processing at NTU Singapore. Fourier transforms, convolutions, frequency domain analysis, probability distributions - all of it was deeply familiar once. But that was decades ago and almost none of it had come up in my day-to-day work since. Going through the maths of attention, backpropagation, and loss functions felt like a slow reconnection with things I had learned and then let go of. It was not just learning something new, it was remembering something old and realising it was more useful than I had thought.

This isn't a step-by-step tutorial. It's more about what the experience of doing it actually felt like, what surprised me technically, and what changed about how I think afterward.

## The modular approach

The thing I liked most about how I structured this: each component is its own module, executable independently. You can run just the tokenizer, just the attention layer, just the training loop, and inspect the output at each stage. That turned out to be the right way to learn it. When something behaved unexpectedly, the problem was always isolated to one module rather than buried somewhere in a chain of connected pieces.

The build order was: tokenization → embeddings → attention → transformer block → full model → pre-training loop. Each step built directly on the one before it, and nothing was wired together until it was understood in isolation.

## Tokenization: not what I expected

The first surprise was tokenization. I had assumed tokens were roughly words. They are not. GPT-2 uses Byte Pair Encoding (BPE), a compression algorithm that iteratively merges the most frequent byte pairs in the training corpus until it reaches a target vocabulary size. The result is a vocabulary of subword units, not whole words.

The practical effect: common words get a single token, rare words get split across multiple tokens, and the model never encounters an "unknown" word - it just sees more tokens. The word "unbelievable" might be three tokens. "AI" is one. Punctuation, spaces, and capitalisation all factor into tokenisation in ways that are not obvious until you run text through the tokenizer and look at the output yourself.

```python
# Using tiktoken (GPT-2's tokenizer)
import tiktoken
enc = tiktoken.get_encoding("gpt2")

enc.encode("Hello, world!")
# [15496, 11, 995, 0]   ← 4 tokens

enc.encode("unbelievable")
# [403, 6667, 18112]    ← 3 tokens
```

Once the tokens are produced, they get passed through an embedding layer, a lookup table that converts each token ID into a dense vector of floats. GPT-2's base model uses 768-dimensional vectors. Every token ID maps to a row in a matrix of shape `(vocab_size, 768)`. Those vectors are what the model actually operates on.

## Positional embeddings: the model has no sense of order by default

Attention, on its own, treats its inputs as a set, with no concept of which token came first. Position has to be injected explicitly. GPT-2 does this with learned positional embeddings: a second embedding matrix of shape `(context_length, 768)` where each row corresponds to a position in the sequence. These are added to the token embeddings before anything else happens.

The token embedding tells the model *what* the token is. The positional embedding tells it *where* it is. Combined, the input to the transformer is a matrix of shape `(sequence_length, 768)`, one vector per token, encoding both identity and position.

## Causal self-attention: the part that actually took time

Multi-head causal self-attention is the core of the transformer and the piece that took the longest to properly understand. The mechanism itself is not complicated in principle - each token produces three vectors (Query, Key, Value) through learned linear projections, and attention scores are computed as the scaled dot product of queries and keys:

```
Attention(Q, K, V) = softmax(QK^T / sqrt(d_k)) * V
```

The scaling by `sqrt(d_k)` is there to stop the dot products from getting very large as dimension grows, which would push the softmax into regions where gradients vanish. Small detail, real consequence.

The "causal" part is the mask. In a decoder-only model like GPT-2, each token is only allowed to attend to tokens that came before it, not tokens that come after. This is enforced by masking out the upper triangle of the attention matrix before the softmax, setting those positions to negative infinity so they become zero after softmax. Without the mask, the model can see the answer it's supposed to predict, which makes training meaningless.

```python
# Causal mask - upper triangle set to -inf before softmax
mask = torch.triu(torch.ones(seq_len, seq_len), diagonal=1)
scores = scores.masked_fill(mask.bool(), float('-inf'))
weights = torch.softmax(scores, dim=-1)
```

Multi-head attention runs this process in parallel across multiple "heads" (GPT-2's base model uses 12), each with its own Q, K, V projections operating on a 64-dimensional subspace (768 / 12). The outputs of all heads are concatenated and projected back to 768 dimensions. The idea is that different heads can learn to attend to different kinds of relationships simultaneously: syntax in one head, semantics in another, positional patterns in another.

## The feedforward layer and why GELU

After attention, each token vector passes through a position-wise feedforward network: two linear layers with a non-linearity between them. The hidden dimension is 4x the model dimension (3072 for GPT-2 base), so the network expands the representation, transforms it, then contracts back.

GPT-2 uses GELU (Gaussian Error Linear Unit) rather than ReLU as the activation. GELU is smoother than ReLU - it doesn't hard-zero negative values, it scales them by the probability of the input being positive under a Gaussian distribution. In practice this tends to train more stably, though understanding exactly why requires getting into gradient dynamics that I won't claim to have fully internalised yet.

## Training loop and debugging validation loss

The pre-training objective is simple: given a sequence of tokens, predict the next one. Cross-entropy loss between the predicted token distribution and the actual next token. Run this over batches, backpropagate, update weights with AdamW.

What the training loop taught me is that validation loss is a much more honest signal than training loss. Training loss will almost always decrease - the model is directly optimising for it. Validation loss tells you whether it's actually learning something general or just memorising the training data. When they diverge, you have overfitting. When validation loss stops improving but training loss keeps dropping, that's the moment to stop.

```python
# Training loop (simplified)
for epoch in range(num_epochs):
    model.train()
    for input_batch, target_batch in train_loader:
        optimizer.zero_grad()
        logits = model(input_batch)
        loss = F.cross_entropy(logits.flatten(0, 1), target_batch.flatten())
        loss.backward()
        optimizer.step()

    # Validation
    model.eval()
    with torch.no_grad():
        val_loss = compute_loss(model, val_loader)
```

With 100 epochs on a small dataset it completes in a few minutes on Colab. The generated text at that scale is not coherent - it just learns statistical patterns in whatever you train on. But watching the loss curve drop and then seeing the inference produce something vaguely English-shaped is genuinely satisfying in a way that reading about it is not.

## What changed afterward

The most concrete change: I read architecture diagrams differently. When someone presents a model architecture now, I can trace what's actually happening at each layer rather than pattern-matching on familiar shapes. I understand why certain design choices exist - why residual connections matter for gradient flow through deep networks, why layer normalisation is placed before the attention sublayer rather than after, why context length is a hard constraint rather than a soft one.

The less concrete but equally real change: I have much better conversations with the engineers building these systems. Not because I can contribute code to a production LLM, but because I can ask more precise questions and understand more of the answers. That's the actual value of doing this as a program manager. You don't need to be an expert. You need to understand the terrain well enough to make good decisions in it.

The repo is at [github.com/sunishbharat/llm-from-the-ground-up](https://github.com/sunishbharat/llm-from-the-ground-up){ target="_blank" }. Each module runs independently - if you want to understand just attention, run just that notebook. That's the point of the structure.

---

- Source: [github.com/sunishbharat/llm-from-the-ground-up](https://github.com/sunishbharat/llm-from-the-ground-up){ target="_blank" }
- Inspired by: [Sebastian Raschka](https://sebastianraschka.com){ target="_blank" } - *Build a Large Language Model (From Scratch)*
