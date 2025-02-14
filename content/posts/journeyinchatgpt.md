---
title: "What happens when you send a message to ChatGPT?"
date: 2025-02-13T18:00:00+01:00
tags: ["Artificial Intelligence", "Transformer", "ChatGPT"]
categories: ["AI"]
description: "A comprehensive explanation of the ChatGPT model in deep learning."
draft: false
---

# What happens when you send a message to ChatGPT?

## 1. Preprocessing the input

The message you send is tokenized into smaller units called tokens. These tokens can be individual words, parts of words, or even characters, depending on the language and context. For example: 

- Input sentence: “I am so fucking handsome!”
- Tokens: [“I”, “am”, “so”, “fucking”, “handsome”, “!”]
- ID: [123, 435, 134, 534, 834, 987]

In models like GPT, each token is mapped to a unique number (an ID) that the model can process.

## 2. Passing the Tokens into the Transformer Model

ChatGPT is based on the Transformer architecture, which uses several layers of computation to process the tokens. The main steps inside the Transformer are:

1. Embedding Layer: Converts tokens into vector representations (high-dimensional numerical arrays) that capture their meaning and relationships.
2. Positional Encoding: Adds information about the order of tokens, since Transformers process tokens in parallel and need to know their sequence.
3. Self-Attention Mechanism:
Each token “looks” at other tokens in the input to determine which are most relevant. For example, in the sentence “I am so fucking handsome”, the word “I” pays close attention to “am” and “handsome”, while ignoring less important words like “so”.
4. Stacking Layers: These processes repeat through multiple layers (often 12–96 layers, depending on the model size) to build a deep understanding of the input.

## 3. Generating a Response

Once the input is processed, the model generates a response token by token, predicting each next token based on the input and previously generated tokens.

1. First Token Prediction:
The model predicts the most likely first word of the response (e.g., “The”).
2. Subsequent Token Prediction:
It predicts the next token (e.g., “capital”), and so on, until it generates a full response or reaches a predefined token limit.

Each token is chosen based on probabilities. For instance, the model might predict:

- “Paris” (85% likelihood)
- “Berlin” (10% likelihood)
- “London” (5% likelihood)

It selects “Paris” because it has the highest probability.
