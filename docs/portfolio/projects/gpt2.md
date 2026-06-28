---
title: GPT-2 Transformer From Scratch
description: Complete GPT-2 decoder-only transformer built from the ground up in Python - tokenization, multi-head attention, training loops, and inference.
---

# GPT-2 Transformer From Scratch

!!! abstract "Summary"
    **Type**: Personal research project
    **Stack**: Python, PyTorch

    **Scope:**

    - Custom byte-pair encoding (BPE) tokenizer
    - Multi-head causal self-attention from first principles
    - Feedforward sublayers, layer normalization, residual connections
    - Weight initialization matching OpenAI's GPT-2 paper
    - Full training and evaluation loop

## Challenge

Reading papers and using pre-built transformer libraries gives a working model but not a deep understanding of how it actually works. The challenge was to build a complete GPT-2 implementation from scratch - no HuggingFace Transformers, no shortcuts - to internalize every component at the implementation level.

## Approach

Implemented the complete GPT-2 decoder-only architecture in Python from the ground up:

- **Tokenizer**: Custom byte-pair encoding (BPE) implementation, building the vocabulary merge table from scratch and handling the encode/decode cycle
- **Attention**: Multi-head causal self-attention with separate Q/K/V projection matrices, scaled dot-product attention, causal masking (upper triangular), and output projection
- **Transformer block**: Combined attention sublayer + feedforward sublayer (two linear layers with GELU activation), each wrapped with pre-layer normalization and residual connections
- **Full model**: Token embeddings + learned positional embeddings, stacked transformer blocks, final layer norm, and unembedding projection to vocabulary logits
- **Training loop**: Cross-entropy loss on next-token prediction, gradient accumulation, learning rate scheduling

The implementation followed the original GPT-2 paper (Radford et al., 2019) and Andrej Karpathy's nanoGPT as a reference cross-check after the initial build.

## Results

Functional GPT-2 architecture that trains and generates coherent text. The primary outcome was depth of understanding: every hyperparameter (n_head, n_layer, n_embd, block_size) has a clear mechanical meaning from having built the code that uses it.

## Tech Stack

| Component | Technology |
|---|---|
| Language | Python |
| Deep learning | PyTorch |
| Tokenization | Custom BPE |
| Reference architecture | GPT-2 (Radford et al., 2019) |

## Links

[View Code (GitHub) :fontawesome-brands-github:](https://github.com/sunishbharat/llm-from-the-ground-up){ .md-button target="_blank" }
